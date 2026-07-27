---
title: "Pydantic AI: Ensuring Reliable and Structured Agent Outputs"
date: 2026-07-20 16:25:00 +0200
categories: [Tools]
tags: [Pydantic AI, AI, Agents, Validation, Skills, Structured-Outputs]
toc: true
math: true
---
# Pydantic AI

I became interested in **Pydantic AI** because I wanted to use **multi‑skills** inside an agent. Pydantic ensures that every output format is respected, which is exactly what I needed.

# What is Pydantic AI?

Pydantic AI is a Python framework for building AI agents especially agents that use LLMs with strong type‑safety, structured outputs, and production‑grade reliability. Pydantic is downloaded over 550M times/month and is used by all FAANG companies and 20 of the 25 largest companies on NASDAQ. 

## Why use Pydantic?

Pydantic is the most widely used data‑validation library in Python. It allows you to control exactly what the output should look like, without suffering from LLM hallucinations.
Simple text prompts, JSON prompts, or XML prompts often lead to inconsistent or random output formats.
This can be fixed by adding a Pydantic validation script next to your Skills to ensure that every output follows the expected structure.

Its core validation engine is written in Rust, making it one of the fastest data‑validation systems available.

Pydantic uses Python type annotations (`int`, `str`, `list[str]`, `dict`, custom models, etc.) to automatically validate incoming data and serialize outgoing data.

## What does it provide?

You define your agent’s expected output using **Pydantic models**.
The LLM must return data that matches the schema, otherwise validation fails.
This eliminates a huge class of runtime errors.

It also works with **every LLM provider**.

## How install Pydantic AI on windows?

> $01$ Open your WSL terminal in VScode.
> 

> $02$ Install python and pip
> 

```bash
python3 --version
sudo apt install python3-pip -y
```

> $03$ Create a virtual environment and active it inside your project
> 

```bash
python3 -m venv .venv
source .venv/bin/activate
```

> $04$ Install Pydantic AI
> 

```bash
pip install pydantic-ai
```

You must use the virtual environment whenever you want to run Pydantic.

<img width="899" height="250" alt="image" src="https://github.com/user-attachments/assets/e6c4dafe-6a9b-42d7-8542-8ac205f3f8f7" />

## How to write Pydantic AI script?

**Validator skeleton: Pydantic model for each agent’s output.**

```python
# output_validator.py

from pydantic import BaseModel, ValidationError
from typing import Any

class OutputValidator:
    def __init__(self, schema: type[BaseModel]):
        self.schema = schema

    def validate(self, data: Any) -> BaseModel:
        try:
            return self.schema.model_validate(data)
        except ValidationError as e:
            raise ValueError(f"Output validation failed:\n{e}") from e
```

**What is Base Model?** 

A `BaseModel` defines the shape, types, and validation rules of your data. `BaseModel` enforces structure which means that every variables must respect its data types. Each file defines a Pydantic `BaseModel` describing the expected output of that agent.

For example: 

```python
# schemas/parser.py
from pydantic import BaseModel

class ParserOutput(BaseModel):
    tokens: list[str]
    metadata: dict
```

**What organization looks like?** 

```python
/skills/
  /parser/
    skill-document-loader/
      SKILL.md
      schema.py #script pydantic here
      references/
	      KB.Json
      tests/
        test_loader.py
```

**What are tests?** 

Tests are small Python scripts that verify each Skill behaves correctly which means that Skills are stable, predictable, and safe to use inside agent pipeline. 

For example, test looks like: 

```python
import json
from schema import RawItemList
from skill_document_loader import run_skill

def test_loader_output_schema():
    output = run_skill("tests/sample_document.xlsx")
    RawItemList.parse_obj(output)  # Pydantic validation
```

## My first script Pydantic AI

My goal was to write 2 smalls scripts for my sub-agent Parser and one for my sub-agent Answer.

#### Sub-agent Parser

