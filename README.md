# AWS Cloud Incident Response Playbook

A practical incident-response playbook for investigating, containing, and recovering from suspected compromised AWS IAM credentials.

This project documents a structured response to suspicious IAM activity, such as an AWS access key being used from an unfamiliar location, an unexpected API call, or a GuardDuty finding that indicates potentially compromised credentials.

## Scenario

An organization receives a security alert indicating that an IAM user's access key may be compromised.

Example indicators include:

- API activity from an unfamiliar source IP address or geographic location
- Unexpected IAM changes, such as new users, policies, roles, or access keys
- Attempts to disable CloudTrail logging or other security controls
- Unexpected EC2 instance launches or resource changes
- GuardDuty findings related to anomalous IAM-user behavior or compromised credentials

The goal is to contain the potential compromise quickly, preserve evidence, determine the scope, rotate credentials safely, and prevent recurrence.

## Objectives

- Identify suspicious IAM or access-key activity.
- Preserve relevant logs and investigation evidence.
- Contain compromised credentials while limiting business disruption.
- Determine what AWS actions and resources may have been affected.
- Eradicate unauthorized access and restore secure operations.
- Document lessons learned and improve detection controls.

## AWS Services and Concepts

| Service or concept | Role in the response |
|---|---|
| AWS CloudTrail | Provides an audit record of AWS API activity. |
| Amazon GuardDuty | Generates findings for suspicious or potentially malicious behavior. |
| AWS IAM | Manages identities, access keys, roles, permissions, and credential rotation. |
| Amazon CloudWatch Logs | Supports centralized log review and alerting workflows. |
| AWS Config | Helps identify resource configuration changes and compliance drift. |
| Least privilege | Limits the permissions and blast radius of identities. |

CloudTrail events can provide key investigation fields such as the acting identity, time, AWS service, API action, Region, source IP address, and user agent. [AWS CloudTrail event documentation](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-event-reference-record-contents.html)

## Response Workflow

```text
Detect
  ↓
Validate and triage
  ↓
Preserve evidence
  ↓
Contain the affected identity
  ↓
Investigate scope and impact
  ↓
Eradicate unauthorized access
  ↓
Recover and monitor
  ↓
Document lessons learned
```

## Project Documents

| File | Description |
|---|---|
| `incident-response-playbook.md` | End-to-end response process for suspected compromised IAM credentials. |
| `detection-and-triage.md` | Alert validation, CloudTrail review, and severity assessment. |
| `containment-and-recovery.md` | Credential containment, remediation, recovery, and monitoring actions. |
| `evidence-checklist.md` | Evidence to collect and preserve during the investigation. |
| `sample-cloudtrail-event.json` | Sanitized example of a CloudTrail event used for analysis practice. |

## Sample CloudTrail Event

This repository includes a fictional CloudTrail management event for hands-on analysis:

[View the sanitized sample event](sample-cloudtrail-event.json)

The event shows an IAM user named `finance-reporting` calling `CreateAccessKey` for a different user named `backup-automation`. It is intentionally suspicious but does not prove compromise on its own. An analyst should validate the source IP address, user agent, authorization model, change record, workload ownership, and related CloudTrail activity before deciding whether to contain the credential.

## Sample Investigation Questions

During triage, an analyst should be able to answer questions such as:

- Which IAM identity or access key performed the suspicious action?
- When did the activity begin, and is it still occurring?
- Which source IP addresses, user agents, Regions, and AWS services were involved?
- Did the identity create users, roles, policies, access keys, or other persistence mechanisms?
- Did the actor access data, modify security controls, or launch new resources?
- Which credentials require rotation, and which affected resources require review?

## Skills Demonstrated

- AWS cloud security
- AWS IAM investigation and credential response
- CloudTrail log analysis
- GuardDuty finding triage
- Incident response and evidence preservation
- Least-privilege remediation
- Security documentation and operational runbook design
- GitHub and Markdown documentation

## Important Note

All examples in this repository use fictional account identifiers, access-key IDs, IP addresses, usernames, and resources. Do not upload real AWS credentials, CloudTrail logs containing sensitive data, customer data, or production account information to a public repository.

## Next Steps

Read the full response procedure in [incident-response-playbook.md](incident-response-playbook.md).
