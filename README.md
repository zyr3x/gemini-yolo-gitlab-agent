# Gemini YOLO GitLab Agent

An AI-powered automation suite for GitLab, leveraging Google's Gemini models and the YOLO (You Only Look Once) execution mode to automate repository management, code reviews, and issue triage. This agent acts as an autonomous AI software engineer, planning and executing tasks directly within your CI/CD pipeline.

## 🚀 Overview

This project provides a set of GitLab CI/CD workflows and AI prompts that allow Google's Gemini to interact directly with your GitLab repository. Unlike traditional LLM integrations, the "YOLO" mode enables the agent to autonomously plan and execute tools (like reading files, searching code, or interacting with the GitLab API) to fulfill complex requests without human intervention for each step.

## 🛠 How It Works

1. **Gemini CLI**: The core engine driving the AI interactions is the `gemini` command-line tool.
2. **YOLO Mode**: Enabled via the `--yolo` flag, this powerful mode allows the AI to "think" through a problem, formulate a plan, and then execute available tools (shell commands, API calls via MCP) until the task is complete.
3. **GitLab CI Integration**: Workflows are triggered by standard GitLab events (Merge Requests, Issues, Pipeline Schedules) ensuring seamless integration into your development process.
4. **MCP (Model Context Protocol)**: Utilizes the GitLab MCP server to provide the AI with structured, secure access to GitLab's API, enabling it to perform repository operations.

## 🏗 Pipeline Architecture

The pipeline is organized into **3 stages** that execute sequentially:

```
┌─────────────┐     ┌─────────────────────────┐     ┌──────────────┐
│  dispatch    │ ──▶ │         yolo            │ ──▶ │    notify     │
│             │     │                         │     │              │
│ • Determine │     │ • yolo-review     (10m) │     │ • yolo-      │
│   command   │     │ • yolo-triage     (10m) │     │   fallback   │
│ • Post ack  │     │ • yolo-invoke     (10m) │     │   (on_failure │
│   comment   │     │ • yolo-plan-exec  (30m) │     │    only)     │
│             │     │ • yolo-sched-triage(15m)│     │              │
└─────────────┘     └─────────────────────────┘     └──────────────┘
```

### Stage 1: `dispatch`

Determines which YOLO command to execute based on the pipeline source:

| Source | Action |
| :--- | :--- |
| Merge Request event | Sets `YOLO_COMMAND=review`, posts an acknowledgment comment 🤖 |
| Manual trigger with `YOLO_COMMAND` variable | Uses the provided command (`triage`, `invoke`, `approve`) |
| Pipeline schedule | Skips dispatch, runs `yolo-scheduled-triage` directly |
| Otherwise | Sets `YOLO_COMMAND=fallthrough` (no action) |

### Stage 2: `yolo`

Runs exactly **one** YOLO job based on `YOLO_COMMAND`. Each job has:

- **Timeout**: Prevents infinite hangs (10–30 minutes depending on the job).
- **Concurrency control**: `resource_group` ensures only one YOLO job runs per MR at a time. A second pipeline for the same MR will queue instead of conflicting.
- **Prompt templating**: Variables like `$REPOSITORY`, `$PULL_REQUEST_NUMBER` are injected into the prompt via `envsubst` before passing to Gemini.
- **Label validation** (triage jobs): Labels selected by the AI are validated against the project's actual label list to prevent prompt injection.

### Stage 3: `notify`

The `yolo-fallback` job runs **only when a YOLO job fails** (`when: on_failure`). It posts an error message to the MR/Issue with a link to the pipeline logs so the user knows something went wrong.

## ✨ Key Features

### 🔍 YOLO Review

Automates comprehensive code reviews for Merge Requests. The AI analyzes the code diff, understands the context by reading relevant files, and provides constructive feedback with severity indicators (🔴 Critical, 🟠 High, 🟡 Medium, 🟢 Low) directly as MR comments.

**Trigger:** Automatic on all Merge Request events (open, synchronize).

### 🏷️ YOLO Triage

Triages and labels a single issue. The AI reads the issue title and description, compares them against available project labels, and applies the most relevant ones.

**Trigger:** Manual pipeline run with `YOLO_COMMAND=triage`.

### ⚡ YOLO Invoke

A flexible command processor for on-demand AI actions. Enables tasks like refactoring, documentation generation, or in-depth bug analysis. The AI creates a "Plan of Action" comment and waits for approval.

**Trigger:** Manual pipeline run with `YOLO_COMMAND=invoke`.

### ✍️ YOLO Plan & Execute

Executes a previously approved plan of action. The AI looks for a comment titled "AI Assistant: Plan of Action" in the issue/MR, then executes each step sequentially using GitLab tools.

**Trigger:** Manual pipeline run with `YOLO_COMMAND=approve`.

### ⏰ Scheduled Triage

A periodic job that scans for all open, unlabeled issues and proactively applies appropriate labels. Results are validated against the project's label list before being applied.

**Trigger:** Automatic via a scheduled pipeline.

## 📖 Usage

### Automatic Review (no action needed)

Simply open or update a Merge Request — the pipeline triggers automatically, and the AI posts a review comment. All required variables (`CI_MERGE_REQUEST_TITLE`, `CI_MERGE_REQUEST_IID`, etc.) are provided by GitLab automatically for MR pipelines.

### Manual Triggers via GitLab UI

Go to **Build → Pipelines → Run pipeline**, select the branch, and add the required variables:

> **Note:** Variables prefixed with `CI_MERGE_REQUEST_*` are auto-filled by GitLab for MR pipelines. Variables prefixed with `CI_ISSUE_*` do **not** exist in GitLab CI by default and **must** be passed manually.

