# AgentPearl

AgentPearl is a small tool for exporting agent core files so they can be read, understood, and migrated by AI across different agent frameworks.

Its job is simple:

1. detect which supported agent framework a repository uses
2. extract only the files that define the agent itself
3. package those files into a clean zip
4. provide documentation that explains what each part of the agent core is responsible for

The intended workflow is:

- use AgentPearl to export a pure agent-core pack
- give that pack plus this repository's documentation to an AI system
- let the AI reconstruct or port the agent core into another framework

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

Once that core is isolated, an AI can read it and transplant it into another framework.

## Core Idea

An exported pack should be:

- source-only
- framework-agnostic at the archive boundary
- small enough to inspect
- rich enough to reconstruct how the agent is designed

AgentPearl is therefore an export-and-portability tool.

It does not hardcode a universal importer because there may be countless target frameworks.

Instead, it produces the right raw material for an AI to do the migration correctly.

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

That document is the key portability layer for AI systems: it explains what each piece of the exported core means, so the AI can map it into a different framework.

## Design Principles

### 1. Source-only archives

The zip is the raw source material.

It is not a target-specific bundle.

It is the minimal portable source package an AI should read before rebuilding the same agent core elsewhere.

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

`AGENT_CORE.md` explains what the exported layers mean, so an AI can reason about portability instead of copying files blindly.

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

- bake target-framework assumptions into the exported zip
- ship a single hardcoded importer for every possible framework
- bundle unrelated source files for convenience

Instead, it is designed for this workflow:

1. export the source agent core
2. give the exported pack and docs to an AI
3. let the AI port the agent core into the target framework

This design is intentional. The number of possible target frameworks is unbounded, so the stable boundary is the exported source pack plus the explanation of what each layer means.

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

That final artifact is meant to be consumed by an AI or human who wants to study or migrate the agent core.

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

- a source pack builder for agent-core migration work
- a framework detector with explicit extraction rules
- a portability aid for cross-framework agent reconstruction

Your expected workflow is:

1. read the exported zip
2. read `MANIFEST.txt`
3. read `AGENT_CORE.md`
4. identify identity, instruction, context, runtime, capability, multi-agent, and composition layers
5. rebuild those layers in the target framework using native target concepts

The most important constraint is:

**the final zip must remain pure agent-core source material**

The portability comes from the combination of:

- the exported source files
- the detected framework label
- the explanation of what each agent-core layer does

If you modify this project, preserve that boundary.
