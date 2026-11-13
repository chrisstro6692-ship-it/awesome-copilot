---
name: aws-resource-query
description: 'Query AWS resources using natural language. Covers EC2, S3, RDS, Lambda, ECS, EKS, Secrets Manager, IAM, VPC, networking, messaging, and more. Strictly read-only — no writes, deletes, or mutations.'
---

# AWS Resource Query

Answer natural language questions about AWS resources by translating intent into read-only AWS CLI commands. This skill **never** runs commands that create, modify, or delete resources.

## Safety Contract

**STRICTLY READ-ONLY.** This skill exclusively uses:
- `aws <service> describe-*`
- `aws <service> list-*`
- `aws <service> get-*`
- `aws sts get-caller-identity`
- `aws configure get`
- `aws resourcegroupstaggingapi get-resources`
- `aws ce get-*`
- `aws support describe-*`

**NEVER** run any of the following, regardless of what the user asks:
`create-*`, `run-*`, `start-*`, `stop-*`, `reboot-*`, `delete-*`, `terminate-*`, `put-*`, `update-*`, `modify-*`, `attach-*`, `detach-*`, `send-*`, `publish-*`, `invoke-*`, `execute-*`

If the user's query implies a write action, respond:
> "This skill is read-only. I can show you the current state of [resource], but I cannot [create/modify/delete] it. Would you like to see what currently exists?"

## Workflow

### Step 1: Parse Intent
Identify: target service(s), scope (all / filtered / specific), detail level, and region.

### Step 2: Confirm Account & Region
```bash
aws sts get-caller-identity --query '{Account:Account,UserId:UserId}'
aws configure get region
```
Append `--region <region>` to all commands when the user specifies one.

### Step 3: Execute & Format
Run the matched read-only command(s) from the reference files linked below and format results as a readable table. For large result sets show a count first and offer to filter further.

---

## Intent → Command Mapping

### Compute, Storage, Databases, Networking Commands

Intent-to-command mappings for these services. See [references/commands-compute-storage-databases-networking.md](references/commands-compute-storage-databases-networking.md) for:

- COMPUTE
- STORAGE
- DATABASES
- NETWORKING

### Security, Messaging, Serverless, Monitoring, Cost, and Cross-Service Commands

Intent-to-command mappings for these services. See [references/commands-security-messaging-monitoring-cost.md](references/commands-security-messaging-monitoring-cost.md) for:

- SECURITY & IDENTITY
- MESSAGING & EVENTS
- API GATEWAY & SERVERLESS
- MONITORING & OBSERVABILITY
- COST & BILLING
- CROSS-SERVICE QUERIES

## Output Formatting Rules

1. Always use `--output table` for list results; use `--output json` only when deep detail is explicitly requested
2. Always use `--query` to extract only relevant fields — never dump raw JSON
3. For large result sets (>20 items), show a count first, then offer to filter
4. When a command returns nothing, explain why (wrong region, no resources, insufficient permissions)
5. Offer to drill into a specific resource: "Found 47 EC2 instances. Filter by state, type, or tag?"

## Error Handling

| Error | Response |
|---|---|
| `AccessDenied` | "You don't have permission to list [resource]. Required: `<service>:<Action>`." |
| `NoCredentialProviders` | "Run `aws configure` or set `AWS_PROFILE`." |
| Empty result | "No [resources] found in [region]. Check another region?" |
| Invalid identifier | "Could not find '[name]'. Check the name or provide the resource ID." |
