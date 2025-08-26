# AWS AgentCore Playground

A playground repository for exploring and experimenting with AWS Bedrock AgentCore capabilities, particularly focusing on long-running asynchronous agentic workflows.

- **Github repository**: <https://github.com/ChrisTowles/towles-agentcore-playground/>
- **AWS Documentation**: [Bedrock AgentCore Runtime Long-Running Tasks](https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/runtime-long-run.html)

## Overview

AWS Bedrock AgentCore Runtime enables building intelligent agents that can process long-running tasks asynchronously. This playground demonstrates:

> **Note**: AWS Bedrock AgentCore is currently in preview and features may change.

## Prerequisites

- Python 3.13+
- AWS Account with Bedrock access
- AWS CLI configured with appropriate credentials

## Installation

```bash
# Clone the repository
git clone https://github.com/ChrisTowles/towles-agentcore-playground.git
cd towles-agentcore-playground

# Install uv if not already installed
# See https://docs.astral.sh/uv/ for more installation options
curl -LsSf https://astral.sh/uv/install.sh | sh

# Install dependencies and create virtual environment
uv sync

# Activate the virtual environment
source .venv/bin/activate  # On Windows: .venv\Scripts\activate
```

## Quick Start

### Using the Agent Start CLI

This repository includes a CLI tool (`agent-start`) for managing AgentCore experiments and utilities. After installation, you can use it as follows:

```bash
# Run commands using uv (recommended)
uv run agent-start --help

# Or if you've activated the virtual environment
agent-start --help
```


## Resources

- [AWS Bedrock AgentCore Documentation](https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/)
- [Runtime Long-Running Tasks Guide](https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/runtime-long-run.html)
- [AWS Bedrock Overview](https://aws.amazon.com/bedrock/)

## Blog Posts

I blog about my learnings with AWS and AI. Check it out here: https://chris.towles.dev/ and follow me on Twitter: https://twitter.com/Chris_Towles
