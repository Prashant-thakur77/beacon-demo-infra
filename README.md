# beacon-demo-infra

The infrastructure repository for the [Beacon Night Shift](https://github.com/Prashant-thakur77/Beacon) demo.
`demo/demo-infra-template.yaml` is the CloudFormation stack the demo runs on (a web service on ECS Fargate
talking to a PostgreSQL RDS instance).

When Beacon fixes an incident at runtime and the engineer says **"open the pull request"**, Beacon opens a
pull request here with the durable fix (the security-group rule declared in the template) and the postmortem
under `docs/incidents/`. Nothing is merged by Beacon.
