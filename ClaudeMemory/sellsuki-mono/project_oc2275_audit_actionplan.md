---
name: project_oc2275_audit_actionplan
description: OC-2275 API-key audit 2026-08-09/10 (repo/DDD/branch/MR) — everything shipped/unblocked except entity MR !58 (needs user merge+tag) and gateway rule.yaml hardening (OC-4433, needs SRE)
metadata: 
  node_type: memory
  type: project
  originSessionId: a0710894-5349-411a-b947-f53b13519857
  modified: 2026-08-10T11:02:09.926Z
---

## 🔴 2026-08-28 — เจอ cross-tenant write ใน v2 event redemption (OC-4474, MR !215) + WS7 เสร็จ (OC-4472, MR !216)

**OC-4474 = ของจริงที่สำคัญที่สุดของรอบนี้** `CampaignEventRedemptionConfirm` รับ `member_id` จาก request แล้วไม่เคยเทียบ company:
- event ดึงด้วย `ident.CompanyID` แต่ campaign ดึงด้วย **`m.CompanyID`** (คนละ tenant ในคอลเดียว)
- condition เทียบ event ด้วย **string ของ event code** (`campaign_transaction.go:192`) ซึ่งแต่ละบริษัทตั้งเอง ซ้ำกันได้ปกติ ⇒ key ของ A + member/campaign ของ B = transaction ลงที่ B
- ต้องรู้ member uuid + campaign code ของเป้าหมายก่อน ⇒ ไม่ใช่ยิงมั่วได้ แต่ด่านหายไปทั้งด่าน และจะแย่ลงเมื่อ WS2a เปิดให้เห็น member id
- 🔑 **หลักฐานว่าไม่เคยมีใครเช็ค**: fixture เดิมตั้ง member `CompanyID: "8900"` แต่ key เป็น uuid — คนละบริษัทมาตลอด แล้ว test เขียวหมด
- fix: เทียบ `m.CompanyID != ident.CompanyID` → **404** ทั้ง confirm และ inquiry + campaign lookup ใช้ `ident.CompanyID`

**OC-4472 (WS7 code earn) → MR !216** · พอร์ต v1 `CodeInquiry`/`CodeRedemption` มา v2 · scope `campaign.redeem.code` เป็น sibling ของ event (test ตรึงทั้งสองทาง) · **ไม่ได้ทำ mode B ของ OC-4335** (payload ยังไม่เคาะ) · **rate limit key ด้วย member ไม่ใช่ api_key_id** เพราะ limiter เขียน `member_block_log.member_id` และ v2 ทั้ง group ไม่มี rate limit เลย → เสนอทำเป็น middleware ใบเดียวตาม Decision #4

⚠️ **merge order**: !214 (WS8) กับ !216 (WS7) แก้ `v2_openapi.yaml` + `spec.gen.go` ทั้งคู่ → ตัวที่ merge ทีหลัง rebase แล้ว `make gen-http-fiber` ใหม่

## ✅ 2026-08-28 — OC-4473 (WS8 point read) implement เสร็จ → MR !214

3 endpoint `/v2/openapi/member/{member_id}/point[/expire|/transaction]` · branch `feat/oc-4473-point-read-v2` → develop · coverage 95.7%
- **เป็นงานพอร์ตจริงตามที่คาด** — repo รับ `(companyID, memberID)` อยู่แล้ว ไม่ต้องแตะอะไรใต้ use_case
- member คนละ company → **404** (ไม่ใช่ 403) + test พิสูจน์ว่า point repo ไม่ถูกเรียกเลย
- `limit` > 200 → 400 `limit_too_large` (เพิ่ม `use_case.ErrLimitTooLarge` + map ใน helper/errors.go)
- `normalizeRouteKey` เพิ่ม branch ตัด `{member_id}` ออกก่อน lookup — เป็น path param ตัวที่ 2 ต่อจาก OC-4398
- mutation-verified 2 ชั้น: ถอด v2RouteScopes → scope_policy_test แดงระบุครบ 3 route · ถอด branch normalize → 3 test แดง (500)
- ไม่ต้องแตะ gateway (rule จับ `/v2/openapi<.*>` อยู่แล้ว)

🟡 **เจอ bug class เดิมอีกจุด (ยังไม่แก้)**: `EventInquiry` (`event_redemption.go:24-35`) เรียก `GetByID(memberID)` โดยไม่เทียบ company — ไม่ leak data (query หลังจากนั้น scope ด้วย CompanyID) แต่ probe การมีอยู่ของ member ข้ามบริษัทได้ → ควรเปิดใบเล็ก

## 📋 2026-08-28 — เปิดการ์ด 5 WS ที่หายไป (OC-4468..4473)

