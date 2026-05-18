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

 
