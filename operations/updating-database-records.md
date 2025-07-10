# Updating Database Records
There might be a reason for needing to access the database directly and quering data / updating records. For example, manually disabling a billing account, changing the account owner associated with workspaces etc.

Please see [Database Client Access Details](services/database.md#client-access) for more information on connecting to one of the platforms databases.

### Update Billing Account Owner
**Prerequisites**: Connect to the `workspaces` database via a PSQL Client of your choice.

Each billing account has it's own `id` associated with it and can be found using

```sql
SELECT * FROM accounts
```

To change the account_owner of a billing account and transfer ownership of the associated workspaces apply:

```sql
UPDATE accounts SET account_owner = '...' WHERE id = '...'
```

Where `id` refers to the accounts `uuid`. The `account_owner` must resemble the username of a user that is already registered in the system and can be found in keycloak. 
Once applied, all workspaces associated with this accounts `id` will have ownership transfered to this user.


### Update Account Approval
**Prerequisites**: Connect to the `workspaces` database via a PSQL Client of your choice.

When a user creates a billing account, by default the `status` of this account is set to `Pending`. An automated email is sent to the EO-DataHub administrators informing them of a new request. 
They can either Approve or Unapprove this request via a dedicated URL described in the email template. 

If the admin wishes to reverse a decision, or temporily disable access to users then they must explicitly set the status within the database.

```sql
UPDATE accounts SET status = 'Unapproved' WHERE id = '...'
```

Where `id` refers to the accounts `uuid`. 

