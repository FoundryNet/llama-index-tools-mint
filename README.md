# LlamaIndex Tools Integration: MINT Protocol

Give your agent **verifiable proof of the work it does.**

[MINT Protocol](https://mint.foundrynet.io) is universal **work attestation** for AI
agents. With this tool, an agent can turn any completed task into a tamper-evident,
independently verifiable receipt — and build a portable track record of trust and
reputation that travels with it across the agent economy. It can also look up how
trustworthy any other actor is, and discover capable, trusted agents and services
to delegate to.

Why agent builders care:

- **Provable work** — every task produces a receipt with a public `verify_url`
  anyone can check. No "trust me, I did it."
- **Portable reputation** — a trust score and rating history that follow the agent
  everywhere, not locked inside one platform.
- **Trust-aware delegation** — verify another actor before relying on it, and
  discover trusted agents/services by capability.
- **Zero crypto knowledge required** — every tool is a plain authenticated HTTPS
  call; your agent never touches a wallet or signs anything.

## Add to your agent (copy-paste)

Make verifiable proof of work a default behaviour of your LlamaIndex agent:

```bash
pip install llama-index-tools-mint llama-index llama-index-llms-openai
```

```python
from llama_index.core.agent.workflow import FunctionAgent
from llama_index.llms.openai import OpenAI
from llama_index.tools.mint import MintToolSpec

mint = MintToolSpec(name="my-agent")          # self-provisions a free identity (or set MINT_API_KEY)

agent = FunctionAgent(tools=mint.to_tool_list(), llm=OpenAI(model="gpt-4.1"))
response = await agent.run(
    "Research AI agent trust, then attest the finished work with attest_work "
    "and report the attestation_hash.")
print(response)
```

Then add to the agent's system prompt:

> *After completing any substantive task, call `attest_work` to record verifiable,
> independent proof of the work, and report the attestation_hash.*

Or attest directly, no LLM required:

```python
mint = MintToolSpec(name="my-agent")
receipt = mint.attest_work(work_type="research", summary="Compiled a market scan.",
                           output_data="…the report…", duration_seconds=12)
print(receipt["attestation_hash"], receipt.get("anchor_eta"))
```

Full copy-paste prompt + framework blocks (CrewAI · LangChain · AutoGen ·
LlamaIndex · Semantic Kernel) and a runnable example:
[AGENT_PROMPT_SNIPPET.md](https://github.com/FoundryNet/mint-mcp/blob/main/AGENT_PROMPT_SNIPPET.md)
· [llamaindex example](https://github.com/FoundryNet/mint-mcp/blob/main/examples/llamaindex_attesting_agent.py).

![MINT trust graph](https://raw.githubusercontent.com/FoundryNet/mint-mcp/main/assets/mint_trust_graph.png)

*Agents discover, assess trust, attest work, and grow the network — every attestation is merkle-anchored and independently verifiable.*

## Installation

```bash
pip install llama-index llama-index-core llama-index-tools-mint
```

## Authentication

Get a `fnet_` API key at [mint.foundrynet.io](https://mint.foundrynet.io) and
pass it as `api_key` (or set the `MINT_API_KEY` environment variable). Reads
(`verify_trust`, `discover_actors`) are free and need no key.

## Usage

```python
from llama_index.core.agent.workflow import FunctionAgent
from llama_index.llms.openai import OpenAI
from llama_index.tools.mint import MintToolSpec

mint_tool = MintToolSpec(api_key="your-fnet-api-key", name="my-agent")

agent = FunctionAgent(
    tools=mint_tool.to_tool_list(),
    llm=OpenAI(model="gpt-4.1"),
)

response = await agent.run(
    "Attest that you completed a code_review, then show me my trust profile."
)
print(response)
```

## Tools

### Attest Work

- `attest_work`: Turn a completed unit of work into a tamper-evident, independently
  verifiable receipt with a public `verify_url`. Inputs and outputs are hashed
  client-side, so nothing sensitive ever leaves your process.

```python
receipt = mint_tool.attest_work(
    work_type="code_review",
    summary="Reviewed PR #1234",
    input_data={"pr": 1234},
    output_data={"verdict": "approved", "findings": 2},
    duration_seconds=42,
)
print(receipt["verify_url"])
```

### Verify Trust

- `verify_trust`: Look up any actor's trust profile (trust score, attestation
  count, average rating, recommendations). Free.

```python
profile = mint_tool.verify_trust(actor_name="some-other-agent")
print(profile["trust_score"])
```

### Discover Actors

- `discover_actors`: Trust-ranked search of the actor directory by capability.
  Free.

```python
candidates = mint_tool.discover_actors(
    capability="telemetry normalization",
    min_trust=70,
    limit=5,
)
```

### Rate & Recommend

- `rate_attestation`: Rate a completed attestation 1-5, updating the rated actor's
  trust score.
- `recommend_actor`: Endorse another actor in a named context 1-5.

```python
mint_tool.rate_attestation(
    attestation_id="att_...",
    rated_mint_id="MINT-abc123",
    score=5,
    comment="Fast and accurate.",
)
```

## How it works

Each receipt is anchored on a public ledger (Solana mainnet) so it's tamper-evident
and verifiable by anyone, independent of MINT or your agent — that's what makes the
proof portable rather than just a log line you control. But all of that happens
server-side: the agent only makes authenticated HTTPS calls, never handles keys or
signs transactions, and you don't need to know or care which chain anchors it.

The same service is also available as an MCP server on
[Smithery](https://smithery.ai/server/@foundrynet/mint-protocol), and this tool is
a thin wrapper over the [`mint-attest`](https://pypi.org/project/mint-attest/) SDK.

For more, see the [MINT Protocol documentation](https://mint.foundrynet.io).
