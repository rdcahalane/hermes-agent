# Hermes Discord Agent Journey

This document is the durable operator brief for Ryan's Hermes Discord setup. It is written for both humans and agents who need to discover the product, understand the value, try it safely, route approvals correctly, and verify that it still works.

## Target Customer

Ryan, using Discord as the primary client for:

- local models on Beast
- cloud agents when local capacity is not enough
- tools, files, images, and autonomous execution
- cron delivery and async follow-up in one place

## Five-Second Value Statement

Use Discord as the control surface for Hermes so Ryan can message one bot to run local or cloud-backed work, inspect files, use tools, receive cron output, and approve risky actions without opening the terminal.

## Discovery Path

1. Start at [README.md](../README.md) for the product overview and install flow.
2. Run `/Users/ryancahalane/.hermes/scripts/discord-agent-doctor.py` for the fastest end-to-end operator snapshot.
3. Read `/Users/ryancahalane/.hermes/docs/discord-agent-runtime-playbook.md` if setup or verification behavior is unclear.
4. Read the general messaging operator guide at `website/docs/user-guide/messaging/index.md` if setup behavior is unclear.
5. Confirm the Discord platform contract in `plugins/platforms/discord/plugin.yaml`.
6. Use `hermes gateway setup` to enable Discord.
7. Configure the minimum Discord runtime inputs:
   - `DISCORD_BOT_TOKEN`
   - `DISCORD_ALLOWED_USERS`
   - `DISCORD_HOME_CHANNEL`
8. Start the gateway with `hermes gateway start`.
9. In Discord, DM the bot or use the configured home channel and send a plain request or slash command.

## Understand The Offer

Hermes on Discord is useful when Ryan wants:

- a messaging-first interface instead of a terminal-first interface
- one thread for chat plus cron delivery plus approval requests
- direct access to local repo inspection, tools, and files
- an agent that can continue work asynchronously and report back into the same channel

Key Discord-specific behaviors already supported in this repo:

- allowlist-based access control
- home-channel delivery for cron and notifications
- slash commands
- thread support
- attachment ingestion for images, audio, and documents
- approval buttons for risky actions

## Trial / Demo Path

Lowest-risk first-use flow:

1. Run `hermes gateway setup` and choose Discord.
2. Add Ryan to `DISCORD_ALLOWED_USERS`.
3. Run the local verification wrapper before touching Discord:
   - `./scripts/verify-discord-agent-journey.sh`
4. Start the runtime with `hermes gateway start` if it is not already running.
5. In Discord, set the delivery destination with `/sethome` or `/set-home` if `DISCORD_HOME_CHANNEL` was not set during setup.
6. Send one constrained task that proves end-to-end usefulness without changing production data, for example:
   - "Inspect the content-performance cron job config and report the exact current values."
   - "What is on backlog? Do not rely on transcript memory."
   - "Summarize the latest failed jobs and likely causes."

Good demo outcomes:

- Hermes answers from files and durable state, not generic promises
- Hermes does not ask Ryan to run commands it can run itself
- Hermes stays inside approval boundaries

## Buy / Approval Path

For Ryan, "buy" is mostly an approval and configuration step, not a checkout flow.

Required approval:

- create or reuse a Discord bot token
- choose the allowed Discord users
- choose the home channel for cron and notifications

Optional approval:

- enable paid/cloud providers if local models are insufficient
- grant any additional credentials beyond the local-model baseline

Default recommendation:

- start with the local-model path already configured in `config.yaml`
- only escalate to paid providers when a concrete task is blocked by local capability

## Agent Entrypoint

Primary operator entrypoints:

- `./scripts/discord-agent-doctor.py`
- `hermes gateway setup`
- `hermes gateway start`
- `/Users/ryancahalane/.hermes/scripts/verify-discord-agent-journey.sh`
- `/Users/ryancahalane/.hermes/scripts/current-operating-state.py`

