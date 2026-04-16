# HAROS Test: clipd — Network Clipboard Daemon

**Date:** April 16, 2026  
**Configuration:** Config 1 — Lean Squad (3 sessions)  
**Orchestrator:** TC (primary session)  
**Purpose:** First real HAROS battalion orchestration test. Build a network clipboard daemon while validating the HAROS-FOV viewer and haros CLI.

---

## Test Objectives

1. **Orchestration:** Can TC coordinate two independent leads via tmux + haros CLI?
2. **FOV Viewer:** Does the web UI display all sessions correctly? Can Dad interact?
3. **haros CLI:** Do list/status/tree/log commands work during a live build?
4. **Deliverable:** Working network clipboard daemon (auto-sync between Nest and Jarvis)

## Build Configuration

```
haros-clipd-orch     → TC (this session, orchestrating)
haros-clipd-lead1    → Nest daemon (Wayland clipboard + TCP server)
haros-clipd-lead2    → Jarvis daemon (Windows clipboard + TCP client)
```

## Timeline

| Time | Event | Notes |
|------|-------|-------|
| | Battalion spawned | |
| | Lead1 briefed | |
| | Lead2 briefed | |
| | Lead1 reports | |
| | Lead2 reports | |
| | Integration test | |
| | Build complete | |

## Observations

*(filled in during the build)*

### FOV Viewer
- [ ] Landing page shows clipd build
- [ ] Build page shows all 3 sessions tiled
- [ ] Terminals render with readable fonts
- [ ] Full-view toggle works
- [ ] Interactive typing works through browser

### haros CLI
- [ ] `haros list` shows clipd
- [ ] `haros status clipd` shows all sessions
- [ ] `haros tree clipd` shows agent hierarchy
- [ ] `haros log` captures session output

### Orchestration
- [ ] Leads received briefs successfully
- [ ] Leads worked independently without collision
- [ ] Orchestrator could monitor progress via FOV
- [ ] Post-work review completed
- [ ] Deliverable is functional

## Result

*(filled in after build completes)*

**Status:** IN PROGRESS  
**Duration:**  
**Token estimate:**  
**Issues encountered:**  
**Lessons learned:**  
