# Can Agent Toolkit for AWS Make Coding Agents Better Engineers?

I gave the same spec to the same coding agent twice: once on its own, and once with the [Agent Toolkit for AWS](https://aws.amazon.com/products/developer-tools/agent-toolkit-for-aws/) installed. Same agent, same model, same prompt, same AWS account and region. The only thing that changed was the toolkit.

The app in both runs is a serverless AI chat backend:

- API Gateway WebSocket API
- AWS Lambda
- Amazon Bedrock (Claude Sonnet 4.5), streamed token by token
- Amazon DynamoDB for session memory
- AWS CDK for deployment

Full write-up: *link coming soon*

## Results at a glance

| | A1: Agent only | B1: Agent + Toolkit |
|---|---|---|
| Deployed | Yes | Yes |
| Streamed responses | Yes | Yes |
| Remembered conversation | Yes | Yes |
| Minutes | 21 | 20 |
| Interventions | 1 hard | Soft only |
| Tokens (approx.) | 45k | 30k |
| IAM wildcard actions | 0 | 0 |
| Lambda runtime | nodejs20.x | nodejs22.x |

Both runs worked. The difference was the path: B1 loaded the `aws-core` skills before writing code and resolved the Bedrock inference profile error and cross-region IAM issue from the skill's troubleshooting guidance, while A1 worked them out from error messages. Both runs missed the same operational concerns (GoneException handling, DynamoDB TTL, timeout alignment, WebSocket reconnect and authentication).

This is one run per condition, so treat it as a field test, not a benchmark.

## Repository layout

```
runs/
  A1/                     Agent only (no toolkit)
    prompt.md             Prompt given to the agent
    RESULT.md             Deployment report
    notes.md              Experiment notes
    README.md             App README written during the run
    work/                 Generated code (CDK, Lambda, React frontend) and design notes
  B1/                     Agent + Agent Toolkit for AWS (aws-core plugin)
    prompt.md             Prompt given to the agent
    SPEC.md               The spec used by both runs
    RESULT.md             Deployment report
    TOOLKIT_USAGE.md      The agent's own log of what the toolkit contributed
    COMPARISON_NOTES.md   Notes on skills, docs and fixes
    infra/  lambda/  frontend/
    screenshots/          Screenshots from the B1 run
probe/test_ws.mjs         WebSocket test client used for A1
results/results.csv       Summary results
```

`TOOLKIT_USAGE.md` and `COMPARISON_NOTES.md` were written by the agent itself, so every claim in them was checked against run A1 and the session transcript.

## Running it yourself

Both stacks were destroyed after testing. To deploy one again you need Node.js, the AWS CDK CLI, AWS credentials, and access to Claude Sonnet 4.5 on Amazon Bedrock in us-east-1.

```bash
# B1 (agent + toolkit)
cd runs/B1/lambda && npm install
cd ../infra && npm install
npx cdk deploy
# test with the WebSocket URL from the stack outputs
WS_URL=wss://YOUR_API_ID.execute-api.us-east-1.amazonaws.com/prod ../test_websocket.sh

# clean up
npx cdk destroy
```

Note: neither stack has authentication on the WebSocket API. Don't leave it deployed, since anyone with the URL can call Bedrock on your account.

To install the toolkit in Claude Code:

```
/plugin marketplace add aws/agent-toolkit-for-aws
/plugin install aws-core@agent-toolkit-for-aws
/reload-plugins
```

## Notes

Account IDs, API IDs, stack IDs and WebSocket endpoints have been redacted from the reports and screenshots.
