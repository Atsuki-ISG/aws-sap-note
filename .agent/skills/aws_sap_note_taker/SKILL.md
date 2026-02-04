---
name: AWS SAP Note Taker
description: Automates the process of creating study notes from exam question screenshots.
---

# AWS SAP Note Taker Skill

This skill guides the agent to process a screenshot of an AWS SAP exam question, transcribe it, generate a study note, and save it to the repository.

## Prerequisites

- The user must provide an image (screenshot of the question).
- The repository must be `/Users/aki/aws-sap-note` (or the current workspace).

## Process

### 1. Analyze Information
1.  **Read the Image**: Look at the provided screenshot.
2.  **Extract Data**:
    -   **ID**: Look for a question ID (e.g., "[SAP-53]"). If not found, use "SAP-XX" or ask the user.
    -   **Category**: Determine the AWS category (e.g., specific to Networking, Compute, Database, etc.). If unsure, default to "Other" or infer from the services mentioned (e.g., CloudFront -> Networking).
    -   **Problem Text**: Transcribe the question body exactly.
    -   **Options**: Transcribe options A, B, C, D...
    -   **Answer**: If a selected answer is visible/marked, note it. If not, the AI will deduce it.

### 2. File Paths Determination
-   **Problem File**: `problems/<Category>/<ID>-<ShortTitle>-problem.md`
-   **Note File**: `notes/<Category>/<ID>-<ShortTitle>.md`
-   *ShortTitle*: A brief, kebab-case summary of the topic (e.g., `cloudfront-5xx-troubleshooting`).

### 3. Create "Problem" File
Create the file at the **Problem File** path using the content:

```markdown
---
id: "<ID>"
title: "<ShortTitle>"
related_note: "../../notes/<Category>/<ID>-<ShortTitle>.md"
---

# Problem Statement

<Transcribed Text>

# Options

A. <Option A Text>
B. <Option B Text>
...

# Answer
<Correct Answer Letter if known>
```

### 4. Generage "Note" Content
Use the **Strategic Architect Mentor (SAM)** prompt stored in `_templates/prompt.md` (or the equivalent system instruction) to generate the explanation.
-   **Input**: The transcribed problem text and options.
-   **Role**: As defined in the prompt (Certified Instructor + Senior Solutions Architect + Exam Specialist).

### 5. Create "Note" File
Create the file at the **Note File** path using the content:

```markdown
---
id: "<ID>"
title: "<ShortTitle>"
category: "<Category>"
domain: "Unknown" # You can attempt to infer this or leave as Unknown
tags:
  - "<Tag1>"
  - "<Tag2>"
created_at: "<Current Date YYYY-MM-DD>"
---

# <ID> <Title>

<Paste Generated Content Here>
```

### 6. Git Operations
1.  Run `git add .`
2.  Run `git commit -m "Add notes for <ID> <ShortTitle>"`
3.  Run `git push origin main`

## Automatic Trigger
If the user uploads an image and says "Save this" or "Analyze this screenshot", follow this process.
