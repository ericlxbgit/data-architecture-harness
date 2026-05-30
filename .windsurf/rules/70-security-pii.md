---
trigger: model_decision
description: Apply when the work involves PII, sensitive data, access control, masking, or compliance (GDPR, HIPAA, SOC2, PCI). Cascade should pull this in when a column suspected to be personal/financial/health data is being added, joined, exposed, or moved across environments.
---

# Security, PII & Sensitive Data

## Classification — required for every column

Every column lives in one of these tiers. Tag at modeling time, not later.

| Tier | Name        | Examples                                          | Handling                                |
|------|-------------|---------------------------------------------------|-----------------------------------------|
| 0    | Public      | Country code, product SKU                         | No restriction.                         |
| 1    | Internal    | Internal user IDs, internal cost data             | No external sharing.                    |
| 2    | Confidential| Revenue, salaries, contract terms                 | Role-based access. No dev copies.       |
| 3    | PII         | Email, name, phone, IP, device ID                 | Masked outside prod. Audit access.      |
| 4    | Sensitive PII| SSN, payment card, health, biometric, gov ID    | Tokenized at ingest. Never in marts raw.|

In dbt this looks like:

```yaml
columns:
  - name: customer_email
    meta:
      classification: pii
      pii_type: email
      masking: hash_sha256
```

## Hard rules

- **Tier 4 data** (`Sensitive PII`) is **tokenized at ingest**. The raw value
  never lands in the warehouse. The token is the join key everywhere.
- **Tier 3 data** in marts is hashed or masked except in views explicitly
  approved for the authorized role.
- **Dev and sandbox** environments **never** receive un-masked Tier 3 or
  Tier 4 data. Use synthetic data or masked subsets.
- **Logs** never contain Tier 3+ values, even truncated. If a log line could
  contain one, redact it explicitly.

## Joins that create identifiability

A join that combines two Tier 2 columns can produce a Tier 3 result. Examples:

- "Birth date" + "ZIP" + "gender" → re-identifiable in many populations.
- Coarse location + timestamp + device class → re-identifiable.

If a model creates such a join, flag it and ask whether the result should be
classified up a tier and access-restricted.

## Right-to-be-forgotten / deletion

- Deletion requests propagate from raw downstream through all derived models.
- A deletion that bypasses lineage (e.g. only deletes from a mart) is a bug.
- Snapshots and SCD2 tables require an explicit deletion strategy: either
  hard-delete with audit log, or replace-with-tombstone.

## Access control

- Permissions live in IaC, not in clicks. Every grant has an owner and a
  reason in code review.
- The principle is least privilege. Default deny. Explicit allow per role.
- Service accounts have scoped roles; humans have roles via groups, not
  individual grants.

## When to refuse

Refuse and escalate when asked to:

- Copy un-masked PII to a sandbox or developer environment.
- Build a model whose grain reveals individuals from aggregate data.
- Disable a PII tag to "make the lineage diff smaller."
- Add a deny-listed identifier (e.g. raw SSN) as a join key in marts.
