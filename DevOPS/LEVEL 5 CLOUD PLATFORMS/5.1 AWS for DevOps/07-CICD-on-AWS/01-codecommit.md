# Chapter 7.1: AWS CodeCommit — Managed Git Repositories

## Overview

AWS CodeCommit is a **fully managed source control service** that hosts private Git repositories. It integrates natively with other AWS CI/CD services and IAM for access control.

> **Note:** AWS has announced CodeCommit is no longer available to new customers as of July 2024. Existing repositories continue to work. For new projects, consider **GitHub**, **GitLab**, or **Bitbucket**.

---

## CodeCommit Architecture

```
┌──── CodeCommit ──────────────────────────────────────────┐
│                                                           │
│  Developer Workstation          AWS CodeCommit            │
│  ┌──────────────────┐          ┌───────────────────┐     │
│  │                  │  HTTPS / │                   │     │
│  │  Local Git Repo  │──SSH────→│  Remote Repo      │     │
│  │  ├── .git/       │          │  ├── main         │     │
│  │  ├── src/        │  push/   │  ├── develop      │     │
│  │  └── README.md   │  pull    │  ├── feature/*    │     │
│  └──────────────────┘          │  └── tags/        │     │
│                                └────────┬──────────┘     │
│                                         │                 │
│  Triggers & Notifications:              │                 │
│  ┌──────────────────────────────────────┘                 │
│  │                                                        │
│  ▼                                                        │
│  ┌───────────┐  ┌───────────┐  ┌───────────────────┐     │
│  │EventBridge│  │ SNS Topic │  │ CodePipeline      │     │
│  │ Rule      │  │ Notify    │  │ (auto-trigger)    │     │
│  └───────────┘  └───────────┘  └───────────────────┘     │
└───────────────────────────────────────────────────────────┘
```

---

## Repository Setup

```bash
# Create a repository
aws codecommit create-repository \
  --repository-name my-app \
  --repository-description "Main application repo" \
  --tags Team=DevOps,Project=MyApp

# List repositories
aws codecommit list-repositories

# Get repository details
aws codecommit get-repository --repository-name my-app
```

### Authentication Methods

```
┌──── HTTPS (Recommended) ────────────────────────────────┐
│                                                          │
│  Option A: Git Credentials (IAM user)                    │
│  1. IAM Console → User → Security Credentials           │
│  2. Generate HTTPS Git credentials                      │
│  3. git clone https://git-codecommit.us-east-1           │
│          .amazonaws.com/v1/repos/my-app                  │
│                                                          │
│  Option B: Credential Helper (AWS CLI)                   │
│  git config --global credential.helper                   │
│    '!aws codecommit credential-helper $@'                │
│  git config --global credential.UseHttpPath true         │
│                                                          │
├──── SSH ─────────────────────────────────────────────────┤
│                                                          │
│  1. Generate SSH key pair                                │
│  2. Upload public key to IAM user                        │
│  3. Note the SSH Key ID                                  │
│  4. Configure ~/.ssh/config:                             │
│     Host git-codecommit.*.amazonaws.com                  │
│       User <SSH-Key-ID>                                  │
│       IdentityFile ~/.ssh/codecommit_rsa                 │
│  5. git clone ssh://git-codecommit.us-east-1             │
│          .amazonaws.com/v1/repos/my-app                  │
└──────────────────────────────────────────────────────────┘
```

---

## Branching Strategy

```bash
# Create a branch
aws codecommit create-branch \
  --repository-name my-app \
  --branch-name feature/add-auth \
  --commit-id abc123def456

# List branches
aws codecommit list-branches --repository-name my-app

# Create pull request
aws codecommit create-pull-request \
  --title "Add authentication module" \
  --description "Implements OAuth2 login flow" \
  --targets repositoryName=my-app,sourceReference=feature/add-auth,destinationReference=main

# Merge pull request (fast-forward)
aws codecommit merge-pull-request-by-fast-forward \
  --pull-request-id 42 \
  --repository-name my-app
```

---

## Triggers and Notifications

```bash
# Create SNS notification rule
aws codecommit put-repository-triggers \
  --repository-name my-app \
  --triggers '[{
    "name": "push-notify",
    "destinationArn": "arn:aws:sns:us-east-1:123456789012:repo-updates",
    "events": ["all"],
    "branches": ["main", "develop"]
  }]'

# EventBridge rule for specific events
# Events: referenceCreated, referenceUpdated, referenceDeleted,
#          pullRequestCreated, pullRequestMerged, commentOnPullRequest
```

---

## IAM Policies for CodeCommit

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowPushToDevelop",
      "Effect": "Allow",
      "Action": [
        "codecommit:GitPush",
        "codecommit:GitPull"
      ],
      "Resource": "arn:aws:codecommit:us-east-1:123456789012:my-app",
      "Condition": {
        "StringEqualsIfExists": {
          "codecommit:References": [
            "refs/heads/develop",
            "refs/heads/feature/*"
          ]
        }
      }
    },
    {
      "Sid": "DenyPushToMain",
      "Effect": "Deny",
      "Action": "codecommit:GitPush",
      "Resource": "arn:aws:codecommit:us-east-1:123456789012:my-app",
      "Condition": {
        "StringEqualsIfExists": {
          "codecommit:References": ["refs/heads/main"]
        },
        "Null": {
          "codecommit:References": "false"
        }
      }
    }
  ]
}
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| `403 Forbidden` on clone | Missing IAM permissions | Attach `AWSCodeCommitPowerUser` or custom policy |
| Credential helper not working | AWS CLI profile issue | Set `AWS_PROFILE` or configure default credentials |
| Push rejected to main | Branch protection or Deny policy | Use PR workflow; check IAM conditions on `GitPush` |
| Large file push fails | File > 6 MB via API | Use Git LFS or split large files |
| Cross-account access | Need role assumption | Use `sts:AssumeRole` with cross-account trust policy |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| **CodeCommit** | Fully managed, private Git repos. IAM-based access control. |
| **Auth Methods** | HTTPS Git credentials, HTTPS credential helper, SSH keys |
| **Triggers** | SNS notifications, Lambda triggers, EventBridge rules |
| **Branch Protection** | IAM policies with `codecommit:References` conditions |
| **Limits** | 25,000 repos per account; single file max 6 MB via API |
| **Status** | No longer available to new customers (July 2024). Use GitHub/GitLab. |

---

## Quick Revision Questions

1. **What authentication methods does CodeCommit support?**
   > HTTPS with IAM Git credentials, HTTPS with AWS CLI credential helper, and SSH with uploaded public keys.

2. **How can you prevent direct pushes to the main branch?**
   > Use an IAM Deny policy with `codecommit:References` condition matching `refs/heads/main` for the `GitPush` action.

3. **What events can trigger actions from CodeCommit?**
   > Branch/tag creation, updates, deletion; pull request events; comment events. Targets include SNS, Lambda, and EventBridge.

4. **How does CodeCommit integrate with CI/CD?**
   > CodePipeline can use CodeCommit as a source stage, automatically triggering pipelines on branch pushes.

5. **What is the current status of CodeCommit for new customers?**
   > No longer available to new AWS customers as of July 2024. Existing users can continue using it. Alternatives: GitHub, GitLab, Bitbucket.

---

[← Previous: 6.5 App Runner](../06-Container-Services/05-app-runner.md) | [Next: 7.2 CodeBuild →](02-codebuild.md)

[← Back to README](../README.md)