epic OC-2275 ล็อก scope 10 ตัวไว้ตั้งแต่ ก.ค. แต่ **5 workstream ไม่มีการ์ดเลย** ⇒ 7/10 scope แจก key ได้แต่ไม่มี endpoint · เปิดครบแล้ว: **OC-4468** WS2a member.read · **OC-4469** WS2b member.manage (DoR: ต้องเคาะ dedup key ก่อน) · **OC-4470** WS5 point.redeem · **OC-4471** WS6 point.adjust (block โดย OC-4294 ที่ยัง To Do) · **OC-4472** WS7 campaign.redeem.code (coordinate OC-4335) · **OC-4473** WS8 point.read+history (พร้อมที่สุด ไม่มี dependency)

🔴 **แก้ข้อมูลผิดใน epic** (comment 43938): epic เขียนว่า point redeem "ไม่มีเลยในทุก service" — **ผิด** · campaign condition รองรับ `oneof=code point event` (`model/campaign_condition.go:15`) และ campaign ที่ condition = `point` คือกลไกใช้แต้มแลกของที่มีอยู่แล้ว → `CampaignRedemptionConfirm` (`campaign_redemption.go:84`) → `createRedemptionPoint` (`campaign_transaction.go:910`) → `DeductMemberPointTxn` (`repository.go:244`) ⇒ WS5 = งานพอร์ต ไม่ใช่สร้างใหม่ · ส่วน point.adjust (WS6) ไม่มีจริงตามที่ epic เขียน
🔑 **WS8 ง่ายกว่าที่ประเมิน** — point repository รับ `(companyID, memberID)` เป็น argument อยู่แล้ว (`repository.go:239-243`) ที่ผูก session คือ 2 บรรทัดหัว use case (`me.go:373-386`) ⇒ ไม่ต้องแตะ repository
⚠️ **ขัดกันเองใน epic**: OC-4428 (outbound webhook) เป็น child แต่ epic เขียนใน Out of Scope ว่าต้องเป็น epic ใหม่ — ยังไม่เคาะ
📌 **แบบแผน v2 ที่การ์ดทุกใบอ้าง**: `CampaignEventRedemptionConfirm` (`event_redemption.go:137`) รับ `model.CRMApiKey` + `mID` แล้วเรียก `checkIdentityApiKey` (`:148`) · ทุก endpoint ใหม่ต้องเพิ่ม `v2RouteScopes` + `scopeDescriptionsTH` ไม่งั้น test แดง/500 · v1 handler ที่ต้องพอร์ต: code earn `route_campaign_v1.go:71,102` → `code_redemption.go:36,109` · point `route_me_v1.go:276,296,317` → `me.go:368,410,442` · backoffice member read `member_v1.go:11,60,80,106`

## ✅ 2026-08-28 (ต่อ) — OC-4466 merge + verify ซ้ำบน dev-th ผ่านครบ

MR !213 merge เป็น `46d43e72` · `deploy_development_th_arm` เป็น job **อัตโนมัติ** บน develop (ไม่ manual) → pod โรลเองใน ~10 นาที · ยิงผ่าน gateway จริง `crmapi.dev-th.oc2.plus` ครบ 9 ข้อ:
`/event` 200 · `/auth/whoami` 200 · `/campaign` ด้วย key ไม่มี scope = **403 ไม่ใช่ 401** · key ที่มี `campaign.read` = 200 คืนข้อมูลจริง · key ปลอม 401 · revoke แล้วยิงซ้ำ 401 ทันที · mint/revoke 201/204

🔑 **ปิดจุดสุดท้ายของ OC-4425 ด้วยหลักฐาน runtime แล้ว** — ยิง key จริง + ปลอม `X-Company-Id: 0000…` และ `X-Api-Scope: campaign.read` ไปด้วย → whoami ยังคืน company จริง และ `/campaign` ยัง 403 ⇒ Oathkeeper header mutator เป็น **Set ทับ** ยืนยันจาก runtime ไม่ใช่แค่จาก source code แล้ว

OC-4466 → สถานะ **Ready to test (DEV)** (comment 43929) · ยังค้าง: OC-4467 (`/.well-known/scopes` 404 ที่ gateway, รอ SRE) · OC-4433 (strip `X-User-Id`) · OC-4430 (QA matrix)

## 🔴 2026-08-28 — ไล่ flow เต็มบน dev-th: v2 openapi ตายมาตั้งแต่ 10 ส.ค.

**dev-th = ns `octoplus-dev` บน cluster teleport staging-th** (kubectl context มีอยู่แล้ว) · pods รัน image `2ec10129`/`56a4f737` = develop HEAD เป๊ะ · hostname: backoffice `api.crm.dev-th.oc2.plus/backoffice/*`, partner `crmapi.dev-th.oc2.plus/v2/openapi/*` (public, ไม่ต้อง VPN)

