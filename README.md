## This repository is for the purpose of learning `Static Secrets: Key/Value Secrets Engine` based on [this](https://learn.hashicorp.com/tutorials/vault/static-secrets?in=vault/secrets-management) tutorial.

## Pre-requirements
- [X] [Vault](https://www.vaultproject.io/docs/install)

## Instructions

### Setup Vault
- Start Vault in DEV Mode:
```shell
vault server -dev -dev-root-token-id root
```
- Export the `VAULT_ADDR` and `VAULT_TOKEN` addr:
```shell
export VAULT_ADDR="http://127.0.0.1:8200"
export VAULT_TOKEN=root
```

- Enable KV secrets engine

**NOTE**: when you start vault in dev mode, it automatically enables kv secrets engine v2
```shell
vault secrets list -detailed
```

- Enable the key/value secrets engine v1 at kv-v1/
```shell
vault secrets enable -path="kv-v1" kv
```

- Store the Google API Key
```shell
vault kv put kv-v1/eng/apikey/Google key=AAaaBBccDDeeOTXzSMT1234BB_Z8JzG7JkSVxI
```

- Read the secret
```shell
vault kv get kv-v1/eng/apikey/Google
```

### Store the root certificate for MySQL

- Generate a mock certificate using OpenSSL
```shell
openssl req -x509 -sha256 -nodes -newkey rsa:2048 -keyout selfsigned.key -out cert.pem

cat cert.pem
```

- Create a secret at path kv-v1/prod/cert/mysql with a cert set to file conents for cert.pem.
```shell
vault kv put kv-v1/prod/cert/mysql cert=@cert.pem
```

### Generate a token for apps

- Create a policy file named apps-policy.hcl that permits the read to the paths kv-v1/eng/apikey/Google and kv-v1/prod/cert/mysql.
```shell
tee apps-policy.hcl <<EOF
# Read-only permit
path "kv-v1/eng/apikey/Google" {
  capabilities = [ "read" ]
}

# Read-only permit
path "kv-v1/prod/cert/mysql" {
  capabilities = [ "read" ]
}
EOF

vault policy write apps apps-policy.hcl
```

- Create a variable that stores a new token with apps policy attached.
```shell
APPS_TOKEN=$(vault token create -policy="apps" -field=token)

echo $APPS_TOKEN
```

`apps` can now use this token to read the secrets


- Retrieve the secrets
```shell
vault kv get kv-v1/eng/apikey/Google
```

- Authenticate with the apps token
```shell
vault login $APPS_TOKEN
```

- Retrieve the secret
```shell
vault kv get kv-v1/prod/cert/mysql
```

- Read only the key field at the path kv-v1/eng/apikey/Google
```shell
vault kv get -field=key kv-v1/eng/apikey/Google
```

- Read only the cert field at the path kv-v1/prod/cert/mysql
```shell
vault kv get -field=cert kv-v1/prod/cert/mysql
```