Primary verification harness:

- `/Users/ryancahalane/.hermes/scripts/verify-discord-agent-journey.sh`
- `/Users/ryancahalane/.hermes/scripts/discord-agent-eval.py`

Agent operating rule for fresh sessions:

- do not trust Discord transcript memory for backlog or status
- reconstruct state with `current-operating-state.py` first

## Verification Command

Run this from `/Users/ryancahalane/.hermes`:

```bash
./scripts/verify-discord-agent-journey.sh
```

This wrapper checks:

- doctor output for customer, approval, entrypoint, and quick-status context
- gateway service presence and staleness via `hermes gateway status`
- behavior on the two highest-signal journey cases

If you need the raw behavioral harness only, run:

```bash
python3 scripts/discord-agent-eval.py --case backlog_question --case inspect_no_pushback --json
```

This verifies two important journey properties:

- Hermes can discover durable operating state instead of relying on chat memory
- Hermes inspects local config directly rather than pushing work back onto Ryan

## Current Verified State

Observed on June 24, 2026 from `/Users/ryancahalane/.hermes`:

- `./scripts/discord-agent-doctor.py` succeeds and reports the Discord plugin and playbook as present.
- `hermes gateway status` reports the launchd gateway service as loaded with `LastExitStatus = 0`, but still warns that the service definition is stale relative to the current Hermes install.
- `./scripts/verify-discord-agent-journey.sh` currently fails before the Discord journey is exercised because both behavioral cases exit with the same local startup error:
  - `backlog_question`
  - `inspect_no_pushback`
- The blocking error is:
  - `Auxiliary compression model qwen2.5-coder-7b-instruct-q4_k_m.gguf has a context window of 8,192 tokens, which is below the minimum 64,000 required by Hermes Agent.`

Interpretation:

- discovery and local operator docs are in place
- the managed gateway is running, but trust is still reduced by stale service metadata
- the core customer/agent journey blocker is now a verified local agent-startup configuration problem in the behavioral eval path, not a missing-docs problem

## Current Unresolved Risks

- Discord cron delivery has recent DNS/connectivity failures in `cron/jobs.json` with errors like `Cannot connect to host discord.com:443`.
- Recent delivery failures in `cron/jobs.json` also include Discord 429 / Cloudflare 1015 rate-limiting responses on June 22, 2026, so live cron delivery trust is still reduced even when the gateway process is up.
- Fresh Discord sessions lose conversational backlog context after reset, so agents must reconstruct state from files and scripts.
- Logs show unknown slash command handling for `/skill`; command discoverability in Discord may still be inconsistent with user expectations.
- The approval and setup path is split across README, code, and runtime config; this document reduces that gap, but the product surface is still broad.
- Verification is currently behavioral and local; it does not prove live Discord network reachability unless the gateway is running and Discord is reachable.
- `hermes gateway status` now reports the launchd service as loaded with `LastExitStatus = 0`, but still warns that the service definition is stale relative to the current Hermes install.
- The two highest-signal behavioral eval cases currently fail before model execution because the configured auxiliary compression model does not meet Hermes Agent's minimum context requirement, so the primary wrapper is not yet a passing proof of usable Discord response behavior.

## What Future Agents Should Remember

- The customer is not evaluating Discord as a generic chat bot. He is evaluating Discord as the main client for local models, tools, files, cron output, and approvals.
- The right first proof is an end-to-end task that inspects durable state and answers concretely.
- The right approval boundary is strict: do not deploy, spend money, contact customers, or change production data without Ryan approval.
- The right recovery move for backlog or status confusion is `python3 /Users/ryancahalane/.hermes/scripts/current-operating-state.py`.
- The fastest repeatable verification path is `./scripts/verify-discord-agent-journey.sh`, not the raw eval harness by itself.
- If the wrapper fails immediately, inspect Hermes compression settings before blaming Discord transport, prompt quality, or eval timeouts.
