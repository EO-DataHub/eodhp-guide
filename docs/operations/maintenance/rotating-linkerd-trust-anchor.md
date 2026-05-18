# Rotating LinkerD Trust Anchor

## Purpose

LinkerD is the service mesh for the platform. Among other things, it provides mTLS between services. It uses a trust anchor certificate to create a Certificate Authority to issue certificates to services. The trust anchor should be rotated approximately every year, but this process is not automatic. While the certificate will rotate every year there is no mechanism to refresh the certificate issuer when the trust anchor is changed, so this process requires manual intervention. System administrators should set reminders when to rotate the certificate.

The consequence of not rotating the certificate issuer is that the certificate issuer will continue to user the previous trust anchor, which will eventually expire. This will result in "Bad Certificate" errors from all LinkerD proxies and will result in services failing to communicate with each other, bringing the whole platform down.

## When to Use

Monitor the LinkerD trust anchor certificate renewal time. You can view the certificate renewal time by inspecting the trust anchor certificate in Kubernetes.

```sh
echo Renewal time: $(kubectl get cert linkerd-trust-anchor -n certs -o jsonpath="{.status.renewalTime}")\n
```

Platform operators should put a reminder into to rotate the trust certificate **before** it automatically rotates.

## Operation

Step by step instructions to execute the procedure.

1. The "active" trust anchor certificate `linkerd-trust-anchor` is generated in `certs` namespace. Make a local copy of it in the `certs` namespace.

   ```sh
   kubectl get secret linkerd-trust-anchor -n certs -o yaml | \
   sed 's/name: linkerd-trust-anchor/name: linkerd-trust-anchor-previous/' | \
   kubectl apply -f -
   ```

2. Temporarily uncomment the following code in apps/certs/base/linkerd.yaml linkerd-identity-trust-roots Bundle manifest:

   ```yaml
   apiVersion: trust.cert-manager.io/v1alpha1
   kind: Bundle
   metadata:
   name: linkerd-identity-trust-roots
   spec:
   sources:
     - secret:
         name: linkerd-trust-anchor
         key: tls.crt
       # Uncomment lines below as part of linkerd trust anchor rotation process
       # - secret:
       #     name: linkerd-trust-anchor-previous
       #     key: tls.crt
   target:
     configMap:
       key: ca-bundle.crt
     namespaceSelector:
       matchLabels:
         linkerd.io/is-control-plane: "true"
   ```

   This will add the current trust anchor certificate to the trust bundle. Commit the change to ArgoCD.

3. Use Cert Manage CLI to renew the trust anchor certificate in the cluster.

   Required: [cmctl](https://cert-manager.io/docs/reference/cmctl/#installation).

   ```sh
   cmctl renew -n certs linkerd-trust-anchor
   ```

4. (Optional) Confirm that the `linkerd-identity-trust-roots` config map now contains subject keys from both secrets.

   Required: [step-cli](https://smallstep.com/docs/step-cli/installation), [jq](https://jqlang.org/download/).

   ```sh
   kubectl get configmap -n linkerd linkerd-identity-trust-roots \
     -o jsonpath='{ .data.ca-bundle\.crt }' \
     | step certificate inspect --bundle --format json \
     | jq -r ".[] | \"Subject: \(.extensions.subject_key_id | .[0:16])... \(.subject_dn)\""
   ```

   You should see output similar to:

   ```
   Subject: c39b67563c5a3684... CN=root.linkerd.cluster.local
   Subject: 120a88a0e760acd2... CN=root.linkerd.cluster.local
   ```

5. Rotate the `linkerd-identity-issuer` in `certs` namespace.

   ```sh
   # (Optional) Inspect current identity issuer
   kubectl get secret -n certs linkerd-identity-issuer \
     -o jsonpath='{ .data.tls\.crt }' \
     | base64 -d | step certificate inspect -
   # Rotate identity issuer
   cmctl renew -n certs linkerd-identity
   # (Optional) Check identity issuer has been rotated
   kubectl get secret -n certs linkerd-identity-issuer \
     -o jsonpath='{ .data.tls\.crt }' \
     | base64 -d | step certificate inspect -
   ```

6. Restart the control plane.

   ```sh
   kubectl rollout restart -n linkerd deploy
   kubectl rollout status -n linkerd deploy
   ```

7. Restart the data plane. This involves running a rollout restart in every namespace.

   A convenience script is provided below. This file is assumed to be executable, on your path and named `kubectl-rollout-all-namespaces`.

   ```sh
   #!/bin/sh

   # Ensure an argument is provided
   if [ -z "$1" ]; then
     echo "Error: Please specify resource type: $0 <resource-type>"
     exit 1
   fi

   kubectl get "$1" --all-namespaces -o custom-columns="NAMESPACE:.metadata.namespace,NAME:.metadata.name" | tail -n +2 | while read namespace name; do
     kubectl rollout restart "$1" "$name" -n "$namespace"
   done
   ```

   Run the convenience script for all deployments and stateful sets. You may need to run it for more resource types as the platform evolves.

   ```sh
   kubectl-rollout-all-namespaces deployments
   kubectl-rollout-all-namespaces statefulsets
   ...
   ```

8. Monitor all of the pods in the cluster to ensure they are restarting successfully

   ```sh
   kubectl get pods -A -w
   ```

9. Once everything has restarted, recomment the lines from step 2, commit to ArgoCD and delete the `linkerd-trust-anchor-previous` secret in `certs` namespace. This will generate a new trust bundle containing only the new trust anchor certificate.

   ```sh
   # After recommenting the lines from step 2
   kubectl delete secret -n certs linkerd-trust-anchor-previous
   ```

## Requirements

- Access to the Kubernetes cluster via kubectl

## Useful Information

If the certificate rotates automatically and you need to resolve the issue, steps 5-8 above are sufficient.
