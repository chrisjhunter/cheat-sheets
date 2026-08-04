# HashiCorp Vault Cheat Sheet

## Setup & auth
```bash
vault server -dev            # quick local dev server (in-memory, unsealed)
export VAULT_ADDR='http://127.0.0.1:8200'
vault status                 # sealed status, HA info
vault operator init          # initialize a new production vault (generates unseal keys)
vault operator unseal <key>  # unseal (repeat with threshold number of keys)
vault login <token>          # authenticate with a token
vault login -method=userpass username=chris
vault token lookup           # info about current token
vault token renew
```

## KV secrets engine
```bash
vault secrets enable -path=secret kv-v2
vault kv put secret/app/db password=hunter2 user=admin
vault kv get secret/app/db
vault kv get -field=password secret/app/db  # extract just one field
vault kv list secret/app                    # list keys under a path
vault kv delete secret/app/db
vault kv metadata get secret/app/db         # version history/metadata
vault kv rollback -version=2 secret/app/db  # roll back to a version
```

## Dynamic secrets (e.g. databases)
```bash
vault secrets enable database
vault write database/config/mydb plugin_name=postgresql-database-plugin \
  connection_url="postgresql://{{username}}:{{password}}@localhost:5432/mydb" \
  allowed_roles="readonly"
vault write database/roles/readonly db_name=mydb \
  creation_statements="CREATE ROLE \"{{name}}\" WITH LOGIN PASSWORD '{{password}}' VALID UNTIL '{{expiration}}';" \
  default_ttl="1h" max_ttl="24h"
vault read database/creds/readonly  # generate short-lived DB creds
```

## Policies
```bash
vault policy write mypolicy policy.hcl
vault policy list
vault policy read mypolicy
vault token create -policy=mypolicy
```
Example policy:
```hcl
path "secret/data/app/*" {
  capabilities = ["read", "list"]
}
```

## Auth methods
```bash
vault auth enable userpass
vault write auth/userpass/users/chris password=hunter2 policies=mypolicy
vault auth enable approle     # for machine-to-machine auth
vault write auth/approle/role/myapp policies=mypolicy
vault read auth/approle/role/myapp/role-id
vault write -f auth/approle/role/myapp/secret-id
vault auth enable kubernetes  # for pods to authenticate via SA tokens
```

## PKI / certificates
```bash
vault secrets enable pki
vault write pki/root/generate/internal common_name="example.com" ttl=8760h
vault write pki/roles/example-dot-com allowed_domains="example.com" allow_subdomains=true
vault write pki/issue/example-dot-com common_name="www.example.com"
```

## Sealing / operations
```bash
vault operator seal             # seal the vault (emergency lockdown)
vault operator raft list-peers  # raft cluster peers (integrated storage)
vault audit enable file file_path=/var/log/vault_audit.log
vault operator rekey            # rotate unseal keys
```

## Useful one-liners
```bash
vault kv get -format=json secret/app/db | jq -r .data.data.password  # extract secret value in scripts
VAULT_TOKEN=$(vault write -field=token auth/approle/login role_id=$ROLE_ID secret_id=$SECRET_ID)
vault list secret/                                                   # list top-level secret paths
vault lease revoke -prefix database/creds/readonly                   # revoke all leases under a role
vault read sys/health                                                # health check (useful for LB probes)
```