```python
# Import Pydantic base class and validator decorators for data validation
from pydantic import BaseModel, field_validator, model_validator
# Import standard library modules for JSON handling, regex pattern matching, and system operations
import json
import re
import sys

# Define the model for individual Question objects
class Question(BaseModel):
    id: str     # Question identifier (must be a string)
    text: str   # The text content of the question

    # Validator applied specifically to the 'id' field
    @field_validator("id")
    @classmethod
    def id_must_match_pattern(cls, v):
        # Check if the ID is blank or contains only whitespace
        if not v.strip():
            raise ValueError("id is empty")
        # Ensure the ID follows a dotted-decimal pattern (e.g., "1.2", "1.2.3")
        if not re.match(r"^\d+(\.\d+)+$", v):
            raise ValueError(f"id '{v}' does not match pattern")
        return v # Return the validated value

    # Validator applied specifically to the 'text' field
    @field_validator("text")
    @classmethod
    def text_must_not_be_empty(cls, v):
        # Check if the text is blank or contains only whitespace
        if not v.strip():
            raise ValueError("text is empty")
        return v # Return the validated value

# Define the top-level model that holds a list of questions
class ParserOutput(BaseModel):
    questions: list[Question] # Expects a list containing Question items

    # Model-level validator executed after field-level validations complete
    @model_validator(mode="after")
    def check_no_duplicates(self):
        # Extract all question IDs into a list
        ids = [q.id for q in self.questions]
        # Find any ID that appears more than once
        duplicates = [id for id in ids if ids.count(id) > 1]
        # Raise an error if duplicate IDs exist
        if duplicates:
            raise ValueError(f"Duplicate IDs found: {set(duplicates)}")
        return self # Return the validated instance

    # Another model-level validator executed after fields are validated
    @model_validator(mode="after")
    def check_not_empty(self):
        # Ensure the questions list is not empty
        if not self.questions:
            raise ValueError("questions array is empty")
        return self # Return the validated instance

# Function to validate an input dictionary against the ParserOutput schema
def validate(data: dict) -> dict:
    try:
        # Attempt to parse and validate input dictionary
        parsed = ParserOutput(**data)
        # Return success metadata if validation passes
        return {
            "status": "valid",
            "question_count": len(parsed.questions)
        }
    except Exception as e:
        # Return error details if validation fails
        return {
            "status": "invalid",
            "question_count": len(data.get("questions", [])),
            "issues": str(e)
        }

# Entry point when script is executed directly from the terminal
if __name__ == "__main__":
    # If a file path argument was passed via CLI
    if len(sys.argv) > 1:
        # Open and load the JSON file
        with open(sys.argv[1], "r") as f:
            data = json.load(f)
    else:
        # Read and load JSON content piped through standard input (stdin)
        data = json.load(sys.stdin)

    # Validate the loaded JSON data
    result = validate(data)
    # Output the result formatted as readable JSON
    print(json.dumps(result, indent=2))
    # Exit with code 0 if valid, or 1 if invalid
    sys.exit(0 if result["status"] == "valid" else 1)
```

#### Sub-agent Answer

