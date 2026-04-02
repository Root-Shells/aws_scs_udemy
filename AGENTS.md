# Agentic Workflow Guidelines - AWS SCS Study Notes

This repository contains study notes for the AWS Certified Security - Specialty (SCS-C02) exam. It is structured as a collection of Markdown files organized by exam domains.

## 1. Project Overview & Workflow

The goal is to transform raw technical learning notes into a polished **reference document for study, review, and long-term retention**.

### Implementation Steps
1. **Identify Domain**: Place files in the correct `domain_X_name` folder.
2. **File Naming**: Use `snake_case.md` for service/topic files. Each file name should start with a numeric prefix (e.g., `01_`, `02_`) to indicate the order in the directory in which it was created to track learning/reference order.
3. **Drafting**: Transform raw notes into well-structured markdown focusing on clarity, correctness, and practical understanding.
4. **Verification**: Cross-reference with AWS documentation or exam guides. Mark uncertainties with `> Needs verification:`.

## 2. Content Structure & Style

Every document should follow this standardized structure unless the content calls for a different one:

# [Topic/Service Name]

## Overview
Brief summary of the topic and why it matters.

## Key Concepts
Explain foundational terms and ideas.

## Detailed Notes
Expanded organized subsections covering capabilities and mechanics.

## Architecture / Flow
Include Mermaid diagram(s) where useful.

## Security Relevance
Explain why this matters from an AWS/cloud security perspective (Preventive vs Detective vs Corrective).

## Operational / Real-World Context
How this appears in enterprise environments (Centralized vs Local, Organizations vs Per-account).

## Common Pitfalls / Misconfigurations
List frequent mistakes, weak points, or misconfigurations.

## Exam / Review Notes
Highlight distinctions, confusing points, decision points/tradeoffs, and key takeaways for the SCS-C02 exam.

## Summary
Short recap for fast review.

## Quick Review Checklist
Concise bullet list of the most important takeaways.

### Markdown Formatting Rules
- **H1**: Topic/Service Name.
- **Emphasis**: Use **bold** for AWS service names, key terms, and specific API actions.
- **Callouts**:
  - `> Note:`: Important detail.
  - `> Caution:`: Potential danger or misconfiguration.
  - `> Exam Tip:`: High-probability exam topic.
  - `> Operational Insight:`: Real-world practice.

### Mermaid Diagrams
- Use Mermaid only where it adds value (e.g., identity federation, IAM role assumption, KMS envelope encryption, incident response workflows).
- **Direction**: Prefer `LR` for simple flows and `TD` for hierarchies.
- **Styling**: Use `subgraph` to group related components (e.g., "Data Sources", "Targets").
- **Node Labels**:
  - Keep node names readable and avoid overly dense diagrams.
  - **New Lines**: Never use the literal `\n` for new lines in node labels as it may not render correctly in all viewers. Instead, use the HTML break tag `<br>` to force a new line (e.g., `NodeA["Line 1<br>Line 2"]`).
  - Always wrap node labels in double quotes (e.g., `NodeA["Label"]`) to ensure compatibility and prevent parsing errors.

## 3. Core Instructions for Agents

### Accuracy and Quality
- **Technical Accuracy**: Do not invent AWS facts. If notes are ambiguous, mark them clearly.
- **Extrapolation**: Expand on implied concepts. Explain *why* something matters, not just *what* it is.
- **Tone**: Professional, practical, technical, and concise.

### Comparison and Distinction
Explicitly distinguish between:
- Identity-based vs Resource-based policies.
- Encryption at rest vs In transit.
- Monitoring vs Enforcement.
- Alerting vs Automated remediation.

### Security Best Practices
- **No Secrets**: Never include real AWS account IDs, access keys, or sensitive configuration.
- **Consistency**: Read existing files in the domain to match the tone and depth.

## 4. Commands & Git Guidelines

### Validation
- **Markdown**: `markdownlint "**/*.md"` (if available).
- **Links**: `find . -name "*.md" -exec markdown-link-check {} \;` (if available).

### Git
- **Commits**: Use descriptive, imperative messages (e.g., "Add CloudWatch Contributor Insights section").
- **Ignored Files**: Local artifacts like `main.pdf` or IDE settings must not be committed. Use the `.gitignore`.
