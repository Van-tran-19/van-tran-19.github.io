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

#### 4th Learning: Don’t use skills as a pydantic script

A skill mustn’t run python script, a skill is used to guide the agent. Schema validation must be done externally, not by the model. A model mustn’t be used as a security tools. 

#### **5th Learning: How to make an agent search only within a specific Confluence URL, even though the agent already has broad access to Kazan Confluence through MCP tools?**

You can define a **Skill** that forces the agent to prioritize or restrict its search to that exact URL, and this approach works because Skills guide and constrain how the agent uses the MCP tools.

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

## Output Contract
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
---
name: skill-kb-loader-matcher 
description: Loads KB.json into memory, caches it for the full run, and finds the most 
relevant KB entry for a given question. Performs no answering, interpretation, or 
content modification.
---

## Purpose
Load KB.json once at startup and cache it for the entire run.
For each input question, search the cache and return the most relevant KB entry with a 
match quality score. Never reload the file between questions.

## Input Contract
json
{
  "question": {
    "id": "string",
    "text": "string",
    "lang": "string"
  }
}
id: question identifier from parser output
text: verbatim question text
lang: detected language code from skill-language-detector
Forbidden: null or empty text
Forbidden: calling this skill before KB.json is loaded

## Output Contract
json
{
  "question_id": "string",
  "kb_entry": {
    "id": "string",
    "question_text": "string",
    "answer_text": "string",
    "tags": ["string"],
    "sources": ["string"],
    "status": "string",
    "needs_review": "boolean",
    "review_note": "string"
  },
  "match_quality": "direct | partial | weak | none"
}
kb_entry: null if match_quality is "none"
match_quality: exactly one of the four allowed values
Forbidden: omitting match_quality
Forbidden: returning multiple KB entries

## Match quality rules:

sql
"direct"  → question_text similarity > 90%, same language, status active
"partial" → question_text similarity 60–90%, or different language match
"weak"    → similarity 30–60%, or needs_review flagged
"none"    → similarity < 30% or no relevant entry found

## Positive Rules
Always load KB.json once and store in memory cache at startup
Always reuse cache, never reload between questions
Always return match_quality even if kb_entry is null
If multiple KB entries match → return highest similarity score only
If KB entry language differs from question lang → still return it, flag as partial

## Negative Rules
Never answer the question
Never modify KB entry content
Never merge multiple KB entries
Never reload KB.json between questions
Never return match_quality = "direct" if needs_review is true
Never omit kb_entry structure when match exists

## Deterministic Logic
sql
STARTUP:
1. Load KB.json from file system
2. Store all entries in memory cache
3. Set cache_ready = true

PER QUESTION:
1. Receive question object
2. Assert cache_ready == true
3. For each KB entry:
   a. Compute text similarity score (question.text vs entry.question_text)
   b. Check language match (question.lang vs entry.language)
   c. Check entry.status and entry.needs_review
4. Select entry with highest similarity score
5. Assign match_quality:
   - score > 90% AND same lang AND not needs_review → "direct"
   - score 60–90% OR different lang                → "partial"
   - score 30–60% OR needs_review == true           → "weak"
   - score < 30% OR no entry found                  → "none"
6. Return output object

## Examples

### Example Input
json
{
  "question": {
    "id": "1.01",
    "text": "Does your organisation have documented Information Security policies?",
    "lang": "en"
  }
}
### Example Output
json
{
  "question_id": "1.01",
  "kb_entry": {
    "id": "SRC-KB-1.1",
    "question_text": "Does your organisation have documented Information Security policies?",
    "answer_text": "Yes, Worldline maintains documented Information Security policies reviewed annually.",
    "tags": ["policy", "security"],
    "sources": ["SG_questionnaire_2025.xlsx"],
    "status": "active",
    "needs_review": false,
    "review_note": ""
  },
  "match_quality": "direct"
}
```

**Skill language-detector**

```markdown
---
name: skill-language-detector 
description: Detects the language of each input question and returns a language code. 
Performs no translation, answering, or content modification.
---

## Purpose
Detect the language of each input question text.
Return a standard language code per question.
All downstream skills must generate answer text in this detected language.

