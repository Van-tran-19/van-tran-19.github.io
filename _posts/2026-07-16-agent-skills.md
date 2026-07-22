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

**Is Agent needed skills to use tools?** 

In a multi-agent architecture, a tool is given associated with a skill. The skill is used to help how using this tool, when, for which request types.  

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

#### Third learning: Skills must be atomic and concise

A Skill is not a giant prompt. It is a compact instruction module. 

Each skill must do one thing at a time. If a Skill becomes too long, it means it is doing too many tasks. Skills must contain only operational instructions. This keeps Skills short and predictable.

### Agent Orchestrator

This agent goal is to route input/output to the correct sub-agent. 

**Skill route-input**

```markdown
---
name: route-input 
description: Classifies the user input as a questionnaire file or a direct question, 
and returns the corresponding routing instruction. Performs no interpretation, 
answering, or transformation of the content.
---

## Purpose
Detect whether the input is a questionnaire file (xlsx, word) or a direct question/set 
of questions. Return a strict routing instruction that tells the orchestrator which 
pipeline to trigger.

## Input Contract
The skill receives the raw user input from the orchestrator.

json
{
  "input": {
    "type": "file | text",
    "value": "string | file_path"
  }
}
type: either "file" or "text"
value: file path if type is file, raw text if type is text
Forbidden: empty input, null values, mixed file+text in one call
Forbidden: the skill must never read the file content only its extension

## Output Contract
json
{
  "route": "parser | answer",
  "input_type": "xlsx | word | question",
  "payload": "file_path | raw_text"
}
route: exactly "parser" or "answer" no other value allowed
input_type: exactly "xlsx", "word", or "question"
payload: unchanged original value passed as-is
The output structure must be identical regardless of input

## Positive Rules
If input type is "file" AND extension is .xlsx → route = "parser", input_type = "xlsx"
If input type is "file" AND extension is .docx or .doc → route = "parser", 
input_type = "word"
If input type is "text" → route = "answer", input_type = "question"
Always return the full output structure with all three fields
Always pass payload unchanged from input

## Negative Rules
Never read or parse file content — extension detection only
Never interpret whether a text is a question or not
Never route to both parser and answer simultaneously
Never infer file type from content
Never return a partial output structure
Never add commentary or explanation to the output
Never activate without a valid input object

## Deterministic Logic
python
1. Receive input object
2. Check input.type:
   a. If "file":
      - Extract file extension from input.value
      - If extension == ".xlsx"         → route = "parser", input_type = "xlsx"
      - If extension == ".docx" | ".doc"→ route = "parser", input_type = "word"
      - If extension is anything else   → return error contract
   b. If "text":
      - route = "answer"
      - input_type = "question"
3. Set payload = input.value unchanged
4. Return output object

## Examples

### Example Input — File XLSX
json
{
  "input": {
    "type": "file",
    "value": "/uploads/questionnaire_SG_2025.xlsx"
  }
}
### Example Output
json
{
  "route": "parser",
  "input_type": "xlsx",
  "payload": "/uploads/questionnaire_SG_2025.xlsx"
}

### Example Input — Direct Question
json
{
  "input": {
    "type": "text",
    "value": "Do you have a patch management policy?"
  }
}
### Example Output
json
{
  "route": "answer",
  "input_type": "question",
  "payload": "Do you have a patch management policy?"
}
```

**Skill error-handle**