#### Triage an issue

| Variable | Required | Description |
| :--- | :---: | :--- |
| `YOLO_COMMAND` | ✅ | Set to `triage` |
| `CI_ISSUE_IID` | ✅ | Issue number (e.g. `42`) |
| `CI_ISSUE_TITLE` | ✅ | Issue title text |
| `CI_ISSUE_DESCRIPTION` | ✅ | Issue description / body text |

#### Invoke a custom command (on a MR)

| Variable | Required | Description |
| :--- | :---: | :--- |
| `YOLO_COMMAND` | ✅ | Set to `invoke` |
| `CI_MERGE_REQUEST_IID` | ✅ | Merge Request number |
| `CI_MERGE_REQUEST_TITLE` | ❌ | Auto-filled for MR pipelines |
| `CI_MERGE_REQUEST_DESCRIPTION` | ❌ | Auto-filled for MR pipelines |

#### Invoke a custom command (on an issue)

| Variable | Required | Description |
| :--- | :---: | :--- |
| `YOLO_COMMAND` | ✅ | Set to `invoke` |
| `CI_ISSUE_IID` | ✅ | Issue number |
| `CI_ISSUE_TITLE` | ✅ | Issue title text |
| `CI_ISSUE_DESCRIPTION` | ✅ | Issue description / body text |

#### Approve and execute a plan

| Variable | Required | Description |
| :--- | :---: | :--- |
| `YOLO_COMMAND` | ✅ | Set to `approve` |
| `CI_MERGE_REQUEST_IID` | ✅ | MR number (or `CI_ISSUE_IID` for issues) |
| `CI_MERGE_REQUEST_TITLE` | ❌ | Auto-filled for MR pipelines |
| `CI_MERGE_REQUEST_DESCRIPTION` | ❌ | Auto-filled for MR pipelines |

### Manual Triggers via API

```bash
# Example: Triage issue #42
curl --request POST \
  --header "PRIVATE-TOKEN: $GITLAB_TOKEN" \
  --form "ref=main" \
  --form "variables[YOLO_COMMAND]=triage" \
  --form "variables[CI_ISSUE_IID]=42" \
  --form "variables[CI_ISSUE_TITLE]=Bug in login page" \
  --form "variables[CI_ISSUE_DESCRIPTION]=Login fails with 500 error" \
  "https://gitlab.example.com/api/v4/projects/PROJECT_ID/pipeline"
```

### Setting up Scheduled Triage

Go to **Build → Pipeline schedules → New schedule**, set a cron (e.g. `0 */6 * * *` for every 6 hours), and the pipeline will automatically triage unlabeled issues.

## 📋 Project Structure

```
.gitlab-ci.yml                          # Pipeline definition (3 stages)
.gitlab/commands/
  ├── yolo-review.md                    # Code review prompt
  ├── yolo-triage.md                    # Issue labeling prompt
  ├── yolo-invoke.md                    # Custom command prompt
  ├── yolo-plan-execute.md              # Plan execution prompt
  └── yolo-scheduled-triage.md          # Bulk triage prompt
.gemini/
  └── settings.json                     # (Generated at runtime) Gemini CLI config
```

## ⚙️ Setup

### Prerequisites

Your GitLab CI/CD runner must have:

- **Node.js** (LTS recommended) and **npm/npx** – for the MCP server wrapper
- **`jq`** – command-line JSON processor
- **`curl`** – for GitLab API interactions
- **`envsubst`** – for prompt variable substitution (part of `gettext` package)
- **`gemini` CLI** – the Google Gemini command-line tool ([Gemini CLI Documentation](https://geminicli.com/))
- A GitLab Runner tagged `self-hosted` (or update the tag in `.gitlab-ci.yml`)

### Environment Variables

Configure these in **Settings → CI/CD → Variables**:

| Variable | Required | Description |
| :--- | :---: | :--- |
| `GITLAB_TOKEN` | ✅ | Personal Access Token with `api` scope for commenting, creating MRs, applying labels |
| `GEMINI_API_KEY` | ❌ | API key for Gemini. Not needed if the runner already has `gemini` CLI authenticated (e.g. via `gemini auth login`) |
| `MCP_SERVER_VERSION` | ❌ | Version of `@modelcontextprotocol/server-gitlab` (default: `0.6.2`) |

### Installation

1. **Copy Files**: Copy `.gitlab/` directory and `.gitlab-ci.yml` into your repository root.
2. **Configure Variables**: Set `GITLAB_TOKEN` and `GEMINI_API_KEY` in CI/CD settings.
3. **Runner Setup**: Ensure your runner has all prerequisites installed.
4. **Test**: Create a test MR — the pipeline should trigger automatically and post a review.

## 🔒 Security

- **Label validation**: All labels selected by the AI are cross-checked against the project's actual label list, preventing prompt injection attacks.
- **Prompt isolation**: All external data (issue titles, MR descriptions) is treated as untrusted context, not executable instructions.
- **Tool restrictions**: The Gemini CLI is configured with a minimal set of allowed shell commands (`cat`, `echo`, `grep`, `head`, `tail`, `jq`, `printenv`).
- **Token fallback**: The MCP server uses `$GITLAB_TOKEN` when available, falling back to the CI job token (`$CI_JOB_TOKEN`) for basic operations.

## 🤝 Contributing

This project is designed to be highly customizable. You can modify the markdown-based prompts in `.gitlab/commands/` to better suit your team's coding standards, triage logic, or to introduce entirely new AI-driven workflows.
