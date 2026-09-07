# Containment and Recovery Guide

## Purpose

This guide provides practical containment, remediation, recovery, and monitoring actions for a suspected or confirmed AWS IAM credential compromise.

Use it after initial validation and evidence preservation. Follow the main [incident response playbook](incident-response-playbook.md) for the complete incident lifecycle.

## Response Principles

- Prioritize stopping active unauthorized access.
- Preserve evidence whenever doing so does not create unacceptable risk.
- Coordinate disruptive changes with incident leadership and workload owners.
- Use the least disruptive containment action that reliably limits the threat.
- Do not restore access until the identity, credentials, affected resources, and security controls have been reviewed.
- Record every containment and recovery decision in the incident timeline.

## Containment Decision Matrix

| Situation | Primary action | Additional action |
|---|---|---|
| Suspected exposed IAM access key with possible misuse | Set the access key to `Inactive`. | Investigate CloudTrail activity and coordinate replacement credentials. |
| Confirmed unauthorized use of an IAM access key | Set the key to `Inactive` immediately. | Scope actions, rotate credentials, review permissions, and search for persistence. |
| Legitimate application depends on the key | Create and securely deploy a replacement key after containment approval. | Validate the application, then remove the old key when safe. |
| Suspicious role-session activity | Restrict or revoke the source identity that can assume the role. | Review the role trust policy, permissions, session source, and dependent workloads. |
| Unauthorized IAM privilege or persistence change | Remove or disable the unauthorized change after preserving evidence. | Review related users, roles, policies, trust policies, and access keys. |
| Unexpected root-user activity | Treat as Critical and escalate immediately. | Secure the root account, review activity, and follow account-recovery procedures. |
| Logging or detection controls changed | Restore approved logging and detection settings. | Identify who changed the controls and review the full time range for concealment activity. |
| Unauthorized compute or automation resource | Isolate, stop, or remove the resource after approval. | Preserve configuration, logs, tags, network details, and associated IAM identity. |

## Access-Key Containment

### 1. Identify the Key and Its Owner

Confirm the following before acting:

- IAM username.
- Access-key ID.
- AWS account ID.
- Last-known use time and AWS service.
- Application, owner, team, or automation associated with the key.
- Suspicious source IP addresses, Regions, API actions, and time range.
- Whether the key is currently active.

### 2. Deactivate the Suspected Key

For active or confirmed misuse, set the access key to `Inactive` as soon as containment is approved.

```bash
aws iam update-access-key \
  --user-name <iam-user-name> \
  --access-key-id <access-key-id> \
  --status Inactive
```

An inactive access key cannot be used for AWS API requests. Deactivation preserves the key record for investigation and can be reversed only with explicit approval.

### 3. Verify Containment

After deactivation:

- Confirm the key status is `Inactive`.
- Monitor CloudTrail for attempted use of the key.
- Check GuardDuty, Security Hub, and CloudWatch for related alerts.
- Confirm whether the application or automation using the key has failed.
- Record the exact containment time in UTC.
- Preserve the key ID and associated CloudTrail events in the evidence record.

### 4. Replace the Credential Safely

If a legitimate workload requires programmatic access:

1. Confirm the workload should continue using a long-lived key rather than an IAM role or temporary credentials.
2. Create a replacement credential only after a secure storage and deployment method is ready.
3. Store the replacement in an approved secrets-management system.
4. Update the application, CI/CD system, or integration through its approved configuration process.
5. Validate the workload with the replacement credential.
6. Apply only the minimum permissions required for the workload.
7. Delete the old inactive key after evidence preservation and a defined recovery period.

Never place an access key, secret access key, token, configuration file containing secrets, or copied production credential in a GitHub repository.

## IAM Role and Temporary-Credential Containment

For a suspected compromised role session:

1. Identify the role ARN, role-session name, source identity, and session start time.
2. Review CloudTrail for `AssumeRole`, `AssumeRoleWithSAML`, `AssumeRoleWithWebIdentity`, and subsequent activity.
3. Determine which user, workload, identity provider, or AWS service assumed the role.
4. Restrict the source identity or remove the ability to assume the role using approved IAM or identity-provider controls.
5. Review and tighten the role trust policy.
6. Review permissions policies, inline policies, permission boundaries, session policies, and service control policies.
7. Confirm that legitimate workloads can continue using a least-privileged replacement role or identity.
8. Monitor for new role sessions, related source IP addresses, and repeated attempts to assume the role.

Temporary AWS credentials expire after a configured period. However, do not rely on expiration alone when active misuse is suspected; take approved containment actions immediately.

## Removing Persistence and Privilege Escalation

After preserving evidence, investigate and remediate unauthorized changes.

