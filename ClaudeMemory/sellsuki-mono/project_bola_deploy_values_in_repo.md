---
name: project_bola_deploy_values_in_repo
description: "BOLA non-secret deploy env (WEBHOOK_BASE_URL, CORS, FRONTEND_BASE_URL) lives in bola-backend repo deployment/values-*.yml — NOT Vault; prod SaaS had WEBHOOK_BASE_URL copy-pasted from rgb72"
metadata: 
  node_type: memory
  type: project
  originSessionId: 41c262a6-ad41-4523-a0a4-bb8e9de6e4b3
---

**bola-backend deploy env split (confirmed 2026-07-17, prod incident):**
- **Non-secret env** (WEBHOOK_BASE_URL, CORS_ALLOWED_ORIGINS, FRONTEND_BASE_URL, S3 buckets, etc.) is **hardcoded as plain `value:` in `backend/bola-backend/deployment/values-{production,staging,development}.yml`** (+ per-cron `values-*-cron-*.yml`) — rendered into the k8s Deployment env directly. **NOT from Vault.** So editing Vault does NOT change these — fix the in-repo values file, merge, ArgoCD sync, then **rollout restart** (pod won't pick up new env otherwise).
- **Secrets** (DB creds, tokens, EXPORT_S3_SECRET_KEY, NOTION_TOKEN…) use `valueFrom.secretKeyRef` (bola-backoffice-secret) — those come from Vault/SRE.
- This partially contradicts [[project_bola_auth_mode_deployment]] ("deploy env in SRE repo"): SECRETS are SRE/Vault, but plain URL/CORS env are in the backend repo's deployment/ folder.

**Prod incident (fixed MR !130):** prod SaaS (ns `bola`, bola.bearyweb.com / API bola-api.sellsuki.com) had `WEBHOOK_BASE_URL` = **rgb72 tenant's domain** `https://lon-api.xn--12car7cvaebaj4dgahdhd8ac4bzmub2b2a3hxhgoz8rrag.com` (= "แบบประเมินการเลี้ยงลูกด้วยนมแม่") — copy-pasted from rgb72 tenant config in both `values-production.yml` and `values-broadcast-cron-production.yml`. → every LineOA added got `webhook_url` baked with rgb72 host (src/use_case/line_oa.go:149 uses global webhookBaseURL at create) → LINE would deliver those OAs' webhooks to rgb72's server (sig fails, but cross-tenant leak + SaaS webhook broken). Correct prod value = `https://bola-api.sellsuki.com` (matches FE VITE_API_URL prod; Kratos cookie domain sellsuki.com). staging/dev were correct (`bola-api.<env>.bearyweb.com`).

**Data cleanup after deploy:** existing LineOA rows keep the stale rgb72 `webhook_url` (baked at create) → must re-save/regenerate; operators must update the webhook URL in LINE Developers Console.

**Code fragility (not fixed):** `UseCase.GetWebhookBaseURL` (src/use_case/auto_push_message.go:408) derives scheme+host from a stored `webhook_setting.WebhookURL` BEFORE falling back to config `webhookBaseURL` → one wrong stored row can poison the base URL. Prefer config as source of truth. Related: [[project_bola_kratos_sso_staging]].