```markdown
---
name: skill-error-handler 
description: Intercepts any failure signal from a skill or agent and returns a clean, 
structured JSON error object. Performs no diagnosis, correction, or retry logic.
---

## Purpose
Catch error signals produced by any skill or agent in the pipeline.
Return a standardised JSON error object that the orchestrator can return directly to
the user.

## Input Contract
The skill receives an error signal from a failed skill or agent.

json
{
  "origin": "skill-name | agent-name",
  "error_type": "string",
  "detail": "string"
}
origin: name of the skill or agent that failed
error_type: one of the allowed error codes below
detail: raw error message string — passed as-is, never rewritten
Forbidden: null or empty origin, null or empty error_type
Forbidden: nested error objects
Allowed error_type values:

python
"unsupported_format"
"parse_failure"
"validation_failure"
"kb_load_failure"
"routing_failure"
"source_unreachable"
"unknown_error"
Output Contract
json
{
  "status": "error",
  "origin": "string",
  "error_type": "string",
  "detail": "string"
}
status: always exactly "error" — never any other value
origin: copied unchanged from input
error_type: copied unchanged from input
detail: copied unchanged from input
The output structure must be identical for every error type

## Positive Rules
Always set status to "error"
Always copy origin unchanged from input
Always copy error_type unchanged from input
Always copy detail unchanged from input
If error_type is not in the allowed list → set error_type = "unknown_error"
Always return the full four-field structure

## Negative Rules
Never diagnose or explain the cause of the error
Never attempt to fix or retry the failed skill
Never rewrite or summarise the detail field
Never omit any of the four output fields
Never add fields beyond the four defined ones
Never return a success contract
Never activate without a valid error signal

## Deterministic Logic
lua
1. Receive error signal object
2. Copy origin → output.origin
3. Check error_type against allowed list:
   - If in allowed list → copy as-is → output.error_type
   - If NOT in allowed list → output.error_type = "unknown_error"
4. Copy detail → output.detail unchanged
5. Set output.status = "error"
6. Return output object

## Examples

### Example Input — Parse Failure
json
{
  "origin": "skill-document-parser",
  "error_type": "parse_failure",
  "detail": "Could not extract rows from sheet 'Maturity Questionnaire EN'"
}
### Example Output
json
{
  "status": "error",
  "origin": "skill-document-parser",
  "error_type": "parse_failure",
  "detail": "Could not extract rows from sheet 'Maturity Questionnaire EN'"
}

### Example Input — Unknown Error Type
json
{
  "origin": "skill-kb-loader",
  "error_type": "memory_overflow",
  "detail": "KB.json exceeded context limit"
}
### Example Output
json
{
  "status": "error",
  "origin": "skill-kb-loader",
  "error_type": "unknown_error",
  "detail": "KB.json exceeded context limit"
}
```

### Sub-agent Parser

The goal of this sub-agent is to extract questions with IDs from an questionnaire in a JSON format. It receives the questionnaire from the agent-orchestrator and it sends back its output. For an human, the task look completely easy but for an LLM, multiple step must be implemented to avoid errors as much as possible. I determine this following steps: 

**skill-document-loader**

```markdown
---
name: skill-load-document 
description: Loads the input file and detects its format based on file extension. Returns raw content and document type. Performs no extraction, classification, or interpretation.
---
 
## Purpose
Receive a file path, load the raw file content, and detect its type.
Return raw content and document type to the next skill.
Nothing else.

## Input Contract
json
{
  "file_path": "string"
}
file_path: absolute or relative path to the input file
Forbidden: null or empty file_path
Forbidden: reading file content before type detection
Forbidden: receiving anything other than a file path

## Output Contract
json
{
  "document_type": "xlsx | word",
  "raw_content": "binary | string",
  "file_path": "string"
}
document_type: exactly "xlsx" or "word" — no other value
raw_content: unmodified raw file content
file_path: original path copied unchanged
Forbidden: partial output structure
Error contract:

json
{
  "error": "unsupported_format",
  "detail": "string"
}

## Positive Rules
If extension is .xlsx → document_type = "xlsx"
If extension is .docx or .doc → document_type = "word"
Always return raw_content unmodified
Always copy file_path unchanged into output
Always return the full output structure
Negative Rules
Never read or parse file content before extension detection
Never infer document type from file content
Never modify or compress raw_content
Never support formats outside xlsx and word
Never return partial output structure
Never add commentary or explanation

## Deterministic Logic
vbnet
1. Receive file_path
2. Extract extension from file_path
3. If extension == ".xlsx"          → document_type = "xlsx"
4. If extension == ".docx" | ".doc" → document_type = "word"
5. If extension is anything else    → return error contract
6. Load raw file content unchanged
7. Return output object

## Examples

### Example Input
json
{
  "file_path": "/uploads/questionnaire_SG_2025.xlsx"
}
### Example Output
json
{
  "document_type": "xlsx",
  "raw_content": "<binary content>",
  "file_path": "/uploads/questionnaire_SG_2025.xlsx"
}
```