```python
import json
import re
import sys
# Pydantic is a library used for data validation.
# BaseModel: base class for creating data models.
# field_validator: validates individual fields/attributes.
# model_validator: validates the model as a whole (e.g., cross-field checks).
from pydantic import BaseModel, field_validator, model_validator

# Pre-defined allowed values for validation
ALLOWED_CONFIDENCE = {"95%", "80%", "60%", "40%", "10%"}
SUPPORTED_LANGUAGES = {"en", "fr", "de", "es", "it", "nl", "pt"}

# Model 1: Individual Answer
# Defines the expected structure and validation rules for one answer.
class Answer(BaseModel):
    id: str
    question: str
    text: str
    confidence: str
    source: str

    # Ensure the 'id' string isn't empty or just whitespace
    @field_validator("id")
    @classmethod
    def id_must_not_be_empty(cls, v):
        if not v.strip():
            raise ValueError("id is empty")
        return v

    # Ensure the 'question' string isn't empty or just whitespace
    @field_validator("question")
    @classmethod
    def question_must_not_be_empty(cls, v):
        if not v.strip():
            raise ValueError("question is empty")
        return v

    # Ensure the 'text' (the actual answer content) isn't empty or whitespace
    @field_validator("text")
    @classmethod
    def text_must_not_be_empty(cls, v):
        if not v.strip():
            raise ValueError("text is empty")
        return v

    # Ensure 'confidence' matches one of the allowed percentage strings
    @field_validator("confidence")
    @classmethod
    def confidence_must_be_valid_bucket(cls, v):
        if v not in ALLOWED_CONFIDENCE:
            raise ValueError(
                f"confidence '{v}' is not a valid bucket. "
                f"Allowed: {ALLOWED_CONFIDENCE}"
            )
        return v

    # Prevent generic/placeholder source named "kb.json"
    @field_validator("source")
    @classmethod
    def source_must_not_be_kb_json(cls, v):
        if v.strip().lower() == "kb.json":
            raise ValueError(
                "source must never be 'KB.json'. "
                "Use the real document name from metadata.sources."
            )
        return v

# Model 2: Answer Output Wrapper
# Represents the root JSON object containing a list of Answer objects.
class AnswerOutput(BaseModel):
    answers: list[Answer]

    # Model-level check: Ensure the 'answers' list is not empty
    @model_validator(mode="after")
    def check_not_empty(self):
        if not self.answers:
            raise ValueError("answers array is empty")
        return self

    # Model-level check: Ensure all answer IDs in the list are unique
    @model_validator(mode="after")
    def check_no_duplicate_ids(self):
        ids = [a.id for a in self.answers]
        duplicates = [id for id in ids if ids.count(id) > 1]
        if duplicates:
            raise ValueError(f"Duplicate answer IDs: {set(duplicates)}")
        return self

# Validation Function
# Takes raw dictionary data and tests it against the Pydantic models.
def validate(data: dict, expected_count: int = None) -> dict:
    try:
        # Pass raw JSON dictionary into the Pydantic schema
        parsed = AnswerOutput(**data)

        # Optional check: verify if the number of answers matches expected count
        if expected_count is not None:
            if len(parsed.answers) != expected_count:
                return {
                    "status": "invalid",
                    "answer_count": len(parsed.answers),
                    "issues": (
                        f"Expected {expected_count} answers, "
                        f"got {len(parsed.answers)}"
                    ),
                }

        # If all checks pass
        return {"status": "valid", "answer_count": len(parsed.answers)}

    except Exception as e:
        # Return error details if validation fails at any point
        return {
            "status": "invalid",
            "answer_count": len(data.get("answers", [])),
            "issues": str(e),
        }

# Main Script Execution
# Allows running the script directly from the command line.
if __name__ == "__main__":
    # Check if a file path argument was passed via command line
    if len(sys.argv) > 1:
        # Read JSON from the provided file path
        with open(sys.argv[1], "r") as f:
            data = json.load(f)
    else:
        # Otherwise, read JSON standard input (e.g., piped data)
        data = json.load(sys.stdin)

    # Read optional second argument: expected number of answers
    expected = int(sys.argv[2]) if len(sys.argv) > 2 else None

    # Run the validation
    result = validate(data, expected_count=expected)

    # Output formatted JSON result to console
    print(json.dumps(result, indent=2))

    # Exit with code 0 for success, 1 for failure
    sys.exit(0 if result["status"] == "valid" else 1)
```

## Sources

1. https://pydantic.dev/docs/ai/overview/
2. https://pydantic.dev/docs/validation/latest/get-started/
3. https://pydantic.dev/docs/ai/overview/install/
