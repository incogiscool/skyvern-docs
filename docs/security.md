# Security

This page covers Skyvern's authentication model, how credentials and artifacts are protected, and what to configure before exposing Skyvern to a network.

## Authentication

Skyvern uses API keys for SDK and direct API access, and JWT tokens for the web UI session.

**API key authentication:** Every request to the API must include an `x-api-key` header. Keys are stored as hashed values in the `organization_auth_tokens` table. The hash is computed with a fingerprinting function so the raw key is never stored.

**JWT for the UI:** When you log in via the web interface, a JWT is issued with HS256 signing (configurable via `SIGNATURE_ALGORITHM`) and stored in an HTTP-only cookie. The secret used for signing is `SECRET_KEY` in your configuration.

For production, set `SECRET_KEY` to a strong random value - the default `"PLACEHOLDER"` must not be used:

```bash
# Generate a strong key
python -c "import secrets; print(secrets.token_hex(32))"
```

Then set it in your environment:

```
SECRET_KEY=your-generated-secret
```

JWT tokens expire after one week by default (`ACCESS_TOKEN_EXPIRE_MINUTES=10080`).

## Authorization

All resources (tasks, workflows, credentials, artifacts) are scoped to an **organization**. An API key authenticates as a specific organization, and can only access resources belonging to that organization. There is no row-level access control within an organization - all members have full access to the organization's resources.

## Credential encryption

Passwords, TOTP secrets, and credit card details are encrypted before being stored in the vault. The encryption key is `ENCRYPTOR_AES_SECRET_KEY` in your configuration. Set this to a strong random value in production:

```
ENCRYPTOR_AES_SECRET_KEY=your-32-char-aes-key
```

The credentials API enforces a strict invariant: **raw secrets are never returned in API responses**. The response models (`PasswordCredentialResponse`, `CreditCardCredentialResponse`, `SecretCredentialResponse`) only include non-sensitive metadata - username, last four digits, or secret label. This is enforced at the Pydantic model level, not by conditional logic, so it cannot be bypassed by adding new endpoints.

## Artifact access

Artifacts (screenshots, recordings, HTML, LLM responses) are stored in S3 or Azure Blob Storage. Access is via pre-signed URLs with a 24-hour expiration:

```
PRESIGNED_URL_EXPIRATION=86400  # seconds (24 hours)
```

Signed URLs are generated on demand and scoped to the requesting organization.

## CORS configuration

By default, CORS allows all origins (`ALLOWED_ORIGINS=["*"]`). For production deployments accessible over a network, restrict this to your frontend's origin:

```
ALLOWED_ORIGINS=["https://your-app.example.com"]
```

Blocked hosts prevent SSRF-style requests to internal services:

```
BLOCKED_HOSTS=["localhost","169.254.169.254"]  # add cloud metadata endpoints
```

## Network security

The Kubernetes deployment guide explicitly warns: do not expose Skyvern to the internet without adding authentication in front of it. The recommended approach is to place Skyvern behind a reverse proxy or API gateway that handles authentication, and access it only from trusted internal networks.

Port exposure in Docker Compose:
- `8000` - Backend API (restrict to internal network in production)
- `8080` - Frontend UI
- `6080` - VNC WebSocket stream (for browser streaming; should not be public)
- `9222` - Chrome DevTools Protocol port (internal only)

For cloud deployments, use a private VPC with no public exposure to ports 6080 and 9222.

## Security best practices

- Set `SECRET_KEY` and `ENCRYPTOR_AES_SECRET_KEY` to strong random values before any network exposure
- Restrict `ALLOWED_ORIGINS` to your known frontend origins
- Add your cloud metadata endpoint (`169.254.169.254`) to `BLOCKED_HOSTS` to prevent SSRF
- Use S3 or Azure Blob for artifact storage rather than local disk in production (better access controls)
- Rotate API keys periodically via the Settings page in the UI
- In Kubernetes, do not expose port 9222 (CDP) or 6080 (VNC) outside the cluster
- Review `BITWARDEN_*` and vault credentials - they grant full access to your password manager