**skill-document-classifier**

```markdown
---
name: skill-document-classifier 
description: Receives raw file content and classifies all document elements into structured JSON categories. Performs no question extraction or answering. Returns a structured map of the document layout.
---
 
## Purpose
Read raw document content and classify every element into predefined categories.
Return a structured JSON object representing the full document layout.
This output is consumed by skill-document-parser.

## Input Contract
json
{
  "document_type": "xlsx | word",
  "raw_content": "binary | string"
}
document_type: exactly "xlsx" or "word"
raw_content: unmodified raw content from skill-load-document
Forbidden: null document_type or raw_content
Forbidden: receiving already-parsed or modified content

## Output Contract
json
{
  "metadata": {
    "document_type": "xlsx | word",
    "language": "string",
    "total_rows": "integer"
  },
  "sections": [
    {
      "id": "string",
      "title": "string",
      "type": "section_header"
    }
  ],
  "questions": [
    {
      "id": "string",
      "text": "string",
      "context": ["string"],
      "type": "question"
    }
  ],
  "ignored": [
    {
      "id": "string",
      "reason": "string"
    }
  ]
}
metadata: always present, all fields required
sections: list of detected section headers — empty array if none
questions: list of detected question rows — never empty
ignored: list of skipped rows with reason — empty array if none
Forbidden: omitting any top-level key

## Positive Rules
If row has a numeric ID pattern → classify as "question"
If row is a domain or sub-domain header → classify as "section_header"
If row is decorative, empty, or a scoring artifact → classify as "ignored"
If question has inline list items (e.g. "ci-après") → include them in context
If document is xlsx → target sheet "Maturity Questionnaire EN" only
Always include all four top-level keys in output

## Negative Rules
Never classify scoring artifacts (e.g. "100%") as questions
Never merge section headers into question text
Never invent IDs or text not present in the document
Never omit ignored rows — always log them with a reason

## Deterministic Logic
sql
1. Receive document_type and raw_content
2. If document_type == "xlsx":
   - Target sheet "Maturity Questionnaire EN"
   - For each row:
     a. If ID column matches ^\d+(\.\d+)+ → type = "question"
     b. If ID column is empty AND text looks like domain title → type = "section_header"
     c. If response column contains "100%" | "0%" → type = "ignored", reason = "scoring_artifact"
     d. If row is empty → type = "ignored", reason = "empty_row"
3. If document_type == "word":
   - For each block:
     a. If block starts with numeric pattern → type = "question"
     b. If block is bold/uppercase with no number → type = "section_header"
     c. Otherwise → type = "ignored"
4. Build output JSON with metadata, sections, questions, ignored
5. Return output object

## Examples

### Example Input
json
{
  "document_type": "xlsx",
  "raw_content": "<binary content>"
}
### Example Output
json
{
  "metadata": {
    "document_type": "xlsx",
    "language": "en",
    "total_rows": 112
  },
  "sections": [
    {"id": "1", "title": "Cyber security management", "type": "section_header"}
  ],
  "questions": [
    {"id": "1.01", "text": "Does your organisation have documented Information Security policies?", "context": [], "type": "question"},
    {"id": "2.07", "text": "If yes, are the same security measures applied?", "context": ["item 1", "item 2"], "type": "question"}
  ],
  "ignored": [
    {"id": "Header.1", "reason": "non_question_label"},
    {"id": "1.07", "reason": "scoring_artifact"}
  ]
}
```