## Input Contract
json
{
  "question": {
    "id": "string",
    "text": "string"
  }
}
text: verbatim question text — minimum 3 characters
Forbidden: null or empty text
Forbidden: pre-labelled language input

## Output Contract
json
{
  "question_id": "string",
  "lang": "string"
}
lang: ISO 639-1 language code — e.g. "en", "fr", "de"
If language cannot be detected → lang = "en" as default
Forbidden: returning null lang
Forbidden: returning full language name instead of code

## Positive Rules
Always return exactly one language code per question
Always use ISO 639-1 two-letter codes
If detection is ambiguous → default to "en"
Always return question_id unchanged from input

## Negative Rules
Never translate question text
Never answer the question
Never return multiple language codes
Never return null or empty lang

## Deterministic Logic
sql
1. Receive question object
2. Analyse question.text for language signals
3. Match against known language patterns
4. If match found   → lang = ISO 639-1 code
5. If no match      → lang = "en"
6. Return {question_id, lang}

## Examples

### Example Input
json
{"question": {"id": "1.01", "text": "Does your organisation have documented policies?"}}
### Example Output
json
{"question_id": "1.01", "lang": "en"}

### Example Input
json
{"question": {"id": "2.01", "text": "Votre organisation effectue-t-elle des évaluations des risques?"}}
### Example Output
json
{"question_id": "2.01", "lang": "fr"}
```

**Skill source-router**

```markdown
---
name: skill-source-router 
description: Decides which sources to query for each question based on KB match quality. Returns an ordered source plan. Performs no answering, fetching, or content generation.
---

## Purpose
Receive the KB match quality for a question.
Return a deterministic ordered list of sources to query.
Minimize source calls never call Confluence or GitLab if KB is sufficient.

## Input Contract
json
{
  "question_id": "string",
  "question_text": "string",
  "match_quality": "direct | partial | weak | none"
}

## match_quality: 
exactly one of the four allowed values
Forbidden: null or missing match_quality

## Output Contract
json
{
  "question_id": "string",
  "sources_to_query": ["kb", "confluence", "gitlab"]
}
sources_to_query: ordered array of source identifiers
Allowed values per item: "kb", "confluence", "gitlab"
Minimum one source always present
Forbidden: empty sources_to_query array

## Routing rules:
css
"direct"  → ["kb"]
"partial" → ["kb", "confluence"]
"weak"    → ["kb", "confluence"]
"none"    → ["kb", "confluence", "gitlab"]

## Positive Rules
Always include "kb" as first source regardless of match quality
If match_quality is "direct" → return ["kb"] only
If match_quality is "partial" or "weak" → return ["kb", "confluence"]
If match_quality is "none" → return ["kb", "confluence", "gitlab"]
Always preserve the order: kb → confluence → gitlab

## Negative Rules
Never call Confluence if match_quality is "direct"
Never call GitLab if Confluence has not been queried first
Never return an empty sources array
Never reorder the source priority
Never add sources outside the three allowed values

## Deterministic Logic
css
1. Receive question_id, question_text, match_quality
2. Switch match_quality:
   - "direct"           → sources = ["kb"]
   - "partial" | "weak" → sources = ["kb", "confluence"]
   - "none"             → sources = ["kb", "confluence", "gitlab"]
3. Return {question_id, sources_to_query: sources}

## Examples
### Example Input
json
{"question_id": "1.01", "question_text": "Do you have an IS policy?", "match_quality": "direct"}
### Example Output
json
{"question_id": "1.01", "sources_to_query": ["kb"]}

### Example Input
json
{"question_id": "7.05", "question_text": "How often do you conduct penetration tests?", "match_quality": "none"}
### Example Output
json
{"question_id": "7.05", "sources_to_query": ["kb", "confluence", "gitlab"]}
```

**Skill confluence-search**

```markdown
---
name: skill-confluence-search 
description: Calls the MCP Confluence tool with a targeted query and returns the raw page content. Only activated when kb-matcher returns partial, weak, or none match quality. Performs no answering or interpretation.
---

## Purpose
Query the approved Confluence space using the MCP Confluence tool.
Return raw page content as plain text.
Never called if KB match quality is direct.

