# Detection and Triage Guide

## Purpose

This guide explains how to validate alerts involving suspected compromised AWS IAM credentials and how to decide whether to close, monitor, escalate, or contain the event.

Use it with the main [incident response playbook](incident-response-playbook.md).

## Detection Sources

Potential credential-compromise signals may come from:

- Amazon GuardDuty findings.
- AWS CloudTrail management events.
- AWS Security Hub findings.
- Amazon CloudWatch alarms or Logs Insights queries.
- IAM access-key exposure alerts from source-code or secret-scanning tools.
- Reports from users, developers, vendors, or security researchers.
- Unexpected AWS billing, resource creation, or account activity.

## Minimum Information to Capture

Record the following before beginning detailed analysis:

| Field | Why it matters |
|---|---|
| Incident ID | Connects investigation notes, evidence, and decisions. |
| Detection time in UTC | Establishes the start of the response timeline. |
| Alert source and finding ID | Allows the original alert to be retrieved and reviewed. |
| AWS account ID and Region | CloudTrail event history and many AWS resources are Region-specific. |
| Affected principal | Identifies the IAM user, role, federated identity, or root user involved. |
| Access-key ID or role session | Connects API activity to the credential or temporary session. |
| Source IP address and user agent | Helps distinguish expected automation from suspicious access. |
| Event name and event source | Identifies the AWS API action and AWS service involved. |
| Event time | Supports timeline creation and correlation. |
| Affected resources | Identifies systems, data, or permissions that may be at risk. |

## Initial Validation Workflow

1. Open the original alert, GuardDuty finding, or CloudTrail event and save its identifier.
2. Confirm the AWS account ID, Region, event time, principal, and affected resource.
3. Check whether the event was expected by reviewing change tickets, deployment records, maintenance windows, and approved automation.
4. Contact the credential owner or application owner through an approved communication channel.
5. Compare the source IP address, user agent, AWS Region, time of day, and API action with the identity's normal activity.
6. Review activity immediately before and after the alert to determine whether it is isolated or part of a sequence.
7. Search other relevant Regions and accounts if the environment uses multiple accounts or centralized identity.
8. Assign an initial severity and decide whether containment must begin immediately.

## CloudTrail Review

CloudTrail management events are the primary source for investigating AWS control-plane activity such as IAM changes, resource creation, policy changes, and logging changes.

### High-Value Event Fields

| Field | Analyst use |
|---|---|
| `eventTime` | Establishes when the API call occurred. |
| `eventSource` | Identifies the AWS service, such as `iam.amazonaws.com`. |
| `eventName` | Identifies the API action, such as `CreateUser` or `AttachUserPolicy`. |
| `userIdentity` | Identifies the actor, credential type, account, and principal ARN. |
| `sourceIPAddress` | Identifies the network origin of the request. |
| `userAgent` | Helps identify the console, AWS CLI, SDK, browser, or automation tool. |
| `awsRegion` | Identifies where the event was recorded. |
| `requestParameters` | Shows relevant action inputs, when recorded. |
| `responseElements` | Can identify newly created resource identifiers. |
| `errorCode` | Reveals failed or denied attempts that may still be suspicious. |
| `resources` | Identifies AWS resources associated with the event. |

### Event Categories to Review

| Category | Examples |
|---|---|
| Credential activity | `CreateAccessKey`, `UpdateAccessKey`, `DeleteAccessKey`, `CreateLoginProfile`. |
| Identity and privilege changes | `CreateUser`, `CreateRole`, `AttachUserPolicy`, `PutUserPolicy`, `PutRolePolicy`, `UpdateAssumeRolePolicy`. |
| Defense evasion | `StopLogging`, `DeleteTrail`, `UpdateTrail`, `DeleteDetector`, `UpdateDetector`. |
| Compute or persistence | `RunInstances`, `CreateFunction`, `CreateStack`, `CreateLaunchTemplate`. |
| Data-access indicators | `GetObject`, `ListBucket`, `GetSecretValue`, when relevant data events are enabled. |
| Reconnaissance | `ListUsers`, `ListRoles`, `GetCallerIdentity`, `DescribeInstances`, `ListBuckets`. |

