# AWS SAP Study Notes

This repository stores study notes and problem transcriptions for the AWS Certified Solutions Architect - Professional (SAP-C02) exam.

## Directory Structure

- `notes/`: Explanation and study notes deeply comparing services and architectures. Organized by category (e.g., Networking, Compute).
- `problems/`: Transcribed problem statements and options from practice exams.
- `_templates/`: Templates for creating new notes and the prompt used for AI generation.

## How to add a new note

1.  **Transcribe the Problem**:
    - Create a new file in `problems/<Category>/` using `_templates/problem_template.md`.
    - Fill in the problem statement and options.
2.  **Generate the Note**:
    - Use the prompt in `_templates/prompt.md`.
    - Paste the problem context.
3.  **Save the Note**:
    - Create a new file in `notes/<Category>/` using `_templates/note_template.md`.
    - Paste the AI-generated content.
    - Ensure the `id` matches the problem file.

## Categories

- **Architecture**: Design patterns, multi-account strategies, etc.
- **Compute**: EC2, Lambda, ECS, EKS...
- **Database**: RDS, DynamoDB, Aurora, ElastiCache...
- **Networking**: VPC, DX, VPN, Route53, CloudFront...
- **Security**: IAM, KMS, WAF, Shield, GuardDuty...
- **Storage**: S3, EBS, EFS, FSx, Storage Gateway...
- **Other**: Migration, Cost, and cross-cutting concerns.
