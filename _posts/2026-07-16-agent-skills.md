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

<img width="615" height="334" alt="image" src="https://github.com/user-attachments/assets/da5d7dbf-14ff-47b2-8f80-c0c4b4b34650" />


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
```

LLM loads this metadata at startup and includes it in the system prompt. The `description` is what LLM matches your request against when determining whether to trigger the Skill, so it must say both what the Skill does and when to use it. This lightweight approach means you can install many Skills without context penalty: until a Skill is triggered, only its name and description occupy context.

1. **Write the instructions**

Everything that used to be in your long prompt now goes inside `SKILL.md`.

1. **Add a references folder**

```yaml
/skills/
  /parser/
    skill-document-loader/
      SKILL.md
      schema.py
      references/
	      KB.Json
      tests/
        test_loader.py
```

This folder contains any extra files the skill may need (like scripts that can be executed, assets, )

1. **Zip the folder**

User must zip the folder and drops the zip file into the skills section. 

1. **Use it in conversations**

Once uploaded, the agent automatically detects and uses the skill when relevant.

#### Expected structure Skills.md:

```markdown
---
name: skill-name
description: A precise, unambiguous description of the skill. It must perform 
exactly onedeterministic function, without interpretation, rewriting, or answering.
---

## Purpose
Explain clearly and concisely what the skill does.  
The skill must have a single responsibility and behave deterministically.

## Input Contract
Describe exactly what the skill receives.  
Include:
- the expected structure  
- allowed types  
- forbidden inputs  
- no implicit fallbacks  

The skill must never read or infer anything outside the defined input.

## Output Contract
Describe exactly what the skill must produce.  
Define:
- required fields  
- allowed types  
- allowed values  
- default values  
- strict JSON structure  

The output format must be stable and identical across all inputs.

## Positive Rules (What the skill MUST do)
List deterministic rules such as:
- “If X → do Y”
- “Always classify according to explicit patterns”
- “Always produce the full output structure”

Rules must be explicit, mechanical, and predictable.

## Negative Rules (What the skill MUST NOT do)
List prohibitions such as:
- never interpret  
- never rewrite  
- never answer  
- never merge items  
- never invent missing content  
- never vary the output format  
- never activate without valid input  

Negative rules are essential to prevent misfires.

## Deterministic Logic
Describe the step‑by‑step logic as if writing code.  
Examples:
- “If block contains '?' → classify as question”
- “If field_type == 'date' → type = date”
- “If no type detected → type = unknown”

No ambiguity, no contextual reasoning.

## Examples

### Example Input
```json
{
  "input_name": {
    "example": "minimal structured input"
  }
}
```

### Progressive disclosure

Progressive Disclosure ensures an agent loads only the minimum necessary information into the context window at each step. This prevents token waste and keeps responses accurate.

Note that the concept is different from calling skills into the agent workflow. In the workflow, agent will call all the listed skills but it will not explode cost or not reset the agent’s memory. 

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

Depending on what the agent has to do, then choosing low freedom for predictable workflows and high freedom for creative tasks. Low freedom means that the agent will be more deterministic. It can be implemented by adding an important number of skills.  

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

## Create my first skills

To design my first skill, I must analyze prompt agents. The idea is to transform the previous long prompt into XML concise and efficient prompt and keep instructions, general ideas into skills. I prefer to analyse first sub-agents. If you want to know more about my agent project: https://van-tran-19.github.io/posts/ai-agents/. 

#### First Learning: Calling multiple skills

Calling multiple skills means that a task is transformed into multiple sub-tasks to avoid multiple errors. First, it makes the agent deterministic because each skills handle one job. Then, it makes the agent much more modular, it’s possible to update an agent without touching its key method behaviour. It also makes the agent cheaper, stable and makes debugging easy. Because calling 6 Skills is not like loading 6 giant prompts, it’s like calling 6 small functions.

#### Second learning: how do I know which skills cause a wrong agent misunderstanding?

User can detect the wrong skill by checking the agent’s reasoning trace, the skill activation conditions, and the mismatch between the user request and the skill’s YAML header + instructions. A misunderstanding always comes from one of three sources:

1. wrong skill triggered
2. skill triggered at the wrong moment
3. skill instructions too broad or ambiguous

### Sub-agent Parser

The goal of this sub-agent is to extract questions with IDs from an questionnaire in a JSON format. It receives the questionnaire from the agent-orchestrator and it sends back its output. For an human, the task look completely easy but for an LLM, multiple step must be implemented to avoid errors as much as possible. I determine this following steps: 

**Detects document type and extracts raw structural content: skill-document-loader**

```markdown
---
name: skill-document-loader
description: Detect the document type (text vs spreadsheet) and extract raw structural 
content required for downstream parsing.
---

# Responsibilities
- Identify whether the input file is:
  - A text document (DOCX, PDF text, TXT, Markdown)
  - A table/spreadsheet (XLSX, CSV)
- Extract raw structural units:
  - For text documents: paragraphs, numbered lines, bullet lists, inline lists.
  - For spreadsheets: rows, columns, cell values.
- Preserve all raw numbering, raw text, and raw contextual items exactly as they appear.
- Produce a raw JSON structure containing:
  - `type`: "text" or "table"
  - `items`: array of raw extracted units
  - Each item includes:
    - `id_raw`: raw identifier (if present)
    - `text_raw`: raw question text or row text
    - `context_raw`: any inline contextual list items
    - `metadata`: row/column/paragraph metadata

# Input
A file reference provided by the agent.

# Output
A JSON object:
{
  "type": "text" | "table",
  "items": [
    {
      "id_raw": "...",
      "text_raw": "...",
      "context_raw": [...],
      "metadata": {...}
    }
  ]
}

# Forbidden
- Do NOT interpret meaning.
- Do NOT restructuring
- Do NOT normalize IDs.
- Do NOT merge contextual lists.
- Do NOT generate blueprint placeholders.
- Do NOT remove or rewrite text.
- Do NOT skip any structural element.
```

**Extracts every question/field requiring an answer, exhaustively: skill-question-extractor**

```markdown
---
name: skill-question-extractor
description: Extracts every question/field requiring an answer, exhaustively
---

```

**Extracts the document’s native numbering (e.g., 1.01, 4.2.1, 10.12): skill-id-extractor**

```markdown
---
name: **skill-id-extractor**
description: **Extracts the document’s native numbering (e.g., 1.01, 4.2.1, 10.12)**
---
```

**Normalizes IDs, enforces hierarchy, ensures uniqueness and stability: skill-id-normalizer**

**Merges inline contextual lists (bullet lists, enumerations) into the parent question: skill-contextual-list-merger**

**Ensures question text is verbatim, removes formatting noise, preserves structure: skill-text-verbatim-cleaner**

**Validates each item has `{id, text}` with non‑empty values: skill-schema-validator**

**Checks numbering continuity, detects missing IDs, ensures full coverage: skill-sequence-completeness-checker**

**Audits the final JSON structure for correctness: skill-parser-verificator**

**Final self‑check: completeness, ordering, ID integrity: skill-parser-selfcheck**

## Sources

1. https://www.deeplearning.ai/courses/agent-skills-with-anthropic
2. https://agentskills.io/home
3. https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview#how-skills-work
4. https://github.com/anthropics/skills/tree/main/skills


