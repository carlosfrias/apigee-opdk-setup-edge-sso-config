# apigee-opdk-setup-edge-sso-config — Apigee Edge SAML SSO Configuration (Dual-Keypair)

> **An Ansible role that configures the `apigee-sso` SAML component of Apigee Edge** — generating the dual keypair (JWT signing + SAML service-provider), binding the IdP metadata, and rendering the `apigee-sso` installer response file.

A SAML SSO bootstrap procedure expressed as code: knowing **why Apigee SSO uses two independent keypairs**, **how the Edge UI becomes an OAuth client of `apigee-sso`**, and **how to bind an IdP by metadata file *or* URL**.

<!-- BEGIN Google Required Disclaimer -->

## Not Google Product Clause

This is not an officially supported Google product.
<!-- END Google Required Disclaimer -->

---

## Why this role is notable

- **Dual-keypair architecture, handled correctly.** Apigee SSO uses **two independent keypairs** — a JWT signing/verification keypair (for the Edge UI ↔ `apigee-sso` OAuth exchange) and a SAML service-provider key/cert (for the SP side of the SAML assertion). Conflating them is a common error this role prevents.
- **IdP binding flexibility.** Bind the IdP by **metadata URL** *or* by **local metadata file** (rewritten to a `file://` URL) — necessary for air-gapped or self-signed-IdP environments.
- **Deliberate escape hatch.** `SSO_SAML_IDPMETAURL_SKIPSSLVALIDATION` is surfaced as an opt-in escape hatch for self-signed IdP certs — not a hardcoded workaround.
- **Response-file-driven install.** Configuration is a rendered response file consumed by `apigee-setup install` — the Apigee installer contract, not raw imperative commands.

---

## What the role actually does

`tasks/main.yml` produces a complete `apigee-sso` configuration:

1. **Cache the key/cert paths** (cacheable facts for downstream install roles).
2. **Generate the SAML service-provider keypair + self-signed certificate** — `create-saml-keys-cert.yml`.
3. **Generate the JWT signing + verification keypair** — `create-jwt-keys.yml`.
4. **Bind the IdP metadata** — if a local metadata file is provided, copy it into place and set `SSO_SAML_IDP_METADATA_URL` to a `file://` URL (so `apigee-sso` reads the metadata locally instead of fetching from an HTTPS endpoint).
5. **Render the installer response file** — `templates/edge-sso-installer-config.conf.j2`.

The template exposes the non-obvious `apigee-sso` contract: `SSO_PROFILE=saml`, separate `SSO_JWT_SIGNINIG_KEY_FILEPATH` / `SSO_JWT_VERIFICATION_KEY_FILEPATH`, `SSO_SAML_SERVICE_PROVIDER_KEY` / `SSO_SAML_SERVICE_PROVIDER_CERTIFICATE`, `SSO_SAML_IDP_METADATA_URL`, and the `SSO_SAML_IDPMETAURL_SKIPSSLVALIDATION` escape hatch for self-signed IdP certs.

---

## Capabilities — what this credentials

> Ansible is the medium. The engineering below is the evidence of the expertise applied.

- **SAML SSO architecture** — Apigee Edge SSO uses two independent keypairs (JWT signing/verification + SAML service-provider key/cert); the role provisions both halves of the trust.
- **IdP binding flexibility** — bind by metadata URL *or* by local metadata file (rewritten to a `file://` URL) for air-gapped/self-signed-IdP environments.
- **Self-signed-IdP escape hatch** — `SSO_SAML_IDPMETAURL_SKIPSSLVALIDATION` surfaced as a deliberate, opt-in escape hatch.
- **Edge UI as OAuth client** — the JWT keypair exists because the Edge UI acts as an OAuth client of `apigee-sso`.
- **Response-file-driven install** — configuration is a rendered response file consumed by `apigee-setup install`.

---

---

## Role variables (selected)

| Variable | Purpose |
|----------|---------|
| `sso_profile` | SSO profile (defaults to `saml`) |
| `sso_saml_service_provider_key_file_path` / `sso_saml_service_provider_certificate_file_path` | SAML SP key + cert |
| `sso_jwt_signinig_key_file_path` / `sso_jwt_verification_key_file_path` | JWT signing + verification keys |
| `sso_saml_idp_metadata_url` | IdP metadata URL (or `file://…` when a local file is supplied) |
| `sso_saml_idp_metadata_local_file_name` | Local IdP metadata file to copy into place (optional — triggers `file://` binding) |
| `sso_saml_idpmetaurl_skipsslvalidation` | Skip SSL validation of the IdP metadata URL (escape hatch, defaults to `n`) |
| `sso_saml_ipd_name` / `sso_saml_ipd_login_text` | IdP display name + login link text |

---

## Provenance

Authored and maintained by **Carlos Frias** during his tenure on Apigee Edge Private Cloud. Part of the `apigee-opdk-*` role corpus; the SAML/TLS expertise is aggregated in the [`apigee-edge-opdk`](https://github.com/carlosfrias/apigee-edge-opdk) framework.

Contributions welcome — see [CONTRIBUTING.md](./CONTRIBUTING.md).

## License

See [LICENSE](./LICENSE).