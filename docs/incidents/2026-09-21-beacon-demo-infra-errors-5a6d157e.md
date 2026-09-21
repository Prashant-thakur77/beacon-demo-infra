# Postmortem: beacon-demo-infra-errors

Incident `5a6d157e-ef0a-4a9c-845c-3120abcc399d` · 22:11:31 IST · status **remediating**

## Summary

The security group sg-08861ee93fee43c6f is missing an ingress rule (tcp/5432-5432 from sg-01eaf3479667c9284) that the golden snapshot says should exist; the service behind it cannot reach its dependency.

## Timeline

| Time (IST) | Event | Detail |
|---|---|---|
| 22:11:26 IST | alarm_received | trigger_type=alarm, alarm_name=beacon-demo-infra-errors |
| 22:11:29 IST | logs_fetched | groups=1, lines=561 |
| 22:11:31 IST | diagnostics_ran | suggested_action=sg.restore_ingress, missing_rules=1 |
| 22:11:31 IST | changes_checked | count=33, top=RevokeSecurityGroupIngress, source=ledger |
| 22:11:31 IST | model_unavailable | reason=triage: litellm.BadRequestError: BedrockException - {"message":"Operation not allowed"} |
| 22:11:31 IST | rca_ready | model=us.amazon.nova-2-lite-v1:0, status=High, suggested_action=sg.restore_ingress |
| 22:11:31 IST | sns_sent |  |
| 22:13:52 IST | fix_proposed | fix_id=1, blast_radius=1 ingress rule on 1 security group: allow tcp/5432-5432 from sg-01eaf3479667c9284 into sg-08861ee93fee43c6f. Nothing else changes., dry_run={'ok': True, 'role': 'arn:aws:sts::283146810291:assumed-role/beacon-remediator-beacon/beacon-remediate-beacon', 'detail': 'Request would have succeeded, but DryRun flag is set.', 'code': 'DryRunOperation'}, blast_radius_spoken=One change: restore the missing security-group rule, the tcp port 5432 rule from the application security group into the database security group. Nothing else changes., action=sg.restore_ingress |
| 22:14:59 IST | approved | "" via assemblyai |
| 22:15:00 IST | executing | approval_id=cf93f5f5-2b9c-4637-8b6d-397550507aae, params={'to_port': 5432, 'source_group_id': 'sg-01eaf3479667c9284', 'from_port': 5432, 'ip_protocol': 'tcp', 'group_id': 'sg-08861ee93fee43c6f'}, action=sg.restore_ingress |
| 22:15:00 IST | executed | executed_at=2026-09-21T16:45:00.474909+00:00, code=Authorized, detail=rule added |
| 22:15:31 IST | verify_attempt | attempt 1 · not yet · alarm_ok_after_fix=no, metric_zero=no, postcondition=ok |
| 22:16:01 IST | verify_attempt | attempt 2 · not yet · alarm_ok_after_fix=no, metric_zero=ok, postcondition=ok |
| 22:16:32 IST | verify_attempt | attempt 3 · not yet · alarm_ok_after_fix=no, metric_zero=ok, postcondition=ok |

Alarm to recovery: not recovered.

## Root cause

RevokeSecurityGroupIngress by user/hackathon-judge-temp at 2026-09-21T16:38:34Z on sg-01eaf3479667c9284, sg-08861ee93fee43c6f shortly before the alarm.

Evidence:

- `- MISSING ingress rule: tcp 5432-5432 from sg-01eaf3479667c9284 into sg-08861ee93fee43c6f`
- `2026-09-21T16:39:45.280000+00:00 2026-09-21T16:39:45 ERROR CRITICAL: Database unreachable. Host=beacon-demo-db.c0bg0ac4umkd.us-east-1.rds.amazonaws.com:5432 consecutive_failures=9 last_error='connection to server at "bea`
- `2026-09-21T16:39:52.828000+00:00 2026-09-21T16:39:52 ERROR CRITICAL: Database unreachable. Host=beacon-demo-db.c0bg0ac4umkd.us-east-1.rds.amazonaws.com:5432 consecutive_failures=10 last_error='connection to server at "be`
- `2026-09-21T16:40:00.858000+00:00 2026-09-21T16:40:00 ERROR CRITICAL: Database unreachable. Host=beacon-demo-db.c0bg0ac4umkd.us-east-1.rds.amazonaws.com:5432 consecutive_failures=11 last_error='connection to server at "be`
- `2026-09-21T16:40:08.701000+00:00 2026-09-21T16:40:08 ERROR CRITICAL: Database unreachable. Host=beacon-demo-db.c0bg0ac4umkd.us-east-1.rds.amazonaws.com:5432 consecutive_failures=12 last_error='connection to server at "be`
- `2026-09-21T16:40:16.548000+00:00 2026-09-21T16:40:16 ERROR CRITICAL: Database unreachable. Host=beacon-demo-db.c0bg0ac4umkd.us-east-1.rds.amazonaws.com:5432 consecutive_failures=13 last_error='connection to server at "be`

## What changed

- `RevokeSecurityGroupIngress` by user/hackathon-judge-temp at 22:08:34 IST on sg-01eaf3479667c9284, sg-08861ee93fee43c6f
- `AuthorizeSecurityGroupIngress` by beacon remediation at 21:18:47 IST on sg-01eaf3479667c9284, sg-08861ee93fee43c6f
- `AuthorizeSecurityGroupIngress` by beacon remediation at 21:19:54 IST on sg-01eaf3479667c9284, sg-08861ee93fee43c6f
- `AuthorizeSecurityGroupIngress` by beacon remediation at 21:19:55 IST on sg-01eaf3479667c9284, sg-08861ee93fee43c6f
- `UpdateFunctionCode20150331v2` by user/hackathon-judge-temp at 21:11:00 IST on n/a
- `UpdateFunctionCode20150331v2` by user/hackathon-judge-temp at 21:11:00 IST on n/a

## The fix

Proposed `sg.restore_ingress` (fix 1). Blast radius: 1 ingress rule on 1 security group: allow tcp/5432-5432 from sg-01eaf3479667c9284 into sg-08861ee93fee43c6f. Nothing else changes.. Dry run: passed (DryRunOperation).

Approved by ? via ? at ?: "". Executed: no.

Approved by voice via assemblyai at 22:14:59 IST: "approve fix 1". Executed: yes.

## Verification

3 attempt(s); final attempt 3 failed:

- alarm_ok_after_fix: not met
- metric_zero: ok
- postcondition: ok

## Sleep Contract

No contract was granted from this incident.

## Cost

Model cost not recorded.

_Generated by Beacon from the incident record; no model was involved._