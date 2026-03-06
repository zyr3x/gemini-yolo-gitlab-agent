# Gemini YOLO GitLab Agent

An AI-powered automation suite for GitLab, leveraging Google's Gemini models and the YOLO (You Only Look Once) execution mode to automate repository management, code reviews, and issue triage. This agent acts as an autonomous AI software engineer, planning and executing tasks directly within your CI/CD pipeline.

## 🚀 Overview

This project provides a set of GitLab CI/CD workflows and AI prompts that allow Google's Gemini to interact directly with your GitLab repository. Unlike traditional LLM integrations, the "YOLO" mode enables the agent to autonomously plan and execute tools (like reading files, searching code, or interacting with the GitLab API) to fulfill complex requests without human intervention for each step.

## 🛠 How It Works

1.  **Gemini CLI**: The core engine driving the AI interactions is the `gemini` command-line tool.
2.  **YOLO Mode**: Enabled via the `--yolo` flag, this powerful mode allows the AI to "think" through a problem, formulate a plan, and then execute available tools (shell commands, API calls via MCP) until the task is complete.
3.  **GitLab CI Integration**: Workflows are triggered by standard GitLab events (Merge Requests, Issues, Pipeline Schedules) ensuring seamless integration into your development process.
4.  **MCP (Model Context Protocol)**: Utilizes the GitLab MCP server to provide the AI with structured, secure access to GitLab's API, enabling it to perform repository operations.

## ✨ Key Features

### 🔍 YOLO Review

Automates comprehensive code reviews for Merge Requests. The AI analyzes the code diff, understands the context by reading relevant files, and provides constructive feedback or actionable suggestions directly as MR comments.

**Trigger:** Automatic on all Merge Request events.

### 🏷️ YOLO Triage

Triages and labels a single issue. The AI reads the issue title and description, compares them against available project labels, and applies the most relevant ones to keep your backlog organized.

**Trigger:** Manual pipeline run with the variable `YOLO_COMMAND=triage`. This job requires issue details (like `CI_ISSUE_IID`) to be passed, which is typical when run from an issue webhook or a manual trigger with specified variables.

### ⚡ YOLO Invoke

A flexible command processor for on-demand AI actions. Enables tasks like refactoring, documentation generation, or in-depth bug analysis based on a high-level prompt.

**Trigger:** Manual pipeline run with the variable `YOLO_COMMAND=invoke`.

### ✍️ YOLO Plan & Execute

Enables multi-step, complex operations that require human approval. The AI generates a detailed plan of action, posts it for review, and once approved, executes the plan autonomously.

**Trigger:** The execution part is triggered by a manual pipeline run with the variable `YOLO_COMMAND=approve`.

### ⏰ Scheduled Triage

A periodic job that scans for all open, untriaged issues and proactively applies appropriate labels, ensuring continuous backlog organization without manual intervention.

**Trigger:** Automatic via a scheduled pipeline.

## 📋 Project Structure

-   `.gitlab-ci.yml`: Defines the CI/CD pipeline, orchestrating the execution of various AI tasks.
-   `.gitlab/commands/`: Contains markdown-based prompt templates that define the AI's persona, instructions, and workflow for different tasks:
    -   `yolo-review.md`: Prompts and instructions for automated code reviews.
    -   `yolo-triage.md`: Prompts and instructions for single-issue labeling.
    -   `yolo-invoke.md`: Prompts for general-purpose custom command execution.
    -   `yolo-plan-execute.md`: Prompts for verifying and executing approved multi-step plans.
    -   `yolo-scheduled-triage.md`: Prompts and instructions for bulk scheduled issue triage.

## ⚙️ Setup

### Prerequisites

To run this agent, your GitLab CI/CD runners must have:

-   **Node.js (LTS recommended) and npm/npx**: Required for the MCP server wrapper.
-   **`jq`**: A lightweight and flexible command-line JSON processor.
-   **`curl`**: For interacting with the GitLab API.
-   **`gemini` CLI**: The Google Gemini command-line tool. Instructions for installation can be found [here](https://gemini.google.com/cli).
-   A GitLab Runner (configured with the `self-hosted` tag as per the current `.gitlab-ci.yml`, or updated to match your environment).

### Environment Variables

The following variables **must** be configured in your GitLab Project or Group (CI/CD Settings > CI/CD > Variables):

| Variable | Description | Recommended Scope |
| :--- | :--- | :--- |
| `GITLAB_TOKEN` | A Personal Access Token (PAT) with `api` scope for the bot to perform actions (e.g., commenting, creating MRs, applying labels). | Protected branches, optionally all branches |
| `GEMINI_CLI_HOME` | (Optional) Path to the Gemini CLI configuration directory. Defaults to `${CI_PROJECT_DIR}/.gemini` in the CI pipeline. | All branches |
| `MCP_SERVER_VERSION` | (Optional) Specifies the version of `@modelcontextprotocol/server-gitlab` to use. Defaults to `0.6.2`. | All branches |

### Installation

1.  **Copy Files**: Copy the entire `.gitlab/` directory and the `.gitlab-ci.yml` file into the root of your GitLab repository.
2.  **Configure Environment Variables**: Set up the required environment variables (`GITLAB_TOKEN`, `GEMINI_CLI_HOME`, `MCP_SERVER_VERSION`) in your GitLab project's CI/CD settings.
3.  **Runner Configuration**: Ensure your GitLab runner has the necessary prerequisites installed (Node.js, npm, jq, curl, gemini CLI) and is tagged appropriately (e.g., `self-hosted` as used in the default `.gitlab-ci.yml`).
4.  **Test**: Trigger a pipeline to ensure everything is set up correctly.

## 🤝 Contributing

This project is designed to be highly customizable. You can modify the markdown-based prompts in `.gitlab/commands/` to better suit your team's coding standards, triage logic, or to introduce entirely new AI-driven workflows.