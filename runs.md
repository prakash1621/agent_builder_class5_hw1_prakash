# Agent Runs

## Run 1 — Happy Path Question

### Question

How many launches were successful?

### Tool Calls

#### Tool: query_launches

```sql
SELECT COUNT(*) AS successful_launches
FROM launches
WHERE success = 1
```

### Observation

```json
[
  {
    "successful_launches": 14
  }
]
```

### Final Answer

14 launches were successful.

---

## Run 2 — Multi-Tool Question

### Question

What is the thrust of the rocket we use for crewed missions?

### Tool Calls

#### Tool 1: query_launches

```sql
SELECT DISTINCT vehicle
FROM launches
WHERE payload_type = 'Crew'
```

### Observation

```json
[
  {
    "vehicle": "Falcon 9"
  }
]
```

#### Tool 2: vehicle_specs

```json
{
  "vehicle": "Falcon 9"
}
```

### Observation

```json
{
  "thrust_kn": 7607,
  "height_m": 70,
  "reusable": true
}
```

### Final Answer

Falcon 9 is the rocket used for crewed missions and it has 7607 kN of thrust.

---

## Run 3 — Ambiguous Question

### Question

Tell me about heavy launches.

### Tool Calls

#### Tool: query_launches

```sql
SELECT *
FROM launches
WHERE vehicle = 'Falcon Heavy'
```

### Observation

```json
[
  {
    "mission_name": "Arabsat-6A",
    "vehicle": "Falcon Heavy"
  },
  {
    "mission_name": "USSF-44",
    "vehicle": "Falcon Heavy"
  }
]
```

### Agent Behavior Observation

The agent interpreted “heavy launches” as launches using the Falcon Heavy rocket instead of using payload weight as the definition.

### Final Answer

The agent returned Falcon Heavy missions instead of interpreting heavy launches as payload-based launches.

---

## Run 4 — Failure Mode Question

### Question

What is the diameter of the Saturn V rocket?

### Tool Calls

#### Tool: vehicle_specs

```json
{
  "vehicle": "Saturn V"
}
```

### Observation

```text
UNKNOWN VEHICLE: 'Saturn V'
```

### Agent Behavior Observation

The available tools did not contain Saturn V data. The agent attempted to use the vehicle_specs tool but could not retrieve valid information.

### Final Answer

The agent could not find information about the Saturn V rocket in the available tools.

---

## Run 5 — Custom Question

### Question

Compare Falcon Heavy and Starship. Which has higher thrust?

### Tool Calls

#### Tool 1: vehicle_specs

```json
{
  "vehicle": "Falcon Heavy"
}
```

### Observation

```json
{
  "thrust_kn": 22819
}
```

#### Tool 2: vehicle_specs

```json
{
  "vehicle": "Starship"
}
```

### Observation

```json
{
  "thrust_kn": 74000
}
```

### Final Answer

Starship has significantly higher thrust than Falcon Heavy. Falcon Heavy produces 22,819 kN while Starship produces 74,000 kN of thrust.

---

# Overall Learnings

 1) 
 One of the most interesting things I observed  tool descriptions affected tool selection.

Initially, I created a `compare()` tool with a simple description explaining what the tool returned. The agent ignored the tool and performed comparisons directly in natural language. for example 

@tool
def compare(a: str, b: str, label: str) -> str:
    """Compare two values and return a formatted comparison string."""
    return f"{label}: a={a}, b={b}"

 this tool is not triggered after running the agent

After I changed the  to  “Use this tool whenever the user asks to compare two values,” the agent started calling the tool correctly. This helped me understand that tool descriptions are effectively prompts that guide agent behavior.
like 
def compare(a: str, b: str, label: str) -> str:
    """Use this tool whenever the user asks to compare two values."""
    return f"{label}: a={a}, b={b}"

 this tool is   triggered after running the agent

2) Next time I would like to  add better hallucination handling and retry logic when tools return unknown values  . I noticed that the agent sometimes continued reasoning even after receiving insufficient information from the tools.

3) Pipeline vs Agent
## Pipeline vs Agent

The pipeline worked better for simple SQL questions because the steps were fixed and easy to follow. It was easier to debug  

The agent worked much better when the question needed multiple tools or multiple data sources. For example, for the question “What is the thrust of the rocket we use for crewed missions?”, the agent first queried the launches database to find the rocket name and then used the JSON specs tool to get the thrust value.

I also understood that agents need strong tool descriptions and a good system prompt to make correct tool-calling decisions. When I first added the `compare()` tool, the agent ignored it because the description only explained what the tool returns.

After I changed the description to clearly say when the tool should be used, the agent started calling it correctly. This showed me that tool descriptions work almost like prompts and strongly affect agent behavior.
