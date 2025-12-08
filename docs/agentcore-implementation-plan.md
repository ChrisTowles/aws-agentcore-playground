# AWS Bedrock AgentCore Implementation Plan

Reference implementation for deploying agents with AWS Bedrock AgentCore using CloudFormation.

## Overview

This plan transforms the `dad-jokes-mcp` package into a fully deployed AgentCore agent with:
- CloudFormation infrastructure-as-code
- Runtime, Gateway, and Memory components
- ARM64 containerized deployment

## Prerequisites

- [ ] AWS CLI configured with credentials
- [ ] Docker with buildx (for ARM64 builds)
- [ ] Claude Sonnet model access enabled in Bedrock console
- [ ] Region: `us-east-1` (AgentCore default)

---

## Phase 1: Foundation Infrastructure

Create base CloudFormation templates for IAM and ECR.

### Tasks
- [ ] Create `infra/cfn/01-foundation.yaml`
  - IAM execution role with AgentCore trust policy
  - ECR repository for agent images
  - CloudWatch log group
- [ ] Validate template: `aws cloudformation validate-template`

### Key Resources
```yaml
AWS::IAM::Role          # AgentCoreExecutionRole
AWS::ECR::Repository    # agent container images
AWS::Logs::LogGroup     # /aws/bedrock-agentcore/runtimes/*
```

---

## Phase 2: AgentCore Runtime Template

Deploy the serverless agent runtime.

### Tasks
- [ ] Create `infra/cfn/02-runtime.yaml`
  - `AWS::BedrockAgentCore::Runtime` resource
  - Container configuration pointing to ECR
  - PUBLIC network mode
  - MCP protocol configuration
- [ ] Create `infra/cfn/03-runtime-endpoint.yaml`
  - `AWS::BedrockAgentCore::RuntimeEndpoint` for versioning

### Required Properties
| Property | Value |
|----------|-------|
| AgentRuntimeName | `dad-jokes-agent` |
| NetworkMode | `PUBLIC` |
| ProtocolConfiguration | `MCP` |
| RoleArn | From Phase 1 |

---

## Phase 3: AgentCore Gateway Template

Expose MCP tools through Gateway.

### Tasks
- [ ] Create `infra/cfn/04-gateway.yaml`
  - `AWS::BedrockAgentCore::Gateway` resource
  - AuthorizerType: `AWS_IAM`
  - ProtocolType: `MCP`
- [ ] Create `infra/cfn/05-gateway-targets.yaml`
  - `AWS::BedrockAgentCore::GatewayTarget` for each tool

---

## Phase 4: AgentCore Memory Template

Add persistent conversation memory.

### Tasks
- [ ] Create `infra/cfn/06-memory.yaml`
  - `AWS::BedrockAgentCore::Memory` resource
  - EventExpiryDuration: 30 days
  - Memory strategies configuration

---

## Phase 5: Update Agent Code

Modify the MCP server for AgentCore compatibility.

### Tasks
- [ ] Update `packages/dad-jokes-mcp/src/dad_jokes_mcp/mcp_server.py`
  - Add FastAPI app with `/invocations` POST endpoint
  - Add `/ping` GET health check endpoint
  - Integrate Strands or keep MCP tools
  - Port 8080
- [ ] Add dependencies to `pyproject.toml`
  - `fastapi`
  - `uvicorn[standard]`
  - `strands-agents` (optional)

### Required Endpoints
```python
@app.post("/invocations")  # Agent invocation
@app.get("/ping")          # Health check -> {"status": "healthy"}
```

---

## Phase 6: Update Dockerfile

ARM64 container for AgentCore.

### Tasks
- [ ] Update `Dockerfile` for ARM64
  - Base: `ghcr.io/astral-sh/uv:python3.13-bookworm-slim`
  - Platform: `linux/arm64`
  - Expose port 8080
  - CMD: uvicorn on 0.0.0.0:8080
- [ ] Test local build: `docker buildx build --platform linux/arm64`

---

## Phase 7: Deployment Scripts

Automation for deployment workflow.

### Tasks
- [ ] Create `scripts/deploy-foundation.sh`
- [ ] Create `scripts/build-and-push.sh`
- [ ] Create `scripts/deploy-runtime.sh`
- [ ] Create `scripts/invoke-agent.py`
- [ ] Add Makefile targets:
  ```makefile
  deploy-foundation  # Deploy IAM/ECR
  build-image        # Build ARM64 container
  push-image         # Push to ECR
  deploy-runtime     # Deploy AgentCore Runtime
  deploy-all         # Full deployment
  invoke             # Test agent invocation
  ```

---

## Phase 8: Deploy ECR & Push Image

### Tasks
- [ ] Deploy foundation stack
  ```bash
  make deploy-foundation
  ```
- [ ] Build ARM64 image
  ```bash
  make build-image
  ```
- [ ] Push to ECR
  ```bash
  make push-image
  ```

---

## Phase 9: Deploy AgentCore Runtime

### Tasks
- [ ] Deploy runtime stack
  ```bash
  make deploy-runtime
  ```
- [ ] Verify deployment status
  ```bash
  aws bedrock-agentcore get-agent-runtime --agent-runtime-id <id>
  ```

---

## Phase 10: Deploy Gateway (Optional)

### Tasks
- [ ] Deploy gateway stack
  ```bash
  make deploy-gateway
  ```
- [ ] Add gateway targets for MCP tools

---

## Phase 11: Test & Validate

### Tasks
- [ ] Create `scripts/invoke-agent.py` test script
- [ ] Test agent invocation
  ```bash
  make invoke PROMPT="Tell me a dad joke"
  ```
- [ ] Verify CloudWatch logs
- [ ] Document API usage in README

---

## File Structure (Target)

```
aws-agentcore-playground/
├── infra/
│   └── cfn/
│       ├── 01-foundation.yaml
│       ├── 02-runtime.yaml
│       ├── 03-runtime-endpoint.yaml
│       ├── 04-gateway.yaml
│       ├── 05-gateway-targets.yaml
│       └── 06-memory.yaml
├── scripts/
│   ├── deploy-foundation.sh
│   ├── build-and-push.sh
│   ├── deploy-runtime.sh
│   └── invoke-agent.py
├── packages/
│   └── dad-jokes-mcp/
│       └── src/dad_jokes_mcp/
│           └── mcp_server.py  # Updated with /invocations
├── Dockerfile  # ARM64
└── Makefile    # New targets
```

---

## References

- [AgentCore CloudFormation](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/AWS_BedrockAgentCore.html)
- [Runtime Resource](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/aws-resource-bedrockagentcore-runtime.html)
- [Getting Started Custom](https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/getting-started-custom.html)
- [Runtime Permissions](https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/runtime-permissions.html)
- [AgentCore Samples](https://github.com/awslabs/amazon-bedrock-agentcore-samples)
