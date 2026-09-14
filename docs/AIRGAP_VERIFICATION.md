# Verifying air-gapped operation

RULIX can write an AI audit report **with no internet connection**. This document
does not ask you to take that on faith. It gives you a procedure you can run.

## Why a screen recording proves nothing

A recording of the tool working guarantees nothing. "It ran fine" is not
evidence that nothing left the machine at that moment. Something could have gone
out quietly in the background, or the recording could have been made with the
network switched off just for the take.

So the proof runs the other way round.

> **An attempt to reach outside must fail.**
> And whoever is checking must be able to reproduce that failure themselves.

## How the block works

With `--offline`, RULIX inspects the **destination IP** at the point the
connection is actually established, and refuses anything that is not loopback
(`127.0.0.1`, `::1`).

- It looks at the **IP after DNS resolution**, not the hostname, so renaming the
  host does not get around it
- **Name resolution is blocked too.** Connections to an external DNS server
  (UDP/TCP 53) are refused as well
- **A proxy cannot be used to slip out.** Going through a proxy would change the
  destination to the proxy's address and make the check meaningless, so
  air-gapped mode does not use a proxy at all
- The process-wide HTTP path (`http.DefaultClient`) is blocked as well. Blocking
  only the client our own code uses would leave a door open for a dependency to
  walk through

### What this guarantees, and what it does not

**It guarantees** that during this run, this process opened no TCP connection
outside loopback. If it tried, no report is produced and the command fails.

**It does not guarantee** a complete audit at the operating-system level. A path
that uses the system resolver (the cgo resolver) may not go through this dialer.
That is why we recommend **pointing the endpoint at a loopback IP directly**
rather than at a hostname.

On a genuinely air-gapped network the network itself is cut off. What this
mechanism adds is that in such an environment the tool does not fail silently or
stall for reasons nobody can name — it **fails while saying exactly what was
blocked**.

## The procedure

### Preparation

You need an in-house inference server. With Ollama:

```bash
ollama pull qwen2.5-coder:7b
ollama serve                      # defaults to 127.0.0.1:11434
```

And a scan result in JSON:

```bash
rulix /path/to/src --profile=all -o json --output-file=scan.json
```

### Step 1 — confirm the block is alive

**This step is supposed to fail.** Point it at an external address and try to
generate a report.

```bash
rulix ai-report scan.json --offline \
    --endpoint https://api.openai.com/v1 --model gpt-4o \
    -o /dev/null
```

Expected: exit code `1`, and the reason it was blocked:

```
🔒 Air-gapped mode: every connection that leaves loopback (127.0.0.1) is blocked
80 issues · 10 files → openai:gpt-4o
Error: The model did not answer for a single finding: explain call failed:
  Post "https://api.openai.com/v1/chat/completions":
  dial tcp: lookup api.openai.com on 10.0.0.1:53:
  Air-gapped mode blocked an outbound connection (10.0.0.1:53).
  Only loopback addresses (127.0.0.1, ::1) are allowed.

The endpoint you set is an external address. Point --endpoint at a local
inference server (for example http://127.0.0.1:11434/v1).
```

Note where it stopped: at the DNS lookup. It never got as far as resolving the
name.

> If this step **succeeds**, the block is not working. In that case the success
> of step 2 proves nothing at all.

### Step 2 — confirm a local server does produce the report

Leave `--offline` on and change only the endpoint to your own server.

```bash
rulix ai-report scan.json --offline \
    --endpoint http://127.0.0.1:11434/v1 --model qwen2.5-coder:7b \
    --project "Example System" -o report.html
```

Expected: exit code `0`, and a report:

```
🔒 Air-gapped mode: every connection that leaves loopback (127.0.0.1) is blocked
400 issues · 413 files → local:qwen2.5-coder:7b
✅ report.html (139s · quality 27 · overall grade E)
```

The report records `local:qwen2.5-coder:7b` as what generated it. What produced
the report is written into the report itself.

### Running both at once

There is a target that runs the two steps in order and fails if step 1 is not
blocked.

```bash
make airgap-verify AIRGAP_SCAN=scan.json
```

### To be surer still — cut the network and run the same procedure

The most convincing check is the one run in the verifier's own environment. Put
the machine on a genuinely isolated network, or disconnect it, and run step 2
exactly as above. The result must be the same.

## Verifying at the source level

The block lives in `internal/netguard`, with tests that pin its behaviour.

```bash
go test ./internal/netguard/ -v
```

| Test | What it guarantees |
|------|--------------------|
| `TestLoopbackIsAllowed` | an in-house inference server is still reachable |
| `TestExternalIsBlocked` | external IPv4 and IPv6 addresses cannot be reached |
| `TestBlockIsImmediate` | the refusal happens before the connection, not as a timeout |
| `TestInstallBlocksDefaultClient` | a dependency's default path is blocked too |
| `TestProxyCannotBypass` | proxy settings cannot be used to get around it |

The addresses used in the tests are `203.0.113.0/24` (TEST-NET-3) and
`2001:db8::/32`, both reserved for documentation. Even if the block failed, no
packet would reach anyone.

## If you are using a cloud API

`--provider claude` and `--provider gemini` are external APIs, so they cannot be
combined with `--offline`; trying to do so is refused before the run starts. An
air-gapped deployment uses only an OpenAI-compatible endpoint — an in-house
Ollama or vLLM server.

---

[한국어](AIRGAP_VERIFICATION.ko.md)
