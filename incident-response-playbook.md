# Incident Response Playbook: Suspected Compromised IAM Credentials

## Purpose

This playbook provides a repeatable process for responding to suspected compromise of an AWS IAM user, access key, role session, or other AWS credentials.

Use it when alerts, logs, or reports indicate possible unauthorized AWS API activity. Examples include an unfamiliar source IP address, an unexpected geographic location, unusual API calls, access-key exposure, unexpected IAM changes, or a GuardDuty finding.

## Scope

This playbook covers:

- IAM users and their access keys.
- IAM roles and temporary role sessions.
- Root-user activity requiring immediate escalation.
- Suspicious AWS management-plane activity recorded in CloudTrail.
- GuardDuty findings related to potentially compromised credentials.
- Evidence preservation, containment, recovery, and post-incident improvement.

This document is a learning and portfolio example. A real organization should adapt it to its account structure, incident-severity definitions, change-management process, business requirements, legal obligations, and communication procedures.

## Roles and Responsibilities

| Role | Responsibilities |
|---|---|
| Incident Commander | Coordinates the response, assigns owners, approves major decisions, and maintains the incident timeline. |
| Cloud Security Analyst | Validates alerts, reviews CloudTrail and GuardDuty data, scopes activity, and records evidence. |
| AWS Administrator | Performs approved IAM, logging, and resource-containment actions. |
| Application Owner | Identifies affected workloads, validates service impact, and updates application credentials. |
| Security Leadership | Receives escalation updates and approves high-impact containment decisions. |

## Severity Assessment

Classify the incident before taking disruptive action, but do not delay urgent containment when active misuse is likely.

| Severity | Example conditions | Initial response target |
|---|---|---|
| Critical | Root-user activity, confirmed credential theft, disabled logging, destructive actions, data exposure, or active attacker behavior. | Escalate immediately and begin containment. |
| High | Confirmed suspicious access-key use, unauthorized IAM changes, new persistence mechanisms, or unexpected resource creation. | Investigate and contain as soon as possible. |
| Medium | Suspicious activity that is not yet confirmed, such as an unusual source IP or atypical API call. | Validate quickly and monitor closely. |
| Low | Benign explanation confirmed, test activity, or a false positive. | Document disposition and improve detection logic if needed. |

## Phase 1: Detect and Validate

1. Record the alert source, detection time, severity, finding ID, and analyst assigned.
2. Determine whether the signal came from GuardDuty, CloudTrail, CloudWatch, Security Hub, a user report, source-code scanning, or another monitoring tool.
3. Identify the affected AWS account, AWS Region, IAM principal, access-key ID when available, and suspected time range.
4. Review CloudTrail events for the identity before and after the alert time.
5. Compare source IP address, user agent, Region, API activity, and access pattern with the identity's expected behavior.
6. Contact the identity owner or application owner through an approved channel to validate whether the activity was expected.
7. Raise severity and begin containment if there is evidence of active or unauthorized use.

### Initial Triage Questions

- What exact event, finding, or report triggered the investigation?
- Which principal performed the action?
- Which access-key ID, role session, or authentication method was used?
- What was the source IP address, user agent, AWS Region, and timestamp?
- Is the activity still occurring?
- Does the behavior match an approved application, deployment, administrator, or automation workflow?
- Did the principal attempt privilege escalation, persistence, defense evasion, data access, or resource creation?

## Phase 2: Preserve Evidence

Preserve evidence before making broad configuration changes whenever practical. Balance evidence preservation against the need to stop ongoing harmful activity.

1. Create an incident record and use a consistent incident identifier.
2. Record all times in UTC and note the source of each timestamp.
3. Export or securely retain relevant GuardDuty findings, CloudTrail events, CloudWatch Logs queries, IAM policy documents, and resource configuration details.
4. Record the affected account ID, principal ARN, access-key ID, role session name, source IP addresses, Regions, and relevant API calls.
5. Capture a timeline that includes first observed activity, alert time, validation actions, containment actions, and recovery actions.
6. Limit evidence access to authorized incident responders.
7. Avoid editing original log files or altering the original evidence location.
8. If enabled, use CloudTrail log-file integrity validation to confirm whether delivered log files were modified, deleted, or unchanged.
9. Record the commands, console actions, personnel, and approvals involved in containment and recovery.

## Phase 3: Contain the Threat

Containment should stop unauthorized use while minimizing unnecessary impact to legitimate workloads.

### For a Suspected Compromised IAM Access Key

1. Identify the IAM user and access-key ID involved.
2. Check recent use with `get-access-key-last-used` when available.
3. Coordinate with the application owner if the key supports an active workload.
4. Set the suspected access key to `Inactive` to block further programmatic use.
5. Do not immediately delete the key unless response leadership determines that deletion is necessary.
6. Create a replacement key only when required and only after the workload owner is ready to update the application securely.
7. Update the application or secret-management system with the replacement credential.
8. Confirm that the application functions with the new credential.
9. Monitor CloudTrail and GuardDuty for additional activity tied to the compromised key, IAM user, source IP, or related resources.

