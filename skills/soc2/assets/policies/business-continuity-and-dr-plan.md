# Business Continuity & Disaster Recovery Plan

| | |
|---|---|
| **Owner** | [PLACEHOLDER: role] |
| **Approver** | [PLACEHOLDER: role] |
| **Review cadence** | Annual, and after any real DR event |
| **Last reviewed** | [PLACEHOLDER: date] |
| **Version** | [PLACEHOLDER: 1.0] |

> **Auditor note:** A backup that has never been restored isn't evidence of anything — auditors
> look for a dated restore test result, not just a backup job succeeding. See
> `references/technical-controls.md`.

## 1. Purpose

Defines how [Your Company, Inc.] maintains and recovers critical systems and data in the event
of a disruption.

## 2. Scope

Applies to [PLACEHOLDER: system boundary] and the infrastructure it depends on.

## 3. Recovery objectives

| System | RPO (max data loss) | RTO (max downtime) |
|---|---|---|
| [PLACEHOLDER] | [PLACEHOLDER] | [PLACEHOLDER] |

## 4. Backup strategy

- **What's backed up:** [PLACEHOLDER]
- **Frequency:** [PLACEHOLDER]
- **Retention:** [PLACEHOLDER]
- **Storage location:** [PLACEHOLDER — ideally isolated from the primary environment]

## 5. Restore testing

Restore is tested at least [PLACEHOLDER: annually], with results (success/failure, time taken)
recorded as evidence.

## 6. Disruption response

1. **Detect and declare** — [PLACEHOLDER: criteria for invoking this plan].
2. **Execute recovery** — [PLACEHOLDER: steps, systems restored in what order].
3. **Communicate** — [PLACEHOLDER: who is notified, internally and externally].
4. **Post-event review** — documented within [PLACEHOLDER: N days].

## 7. Review history

| Version | Date | Reviewed by | Changes |
|---|---|---|---|
| [PLACEHOLDER] | [PLACEHOLDER] | [PLACEHOLDER] | Initial version |
