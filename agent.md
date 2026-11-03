# Forgejo API and Actions Guidance

Use these notes when adding or changing Forgejo API calls or Forgejo Actions workflows. Consult the official documentation linked below when behavior may depend on the Forgejo server or runner version.

## Forgejo API

- The API is usually available at `https://<instance>/api/v1`; check whether the instance administrator has disabled it. Prefer the Actions context/environment URL (`forgejo.api_url` / `FORGEJO_API_URL`) over hard-coding the instance domain. `forgejo.server_url` / `FORGEJO_SERVER_URL` is the instance origin and does not include `/api/v1` by assumption.
- Authenticate with an access token using an HTTP authorization header. Forgejo documents `Authorization: Bearer <token>` and the historical `Authorization: token <token>` forms. Never put tokens in URLs, source, logs, or exposed command arguments. Scope credentials to only the step that needs them.
- Create access tokens through the Forgejo UI. Give each only the scopes required for the endpoint and instance version. Token values are shown only once; never request or store a user's password to create one.
- Prefer short-lived Authorized Integrations/OIDC over long-lived personal access tokens when supported. For same-instance Forgejo Actions, configure a Forgejo Actions (Local) Authorized Integration, restrict it to the needed repository/workflow/ref/event, and grant only required capabilities. Enable OIDC with `enable-openid-connect: true`, request a JWT using the runner-provided OIDC variables and the integration audience, and use `Authorization: Bearer <jwt>`. The automatic `${{ forgejo.token }}` is limited: `pull_request` runs do not have repository write permission. Do not assume it can create issues or releases.
- For API access to another Forgejo instance, use a Generic JWT Authorized Integration on the target instance with tightly constrained issuer and claim rules. Prefer local integrations when applicable.
- Check endpoint paths, request/response schemas, and required scopes against the target instance's OpenAPI (`/swagger.v1.json`) or Swagger UI (`/api/swagger`). Forgejo API compatibility applies within a major version; recheck behavior across major versions.
- Handle failed HTTP status codes, pagination, response-size limits, and retries deliberately. Follow `Link` headers/page and limit parameters rather than assuming a single response is complete. Do not enable verbose HTTP traces that could expose credentials.
- Build API URLs from trusted instance/context values, encode URL components correctly, and never attach credentials to a host taken directly from untrusted event input.

## Forgejo Actions

- Store workflows in `.forgejo/workflows/`. Forgejo Actions is familiar to, but not fully compatible with, GitHub Actions; verify syntax and context fields against Forgejo's reference.
- Preserve the configured runner environment. `runs-on` selects an available runner label; it does not promise a particular OS or installed tools. Many runners use minimal Debian/Node environments rather than GitHub's `ubuntu-latest`; install tools explicitly or choose a known configured image.
- Fully qualify remote action URLs to avoid dependency on `DEFAULT_ACTIONS_URL`. Pin security-sensitive actions to immutable commit SHAs and verify their provenance/compatibility; tags and branches may move.
- Workflow/job `permissions` and job-level `continue-on-error` are ignored. Use `enable-openid-connect: true`, not `permissions: id-token: write`, to permit OIDC token generation. Never rely on ignored keys to enforce security.
- `github` is an alias of `forgejo`, but some GitHub context fields may be absent. Prefer documented `forgejo` fields and check the event payload for the chosen trigger.
- Secrets are unavailable to fork-originated `pull_request` runs, and the automatic token is read-only there. Treat `pull_request_target` as privileged: it executes workflow code from the base/default branch with secrets and a write-capable token. Do not checkout or execute untrusted PR code in that context.
- Do not pass API or release credentials to an AI/code-review agent that can inspect or execute repository-controlled code unless this exposure is an explicit, reviewed decision. Prefer splitting untrusted analysis from a separate, narrowly scoped, human-approved publishing step. Agent-generated requests to create issues or releases are untrusted input and require validation.
- Scheduled workflows run only when present on the default branch. Cron is UTC by default; time zones are configurable. Observe Forgejo's daylight-saving-time behavior if using local time.
- For workflow dispatch through the API, check the target instance's API schema and Forgejo version for the supported endpoint and event behavior.

## Official documentation

- [API usage and authentication](https://forgejo.org/docs/latest/user/api/usage/)
- [Authorized Integrations and Actions OIDC](https://forgejo.org/docs/latest/user/api/authorized-integrations/)
- [Forgejo Actions workflow reference](https://forgejo.org/docs/latest/user/actions/reference/)
- [Forgejo Actions security](https://forgejo.org/docs/latest/user/actions/security/)
- [Forgejo Actions and GitHub Actions differences](https://forgejo.org/docs/latest/user/actions/github-actions/)
- [Forgejo Actions administrator guide](https://forgejo.org/docs/latest/admin/actions/)