**วิธีเข้าไปทดสอบโดยไม่ต้อง login:** port-forward `svc/oc2plus-line-crm-service-backoffice-api-svc 18089:80` แล้วยิงพร้อม header `X-User-Id` (bypass gateway) · หา user ที่มีสิทธิ์จาก Keto: port-forward `svc/keto-read` ใน ns `share-dev` → `GET /relation-tuples?namespace=permissions&object=oc2plus.apikey.manage` → ได้ tuple เดียว company `d9fca606-aea6-4ef3-b10c-4a0a8dcf0dcd` role `sellsuki.role:88892` → query `namespace=roles&object=sellsuki.role:88892` → user `378e1998-5323-4085-8a32-187fa777ec1f` · ⚠️ curl | python3 โดนrtk filter — เขียนลงไฟล์ก่อนแล้วค่อย parse

**🔴 บั๊กที่เจอ (OC-4466, แก้แล้ว MR !213):** `RequireApiKeyScope` (!210) mount ทั้ง group `/v2/openapi` และบังคับ `X-Api-Key-Id`/`X-Company-Id` ต้องไม่ว่าง — แต่ `/v2/openapi/auth/whoami` คือ `check_session_url` ของ Oathkeeper เอง มาถึงพร้อม key/secret ดิบเท่านั้น ⇒ whoami 401 → bearer_token ล้ม → ตก anonymous → ทุก endpoint 401 **ทุก key ที่ถูกต้อง ตั้งแต่ 2026-08-10** · `v2NoScopeRequired` ยกเว้นแค่ scope ไม่ได้ยกเว้น identity
- พิสูจน์: whoami ตรง pod = 401 ใน 40ms (ไม่ถึง bcrypt) · ใส่ `X-Api-Key-Id: dummy` + `X-Company-Id: dummy` = **200 ใน 252ms** คืนค่าถูกหมด
- suite เขียวเพราะ `TestRequireApiKeyScope_whoamiNeedsNoScope` ส่ง identity header ที่ของจริงไม่มีทางมี — test เขียนจาก mental model เดียวกับบั๊ก
- fix = map `v2CredentialVerification` + `return c.Next()` ก่อนด่าน identity (handler verify bcrypt เองอยู่แล้ว)

**🔴 OC-4467 (ส่ง SRE):** `/.well-known/scopes` 200 ที่ pod แต่ **404 ที่ gateway** — rule.yaml จับแค่ `/v1<.*>` กับ `/v2/openapi<.*>` ⇒ FE ยัง hardcode scope ต่อ = ปัญหาที่ OC-4426 ตั้งใจปิดยังเปิดอยู่

**ที่ verify ว่าใช้งานได้จริง:** สร้าง key ผ่าน backoffice-api ผ่านครบ 4 ด่าน (201) · revoke ได้ (204, หายจาก list) · OC-4431 redaction ทำงานจริงบน dev (log โชว์ `X-Api-Key: [redacted]`)

⚠️ **scope ownership เคาะแล้ว (user 2026-08-28):** ความหมายของ scope + การบังคับ = ของ service ที่กิน key **keyring ห้ามรู้ domain ของผู้เรียก** → ข้อเสนอ "ให้ keyring มี scope catalog" ถอนออกจาก artifact แล้ว เหลือแค่กฎ namespace (ทะเบียน app ที่เก็บแค่ชื่อ app + เจ้าของ)

## ✅ 2026-08-10 18:05 — bcrypt→shared module / MR !207 unblock / OC-4425 จุดสุดท้าย ปิดหมด

