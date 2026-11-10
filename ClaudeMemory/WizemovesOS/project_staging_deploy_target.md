---
name: project-staging-deploy-target
description: Where WizemovesOS staging deploys (Sellsuki k8s behind Teleport) and what was set up vs still pending
metadata:
  node_type: memory
  type: project
  originSessionId: 202c78fd-87b6-44c7-bd9e-ce11e2a73ba1
  modified: 2026-11-10T13:16:57.923Z
---

Web app staging target is **Sellsuki's Kubernetes cluster** (user said "k8s ของ sellsuki", 2026-11-10); CI is GitLab (`gitlab.sellsuki.com`). The user's default kubeconfig points at `teleport.internal.staging-th.sellsuki.com`, so the cluster is reached through Teleport, not a plain kubeconfig.

Set up 2026-11-10 (uncommitted at the time): `apps/web/Dockerfile` + `nginx.conf`, `deploy/k8s/staging/web.yaml` (tokens `__IMAGE__ __NAMESPACE__ __HOST__ __INGRESS_CLASS__`), CI jobs `image:web` (kaniko) and `deploy:staging:{db,functions,web}`. Namespace, host, ingress class, resource names were guesses, not Sellsuki conventions. Backend is a Supabase staging project (edge functions + `supabase db push`); `seed.sql` must NOT run on shared envs (known dev password).

CI auth to the cluster: user chose **Teleport** (Machine ID / `tbot`, GitLab join method, `kubernetes/v2` service). `deploy:staging:web` installs tbot at the proxy's version, gets a kubeconfig, runs kubectl. One-time Teleport role/bot/token + namespaced k8s RBAC are documented in `deploy/teleport/README.md` and were NOT applied by Claude (sandbox can't `tsh kube login`: macOS keychain TLS check fails; user must run `deploy/teleport/setup-staging.sh --apply` and `scripts/set-staging-ci-vars.sh --apply` themselves; tctl may need a Teleport admin). Work is committed locally on branch `chore/staging-deploy-readiness`, not pushed. Confirmed names: Teleport cluster = kube cluster = proxy host `teleport.internal.staging-th.sellsuki.com`. Unknown/unverified: Teleport cluster name, kube cluster name, whether the runner can reach the proxy, tbot tarball URL pattern.

**How to apply:** infra conventions (Helm/ArgoCD, namespace, TLS annotations) are still unconfirmed with the infra team. Never run kubectl against the user's default context.

See also [[sandbox-tooling-gotchas]].