### IAM Changes to Review

- New IAM users, roles, groups, or identity-provider configurations.
- New or modified access keys.
- New login profiles or multi-factor authentication device changes.
- Attached managed policies and inline policies.
- Role trust-policy modifications.
- Permission-boundary changes.
- Cross-account trust relationships.
- Changes to AWS Organizations policies, where applicable.

### Example Investigation Commands

```bash
aws iam list-access-keys \
  --user-name <iam-user-name>
```

```bash
aws iam list-attached-user-policies \
  --user-name <iam-user-name>
```

```bash
aws iam list-user-policies \
  --user-name <iam-user-name>
```

```bash
aws iam get-role \
  --role-name <role-name>
```

Use these commands only in an authorized environment. Preserve relevant output in an approved evidence location, not in a public repository.

## Resource Containment

Investigate resources created, modified, or accessed by the affected identity.

| Resource type | Review and containment actions |
|---|---|
| Amazon EC2 | Preserve instance ID, AMI, security groups, volumes, network interfaces, tags, and CloudTrail activity. Isolate or stop the instance according to the response plan. |
| AWS Lambda | Review function code, execution role, environment variables, layers, triggers, destinations, and recent changes. Disable triggers or restrict permissions if necessary. |
| Amazon S3 | Review bucket policy, object access, encryption, public-access settings, access logs, and data events if enabled. Restrict access without destroying evidence. |
| AWS Secrets Manager | Identify secrets accessible to the affected identity. Rotate exposed or potentially exposed secrets and review resource policies. |
| Networking | Review security groups, network ACLs, route tables, VPC endpoints, and firewall controls for unauthorized changes. |
| AWS CloudFormation | Review stacks, change sets, templates, execution roles, and recently created resources for automated persistence. |
| CI/CD systems | Review pipelines, deployment credentials, secrets, build logs, repositories, webhooks, and federated access relationships. |

## Recovery

Recovery begins only when containment is stable and the affected environment has been reviewed.

1. Confirm unauthorized credentials, sessions, permissions, and resources have been disabled, removed, or isolated.
2. Rotate credentials that were exposed, used suspiciously, or accessible to the compromised identity.
3. Restore legitimate applications with least-privileged identities and approved secret storage.
4. Verify that CloudTrail, GuardDuty, AWS Config, Security Hub, logging destinations, alerting, encryption, and retention settings are operating as expected.
5. Verify that access to sensitive data, secrets, backups, and production systems is appropriately restricted.
6. Monitor the environment for recurrence using indicators from the incident.
7. Obtain confirmation from system owners that critical services are functioning normally.
8. Document recovery completion, remaining risks, and follow-up tasks.

## Post-Recovery Monitoring

Monitor for a defined period based on incident severity and business risk.

Focus on:

- Reuse attempts involving the disabled access key or affected principal.
- Activity from suspicious source IP addresses, user agents, or Regions.
- New IAM users, access keys, roles, policies, and trust relationships.
- Changes to CloudTrail, GuardDuty, AWS Config, logging, monitoring, or encryption.
- Unexpected resource creation, especially compute, serverless, networking, and automation resources.
- Unusual access to secrets, S3 data, backups, databases, or key-management resources.
- Increased error rates or authentication failures from workloads that received replacement credentials.

## Long-Term Improvements

After recovery, reduce the chance and impact of future credential compromise.

- Prefer IAM roles and short-lived credentials over long-lived IAM user access keys.
- Use federation for human access where possible.
- Store application secrets in AWS Secrets Manager or an approved equivalent.
- Rotate secrets and access keys on a defined schedule and after suspected exposure.
- Apply least privilege and routinely remove unused permissions and credentials.
- Require multi-factor authentication for privileged access.
- Enable CloudTrail across required Regions and protect log destinations.
- Enable GuardDuty and route high-priority findings to an owned response process.
- Configure AWS Config to record relevant resource configuration changes.
- Use secret scanning, pre-commit hooks, and repository protections to reduce accidental credential exposure.
- Test this playbook through tabletop exercises and update it after each exercise or incident.

## Recovery Checklist

```text
[ ] Suspicious credential or session contained.
[ ] Evidence preserved and incident timeline updated.
[ ] Related IAM changes reviewed.
[ ] Unauthorized persistence removed.
[ ] Unauthorized resources isolated, stopped, or removed.
[ ] Affected secrets and credentials rotated.
[ ] Legitimate applications restored and tested.
[ ] Logging, detection, and configuration monitoring verified.
[ ] Enhanced monitoring period started.
[ ] Post-incident review scheduled.
[ ] Corrective actions assigned owners and due dates.
```
