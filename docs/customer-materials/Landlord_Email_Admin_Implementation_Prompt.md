# Implementation prompt for delegated customer email administration

Copy the prompt below into a coding assistant with access to `NovaWorksLLC/Landlord-TenantSite`. This is a development specification, not authorization to change production accounts or send customer emails. Use it alongside `Nova_Works_Email_and_Admin_Setup_Guide.docx`.

## Prompt

You are updating the Nova Works landlord application so an authorized assistant can help Jesse Estes fulfill customer email requests and administer approved company licenses. Review the current repository before editing. Preserve existing tenant request workflows, company isolation, licensing behavior, billing integrations, and historical records.

Work on the existing branch. Do not create a new branch. Implement and test repository changes, but do not deploy database migrations to production, change live customer entitlements, connect real mailboxes, or send live messages as part of development. Use test data and document the final production setup steps.

### 1 Review existing capabilities and define the implementation scope

Read repository instructions, authentication and authorization code, database schemas and migrations, server functions, provider administrator UI, company owner UI, license issuance and redemption, billing events, and audit mechanisms. Identify the deployed hosting and server execution environment instead of assuming which provider runs it.

Relevant previously reviewed files include `js/ui/views/adminplus.js`, `js/ui/views/owner.js`, and `js/api/supabase.js`; these paths are starting points, not a complete inventory. The administration UI already includes reason prompts, support lookup, billing event handling, and other administrator functions. Reuse the existing mechanisms where appropriate. Do not duplicate audit or permission infrastructure without checking its server enforcement.

Deliver a brief inventory distinguishing existing working features, missing features, and features requiring configuration. Verify whether more than one provider administrator can be provisioned safely and whether current roles support delegated access. Do not equate a customer's owner or manager account with the Nova Works provider administrator.

Build the assisted administration path first. Make mailbox ingestion and unattended processing an optional second phase behind a disabled feature flag. The first phase must work without an email plugin: an operator can record a customer request, review a proposed action, and execute an authorized change through the site. No live email provider credentials are needed to test that phase.

### 2 Add a restricted delegated administration identity

Support a separate delegated operator account with its own login and audit identity. Keep the primary Nova Works administrator in control of provisioning, permission changes, revocation, and recovery. Reuse the authentication provider and available MFA features.

Define narrow permissions for company lookup, entitlement lookup, request review, issuing already authorized licenses, and permitted invitation actions. Keep refunds, changes to prices or contracts, deletion, ownership transfer, unrestricted tenant exports, and permission administration outside the default delegated role.

Enforce authorization in every server endpoint and applicable database policy. Hiding controls in the browser is insufficient. Support account disabling and effective session or token revocation. Do not hardcode a username, password, or provider administrator email in client code. Do not require sharing the primary owner's credentials with the assistant.

### 3 Record customer authority and commercial entitlement

Provide a controlled register of company identifiers, authorized owner contacts, approved delegates, and permitted request types. Only an authorized operator may update this register. Changing an authorized contact must not be approved solely from an email requesting the change.

Record or reference the authoritative signed order, purchased account allowance, active unit allowance, term, effective dates, owner pricing treatment, and payment evidence. Preserve the existing billing source of truth. Never treat a customer's statement that payment was made as verified payment.

Separate company rental unit capacity from owner and manager account allocation. Count a unit once across the company even if multiple managers can access it, subject to the actual signed order. Do not impose the earlier illustrative 25 or 100 unit pricing tiers or overwrite existing license limits. Identify any difference between current enforcement and the proposed company level model; implement an explicit compatible migration only if required and document its effect.

License issuance must not silently create a free entitlement, bill an unapproved amount, or override special terms. Distinguish an invitation issued, an account provisioned, and a key redeemed.

### 4 Implement an explicit authorization policy

Store versioned operating rules controlled by the owner: permitted action types, applicable companies, maximum quantity per request, permitted terms, commercial constraints, required verification evidence, and rules for replying to customers. Missing or conflicting policy values route the request to review.

Use a policy evaluator on the server. The assistant may propose a structured action; it must not grant itself authority, change the policy, select a more permissive version, or override failed checks. Record which policy version and entitlement were used to authorize the action. Recheck policy and entitlement immediately before executing an approved change.

Initial default behavior: allow reading within assigned scope and preparing drafts; require review of writes until the owner explicitly enables a defined action. Support a narrow standing rule for granting a license already purchased by an authenticated authorized company requester. Revocation, reassignment, new charges, discounts, refunds, and contract exceptions remain reviewed by default.

### 5 Build a customer request queue and review screen

Add request intake with the source mailbox or manual source, stable message identifier where available, thread identifier, company, requester, requested action, quantity, intended recipient, effective date, evidence references, and operator notes. Keep original request content distinct from interpreted action fields.

Track received, needs information, pending review, approved, executing, changed, communication pending, completed, rejected, and failed states, or an equivalent well defined model. Distinguish a failed change from a completed change whose reply failed. Include assigned operator, state timestamps, and a clear exception reason.

Display the correct company and current allocation beside the proposed result. Approval must bind to a specific action payload, company, recipient, quantity, policy version, and relevant current state. Expire or invalidate approval when those material fields change. Do not implement a generic approval that can be reused for unrelated actions.