## Input Contract
json
{
  "question_id": "string",
  "query": "string",
  "approved_spaces": ["string"]
}
query: verbatim question text used as search query
approved_spaces: list of allowed Confluence space keys
Forbidden: querying spaces not in approved_spaces
Forbidden: calling this skill if match_quality is "direct"

##Approved spaces:
python
"TA-Security"
"Worldline-Policies"
Output Contract
json
{
  "question_id": "string",
  "confluence_result": "string | null",
  "page_title": "string | null",
  "page_url": "string | null"
}
confluence_result: raw plain text content of the page — null if nothing found
page_title: exact title of the Confluence page — null if nothing found
page_url: full URL of the page — null if nothing found
Forbidden: returning HTML or markdown-formatted content
Forbidden: returning multiple pages merged into one result

## Positive Rules
Always restrict search to approved_spaces only
Always return plain text content — strip navigation, headers, footers
Always return page_title and page_url alongside content
If no result found → return null for all three result fields
Always return question_id unchanged

## Negative Rules
Never query unapproved Confluence spaces
Never interpret or summarise page content
Never merge content from multiple pages
Never call this skill if match_quality is "direct"
Never return HTML tags in confluence_result

## Deterministic Logic
sql
1. Receive question_id, query, approved_spaces
2. Call MCP Confluence tool with query
3. Filter results to approved_spaces only
4. If results found:
   a. Select top result
   b. Extract plain text content — strip HTML
   c. Extract page_title and page_url
   d. Return full output object
5. If no results:
   a. Return {question_id, confluence_result: null, page_title: null, page_url: null}

## Examples

### Example Input
json
{
  "question_id": "7.05",
  "query": "How often do you conduct penetration tests?",
  "approved_spaces": ["TA-Security", "Worldline-Policies"]
}
### Example Output
json
{
  "question_id": "7.05",
  "confluence_result": "Worldline conducts external penetration tests every 24 months on infrastructure processing client data...",
  "page_title": "Vulnerability Management Policy",
  "page_url": "https://confluence.worldline.com/display/TA-Security/Vulnerability+Management"
}
```

**Skill code-source**

```markdown
---
name: skill-code-source 
description: Calls the MCP GitLab tool with a targeted query and returns relevant code references. Only activated when both KB and Confluence returned no usable result. Performs no answering or interpretation.
---

## Purpose
Query the approved GitLab repository using the MCP GitLab tool.
Return relevant code comments, README excerpts, or configuration references.
Never called unless KB and Confluence both returned null results.

## Input Contract
json
{
  "question_id": "string",
  "query": "string",
  "approved_repos": ["string"]
}
query: verbatim question text used as search query
approved_repos: list of allowed repository paths
Forbidden: querying repos not in approved_repos
Forbidden: calling this skill if KB or Confluence returned a usable result

## Output Contract
json
{
  "question_id": "string",
  "gitlab_result": "string | null",
  "file_path": "string | null",
  "repo": "string | null"
}
gitlab_result: relevant extracted text — null if nothing found
file_path: exact file path within the repository — null if nothing found
repo: repository name — null if nothing found
Forbidden: returning full file contents — extract only relevant lines

## Positive Rules
Always restrict search to approved_repos only
Always return only relevant lines — not the full file
Always return file_path and repo alongside content
If no result found → return null for all three result fields

## Negative Rules
Never query unapproved repositories
Never return full file content
Never interpret or summarise code
Never call this skill if KB or Confluence returned usable content
Never answer the question

## Deterministic Logic
sql
1. Receive question_id, query, approved_repos
2. Call MCP GitLab tool with query
3. Filter results to approved_repos only
4. If results found:
   a. Extract relevant lines only (README, comments, config)
   b. Extract file_path and repo name
   c. Return full output object
5. If no results:
   a. Return {question_id, gitlab_result: null, file_path: null, repo: null}

## Examples

### Example Input
json
{
  "question_id": "9.06",
  "query": "automated source code analysis security",
  "approved_repos": ["worldline/ta-security", "worldline/ta-policies"]
}
### Example Output
json
{
  "question_id": "9.06",
  "gitlab_result": "# SAST is enforced via SonarQube on every merge request to main branch.",
  "file_path": "docs/SDLC.md",
  "repo": "worldline/ta-security"
}
```

**Skill kb**

```markdown
---
name: skill-kb 
description: Generates the final answer text for a question using the best available source. Performs faithful reformulation only. Never invents facts or adds content absent from the source.
---

