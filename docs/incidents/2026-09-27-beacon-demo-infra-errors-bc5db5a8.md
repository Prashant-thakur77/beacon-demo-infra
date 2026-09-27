# Postmortem: beacon-demo-infra-errors

Incident `bc5db5a8-e6eb-432d-aa57-14f6124d29e1` · 00:14:37 IST · status **resolved**

## Summary

The security group sg-08861ee93fee43c6f is missing an ingress rule (tcp/5432-5432 from sg-01eaf3479667c9284) that the golden snapshot says should exist; the service behind it cannot reach its dependency.

## Timeline

| Time (IST) | Event | Detail |
|---|---|---|
| 00:14:35 IST | alarm_received | alarm_name=beacon-demo-infra-errors, trigger_type=alarm |
| 00:14:36 IST | logs_fetched | groups=1, lines=605 |
| 00:14:37 IST | diagnostics_ran | missing_rules=1, suggested_action=sg.restore_ingress |
| 00:14:37 IST | changes_checked | count=4, top=RevokeSecurityGroupIngress, source=ledger |
| 00:14:37 IST | model_unavailable | reason=triage: litellm.BadRequestError: BedrockException - {"message":"Operation not allowed"} |
| 00:14:37 IST | rca_ready | suggested_action=sg.restore_ingress, model=us.amazon.nova-2-lite-v1:0, status=High |
| 00:14:37 IST | sns_sent |  |
| 00:15:57 IST | fix_proposed | blast_radius=1 ingress rule on 1 security group: allow tcp/5432-5432 from sg-01eaf3479667c9284 into sg-08861ee93fee43c6f. Nothing else changes., blast_radius_spoken=One change: restore the missing security-group rule, the tcp port 5432 rule from the application security group into the database security group. Nothing else changes., action=sg.restore_ingress, dry_run={'detail': 'Request would have succeeded, but DryRun flag is set.', 'role': 'arn:aws:sts::283146810291:assumed-role/beacon-remediator-beacon/beacon-remediate-beacon', 'code': 'DryRunOperation', 'ok': True}, fix_id=1 |
| 00:16:06 IST | proposal_withdrawn | fix_id=1, reason=engineer interrupted the read-back |
| 00:16:28 IST | fix_proposed | action=sg.restore_ingress, blast_radius_spoken=One change: restore the missing security-group rule, the tcp port 5432 rule from the application security group into the database security group. Nothing else changes., dry_run={'code': 'DryRunOperation', 'detail': 'Request would have succeeded, but DryRun flag is set.', 'role': 'arn:aws:sts::283146810291:assumed-role/beacon-remediator-beacon/beacon-remediate-beacon', 'ok': True}, fix_id=2, blast_radius=1 ingress rule on 1 security group: allow tcp/5432-5432 from sg-01eaf3479667c9284 into sg-08861ee93fee43c6f. Nothing else changes. |
| 00:16:58 IST | approved | "" via assemblyai |
| 00:16:59 IST | executing | params={'from_port': 5432, 'to_port': 5432, 'group_id': 'sg-08861ee93fee43c6f', 'source_group_id': 'sg-01eaf3479667c9284', 'ip_protocol': 'tcp'}, approval_id=4ac8a5c7-aa13-4ce0-a793-6f4bbf469812, action=sg.restore_ingress |
| 00:17:00 IST | executed | executed_at=2026-09-27T18:47:00.147869+00:00, code=Authorized, detail=rule added |
| 00:17:30 IST | verify_attempt | attempt 1 · not yet · alarm_ok_after_fix=no, metric_zero=no, postcondition=ok |
| 00:18:01 IST | verify_attempt | attempt 2 · not yet · alarm_ok_after_fix=no, metric_zero=ok, postcondition=ok |
| 00:18:31 IST | verify_attempt | attempt 3 · passed · alarm_ok_after_fix=ok, metric_zero=ok, postcondition=ok |
| 00:18:32 IST | resolved | handled_by=voice, attempts=3 |

Alarm to recovery: **3 m 54 s**

## Root cause

RevokeSecurityGroupIngress by user/hackathon-judge-temp at 2026-09-27T18:42:52Z on sg-01eaf3479667c9284, sg-08861ee93fee43c6f shortly before the alarm.

Evidence:

- `- MISSING ingress rule: tcp 5432-5432 from sg-01eaf3479667c9284 into sg-08861ee93fee43c6f`
- `2026-09-27T18:42:59.803000+00:00 2026-09-27T18:42:59 WARNING ConnectionPool WARNING: pool utilization rising. db_host=beacon-demo-db.c0bg0ac4umkd.us-east-1.rds.amazonaws.com query_timeout=5002ms error='connection to serv`
- `2026-09-27T18:43:06.981000+00:00 2026-09-27T18:43:06 WARNING ConnectionPool WARNING: pool utilization rising. db_host=beacon-demo-db.c0bg0ac4umkd.us-east-1.rds.amazonaws.com query_timeout=5006ms error='connection to serv`
- `2026-09-27T18:43:15.695000+00:00 2026-09-27T18:43:15 ERROR POST /api/v2/payments 503 5002ms - service=payments-service error='connection_timeout' db_host=beacon-demo-db.c0bg0ac4umkd.us-east-1.rds.amazonaws.com db_pool=EX`
- `2026-09-27T18:43:23.481000+00:00 2026-09-27T18:43:23 ERROR GET /api/v2/recommendations 503 5008ms - service=recommendations-service error='connection_timeout' db_host=beacon-demo-db.c0bg0ac4umkd.us-east-1.rds.amazonaws.c`
- `2026-09-27T18:43:31.442000+00:00 2026-09-27T18:43:31 ERROR POST /api/v2/payments 503 5007ms - service=payments-service error='connection_timeout' db_host=beacon-demo-db.c0bg0ac4umkd.us-east-1.rds.amazonaws.com db_pool=EX`

## What changed

- `RevokeSecurityGroupIngress` by user/hackathon-judge-temp at 00:12:52 IST on sg-01eaf3479667c9284, sg-08861ee93fee43c6f
- `CreateNetworkInterface` by role/AWSServiceRoleForECS at 00:09:10 IST on sg-01eaf3479667c9284, subnet-00016962bbff15779
- `DeleteNetworkInterface` by role/AWSServiceRoleForECS at 00:11:13 IST on eni-06ed27886fcd1888d
- `UpdateService` by user/hackathon-judge-temp at 00:08:49 IST on n/a

## The fix

Proposed `sg.restore_ingress` (fix 2). Blast radius: 1 ingress rule on 1 security group: allow tcp/5432-5432 from sg-01eaf3479667c9284 into sg-08861ee93fee43c6f. Nothing else changes.. Dry run: passed (DryRunOperation).

Approved by ? via ? at ?: "". Executed: no.

Approved by ? via ? at ?: "". Executed: no.

Approved by voice via assemblyai at 00:16:58 IST: "Approve fix 2.". Executed: yes. Attested: AssemblyAI session recording `sess_d32506700b764ad898637798577895fb`.

## Verification

3 attempt(s); final attempt 3 passed:

- alarm_ok_after_fix: ok
- metric_zero: ok
- postcondition: ok

## Sleep Contract

No contract was granted from this incident.

## Cost

Model cost not recorded.

_Generated by Beacon from the incident record; no model was involved._