Offer an assisted workflow to create a request, review it, approve it if authorized, execute it, verify the resulting state, and prepare a customer response. Provide filters for pending review, failures, and completed requests. Keep tenant sensitive information out of broad company lookup results unless explicitly needed and authorized.

### 6 Make changes safe to retry and auditable

Use an idempotency key tied to the source message and logical action. Enforce uniqueness server side; do not rely only on a UI check. Prevent simultaneous workers from issuing the same entitlement twice. Use transactions or appropriate locking for capacity checks and issuance; recheck allowance at execution time.

Persist the operation result so retrying after a timeout retrieves or reconciles the prior result before attempting another issuance. Queue outgoing communication after a successful change using a durable outbox or equivalent mechanism. A send failure must not cause another license to be issued.

Record the authenticated actor or service identity, initiating customer request, company, reason, policy version, approval identity if required, previous and resulting state, timestamp, and outcome. Make records tamper resistant against ordinary operators using existing audit facilities where possible. Do not put passwords, tokens, full invitation secrets, or unnecessary tenant documents in logs.

Verify the resulting site state after execution. Report a partial or failed result accurately. Preserve existing case timelines and PDF fingerprints without implying those features prove the truth of customer requests.

### 7 Provide a narrow integration interface where needed

For the assisted browser workflow, accessible, clearly labeled UI controls and dependable server authorization may be sufficient. Avoid requiring a custom integration merely to support supervised site use.

For API based operation, expose versioned, schema validated actions limited to request creation, company and entitlement lookup, action proposal, authorized execution, status lookup, and reply preparation. Use scoped authentication and revocable credentials. Do not expose arbitrary SQL, unrestricted table writes, arbitrary URL fetching, or a general administrator command endpoint.

Never place a Supabase service role key, OAuth refresh token, email provider secret, or integration signing secret in browser code, a prompt, or the repository. If elevated server credentials are needed internally, keep them in server secrets and require caller authentication and authorization before use. Document credential rotation, environment variables, and account disablement.

An MCP integration is optional if the intended assistant environment supports it. Document available operations and authentication; do not claim that creating endpoints automatically makes them available to ChatGPT.

### 8 Add optional mailbox ingestion and reply delivery

Keep this phase separate and disabled until configured. Confirm the actual mailbox provider. Gmail and Microsoft email connectors may differ in reading, drafting, sending, attachments, shared mailbox access, and triggers. Do not assume that ChatGPT has a send tool or continuous browser session.

If the server receives mailbox events, verify provider signatures or subscription authentication, validate the intended mailbox, and deduplicate notifications and message processing. Retrieve only the required thread context. Sender matching is one check; use provider authenticated message information where available and route identity conflicts or sensitive requests to additional verification. A recognized display name alone is insufficient.

Treat message bodies and attachments as untrusted request content. Never allow instructions in them to alter the operating policy, reveal secrets, access another company, or redirect admin actions to a supplied URL. Do not execute attachment code. Any model classification output must pass a strict action schema and the server policy checks.

Draft replies by default. Enable sending only with an explicit policy and confirmed provider capability. Verify the recipient against the authorized customer record and thread before sending. Use approved templates; do not invent prices, contract promises, payment confirmation, or delivery guarantees. Deliver invitation secrets only through the intended protected invitation channel, not routine logs. Track draft, queued, sent, and failure states; record delivery confirmation only if actually available.

### 9 Test meaningful security and failure cases

Use a demonstration company and controlled recipients. Test server authorization directly in addition to UI behavior:

- A delegated account cannot grant itself more permissions or access excluded company or tenant records.
- An approved request within a purchased allowance creates exactly one license with the correct recipient and term.
- Unknown senders, wrong companies, expired policies, missing evidence, and exceeded allowances make no unapproved change.
- Owner and manager allocation obey the authoritative order and company unit counts do not multiply across managers.
- Duplicate emails, repeated notifications, simultaneous workers, retries, and timeouts cannot duplicate issuance.
- A successful change followed by a send failure resumes communication without another change.
- Approval becomes invalid when the proposed recipient, company, quantity, or relevant entitlement changes.
- Disabling the delegated account or revoking an integration credential blocks subsequent operations.
- Malicious instructions in an email or attachment cannot bypass authorization or expose secrets.
- Existing tenant submissions, manager assignments, licensing, billing, and record export workflows still behave correctly.

Use the repository's existing test framework. Add meaningful tests for authorization, transactions, state transitions, and idempotency; do not add tests that only mirror UI text.

### 10 Deliver a reviewable implementation and setup instructions

Provide the final change summary, configuration requirements, migrations and rollback considerations, tests run, limitations, and production activation steps. Include a simple owner guide covering delegated account provisioning, policy setup, customer contact verification, approving exceptions, reviewing logs, and disabling access.

Separate implemented functionality from features still requiring mailbox consent, secrets, hosting configuration, or assistant integration. Do not enable automation or send email until explicitly authorized. If a platform limitation blocks the full workflow, deliver the working assisted path and identify the exact remaining dependency.