**skill-document-parser**

```markdown
---
name: skill-document-parser 
description: Receives the classified document JSON and extracts only the questions array into a strict normalized JSON output. Performs no classification, answering, or interpretation.
---

## Purpose 
Consume the classified document JSON from skill-document-classifier.
Extract only the questions array.
Normalize IDs and merge contextual lists into question text.
Return a strict JSON object ready for Pydantic validation.

## Input Contract
json
{
  "questions": [
    {
      "id": "string",
      "text": "string",
      "context": ["string"],
      "type": "question"
    }
  ]
}
questions: non-empty array of classified question items
context: may be empty array — never null
Forbidden: receiving section_header or ignored items
Forbidden: receiving raw file content directly

## Output Contract
json
{
  "questions": [
    {
      "id": "string",
      "text": "string"
    }
  ]
}
id: normalized string matching ^\d+(\.\d+)+$
text: verbatim question text with context merged if present
Forbidden: any key other than id and text per item
Forbidden: empty id or empty text
Forbidden: duplicate IDs

## Positive Rules
Always preserve original document order
If context array is non-empty → append items to text with newline separator
Always normalize ID format to match ^\d+(\.\d+)+$
Always output every item from input questions array
Always produce exactly two keys per item: id and text

## Negative Rules
Never skip a question because its text seems empty or unclear
Never interpret or rewrite question text
Never answer any question
Never add fields beyond id and text
Never invent or complete missing IDs
Never merge two separate questions into one
Never reorder questions

## Deterministic Logic
vbnet
1. Receive classified questions array
2. For each item:
   a. Copy id → normalize to ^\d+(\.\d+)+$ format
   b. Copy text verbatim
   c. If context is non-empty:
      → append each context item to text separated by "\n"
   d. Build output item: {"id": normalized_id, "text": merged_text}
3. Preserve original order
4. Return {"questions": [output_items]}

## Examples

### Example Input
json
{
  "questions": [
    {
      "id": "1.01",
      "text": "Does your organisation have documented Information Security policies?",
      "context": [],
      "type": "question"
    },
    {
      "id": "2.07",
      "text": "Are the same security measures applied to the following?",
      "context": ["Dev environment", "UAT environment"],
      "type": "question"
    }
  ]
}
### Example Output
json
{
  "questions": [
    {
      "id": "1.01",
      "text": "Does your organisation have documented Information Security policies?"
    },
    {
      "id": "2.07",
      "text": "Are the same security measures applied to the following?\nDev environment\nUAT environment"
    }
  ]
}
```

### Sub-agent Answer

This agent is responsible to answer question from questionnaire or simple question. 

**Skill kb-loader-matcher**

```markdown
charge KB.json for memory
store in cache
avoid rereading
```

**Skill language-detector**

```markdown
detect language for input
all generated question must be in this language 
```

**Skill source-router**

```markdown
decide which sources must be called for each question
KB if match 
KB+ confluence if ...
KB+confluence+gitlab
```

**Skill kb-matcher**

```markdown
search for the most pertinent response for the question
```

**Skill confluence-search**

```markdown
call the MCP confluence with a specific url
used if kb-matcher is weaken
```

**Skill code-source**

```markdown
call MCP gitlab with specific url
```

**Skill kb**

```markdown
generate the final response with the most useful source available 
```

**Skill confidence-scorer**

```markdown
give a bucket confidence score for each generated response 
```

**Skill Source-resolver**

```markdown
determine real name of the source to cite
KB: metadat.sources
confluence: web title page
Gitlab: path repo
```

## Sources

1. https://www.deeplearning.ai/courses/agent-skills-with-anthropic
2. https://agentskills.io/home
3. https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview#how-skills-work
4. https://github.com/anthropics/skills/tree/main/skills