## Purpose
Receive the best available source result (KB, Confluence, or GitLab).
Generate a clean, professional answer in the question's detected language.
Apply strict source fidelity, reformulation allowed.

## Input Contract
json
{
  "question_id": "string",
  "question_text": "string",
  "lang": "string",
  "kb_entry": "object | null",
  "confluence_result": "string | null",
  "gitlab_result": "string | null"
}
At least one source must be non-null or produce a refusal entry
lang: ISO 639-1 code from skill-language-detector
Forbidden: generating content without a source
Forbidden: combining facts from multiple sources in one answer

## Output Contract
json
{
  "question_id": "string",
  "answer_text": "string",
  "source_used": "kb | confluence | gitlab | none"
}
answer_text: professional answer in detected language — never empty
source_used: exactly one of the four allowed values
If source_used is "none" → answer_text = refusal message in detected language
Refusal messages:

json
"en": "Source information missing"
"fr": "Information source manquante"

## Positive Rules
Priority order: kb_entry → confluence_result → gitlab_result → refusal
Always generate answer in the detected language
Always use the highest-priority non-null source available
If kb_entry exists → use answer_text, faithfully rephrased
If only confluence_result → extract and rephrase relevant content
If only gitlab_result → extract and rephrase relevant content
If all sources null → return refusal message in detected language

## Negative Rules
Never invent facts absent from the source
Never combine content from multiple sources
Never answer in a different language than detected
Never return empty answer_text
Never skip the refusal entry if all sources are null

## Deterministic Logic
sql
1. Receive all source inputs
2. Select source by priority:
   a. If kb_entry != null       → source = "kb",         use kb_entry.answer_text
   b. Else if confluence != null → source = "confluence", use confluence_result
   c. Else if gitlab != null    → source = "gitlab",     use gitlab_result
   d. Else                      → source = "none",       use refusal message[lang]
3. Rephrase selected content in lang — no new facts added
4. Return {question_id, answer_text, source_used}

## Examples

### Example Input
json
{
  "question_id": "1.01",
  "question_text": "Does your organisation have documented IS policies?",
  "lang": "en",
  "kb_entry": {"answer_text": "Yes, Worldline maintains documented IS policies reviewed annually."},
  "confluence_result": null,
  "gitlab_result": null
}
### Example Output
json
{
  "question_id": "1.01",
  "answer_text": "Yes, Worldline maintains documented Information Security policies, reviewed on an annual basis.",
  "source_used": "kb"
}
```

**Skill confidence-scorer**

```markdown
---
name: skill-confidence-scorer 
description: Assigns exactly one confidence bucket to a generated answer based on source quality and match characteristics. Returns a percentage string. Performs no content generation or modification.
---

Purpose
Receive the source used, match quality, and KB entry metadata.
Return exactly one confidence bucket as a percentage string.
No interpretation — purely rule-based assignment.

## Input Contract
json
{
  "question_id": "string",
  "source_used": "kb | confluence | gitlab | none",
  "match_quality": "direct | partial | weak | none",
  "needs_review": "boolean"
}
Forbidden: null source_used or match_quality
Forbidden: receiving values outside allowed enums

## Output Contract
json
{
  "question_id": "string",
  "confidence": "95% | 80% | 60% | 40% | 10%"
}
confidence: exactly one of the five allowed string values
Forbidden: words like "high", "low", "medium"
Forbidden: bare numbers without % sign
Forbidden: values outside the five allowed buckets

## Positive Rules
Always return exactly one confidence string
If source_used = "none" → always return "10%"
If needs_review = true → downgrade one bucket level
Apply bucket strictly from the logic table below
Bucket assignment table:

sql
source=kb,         match=direct,  needs_review=false → "95%"
source=kb,         match=direct,  needs_review=true  → "80%"
source=kb,         match=partial, needs_review=false → "80%"
source=kb,         match=partial, needs_review=true  → "60%"
source=kb,         match=weak                        → "40%"
source=confluence, any match                         → "80%"
source=confluence, match=partial                     → "60%"
source=gitlab,     any match                         → "40%"
source=none                                          → "10%"

