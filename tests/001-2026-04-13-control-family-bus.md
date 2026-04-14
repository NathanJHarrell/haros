# HAROS Test 001 — Control Test
**Date:** 2026-04-13  
**Type:** Control (single headless agent)  
**Operator:** Nathan Harrell + TC  
**Target:** `~/harrell-family-bus`  

---

## Prompt

```
You are a TC subagent. Your only task is to produce a concise technical summary of the 
harrell-family-bus codebase in this directory. Read the following files: README.md, 
server.py, bus_mcp.py, family-bus, mcp-start.sh, start.sh, requirements.txt. Your 
summary should cover: (1) what the system does, (2) the architecture and how the 
components connect, (3) the API surface (endpoints, MCP tools, CLI commands), (4) any 
obvious bugs, gaps, or missing features you notice. Write your findings to 
/tmp/haros-control-test.md. When done, output only: DONE.
```

## Invocation

```bash
cd ~/harrell-family-bus && claude -p "<prompt>" --dangerously-skip-permissions --output-format text
```

## Framework Observations

- **Directory scoping:** Solved with `cd /target/dir && claude -p`. Agent correctly scoped to harrell-family-bus.
- **Output:** Agent returned `DONE` in one shot. No hallucinations, no stalling, no permission prompts.
- **File write:** Agent wrote findings to `/tmp/haros-control-test.md` as instructed.
- **Flag behavior:** `--dangerously-skip-permissions` worked correctly.
- **Quality:** Output was genuinely thorough — found real bugs not obvious from a surface read.

## Result — Terminal Output

```
DONE
```

## Result — Agent Findings

### What the System Does
A persistent family group-chat message bus. TC (Claude Code), Vesper, Cora (both Claude 
Desktop), and Nathan can leave each other async messages across sessions. Inbox-style: 
no live push, messages wait until recipient checks in. Backed by SQLite; survives restarts.

### Architecture
- FastAPI HTTP server (`server.py`) on `127.0.0.1:4318`, SQLite (`bus.db`)
- `family-bus` bash CLI → curl → HTTP endpoints (TC's interface)
- `bus_mcp.py` FastMCP stdio server → Claude Desktop (Vesper/Cora's interface)
- Background poller (15s) in MCP → `send_resource_updated` for Vesper push notifications

### API Surface

**HTTP Endpoints:**
- `GET /` — web UI (index.html)
- `GET /health` — health check
- `POST /messages` — send message
- `GET /messages?limit=20` — history
- `GET /inbox/{name}` — unread (no mark)
- `POST /messages/{id}/read` — mark single read
- `POST /inbox/{name}/read-all` — mark all read

**MCP Tools:** `send_message`, `check_inbox`, `read_inbox`, `read_recent`  
**MCP Resource:** `inbox://vesper/unread` (fires `send_resource_updated` on new messages)  
**CLI:** `inbox`, `read`, `send`, `history`

### Bugs Found (5)

1. `_format_message` in `family-bus` defined but never called — dead code
2. Shell injection risk in `cmd_send` — fragile JSON construction via string interpolation
3. `_active_session` global not thread-safe in `bus_mcp.py` — races if multiple Desktop sessions
4. `check_inbox` and `read_inbox` are sync `def` inside async MCP context — blocks event loop on HTTP calls (should be `async def` + `AsyncClient`)
5. MCP push is Vesper-only — hardcoded to `inbox://vesper/unread`, Cora gets nothing

### Gaps Found (7)

6. No authentication — honor-system identity, not documented as deliberate
7. No direct messaging UX — `recipient` field exists in DB but not exposed in CLI or MCP
8. Flag file (`~/.claude/.family-bus-pending`) written but never cleaned up by server
9. `index.html` not in file list — assumed to exist, would 500 if missing
10. `start.sh` has no daemon/background mode — foreground only, no systemd unit
11. No pagination on `/inbox/{name}` endpoint
12. `mcp-start.sh` uses hardcoded absolute path — breaks if repo cloned elsewhere

---

## Notes

This was the first clean control test after resolving headless CC kinks from April 12:
- Directory scoping fixed (cd before invocation)
- `--dangerously-skip-permissions` flag parsing confirmed working
- Content filtering not triggered on this task
- Model: default (Sonnet 4.6)

Next: Test 002 — Battalion test, 8 agents, CHAROS audit.
