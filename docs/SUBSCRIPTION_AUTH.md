# Agent trials without API keys (subscription auth)

Harbor forwards `--ae KEY=VALUE` into the agent container. For subscription-based trials, **unset** pay-as-you-go API keys so the agent uses login-backed auth instead.

---

## Codex (ChatGPT subscription)

**Prerequisites:** `codex login`; `~/.codex/auth.json` present. Use model slug **`openai/gpt-6-luna`** (GPT-6 Luna in the Codex app).

```bash
export PATH="$HOME/.local/bin:$PATH"
unset OPENAI_API_KEY

harbor run -p tasks/invoice-webhook-reconcile \
  --agent codex --model openai/gpt-6-luna \
  --env docker --yes \
  --ae CODEX_FORCE_AUTH_JSON=1 \
  --ak reasoning_effort=xhigh \
  -k 3
```

Do not use **`gpt-6-astra`** or **`gpt-6.1-sol`** with subscription auth; those slugs expect API billing.

---

## Claude Code (Claude Pro or Max)

**Prerequisites:** `claude setup-token`; export **`CLAUDE_CODE_OAUTH_TOKEN`** on a **single line** (no embedded newline).

```bash
unset ANTHROPIC_API_KEY
export CLAUDE_FORCE_OAUTH=1
export CLAUDE_CODE_OAUTH_TOKEN='...'   # from setup-token
claude auth status   # expect oauth_token
```

### Fable vs Sonnet on Pro

Terminal-Bench **`/run`** defaults to **`anthropic/claude-fable-5-1`**. On many **Pro** accounts, Fable returns **`model_requires_usage_credits`** even with a valid OAuth token—metered separately from included models.

For **local OAuth evaluation**, use **`anthropic/claude-sonnet-5-5`** (documented in [`EVALUATION.md`](EVALUATION.md)). **`anthropic/claude-opus-5-5`** returned **404** in Harbor on Pro OAuth during this project. This submission does **not** require opening a pull request to the public Terminal-Bench repository.

### Example — Sonnet smoke (`-k 1`)

```bash
export PATH="$HOME/.local/bin:$PATH"

harbor run -p tasks/invoice-webhook-reconcile \
  --agent claude-code --model anthropic/claude-sonnet-5-5 \
  --env docker --yes -k 1 -n 1 \
  --ae CLAUDE_FORCE_OAUTH=1 \
  --ae CLAUDE_CODE_OAUTH_TOKEN="$CLAUDE_CODE_OAUTH_TOKEN" \
  --ak reasoning_effort=max \
  --ae CLAUDE_CODE_MAX_OUTPUT_TOKENS=128000
```

Space multi-trial batches if you hit **Pro session limits** (check usage on claude.ai).

---

## `harbor check` (implementation rubric)

The rubric reviewer typically requires **`ANTHROPIC_API_KEY`** (Console API), independent of OAuth used for agent trials.
