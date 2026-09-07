# AWS Incident Evidence Checklist

## Purpose

This checklist identifies evidence to collect, preserve, and document during an investigation of suspected compromised AWS IAM credentials.

Use it together with the [incident response playbook](incident-response-playbook.md), [detection and triage guide](detection-and-triage.md), and [containment and recovery guide](containment-and-recovery.md).

## Evidence-Handling Principles

- Begin preserving relevant evidence as soon as an incident is identified.
- Balance evidence preservation with the need to contain active unauthorized activity.
- Preserve original data whenever possible and avoid modifying source logs.
- Record where evidence came from, who collected it, when it was collected, and how it was stored.
- Use UTC for timestamps.
- Restrict evidence access to authorized responders.
- Do not upload real production logs, credentials, customer data, or sensitive evidence to a public repository.

## Incident Metadata

Capture this information at the start of the investigation.

| Item | Capture |
|---|---|
| Incident ID | A unique identifier used across notes, tickets, exports, and communications. |
| Incident title | A concise description, such as “Suspected compromised IAM access key.” |
| Detection source | GuardDuty, CloudTrail, Security Hub, secret scanning, user report, or other source. |
| Detection time | Time in UTC when the organization first became aware of the event. |
| Incident commander | Person responsible for coordinating the response. |
| Assigned analyst | Person responsible for investigation and evidence collection. |
| Initial severity | Critical, High, Medium, or Low, with supporting rationale. |
| AWS account ID | Affected account or accounts. |
| AWS Organization context | Management account, member account, organizational unit, or delegated administrator when applicable. |
| Affected Region or Regions | Regions in scope for the investigation. |
| Affected principal | IAM user, role, assumed-role session, federated user, workload identity, or root user. |
| Access-key ID or session identifier | Identifier tied to the suspected activity. |
| Incident status | Open, investigating, contained, recovering, monitoring, or closed. |

## Alert and Finding Evidence

Collect the original alert and its details before making broad configuration changes.

- GuardDuty finding ID, type, severity, title, description, creation time, update time, and archived status.
- Security Hub finding ID, product ARN, severity, workflow status, and remediation recommendation.
- CloudWatch alarm name, state-change time, metric, threshold, evaluation period, and linked logs.
- Secret-scanning alert source, repository or location, detection time, exposure status, and remediation details.
- User, customer, vendor, or researcher report, including the original message and the channel used.
- Ticket number, case number, or incident-management record.
- Screenshots only when exports or original data are not available; record the source and capture time.

## CloudTrail Evidence

CloudTrail is a primary source of AWS management-plane activity.

### Event Details to Preserve

For each relevant event, collect:

| Field | Why it matters |
|---|---|
| `eventID` | Unique identifier for the event record. |
| `eventTime` | Establishes the time of the API call. |
| `eventSource` | Identifies the AWS service receiving the request. |
| `eventName` | Identifies the API operation performed. |
| `awsRegion` | Identifies where the event was recorded. |
| `userIdentity.type` | Identifies the credential or identity type. |
| `userIdentity.arn` | Identifies the acting principal. |
| `userIdentity.accountId` | Identifies the AWS account associated with the identity. |
| `userIdentity.accessKeyId` | Connects the action to an access key when present. |
| `sourceIPAddress` | Identifies the request origin. |
| `userAgent` | Indicates console, CLI, SDK, browser, or automation use. |
| `requestParameters` | Captures important requested action inputs when recorded. |
| `responseElements` | May identify a created or changed resource. |
| `errorCode` and `errorMessage` | Preserve failed, denied, or attempted actions. |
| `resources` | Links the event to associated AWS resources. |
| `recipientAccountId` | Identifies the AWS account receiving the action. |
| `additionalEventData` | May provide further authentication or request context. |

### Collection Checklist

```text
[ ] Export relevant CloudTrail events for the full investigation window.
[ ] Record the AWS Region associated with each event.
[ ] Preserve event IDs and original JSON where available.
[ ] Search activity before and after the alert to build a timeline.
[ ] Search by affected principal, access-key ID, source IP address, event name, and resource.
[ ] Review CloudTrail events in other in-scope Regions.
[ ] Record failed API calls and denied actions, not only successful events.
[ ] Identify possible privilege escalation, persistence, defense evasion, discovery, resource creation, and data access.
[ ] Retain query parameters, commands, and time ranges used to collect evidence.
[ ] Record the collector, collection time in UTC, and evidence-storage location.
```

## IAM Evidence

Collect the current state of the affected identity and related permission paths.