### Example AWS CLI Containment Command

```bash
aws iam update-access-key \
  --user-name <iam-user-name> \
  --access-key-id <access-key-id> \
  --status Inactive
```

Replace the placeholder values only in an authorized environment. Never place real credentials, account IDs, or sensitive incident details in a public repository.

### For a Suspected Compromised IAM Role Session

1. Identify the role ARN, session name, source identity, and session start time.
2. Review the role's trust policy and permissions policies.
3. Identify the system, user, federation provider, or workload that assumed the role.
4. Restrict or revoke the source identity according to the organization's approved access model.
5. Update overly permissive trust or permissions policies after assessing business impact.
6. Review related role sessions and dependent workloads for additional exposure.

### For Root-User Activity

1. Treat unexpected root-user activity as Critical severity.
2. Escalate immediately to designated account owners and security leadership.
3. Confirm that multi-factor authentication is enabled for the root user.
4. Review root-user CloudTrail activity, access methods, source IP addresses, and affected services.
5. Follow AWS account-recovery and organizational incident procedures as appropriate.
6. Do not use root credentials for routine remediation when a least-privileged administrative role can perform the action.

## Phase 4: Investigate Scope and Impact

Determine what the affected identity did, what it could access, and whether an attacker established persistence.

1. Search CloudTrail for all activity associated with the affected principal, access key, role session, source IP, and time range.
2. Review management events for IAM changes, logging changes, network changes, account changes, and resource creation.
3. Look for common persistence actions, including new IAM users, access keys, roles, inline policies, managed-policy attachments, trust-policy changes, and federation changes.
4. Review attempts to weaken security controls, such as stopping or modifying CloudTrail, changing log destinations, changing GuardDuty settings, or modifying security groups and network controls.
5. Identify created or modified resources, including EC2 instances, Lambda functions, S3 buckets, IAM identities, keys, security groups, and CloudFormation stacks.
6. Review data-access events when they are enabled and relevant to the incident, especially for sensitive S3 buckets and secrets.
7. Identify related accounts, Regions, VPCs, workloads, repositories, CI/CD systems, and third parties that may use the affected credential.
8. Determine whether the credential may have been exposed through source code, a CI/CD log, local configuration file, browser history, support ticket, chat message, public repository, or third-party integration.
9. Keep a written list of confirmed facts, hypotheses, evidence sources, and unresolved questions.

## Phase 5: Eradicate and Recover

1. Remove unauthorized IAM users, roles, policies, access keys, resource policies, and trust relationships after preserving evidence and obtaining approval.
2. Remove or isolate unauthorized compute, storage, networking, serverless, and automation resources.
3. Rotate affected credentials, including access keys, application secrets, API tokens, and third-party credentials that could have been exposed.
4. Apply least-privilege permissions to the affected IAM identity and dependent workloads.
5. Confirm CloudTrail, GuardDuty, AWS Config, log destinations, encryption, retention, and alerting configurations remain enabled and correctly configured.
6. Re-enable legitimate access only after the new credential, role, or application configuration has been validated.
7. Monitor for recurrence using affected identities, source IP addresses, API actions, resource identifiers, and indicators discovered during the investigation.
8. Close the incident only after the incident commander and relevant resource owners agree that containment, remediation, and monitoring requirements are met.

## Phase 6: Post-Incident Review

Conduct a documented review after the incident.

- Summarize what happened, how it was detected, and the confirmed impact.
- Maintain a timeline from initial activity through recovery.
- Identify the root cause or most likely credential-exposure path.
- Identify controls that worked and controls that failed or were missing.
- Create corrective actions with owners and due dates.
- Improve alerting, log coverage, key rotation, secret management, permissions, documentation, and training.
- Update this playbook when the incident reveals gaps or better response methods.

## Investigation Record Template

```text
Incident ID:
Date and time detected (UTC):
Incident Commander:
Analyst:
AWS account ID:
Affected Region(s):
Affected IAM principal:
Access-key ID or role session:
Alert source and finding ID:
Initial severity:
Source IP address(es):
User agent(s):
Suspicious API action(s):
Affected resource(s):
Containment action(s):
Evidence location:
Business impact:
Recovery status:
Follow-up actions:
```

## Security Notes

- Prefer temporary credentials and IAM roles over long-lived IAM user access keys where possible.
- Store secrets in an approved secrets-management system rather than source code, plain-text files, or public repositories.
- Apply least privilege and regularly review unused identities, access keys, policies, and permissions.
- Enable and protect centralized audit logging before an incident occurs.
- Treat public exposure of an AWS access key as a security event, even if unauthorized activity has not yet been confirmed.
