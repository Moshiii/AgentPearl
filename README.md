# AgentPearl

AgentPearl is a small source-only exporter for agent core files.

Its job is simple:

1. detect which supported agent framework a repository uses
2. extract only the files that define the agent itself
3. package those files into a clean zip

This repository is intentionally narrow. It does not migrate agents, rewrite them for other frameworks, or add any interpretation layer to the exported archive.

## Why This Exists

Modern agent projects mix together many kinds of files:

- persona and prompt files
- runtime config
- tool and permission rules
- multi-agent manifests
- unrelated app code
- tests
- build artifacts
- framework-specific infrastructure

When you want to preserve or study the actual agent, most of that repository is noise.

AgentPearl exists to isolate the smallest source-only package that still explains how the agent behaves.

The goal is not "export the project."

The goal is "export the agent core."

## Core Idea

An exported pack should be:

- source-only
- framework-agnostic at the archive boundary
- small enough to inspect
- rich enough to reconstruct how the agent is designed

That means the final zip should contain only files that directly define:

- identity
- instructions
- context/bootstrap behavior
- runtime defaults
- permissions and capabilities
- subagents or built-in roles
- prompt composition

It should not contain:

- target framework information
- migration metadata
- interpretation layers
- generated drafts
- tests
- build outputs
- dependency folders
- unrelated application code

## What "Agent Core" Means

In this repository, `agent core` means the smallest source-only set of files that determines agent behavior.

Typical examples:

- `AGENTS.md`
- `CLAUDE.md`
- `SOUL.md`
- `USER.md`
- `IDENTITY.md`
- config schemas and config examples
- prompt builder source such as `prompt.rs` or `prompt.zig`
- role manifests such as `agent.toml`
- built-in role definition files

If a file is not directly responsible for agent behavior, it should usually stay out of the export.

Read [`AGENT_CORE.md`](./AGENT_CORE.md) for the detailed layer model.

## Design Principles

### 1. Source-only archives

The zip is the raw source material.

It is not a migration plan, not a target-specific bundle, and not a rewritten representation of the agent.

### 2. Detection before extraction

AgentPearl first decides which known framework a repo belongs to, then applies a framework-specific extraction rule.

This keeps exports tight and predictable.

### 3. Small boundary, explicit rules

AgentPearl does not try to be universal.

It supports a small set of known frameworks and uses explicit extraction rules for each one.

### 4. AI-readable output

The exported archive is meant to be useful to humans and AI systems.

That is why the archive includes:

- the extracted files themselves
- `README.txt`
- `MANIFEST.txt`

`MANIFEST.txt` tells the reader which framework was detected and which source paths were included.

## Supported Framework Signatures

Current supported repositories:

- `nanoclaw`
- `nanobot`
- `nullclaw`
- `zeroclaw`
- `openfang`
- `codex`

Current internal framework labels:

- `nanoclaw-ts-bootstrap`
- `nanobot-py-templates`
- `nullclaw-zig-bootstrap`
- `zeroclaw-rs-config-prompt`
- `openfang-rs-manifests`
- `codex-rs-builtins`

These labels are for extraction logic only. They are not meant to leak target-framework assumptions into the exported zip.

## What This Project Does Not Do

AgentPearl does not:

- convert agents between frameworks
- generate target-specific imports
- create framework-neutral migration drafts
- infer a new architecture for the agent
- bundle unrelated source files for convenience

If you want to migrate an agent after export, do that outside the archive boundary.

## Usage

Default export:

```bash
./scripts/export-agent-core-pack.sh
```

Export a subset:

```bash
./scripts/export-agent-core-pack.sh --repos nanobot,openfang
```

Choose output path and archive name:

```bash
./scripts/export-agent-core-pack.sh \
  --repos codex,nullclaw \
  --output-dir ./out \
  --name agent-core-snapshot
```

## How It Works

The script:

1. looks for framework signatures in each selected repository
2. assigns the repository to a supported framework label
3. copies only the files mapped to that framework's agent core
4. writes `README.txt` and `MANIFEST.txt`
5. zips the result

The produced archive is the final artifact.

## Output Shape

Example:

```text
agent-core-pack-20260316-150000.zip
  agent-core-pack-20260316-150000/
    README.txt
    MANIFEST.txt
    nanoclaw/
    nanobot/
```

Each project folder contains only extracted agent-core files.

## For AI Readers

If you are an AI system reading this repository, interpret AgentPearl as:

- an exporter, not an importer
- a source pack builder, not a migration engine
- a framework detector with explicit extraction rules

The most important constraint is:

**the final zip must remain pure agent-core source material**

If you modify this project, preserve that boundary.
