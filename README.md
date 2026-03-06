# Gemini YOLO GitLab Agent

An AI-powered automation suite for GitLab, leveraging Google's Gemini models and the YOLO (You Only Look Once) execution mode to automate repository management, code reviews, and issue triage.

## 🚀 Overview

This project provides a set of GitLab CI/CD workflows and AI prompts that allow Google's Gemini to interact directly with your GitLab repository. Unlike traditional LLM integrations, the "YOLO" mode allows the agent to autonomously execute tools (like reading files, searching code, or using the GitLab API) to fulfill complex requests.

## 🛠 How It Works

1. **Gemini CLI**: The core engine is the `gemini` command-line tool.
2. **YOLO Mode**: Enabled via the `--yolo` flag, this allows the AI to "think" and then execute available tools (shell commands, API calls via MCP) until the task is complete.
3. **GitLab CI Integration**: Workflows are triggered by GitLab events (Merge Requests, Issues, Pipeline Schedules).
4. **MCP (Model Context Protocol)**: Uses the GitLab MCP server to give the AI structured access to GitLab's API.

## ✨ Key Features

### 🔍 YOLO Review

Automated code reviews for Merge Requests. The AI analyzes the diff, understands the context by reading relevant files, and provides feedback or suggestions directly on the MR.

### 🏷 YOLO Triage

Automatically labels new issues. The AI reads the issue title and description, compares them against available project labels, and applies the most relevant ones.

### ⚡ YOLO Invoke

A flexible command processor. You can trigger custom AI actions by mentioning the bot or using specific keywords in comments, allowing for on-demand refactoring, documentation generation, or bug analysis.

### ⏰ Scheduled Triage

A periodic job that scans for untriaged issues and applies labels, keeping your backlog organized without manual intervention.

## 📋 Project Structure

- `.gitlab-ci.yml`: Defines the CI/CD pipeline and the high-level orchestration of AI tasks.
- `.gitlab/commands/`: Contains markdown-based prompt templates for different AI workflows:
  - `yolo-review.md`: Prompts for code review.
  - `yolo-triage.md`: Prompts for issue labeling.
  - `yolo-invoke.md`: Prompts for custom command execution.
  - `yolo-plan-execute.md`: Prompts for executing complex multi-step plans.

## ⚙️ Setup

### Prerequisites

- A GitLab Runner (configured with the `self-hosted` tag as per the current `.gitlab-ci.yml`, or updated to match your environment).
- Gemini CLI installed and configured on the runner.

### Environment Variables

The following variables must be configured in your GitLab Project or Group (CI/CD Settings):

| Variable | Description |
| :--- | :--- |
| `GITLAB_TOKEN` | A Personal Access Token (PAT) with `api` scope for the bot to perform actions. |
| `GEMINI_CLI_HOME` | (Optional) Path to the Gemini CLI configuration directory. Defaults to `${CI_PROJECT_DIR}/.gemini`. |

### Installation

Simply copy the `.gitlab/` directory and `.gitlab-ci.yml` file to your repository and configure the required environment variables.

## 🤝 Contributing

This project is designed to be highly customizable. You can modify the prompts in `.gitlab/commands/` to better suit your team's coding standards or triage logic.
