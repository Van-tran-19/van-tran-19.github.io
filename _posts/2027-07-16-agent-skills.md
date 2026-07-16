---
title: "Stop Repeating Prompts: Build Real Agent Workflows with Skills"
date: 2026-07-16 11:50:00 +0200
categories: [Tools]
tags: [Skills, Automation, Claude, MCP, Tools, AI, Agents]
toc: true
---
# Agent skills

My interest in skills emerged when I noticed my agent instructions becoming repetitive. I often typed the same prompt for my sub‑agents, again and again. Naturally, I started searching for a way to package these repeated instructions which led me to learn how Skills work for agents.

## What are Skills?

Skills are folders of instructions that extend an agent’s capabilities with:

- specialized knowledge
- repeated workflows
- custom logic
- structured tasks

Instead of rewriting the same long prompt every time, a skill becomes a reusable module that the agent can load automatically.

**Skills Architecture** https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview#how-skills-work

<img width="615" height="334" alt="image" src="https://github.com/user-attachments/assets/41e0e3a9-6fcc-4a0a-ab86-144378f6d368" />

### Why skills are useful?

Skills package all instructions into a reusable module. They spare the user from writing long prompts, repeating manual steps, and overloading the context window. Because skills follow an open standard, they work across major LLM providers. When a CSV file is dropped into the conversation, the LLM automatically reads the instructions defined inside the skill, applies them to the CSV, loads additional reference files only when needed, and executes the entire workflow all without requiring long prompts from the user. 

### How to create a skill?

The agent workflow is turned into a Skills.md file. 

1. **Add a YAML header**

Into this file, a YAML header is added with metadata including:

- name
- description

```yaml
---
name: parsing-questionnaire
description: Reads the raw Word/Markdown file, extracts the list of questions into a 
JSON array.
---

# Your Skill Name

## Instructions
[Clear, step-by-step guidance for Claude to follow]

## Examples
[Concrete examples of using this Skill]
```

LLM loads this metadata at startup and includes it in the system prompt. The `description` is what LLM matches your request against when determining whether to trigger the Skill, so it must say both what the Skill does and when to use it. This lightweight approach means you can install many Skills without context penalty: until a Skill is triggered, only its name and description occupy context.

1. **Write the instructions**

Everything that used to be in your long prompt now goes inside `SKILL.md`.

1. **Add a references folder**

```yaml
agent-fill-form-expert/
├── SKILL.md
└── references/
    └── KB.json etc...
```

This folder contains any extra files the skill may need (like scripts that can be executed, assets, )

1. **Zip the folder**

User must zip the folder and drops the zip file into the skills section. 

1. **Use it in conversations**

Once uploaded, the agent automatically detects and uses the skill when relevant.

### Progressive disclosure

Progressive Disclosure ensures an agent loads only the minimum necessary information into the context window at each step. This prevents token waste and keeps responses accurate.

**Why it exists?**

LLMs have a limited context window. Too much data leads to:

- token consumption
- context pollution
- degraded responses
- higher cost
- hallucinations

**How it works?**

When a skill is available, the agent initially loads **just** `name + description`

This allows the model to know if the skill exists and when it should be triggered. When the skill is triggered, SKILL.md is loaded. Then, additional files are loaded only if needed. Any other files, like scripts are executed outside the context window. This avoids polluting the context with code. 

### How skills fit into the agent ecosystem?

Modern agent systems are built from four components: 

- Skills
- Tools: run code, access files, call APIs
- MCP: connects agents to external systems and data
- Sub-agents: isolated, parallel workers with their own context

#### Skills vs MCPs

MCP brings external data and tools into the agent environment. Whenever the agent needs information the model doesn’t know, MCP provides it. Skills tell the agent how to use that external data. They define the procedural workflow:

- how to analyze
- how to compute
- how to structure output
- how to apply domain rules

#### Skills vs Tools

Tools are low level capabilities. They always live in the context window. Skills are higher level workflows and their are loaded when needed. 

### List of pre-built skills

https://github.com/anthropics/skills/tree/main/skills. For my agent, xlsx skills could be interesting because it allows agent to create and manage excel. 

### Requirements for skill designs

#### Naming

Name must be lowercase letters, numbers, hyphens, preference is given for verb‑ing format and it has to be concise but descriptive. 

#### Description

Description must explain what the skill does, explain when it should be used and include keywords that help the agent trigger it. 

#### Structure

Structure must keep `SKILL.md` under ~500 lines, break complex workflows into multiple skills, use step‑by‑step instructions and use forward slashes for paths. 

#### Freedom vs Determinism

Depending on what the agent has to do, then choosing low freedom for predictable workflows and high freedom for creative tasks. 

#### Optional metadata

It can include license, compatibility or custom key value pairs. 

#### How to evaluate skills?

To evaluate skills with Claude, run a sub-agent which contains the anthropic skill creator and evaluates the skill. The skill creator checks front-matter quality, workflow clarity, duplication, conciseness, structure, references, assets and scripts. 

**Unit-testing skills**

It means checking that a skill behaves correctly, step‑by‑step, just like we test software.
Because skills are reusable workflows, we want to make sure they:

- run the right steps
- load the right files
- use the right scripts
- produce the right output
- behave consistently across models

### Skills with an LLM API

An LLM API is a programming interface that lets the user to send text to an LLM and gets a response back. With an API, user must manually provide a sandboxed container which contains limited RAM, CPU and disk, no internet access, preinstalled libraries, a isolated environment, a file system and code execution. 

## Sources

1. https://www.deeplearning.ai/courses/agent-skills-with-anthropic
2. https://agentskills.io/home
3. https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview#how-skills-work
4. https://github.com/anthropics/skills/tree/main/skills