## Negative Rules
Never output a word instead of a percentage string
Never output a value outside the five allowed buckets
Never assign "95%" if needs_review is true
Never assign anything other than "10%" for source=none

## Deterministic Logic
sql
1. Receive question_id, source_used, match_quality, needs_review
2. If source_used == "none"       → confidence = "10%"
3. If source_used == "gitlab"     → confidence = "40%"
4. If source_used == "confluence":
   - match == "direct" | "partial" → confidence = "80%"
   - match == "weak"               → confidence = "60%"
5. If source_used == "kb":
   - match == "direct" AND needs_review == false → confidence = "95%"
   - match == "direct" AND needs_review == true  → confidence = "80%"
   - match == "partial" AND needs_review == false → confidence = "80%"
   - match == "partial" AND needs_review == true  → confidence = "60%"
   - match == "weak"                              → confidence = "40%"
6. Return {question_id, confidence}

## Examples

### Example Input
json
{"question_id": "1.01", "source_used": "kb", "match_quality": "direct", "needs_review": false}
### Example Output
json
{"question_id": "1.01", "confidence": "95%"}

### Example Input
json
{"question_id": "7.05", "source_used": "none", "match_quality": "none", "needs_review": false}
### Example Output
json
{"question_id": "7.05", "confidence": "10%"}
```

**Skill Source-resolver**

```markdown
---
name: skill-source-resolver 
description: Determines the real document name or path to cite in the source field of the answer output. Performs no content generation or answering. Returns a string or empty string.
---

## Purpose
Receive the source used and associated metadata.
Return the exact document name, page title, or file path to cite.
Never return "KB.json" or any internal system reference.

## Input Contract
json
{
  "question_id": "string",
  "source_used": "kb | confluence | gitlab | none",
  "kb_entry": {
    "sources": ["string"]
  },
  "page_title": "string | null",
  "file_path": "string | null",
  "repo": "string | null"
}
kb_entry.sources: array from KB metadata — may be empty
Forbidden: null source_used
Forbidden: passing internal system filenames as source

## Output Contract
json
{
  "question_id": "string",
  "source": "string"
}
source: real document name, page title, or repo/file path
If no source determinable → source = ""
Forbidden: returning "KB.json" as source value
Forbidden: returning null — always return string or empty string

## Positive Rules
If source_used = "kb" → use first entry in kb_entry.sources
If source_used = "confluence" → use page_title
If source_used = "gitlab" → use "repo/file_path" format
If source_used = "none" → return ""
If kb_entry.sources is empty → return ""

## Negative Rules
Never return "KB.json" as source
Never return null
Never invent a source name not provided in input
Never combine multiple source names into one string

## Deterministic Logic
bash
1. Receive question_id, source_used, and metadata
2. Switch source_used:
   a. "kb":
      - If kb_entry.sources is non-empty → source = sources[0]
      - If kb_entry.sources is empty     → source = ""
   b. "confluence":
      - If page_title != null → source = page_title
      - If page_title == null → source = ""
   c. "gitlab":
      - If repo and file_path != null → source = repo + "/" + file_path
      - Otherwise                     → source = ""
   d. "none" → source = ""
3. Return {question_id, source}

## Examples

### Example Input — KB source
json
{
  "question_id": "1.01",
  "source_used": "kb",
  "kb_entry": {"sources": ["Knowledge_Base.xlsx"]},
  "page_title": null,
  "file_path": null,
  "repo": null
}
### Example Output
json
{"question_id": "1.01", "source": "Knowledge_Base.xlsx"}

### Example Input — GitLab source
json
{
  "question_id": "9.06",
  "source_used": "gitlab",
  "kb_entry": {"sources": []},
  "page_title": null,
  "file_path": "docs/SDLC.md",
  "repo": "worldline/ta-security"
}
### Example Output
json
{"question_id": "9.06", "source": "worldline/ta-security/docs/SDLC.md"}
```

## Sources

1. https://www.deeplearning.ai/courses/agent-skills-with-anthropic
2. https://agentskills.io/home
3. https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview#how-skills-work
4. https://github.com/anthropics/skills/tree/main/skills
