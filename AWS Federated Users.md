# AWS Federated Users

## AWS OIDC Provider
An OIDC provider can be created to allow authentication to AWS IAM roles from federated OIDC providers, e.g. Keycloak.

OIDC providers can be created using the AWS console, CLI or declaratively using the aws-controllers-k8s iam-chart.

Below is an example of the OIDCProvider Kubernetes custom resource for the IAM chart, but the process is similar for AWS console or CLI creation.

```yaml
apiVersion: iam.services.k8s.aws/v1alpha1
kind: OpenIDConnectProvider
metadata:
  name: aws-oidc-provider
  namespace: aws
spec:
  url: test.eodatahub.org.uk
  clientIDs:
    - oauth2-proxy
    - oauth2-proxy-workspaces
  thumbprints: 
    - 7621B110E29FD3BAEF9433832BBED5B879121373  # platform thumbprint
    - 77A1F8C598DCC4FD8ACA18A6DC1DDCE1AB157E4E  # workspaces thumbprint
```

To calculate the thumbprints required, the following commands can be used ([Obtain the thumbprint for an OpenID Connect identity provider - AWS Identity and Access Management (amazon.com)](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_providers_create_oidc_verify-thumbprint.html)).

```bash
kubectl get -n certs secret <tls-secret-name> -o jsonpath='{.data.tls\.crt}' | base64 --decode > cert.pem
openssl x509 -in cert.pem -fingerprint -sha1 -noout | sed 's/://g' | awk -F= '{print $2}'
rm cert.pem
```

## AWS Federated User Policies

An example of a federated policy is shown below. Note that it is possible, through your OIDC claims, to pass in arbitrary "Principal Tags" to AWS. The process for this will be described in a subsequent section.

### Trust Policy
The trust policy references the AWS OIDC provider ARN from which users will be allowed to authenticate. It can also define the OIDC client (claims.aud) that is allowed to authenticate.
Note that the `sts:TagSession` action is required to permit passing session tags to be used as principal tags in the permission policy.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::312280911266:oidc-provider/test.eodatahub.org.uk/keycloak/realms/eodhp"
      },
      "Action": [
        "sts:AssumeRoleWithWebIdentity", 
        "sts:TagSession"
      ],
      "Condition": {
        "StringEquals": {
          "test.eodatahub.org.uk/keycloak/realms/eodhp:aud": "oauth2-proxy"
        }
      }
    }
  ]
}
```

### Permission Policy
The permission policy is much the same as usual, but now PrincipalTag values can be used in the conditions to facilitate ABAC.

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
          "s3:GetObject",
          "s3:PutObject",
          "s3:DeleteObject",
          "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::eodhp-dev-workspaces",
        "arn:aws:s3:::eodhp-dev-workspaces/*"
      ],
      "Condition": {
        "StringLike": {
          "s3:prefix": "${aws:PrincipalTag/username}/*"
        }
      }
    },
    {
      "Effect": "Allow",
      "Action": [
          "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::eodhp-dev-workspaces",
        "arn:aws:s3:::eodhp-dev-workspaces/*"
      ],
      "Condition": {
        "StringLike": {
          "s3:prefix": "${aws:PrincipalTag/username}"
        }
      }
    }
  ]
}
```

### Boto3 Python Example
The example below assumes that `token` is a JWT encoded set of claims from your OIDC provider, that `username` is the user's OIDC preferred_username and that `role_arn` is the role ARN of the AWS role the user wishes to assume. 

The token may contain principal tags that will be passed to the `assume_role_with_web_identity` call and made available to parameterised policies.

```python3
sts_client = boto3.client("sts")
role = sts_client.assume_role_with_web_identity(
    RoleArn=role_arn,
    RoleSessionName=f"{username}-session",
    WebIdentityToken=token,
)
creds = role["Credentials"]
s3_client = boto3.client(
    "s3",
    aws_access_key_id=creds["AccessKeyId"],
    aws_secret_access_key=creds["SecretAccessKey"],
    aws_session_token=creds["SessionToken"],
)
response = s3_client.list_objects_v2(
    Bucket="my-bucket", Prefix=username,
)
```

### Passing OIDC Claims as Principal Tags
AWS expects any intended principal tags to be contained encoded within the claims token in a specific format.

```json
{
  "sub": "johndoe",
  "aud": "oauth2-proxy",
  "jti": "ZYUCeRMQVtqHypVPWAN3VB",
  "iss": "https://test.eodatahub.org.uk",
  "iat": 1566583294,
  "exp": 1566583354,
  "auth_time": 1566583292,
  "https://aws.amazon.com/tags": {
    "principal_tags": {
      "username": [
        "johndoe"
      ]
    }
  }
}
```

Note that all principal tags must be under the https://aws.amazon.com/tags > principal_tags path. Each principal tag should be a list of strings, even if only a single value is intended.

Reference: https://docs.aws.amazon.com/IAM/latest/UserGuide/id_session-tags.html#id_session-tags_adding-assume-role-idp

### Configuring Principal Tag Claims in Keycloak
1. Select the realm of your application
2. Select Clients, then click into your Client
3. Client Scopes
4. Click into {{your-client}}-dedicated from the list
5. Add mapper > By configuration
6. Mapper type: User Attribute (may differ based on your needs)
   Name: Choose a name for the mapper
   User Attribute: Select from available options (may differ depending on Mapper type)
   Token Claim Name: Set the path that the attribute will be set in the claims. For above example `https://aws\.amazon\.com/tags.principal_tags.username` will do. Note that `.` need to be escaped with `\.` if they are not used to specify a subpath.
   Claim JSON Type: String
   Add to ID token: No
   Add to access token: Yes
   Add to lightweight access token: No
   Add to userinfo: Yes
   Add to token introspection: Yes
   Multivalued: Yes (this is what makes the field a list of strings and not just a string)
   Aggregate attribute values: No
7. Save
8. You can evaluate a user token from Clients > {{your-client}} > Client scopes tab > Evaluate tab > select a user and Generate access token (right sidebar) to ensure the token appears as expected.