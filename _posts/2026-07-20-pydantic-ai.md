---
title: "Pydantic AI: Ensuring Reliable and Structured Agent Outputs"
date: 2026-07-20 16:25:00 +0200
categories: [Tools]
tags: [Pydantic AI, AI, Agents, Validation, Skills, Structured-Outputs]
toc: true
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

## Sources

1. https://pydantic.dev/docs/ai/overview/
2. https://pydantic.dev/docs/validation/latest/get-started/
3. https://pydantic.dev/docs/ai/overview/install/