A single event rarely proves compromise. Investigate combinations of events, sequence, identity, source IP address, user agent, affected resources, and whether the actions match normal business activity.

## Example CloudTrail Lookup Commands

Use only in an authorized AWS environment. Replace placeholder values with information from the incident.

### Search by access-key ID

```bash
aws cloudtrail lookup-events \
  --lookup-attributes AttributeKey=AccessKeyId,AttributeValue=<access-key-id> \
  --start-time <start-time-utc> \
  --end-time <end-time-utc> \
  --region <aws-region>
```

### Search by IAM username

```bash
aws cloudtrail lookup-events \
  --lookup-attributes AttributeKey=Username,AttributeValue=<iam-user-name> \
  --start-time <start-time-utc> \
  --end-time <end-time-utc> \
  --region <aws-region>
```

### Search by event name

```bash
aws cloudtrail lookup-events \
  --lookup-attributes AttributeKey=EventName,AttributeValue=CreateAccessKey \
  --start-time <start-time-utc> \
  --end-time <end-time-utc> \
  --region <aws-region>
```

CloudTrail Event history supports searches using a single attribute at a time. Use a narrow time window, export relevant results, and correlate them with other sources rather than relying on one lookup alone.

## GuardDuty Triage

For a GuardDuty finding, capture:

- Finding ID and type.
- Severity and creation time.
- Account ID and Region.
- Affected resource and principal.
- Source and destination network details, when included.
- Action details and service information.
- Related CloudTrail events.
- Whether the finding is new, recurring, archived, or previously investigated.

GuardDuty severity should guide urgency, but severity alone does not establish business impact. A high-severity finding warrants rapid validation and remediation because it can indicate a compromised resource or credentials actively used for unauthorized purposes.

## Initial Severity Decision

| Decision | Conditions | Action |
|---|---|---|
| Close as benign | The activity is verified, documented, and consistent with approved behavior. | Record evidence and rationale; consider tuning the alert. |
| Monitor | The activity is unusual but no unauthorized action is confirmed. | Increase observation, obtain owner confirmation, and set a review time. |
| Escalate | Suspicious activity is plausible but scope or impact is unclear. | Notify the incident lead and continue investigation. |
| Contain immediately | Active misuse, known credential exposure, privilege escalation, logging changes, destructive actions, or data-access risk is identified. | Follow the containment phase of the main playbook without delay. |

## Red Flags That Require Urgent Escalation

- Root-user activity that cannot be immediately explained.
- A confirmed exposed access key or secret.
- `CreateAccessKey`, `CreateUser`, `CreateRole`, `AttachUserPolicy`, or policy changes not tied to approved work.
- Changes intended to reduce logging, monitoring, detection, encryption, or network controls.
- New compute, serverless, or automation resources created unexpectedly.
- Access to sensitive data stores, secrets, backups, or customer information.
- Repeated failed API calls followed by successful privileged actions.
- Activity from unfamiliar infrastructure combined with IAM discovery or privilege changes.

## Triage Record Template

```text
Incident ID:
Analyst:
Alert source:
Finding ID or event ID:
Date and time detected (UTC):
AWS account ID:
AWS Region:
Affected principal:
Access-key ID or role session:
Source IP address:
User agent:
Event source:
Event name:
Affected resources:
Expected or unexpected:
Evidence reviewed:
Initial severity:
Decision:
Escalation contact:
Next review time:
```

## Documentation Standards

- Record facts separately from assumptions and hypotheses.
- Use UTC for investigation and response timelines.
- Preserve original event identifiers and unmodified exports where possible.
- Do not paste secrets, full credentials, customer data, or sensitive production log data into tickets, chat tools, or public repositories.
- Record why a decision was made, especially when deciding to close an alert or delay containment.
