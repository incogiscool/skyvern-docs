# Credentials & Auth

Skyvern stores login credentials, credit card details, and API keys in an encrypted vault so your automations can authenticate without hardcoding secrets into workflows or scripts.

This page explains the credential types, how to create and manage them, how to use them inside workflows and tasks, how TOTP (2FA) works, and how to connect to external vault providers.

## Credential types

**Password** - A username and password pair. Optionally includes a TOTP secret for 2FA via authenticator app, or a TOTP identifier for email/SMS-based codes.

**Credit card** - Full card details (number, CVV, expiration, brand, holder name). Skyvern stores all fields but only returns the last four digits and brand in API responses.

**Secret** - A generic key-value secret (API tokens, connection strings, etc.) with an optional label.

## Creating a credential

### Via the API

```bash
curl -X POST https://api.skyvern.com/api/v1/credentials \
  -H "x-api-key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Amazon Login",
    "credential_type": "password",
    "credential": {
      "username": "user@example.com",
      "password": "mypassword",
      "totp": "JBSWY3DPEHPK3PXP",
      "totp_type": "authenticator"
    }
  }'
```

The response includes a `credential_id` (e.g., `cred_abc123`). Save this - you'll reference it when using the credential in workflows.

For a credit card:

```bash
curl -X POST https://api.skyvern.com/api/v1/credentials \
  -H "x-api-key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Company Visa",
    "credential_type": "credit_card",
    "credential": {
      "card_number": "4111111111111111",
      "card_cvv": "123",
      "card_exp_month": "12",
      "card_exp_year": "2027",
      "card_brand": "visa",
      "card_holder_name": "Jane Smith"
    }
  }'
```

### Via the Python SDK

```python
from skyvern import Skyvern

skyvern = Skyvern(api_key="YOUR_API_KEY")

credential = await skyvern.create_credential(
    name="Amazon Login",
    credential_type="password",
    credential={
        "username": "user@example.com",
        "password": "mypassword",
    }
)
print(credential.credential_id)
```

## Security: what the API returns

Raw secrets are **never** returned by any Skyvern API endpoint. When you create or retrieve a credential, the response contains only non-sensitive metadata:

- Password credentials: `username`, `totp_type`, `totp_identifier`
- Credit card credentials: `last_four`, `brand`
- Secret credentials: `secret_label`

This is enforced at the model level - the `PasswordCredentialResponse`, `CreditCardCredentialResponse`, and `SecretCredentialResponse` Pydantic models do not have fields for the raw secret values. Any caller with a valid API key can list credentials and see metadata, but cannot extract the actual passwords or card numbers.

## Using credentials in workflows

Reference a credential in a workflow with a `credential` parameter:

```yaml
parameters:
  - parameter_type: credential
    key: login_creds
    credential_id: cred_abc123

blocks:
  - block_type: login
    label: sign_in
    url: "https://example.com/login"
    parameter_keys: [login_creds]
```

When the `login` block runs, Skyvern injects the decrypted credential data into the agent's context. The agent fills in the username and password fields without ever exposing the values in logs or API responses.

For a `task` block that needs credentials, use the same pattern:

```yaml
blocks:
  - block_type: task
    label: checkout
    url: "https://shop.example.com/checkout"
    navigation_goal: "Complete checkout using the stored credit card"
    parameter_keys: [card_creds]
```

## Using credentials in tasks (direct)

For a one-off task, pass the credential ID directly:

```python
task = await skyvern.run_task(
    prompt="Log in and check my order status",
    url="https://example.com/login",
    # Credential lookup happens via the credential_id embedded in the prompt context
)
```

For cases where you need to specify the credential explicitly, use the credential parameter in the task request. The agent knows to look for stored credentials by their label when they appear in the browser context.

## TOTP (2FA) handling

Skyvern supports three types of two-factor authentication:

**Authenticator app (`totp_type: authenticator`)** - You store the TOTP secret (the base32 string shown when setting up an authenticator app). Skyvern generates time-based codes automatically using that secret, without any human intervention.

**Email or SMS codes (`totp_type: email` or `totp_type: text`)** - The 2FA code is sent to an email or phone when you log in. Your application must forward the code to Skyvern's TOTP endpoint as soon as it arrives:

```bash
curl -X POST https://api.skyvern.com/api/v1/credentials/totp \
  -H "x-api-key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "content": "Your verification code is 847291",
    "totp_identifier": "user@example.com",
    "task_id": "tsk_abc123"
  }'
```

Skyvern parses the code from the message content automatically. Set `totp_identifier` on the task to tell Skyvern which identifier to poll for, and `totp_url` if you have an endpoint that serves codes:

```json
{
  "prompt": "Log in to my account",
  "url": "https://example.com/login",
  "totp_identifier": "user@example.com",
  "totp_url": "https://my-app.com/api/totp-codes"
}
```

## Vault integrations

For teams that already manage secrets in an external vault, Skyvern can pull credentials from:

### Bitwarden / Vaultwarden

Configure these environment variables:

```
CREDENTIAL_VAULT_TYPE=bitwarden
SKYVERN_AUTH_BITWARDEN_CLIENT_ID=user.your-client-id
SKYVERN_AUTH_BITWARDEN_CLIENT_SECRET=your-client-secret
SKYVERN_AUTH_BITWARDEN_MASTER_PASSWORD=your-master-password
SKYVERN_AUTH_BITWARDEN_ORGANIZATION_ID=your-org-id
```

For self-hosted Vaultwarden, also set:
```
BITWARDEN_SERVER=http://localhost
BITWARDEN_SERVER_PORT=8002
```

Reference Bitwarden credentials in workflows with a `bitwarden_login_credential` parameter:

```yaml
parameters:
  - parameter_type: bitwarden_login_credential
    key: bitwarden_creds
    bitwarden_client_id_aws_secret_key: BW_CLIENT_ID
    bitwarden_client_secret_aws_secret_key: BW_CLIENT_SECRET
    bitwarden_master_password_aws_secret_key: BW_MASTER_PASSWORD
    url_parameter_key: target_url
```

### 1Password

Set the service account token:

```
OP_SERVICE_ACCOUNT_TOKEN=your-token
```

Reference in workflows with a `onepassword` parameter:

```yaml
parameters:
  - parameter_type: onepassword
    key: op_creds
    vault_id: your-vault-id
    item_id: your-item-id
```

### Azure Key Vault

```
AZURE_STORAGE_ACCOUNT_NAME=your-account
AZURE_STORAGE_ACCOUNT_KEY=your-key
```

Reference in workflows with `azure_vault_credential`:

```yaml
parameters:
  - parameter_type: azure_vault_credential
    key: azure_creds
    vault_name: my-keyvault
    username_key: my-username-secret
    password_key: my-password-secret
    totp_secret_key: my-totp-secret
```

## Testing a credential

Before using a credential in production automation, you can verify it works by running a login test:

```bash
curl -X POST https://api.skyvern.com/api/v1/credentials/{credential_id}/test \
  -H "x-api-key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"url": "https://example.com/login"}'
```

This creates a temporary workflow run that navigates to the URL and attempts to log in. The response includes a `workflow_run_id` you can poll to check the result. If the login succeeds, a browser profile is saved so future runs with this credential can skip the login step.

## Managing credentials

List all credentials (metadata only, no secrets):

```bash
GET /api/v1/credentials
```

Delete a credential:

```bash
DELETE /api/v1/credentials/{credential_id}
```

Rename or update metadata (without changing the stored secret):

```bash
PATCH /api/v1/credentials/{credential_id}
```