- IAM username, user ID, ARN, creation date, and tags.
- Access-key IDs, status, creation dates, last-used data, and owning workload.
- MFA device status and recent changes.
- Login profile status, if applicable.
- Group memberships.
- Attached managed policies.
- Inline policies.
- Permission boundaries.
- IAM role trust policies.
- Role permissions policies and inline policies.
- Cross-account access and external ID relationships.
- Federated identity provider configuration and session details.
- Service control policies and resource policies that may affect access.
- Recent IAM changes from CloudTrail.

Avoid including secret access keys, passwords, session tokens, recovery codes, or full sensitive policy exports in public documentation.

## Resource and Configuration Evidence

Capture state for resources created, changed, accessed, or potentially exposed by the affected identity.

| Area | Evidence to collect |
|---|---|
| EC2 | Instance ID, AMI, launch time, instance profile, tags, security groups, network interfaces, volumes, key-pair name, and CloudTrail events. |
| Lambda | Function name, runtime, code location or hash, execution role, environment-variable metadata, layers, triggers, destinations, and recent changes. |
| S3 | Bucket name, bucket policy, public-access settings, encryption configuration, access logs, object-level activity when data events are enabled, and relevant object metadata. |
| Secrets | Secret ARN, owning workload, resource policy, rotation status, access history where available, and rotation actions performed. |
| KMS | Key ARN, key policy, grants, key usage, and related CloudTrail events. |
| Networking | Security groups, network ACLs, route tables, VPC endpoints, Internet gateways, NAT gateways, and firewall-related changes. |
| CloudFormation | Stack name, template source, execution role, change history, created resources, and deletion-protection status. |
| CI/CD | Pipeline identity, build logs, secret references, artifact location, repository changes, deployment events, and federated-access relationships. |
| AWS Config | Resource configuration items, configuration history, compliance status, and rules relevant to the changed resource. |

## Network and Authentication Evidence

When available and relevant, preserve:

- Source IP addresses and whether they belong to known corporate, VPN, cloud, or third-party networks.
- User-agent strings and authentication mechanisms.
- VPC Flow Logs for affected workloads or network interfaces.
- CloudFront, load balancer, WAF, API Gateway, Route 53, or application logs.
- Identity-provider sign-in logs, federation events, MFA events, and SSO audit logs.
- DNS activity and threat-intelligence enrichment from approved sources.
- Geographic or autonomous-system information only as supporting context, not as proof of malicious activity.

## Log Integrity and Storage

Document the condition and location of collected evidence.

| Item | Record |
|---|---|
| Original evidence location | CloudTrail Event history, S3 log bucket, CloudTrail Lake, CloudWatch Logs group, GuardDuty console, Security Hub, or another approved source. |
| Export location | Approved case system, protected evidence bucket, or restricted investigation workspace. |
| Collection method | Console export, AWS CLI, API, query, automated collection, or approved tool. |
| Collector | Person or automated process that collected the evidence. |
| Collection time | UTC timestamp for collection. |
| Integrity status | CloudTrail log-file validation result, checksum, access controls, or other integrity information. |
| Access controls | Who can view, modify, or delete the evidence. |
| Retention requirement | Required retention period based on policy, legal hold, compliance, or business requirements. |

CloudTrail log-file integrity validation can help determine whether a delivered CloudTrail log file was modified, deleted, or unchanged after delivery.

## Timeline Template

Use a single UTC timeline throughout the investigation.

```text
Timestamp (UTC) | Event / Action | Source | Actor | Evidence Reference | Notes
----------------|----------------|--------|-------|--------------------|------
                |                |        |       |                    |
                |                |        |       |                    |
                |                |        |       |                    |
```

## Chain-of-Custody Template

Use this when evidence is copied, transferred, or accessed outside its original AWS location.

```text
Evidence ID:
Incident ID:
Description:
Original source:
Original location:
Collected by:
Collection time (UTC):
Collection method:
Integrity check:
Stored at:
Access restrictions:
Transferred to:
Transfer time (UTC):
Transfer approved by:
Notes:
```

## Final Evidence Review

Before closing an incident, confirm:

```text
[ ] The initial alert or report is preserved.
[ ] Relevant CloudTrail and GuardDuty evidence is retained.
[ ] Affected identity and credential details are documented.
[ ] Relevant IAM policies, trust relationships, and configuration changes are captured.
[ ] Affected resources and potential data exposure are assessed.
[ ] Containment and recovery actions are recorded with timestamps.
[ ] The incident timeline is complete and uses UTC.
[ ] Evidence locations and access restrictions are documented.
[ ] Sensitive data has not been copied into public repositories or unapproved tools.
[ ] Follow-up actions and evidence-retention requirements have owners.
```