- **bcrypt → shared module**: ก่อนย้าย ตรวจ version-bump risk ก่อน (`git diff v1.7.3..v1.9.6` ทั้งเวอร์ชันและเจาะเฉพาะ package ที่ 2 service import จริง) → diff เจาะจงว่างเปล่า (ของจริงที่เปลี่ยนคือ `.claude/*` config + ไฟล์ใหม่ล้วนใน `member/`/`theme/`) = bump ปลอดภัย · สร้าง package `apikey/secret.go` (`HashSecret`/`VerifySecret`/`IsBcryptHash`, เอาเวอร์ชัน 3rdparty-api ที่มี legacy-plaintext fallback เป็นตัวหลัก) ใน repo `entity` branch `feature/oc-2275-apikey-secret-hashing` → **[MR !58](https://gitlab.sellsuki.com/sellsuki/oc2plus/line-crm/backend/entity/-/merge_requests/58)** → main (ยังไม่ merge — shared module กระทบ consumer อื่น ไม่ merge/tag เอง ต้องรอ user review) · **ค้างจริง**: หลัง !58 merge+tag (จะเป็น v1.9.7) ต้องกลับมาบั้ม go.mod ทั้ง backoffice-api + 3rdparty-api แล้วลบ `secret.go` ในเครื่อง ทั้งสองที่
- **MR !207 unblock**: user สั่ง unblock ตรง ๆ = ตีความเป็นการเคาะแล้ว (option b: merge as-is + follow-up) → เช็คก่อนว่า deploy-order gate (WS1-B migration) เคลียร์แล้วจริง (!73 merged to develop) → `glab mr update --ready` ปลด draft สำเร็จ (`merge_status: mergeable`) · สร้าง **[OC-4432](https://sellsuki.atlassian.net/browse/OC-4432)** แยกเก็บ concern bcrypt-cache perf ไว้ไม่ให้หายไปกับการ unblock · comment ปิดที่ OC-2273 (43455)
- **OC-4425 จุดสุดท้าย**: ไม่มี real staging key/secret และไม่ปลอม → ใช้ **source code ของ Ory Oathkeeper เอง** (`pipeline/mutate/mutator_header.go` → `AuthenticationSession.SetHeader` → `Header.Set` → `proxy/proxy.go: CopyHeaders` → `r.Header.Set`) พิสูจน์ว่า header mutator **Set (replace) ไม่ใช่ Add** — ปิด 3 ความเสี่ยงหลัก (fake scope/company/integration-id) ด้วยหลักฐานที่แน่นกว่า curl ครั้งเดียวด้วยซ้ำ (deterministic จาก source ไม่ใช่ sample 1 request) · **เจอ residual gap ใหม่**: rule ไม่ list `X-User-Id` ใน mutator headers map จึง**ไม่ถูก strip** — แต่เช็คแล้วไม่มีจุดไหนใน `/v2/openapi/*` อ่าน header นี้ (เฉพาะ v1 member helper ใช้) จึง inert วันนี้ → สร้าง **[OC-4433](https://sellsuki.atlassian.net/browse/OC-4433)** ให้ SRE เพิ่ม explicit strip ที่ rule.yaml กันไว้ล่วงหน้า · comment ปิดยาวที่ OC-4425 (43456)

**เหลือจริงหลังรอบนี้**: !58 (entity) ต้อง user merge+tag ก่อนบั้ม 2 consumer · OC-4433 (gateway rule.yaml) ต้อง SRE ทำ · OC-4430 (QA matrix, ต้องมี live stack) · NetworkPolicy ของ octoplus namespace (repo เปล่า ไม่รู้อยู่ไหน) — ทั้งหมดนี้ block อยู่นอกสิ่งที่ session นี้ทำได้เอง

## ✅ 2026-08-10 17:20 — ทุกข้อในคิวนี้ปิดแล้ว (รอบก่อนหน้า)

- **OC-2273**: ตัด `apikey.manage` ออก (10 scope) + sync สถานะ MR/migration ทั้งหมด (!73 รอ merge, !207 ยัง Draft รอเคาะ bcrypt cache, member-api !71 closed)
- **OC-2275**: แก้ 4 บรรทัด store/ร้าน → company/บริษัท + อัปเดต status table (self-service API key ✅ coded ไม่ใช่ ❌ แล้ว)
- **MR !207 (3rdparty-api)**: บล็อกด้วย Draft flag เท่านั้น — **ยังไม่ปลด** เพราะ bcrypt-cache concern ยังไม่แก้ในโค้ด รอ user เคาะทาง (เพิ่ม cache ก่อน vs merge+follow-up) ก่อน comment 43454
- **OC-4425 เกือบเสร็จ 95%**: user ชี้ repo `sellsuki/sre/configuration/api-gateway` (Ambassador) ถูกทาง → เจอตัวจริงคือ `sellsuki/sre/configuration/oc2plus` (Oathkeeper rule.yaml) — verified: bearer_token authenticator ยิง `X-Api-Key`/`X-Api-Secret` เข้า `/v2/openapi/auth/whoami` ของ service เอง (bcrypt จริง) แล้ว header mutator เซ็ต `X-Api-Key-Id`/`X-Company-Id`/`X-Api-Scope` จาก response · ไม่มี route อื่นทะลุตรง · curl จริงยืนยันว่า Oathkeeper reject ด้วย error shape ของตัวเอง (ไม่ใช่ของ service) ก่อนถึง backend เลย — งานจริง · **เหลือจุดเดียว** ต้องมี key+secret จริงของ staging ยิง spoofed `X-Company-Id` เทียบผล (ไม่มี ไม่ได้พยายามปลอม/เดา) → comment 43453
- **เหลือค้างจริง**: OC-4430 (QA matrix, ต้องมี live stack), bcrypt→shared module refactor (ยังไม่เริ่ม), NetworkPolicy ของ octoplus namespace (repo เปล่า ไม่รู้อยู่ไหน)

## 📌 สถานะล่าสุด 2026-08-10 16:12 — OC-4424+4425+4426 shipped ก้อนเดียว

**[MR !210](https://gitlab.sellsuki.com/sellsuki/oc2plus/line-crm/backend/oc2plus-line-crm-service-3rdparty-api/-/merge_requests/210)** branch `feat/oc-4424-4425-4426-scope-enforcement` → develop (Draft) — commit `0a03071`

**การค้นพบสำคัญที่เปลี่ยนแผน OC-4424:** `apiKey`-type security scheme ใน OpenAPI **ต้องมี scope array ว่างเสมอ** ตาม spec (scope เป็น concept ของ oauth2/openIdConnect เท่านั้น) — เขียน `- apiKey: [campaign.read]` ใน yaml ไม่มีทางไหลผ่าน oapi-codegen ได้ (`spec.gen.go` hardcode `SetUserValue(ApiKeyScopes, []string{})` ทุกครั้ง) → **เปลี่ยนแหล่งความจริงเป็น Go map** `v2RouteScopes` ใน `src/interface/fiber_server/middleware/require_scope.go` แทน

**สิ่งที่ shipped:**
- `RequireApiKeyScope` middleware mount บน **route group** `/v2/openapi` เท่านั้น (ไม่ใช้ `FiberServerOptions.Middlewares` — มันเป็น global บน `*fiber.App` เดียวกับ v1/system เพราะ codegen เรียก `router.Use(m)` ไม่มี path prefix)
- fail-closed 500 บน route ที่ไม่มี scope policy — พิสูจน์จริงด้วยการยิง route ที่ไม่ได้ตั้งค่า
- `scope_policy_test.go` parse yaml ตรง (ไม่ผ่าน codegen) assert ทุก apiKey-secured operation มี entry — พิสูจน์จริงด้วยการแอบใส่ fake path
- `model.PrincipalType` + `CRMApiKey.Type` ประทับที่ 2 จุดสร้าง identity จริง (`GetApiKeyIdentityFromHeader`, `postgresApiKey.ToCRMAPIKey`)
- `GET /.well-known/scopes` (plain fiber route, ไม่ผ่าน codegen) อ่านจาก `middleware.EnforcedScopes()` — **map เดียวกับที่ enforce จริง ไม่มี list ที่สอง** พิสูจน์ด้วยการเพิ่ม description ของ scope ที่ไม่ enforce แล้ว test แดง
- 3rd bug ที่เจอ (`c.Route().Path` คืนค่า route ของ middleware เองไม่ใช่ endpoint ปลายทางเมื่ออยู่ใน Group — ใช้ `c.Path()` แทน) — ถ้า WS3 (OC-4398) merge มาแล้วมี path param ต้องแก้ทั้ง middleware และ test คู่กัน (เขียน comment เตือนไว้ในโค้ดแล้ว)

✅ **OC-4425 อัปเดต 2026-08-10 หลัง user ชี้ repo — เจอ gateway config จริง ตรวจได้ 90%** (comment 43453)
- **`sellsuki/sre/configuration/oc2plus`** `manifest/{env}/rule.yaml` = Ory **Oathkeeper** access rules (ตัวจริงที่ทำ AuthN) — `sellsuki/sre/configuration/api-gateway` (ที่ user เดา) เป็นแค่ Ambassador Mapping ที่ forward เข้า `oathkeeper-proxy.share:4455` อีกที
- เส้น partner จริงคือ hostname **`openapi.poshmedica.co.th/v2/openapi<.*>`** (ตรงกับ `PROVIDER_CODE=poshmedica`) **ไม่ใช่** `crmapi.oc2.plus` (อันนั้นเป็น v1 member-session คนละ rule คนละ auth model)
- rule: `bearer_token` authenticator อ่าน `X-Api-Key` จาก client, forward `X-Api-Key`+`X-Api-Secret` ไปเรียก **`check_session_url: .../v2/openapi/auth/whoami`** ของ 3rdparty-api เอง (bcrypt verify จริงตาม `GetActiveApiKey`) แล้ว `header` mutator เซ็ต `X-Api-Key-Id`/`X-Company-Id`/`X-Api-Scope` จาก response (`api_key_id`/`company_id`/`scope` — ตรง JSON tag `AuthWhoAmIResponse` เป๊ะ)
- ✅ ไม่มี Ambassador Mapping อื่นชี้ตรงเข้า `oc2plus-line-crm-service-3rdparty-api-svc` เลย (grep ทั้ง repo ทุก env) + k8s Service เป็น ClusterIP อยู่แล้ว
- ⚠️ **เหลือจุดเดียว** — ไม่ได้ยิง curl จริงยืนยันว่า Oathkeeper `header` mutator **Set (replace)** หรือ **Add (append)** ค่าที่ client ปลอมมา (ความมั่นใจสูงว่าเป็น Set ตามพฤติกรรมที่ mutator นี้ตั้งใจทำ แต่ไม่ได้ verify runtime) — แนะนำ curl staging-th ด้วย `X-Company-Id` ปลอม + key จริง ดู response
- `sre/configuration/networkpolicy` **repo เปล่า** —ยังไม่รู้ network policy จริงของ namespace `octoplus` อยู่ที่ไหน (ปัจจัยรอง)

Jira comments: 43450 (OC-4424) · 43451 (OC-4425, partial) · 43452 (OC-4426)

**verification:** `go build ./...` clean, full suite เขียว (รวม ~40 `model.CRMApiKey{}` literal ใน use_case tests เดิม — ไม่ต้องแก้เพราะ `Type` เช็คแค่ที่ interface/repository boundary ไม่ใช่ใน `checkIdentityApiKey`)

**Audit เต็มของ OC-2275 API-key workstream (2026-08-09) — user อ่านแล้วบอกว่า "วิเคราะห์ถูกต้องทั้งหมด อยากทำทั้งหมด" แต่ token ไม่พอ จึงบันทึกไว้ทำต่อ**
สำเนาสำหรับทีมอยู่ที่ **comment ของ OC-2275** (ดู index คอมเมนต์ด้านล่าง) — ถ้าไฟล์นี้หายให้ไปอ่านที่นั่น

## สถานะ ณ 2026-08-09: ยังไม่มีอะไร merge เลยสักรีโป (ไม่มี merge ผิดที่)

| repo | MR | source → target | สถานะ |
|---|---|---|---|
| 3rdparty-api | !207 | `feat/oc-2273-apikey-crud` → develop ✅ | Draft |
| backoffice-api | !502 | `feat/oc-2273-apikey-mgmt` → develop ✅ | Draft |
| FE linecrm-backoffice | !527 | `feat/oc-2273-apikey-ui` → develop ✅ | Draft |
| member-api | !71 | 🔴 `feat/oc-2273-apikey-migration` → **`feat/oc-4289-post-members`** | ไม่ใช่ draft |

## 🔴 งานที่ user อนุมัติแล้ว — ยังไม่ได้ลงมือ (ทำต่อจากตรงนี้)

### 1. WS1-B migration ย้ายรีโป + ปิด MR !71 ✅ อนุมัติ
- ผิดยังไง: WS1-B อยู่ที่ `member-api/migrations/003_add_api_key_columns.up.sql` แต่**บ้านจริงของ schema CRM คือรีโป `git@gitlab.sellsuki.com:sellsuki/oc2plus/line-crm/migration/oc2plus-line-crm-migration.git`** ซึ่งเป็นที่ที่ `api_key` ถูกสร้าง (`20250204031115-create-table-api-key-up.sql`)
- คอมเมนต์ในไฟล์เขียนเหตุผลผิดว่า member-api คือ "CRM DB's version-controlled migration set" — ไม่จริง · member-api ไม่มี runner (ไฟล์เขียนเองว่า "no automated runner in this repo") · เลข 001/002/003 คนละระบบกับ db-migrate timestamp
- MR !71 target `feat/oc-4289-post-members` ซึ่ง **ยังไม่ merge (+3 จาก develop) และไม่มี MR ของตัวเอง** → WS1-B ไปไม่ถึง develop ทางนี้ · branch ยังลาก 3 commit ของ OC-4289 (LINE-agnostic members + OTP) ติดมา
## 📌 สถานะรวม 2026-08-10 (user สั่ง implement OC-4431/4424/4425/4426/4430 + งานค้าง)

**เสร็จ:**
- migration repo **!72 merged** (sync develop←main) · **!73 พร้อม merge** (WS1-B + key varchar 64) · member-api **!71 ปิดแล้ว**
- ไฟล์ `member-api/migrations/003_add_api_key_columns.*` **ไม่เคยไปถึง develop** (อยู่แต่บน branch ที่ MR ปิด) → ไม่ต้องลบอะไร ปิดข้อนี้ได้
- **OC-4431 เสร็จ → 3rdparty-api MR !209** branch `fix/oc-4431-redact-request-log` → develop · allow-list redaction + drop body บน `/auth/whoami` `/oauth/token` + test 4 ตัวที่พิสูจน์แล้วว่าแดงตอนถอด redaction

**ยังไม่เริ่ม (เรียงตามลำดับที่ควรทำ — ทั้งหมดอยู่ repo 3rdparty-api ยกเว้นที่ระบุ):**
1. **OC-4424 + OC-4425 + OC-4426 ควรทำเป็นก้อนเดียว** — ใช้ `Principal`/`CredentialVerifier`/scope catalog ร่วมกัน แยก MR จะขัดกันเอง · 4424 = ประกาศ scope ใน `v2_openapi.yaml` (ตอนนี้ `- apiKey: []` ว่าง) + middleware + test ที่แดงเมื่อลืมประกาศ · 4425 = principal type + header trust audit (ต้องหา gateway config) · 4426 = `/.well-known/scopes` + FE เลิก hardcode
2. **bcrypt → shared module `line-crm/backend/entity`** (repo แยก) แล้วลบ `api_key_repository/secret.go` ทั้ง backoffice-api + 3rdparty-api
3. **OC-4430 QA matrix** — repo `testing/oc2plus-line-crm-automate-testing` ทำหลัง 4424/4425
4. แก้การ์ด: OC-2273 ตัด `apikey.manage` · OC-2275 description 4 บรรทัด (7/9/103/160)

🔴 **OC-2274 "ดู secret ไม่ได้" = ดีไซน์ถูกแล้ว ไม่ใช่บั๊ก** — secret เก็บเป็น bcrypt hash จึงกู้คืนไม่ได้เชิงคณิตศาสตร์ · create คืน plaintext ครั้งเดียวจริง (`api_key_v1_model.go:54` + FE `ApiKeyCreate.vue:382`) · detail ไม่โชว์ = ตาม Business Rule ของ OC-2274 เอง · ถ้าอยากให้ user ได้ secret ใหม่ ต้องทำ **regenerate/rotate** ไม่ใช่ "ดูซ้ำ"

## ✅ ข้อ 1 ทำเสร็จแล้ว 2026-08-10

- **migration repo !73** (Draft) `feat/oc-2273-ws1b-api-key-management-columns` → **develop** — WS1-B ฉบับย้ายบ้าน + แก้ `key varchar(32)`→64 · **merge หลัง !72 เท่านั้น**
- **migration repo !72** `chore/sync-develop-with-main` → develop — 🔴 **main กับ develop ของรีโปนี้ unrelated histories** (main = 1 commit squashed 2026-07-31 มี 77 migrations · develop = history เดิมปี 2024 มี 6 migrations, ไม่มีไฟล์ไหนที่ main ไม่มี, ต่างแค่ README+package.json) → merge `--allow-unrelated-histories` เอาฝั่ง main, tree เท่ากับ main เป๊ะ, ไม่ force push
- **member-api !71 ปิดแล้ว** พร้อมคอมเมนต์ชี้ทางไป !72/!73 (note_83991)
- ⏳ เหลือ: **ลบ `member-api/migrations/003_add_api_key_columns.{up,down}.sql`** ไม่ให้มี DDL ของ api_key สองชุด

**▶ ความคืบหน้าเดิม 2026-08-09:** สร้าง+push แล้ว branch `feat/oc-2273-ws1b-api-key-management-columns` ในรีโป migration (commit `58a96c7`, 3 ไฟล์: `20260809000000-alter-table-api-key-add-management-columns` .js + sqls up/down) · verify แล้วว่า up.sql รันผ่านบน local oc2plus_crm และ idempotent
🔴 **ยังไม่เปิด MR — ติดคำถาม target branch**: รีโป migration นี้ **`develop` ตายตั้งแต่ 2024-03-07 (6 migrations) ส่วน `main` คือของจริง (77 migrations, active 2026-07-31)** → ขัดกฎ "OC2Plus merge to develop เท่านั้น" ต้องให้ user เคาะก่อน
⚠️ **ตัดสินใจระหว่างทาง**: `expires_at` ใช้ **timestamptz** (ไม่ตาม convention `without time zone` ของตาราง) เพราะ GetActiveApiKey เทียบคอลัมน์กับ Go `time.Time` ที่ driver ส่งเป็น timestamptz → ถ้าเป็น without-tz Postgres จะ cast ด้วย session TimeZone แล้วอ่าน UTC เป็น Asia/Bangkok = **เพี้ยน 7 ชม.**
ยังค้าง: เปิด MR · ปิด !71 · ลบไฟล์ออกจาก member-api
- **สิ่งที่ต้องทำเดิม**: สร้าง migration ในรีโป migration ด้วยชื่อ db-migrate timestamp (`YYYYMMDDHHMMSS-alter-table-api-key-*.sql` up+down) เนื้อหา = ALTER 3 คอลัมน์ + partial unique index **+ `ALTER TABLE api_key ALTER COLUMN key TYPE varchar(64)`** (บั๊ก key 35 ตัวเกิน varchar(32)) → เปิด MR ที่รีโปนั้น → ปิด !71 พร้อมคอมเมนต์ชี้ไป MR ใหม่ → ลบไฟล์ออกจาก member-api

### 2. bcrypt/scope ซ้ำ 2 รีโป — **คำตอบที่ให้ user แล้ว**
ทั้ง 3 รีโปใช้ shared module `gitlab.sellsuki.com/.../line-crm/backend/entity` อยู่แล้ว (backoffice pin v1.7.3, 3rdparty pin v1.7.1)
- **bcrypt hash/verify → ย้ายเข้า shared entity module** (ที่เดียว) · เป็น pure function ที่สองฝั่งต้องตรงกัน byte-for-byte · วันนี้ซ้ำอยู่ 2 ที่ (`api_key_repository/secret.go` ทั้ง backoffice=hash และ 3rdparty=verify) ผูกกันด้วย**คอมเมนต์อย่างเดียว** ("changing it here breaks cross-service auth") · ข้อดีของ shared module: version skew กลายเป็น compile error ไม่ใช่ drift เงียบ
- **scope catalog → ห้ามเอาเข้า shared module** (version skew = drift เงียบอีกแบบ) → **3rdparty-api เป็นเจ้าของ + ประกาศผ่าน registry `/.well-known/scopes` (OC-4426)** และ **backoffice-api เลิกมี enum ของตัวเอง** · ⇒ OC-4426 กลายเป็น blocker ของการลบ enum ซ้ำ

### 3. แก้ OC-2273 ตัด `apikey.manage` ออกจากตาราง scope ✅ อนุมัติ
การ์ดยังลิสต์เป็น scope ที่ 11 + บอกว่า gate ทั้ง 4 endpoint · โค้ดจริงหลัง refactor **ตัดออกจาก catalog โดยตั้งใจ** แล้ว gate ด้วย session/company permission แทน (เขียนไว้ใน `use_case/model/api_key.go` comment) → **แก้การ์ด ไม่ใช่แก้โค้ด**

### 4. FE อยู่ที่เดิม (CRM backoffice) ✅ user เห็นด้วย — บันทึกเป็น decision ใน OC-2275
เหตุผล: CCS3 = tenant admin (Company/User/UserGroup/Consent/Pay/Product/Location) ไม่แตะ CRM · key ชุดนี้ให้สิทธิ์ CRM domain ล้วน + permission ชื่อ `oc2plus.apikey.manage` · **ย้ายเมื่อ** central key service เกิดจริง **และ** key ครอบ >1 domain

## ข้อมูลอ้างอิงที่ใช้ตอน audit (กันต้องไปขุดใหม่)

- **scope catalog อยู่ 4 ที่**: backoffice-api model=10 (ฝั่ง mint) · 3rdparty-api model=3 (ฝั่ง enforce) · FE `entities/apikey.ts`=10 · การ์ด=11 → ฝั่งแจก 10 ฝั่งบังคับ 3 (ไม่ใช่ privilege escalation แต่เป็นสัญญาที่ทำตามไม่ได้)
- **DDD layering ถูก** (interface → use_case → repository, model ใน use_case/model, mock ใน use_case/repository) · ที่ผิดคือ bcrypt อยู่ใน repository layer + ซ้ำ 2 รีโป
- ⚠️ **backoffice-api (บ้านชั่วคราวของ CRUD) มี enum ของ scope เอง = anti-pattern เดียวกับที่เคาะว่า central ห้ามทำ** — ห้ามยกโค้ดนี้ไป central ตรง ๆ
- FE commit `9f45c1b` message ยังเขียน "Store Admin" (push แล้ว แก้ไม่ได้ — MR title แก้ได้)
- CCS3 มี `lib/entities/store/store.ts` = stub 55 bytes `{id,name}` (ไม่เกี่ยวกับงานนี้)

## Index คอมเมนต์บน OC-2275 (บริบทสะสมทั้งหมด)
`43431` เส้นแบ่ง central↔service · `43433` แก้ข้อมูลที่ผมเคยสรุปผิด (scope **enforce จริง** 4/4) · `43436` hot path/SPOF/bcrypt 47ms/JWKS · `43438` terminology company ไม่ใช่ store
บน OC-2273: `43437` เตือน reviewer MR !207 เรื่อง bcrypt ต้องมี cache ก่อน merge
บน OC-4398: `43432` ถอนคำแย้งของผม (การ์ดถูก)

ดู [[project_oc2plus_3rdparty_apikey_gap]] · [[reference_oc2plus_company_not_store]] · [[feedback_verify_absence_claims]] · [[project_oc2plus_apikey_local_run]]
