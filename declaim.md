# Declaim Prompt Format (v0.2)

**Declaim** is a declarative interaction language designed to break down large language model (LLM) tasks into structured, context-aware subtasks that can be interpreted, tracked, and executed deterministically. This format is intended to be used both by humans and LLMs to produce valid Declaim YAML.

## Purpose

- Split large prompts into structured workflows  
- Maintain and share context across steps  
- Allow composition of subtasks into a final product  
- Keep parsing simple and LLM-independent on the compiler side  

## Input Format (Human-Written Prompt)

Each prompt consists of:

1. **Optional global context declaration**  
2. One or more **task blocks** using the strict format below  

---

### 1. Context Block (Optional)

```yaml
context:
  main: <context name>
  shared:
    - <shared_context_1>
    - <shared_context_2>
```

If omitted, `main` defaults to `app_spec`, and `shared` is inferred from task outputs.

---

### 2. Task Blocks

Each block must follow a strict structure:

```
---
task: <task_type>
desc: <short human-readable description>
prompt: <free-form input prompt for the task>
output: <output context name>
(optional) depends_on: <context name or comma-separated list>
(optional) output_file: <file path>
(optional) output_format: <md | json | txt | yaml>
(optional) output_context: <alias for output>
(optional) append_to_context: <context name to append output>
```

---

### Supported `task` types:

- `codegen`: Generate code or configurations from description  
- `synthesis`: Combine outputs of previous tasks into one artifact  
- `doc`: Generate documentation based on context  

---

## Full Example

```yaml
context:
  main: app_spec
  shared:
    - frontend
    - backend
    - schema

---
task: codegen
desc: Create a database schema for users and sessions
prompt: Design schema using raw SQL and PostgreSQL
output: schema
output_context: schema
append_to_context: app_spec

---
task: codegen
desc: Implement login API
prompt: Use Go with Gin to implement login endpoint
depends_on: schema
output: backend
output_context: backend
append_to_context: app_spec

---
task: codegen
desc: Create frontend login page and landing page
prompt: Use HTMX to create a login form and conditional landing page
depends_on: backend
output: frontend
output_context: frontend
append_to_context: app_spec

---
task: synthesis
desc: Combine all parts into a markdown output
prompt: Combine schema, backend, and frontend into documentation
depends_on: schema, backend, frontend
output_file: ./spec/output.md
output_format: md
output: app_doc
```

---

## Rules

1. All blocks must begin with `---`  
2. All fields except `depends_on`, `output_file`, `output_format`, `output_context`, and `append_to_context` are required  
3. Task blocks are evaluated in order  
4. The `output` becomes a named context available to future steps  
5. `output_context` and `append_to_context` control how results enter shared memory  
6. The final output block can aggregate multiple contexts  

---

## Example Usage with LLMs

When prompting ChatGPT or other models:

> "Please follow the prompt format described in [github.com/tiki-systems/declaim/blob/main/declaim.md](https://github.com/tiki-systems/declaim/blob/main/declaim.md), then transform the following application request into a Declaim spec: [your prompt]"

---

This allows the compiler to parse the output into deterministic, LLM-free YAML.

---

## Roadmap

- [ ] Support for templated prompt blocks  
- [ ] Support `type: workflow` for reusable subroutines  
- [ ] Validate structure using JSON Schema  
