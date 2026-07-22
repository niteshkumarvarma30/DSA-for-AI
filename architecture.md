# Updated Workflow Architecture

> Current architecture note: the active runtime JSON architecture is documented in [runtime_json_architecture_workflow.md](runtime_json_architecture_workflow.md). The older notes below describe the earlier structured-rule migration.

## Current Problem

The current system sends user constraints directly to Agent 1.

Agent 1 then writes Python code with Z3 constraints.

This works only if Agent 1 correctly translates every user rule into mathematical constraints.

For example:

```text
Kitchen should be southeast.
```

Z3 cannot understand this sentence directly. Z3 only understands mathematical constraints such as:

```python
solver.add(kitchen_x1 + kitchen_x2 > master_x1 + master_x2)
solver.add(kitchen_y1 + kitchen_y2 < living_y1 + living_y2)
```

So if Agent 1 forgets or incorrectly translates the rule, Z3 may still return a layout, and the current Python verifier may still pass it because custom semantic rules are not independently re-checked.

## Current Stage 3 Full-Utilization Issue

The first-floor pipeline was validating room correctness, but it was not
requiring exact rooftop/buildable-area coverage.

Root causes:

- `z3_solver_config.json` had `require_full_coverage: false`, so final
  verification explicitly skipped the coverage check.
- `system_constraints.json` did not clearly state that the target floor must
  use the full reference/buildable footprint.
- The final coverage check compared summed room rectangle area to footprint
  area. That can report a good number while still missing visible corner
  polygons, because it is not a polygon union/difference check.
- The Stage 3 model still stores rooms as `[x1, y1, x2, y2]` rectangles. For
  arbitrary rooftop shapes, the pipeline must either support polygon rooms or
  decompose uncovered buildable polygons into filler rectangles.

Implemented fix direction:

```text
1. Set require_full_coverage = true in solver/runtime policy.
2. Make system policy explicit:
   target_floor_must_use_full_reference_footprint = true.
3. Verify final coverage with exact geometry:
   uncovered = buildable_polygon - union(first_floor_room_rectangles)
   overflow = union(first_floor_room_rectangles) - buildable_polygon
4. Reject final layouts when uncovered or overflow area exceeds tolerance.
5. Add deterministic Stage 3 coverage-fill zones for any uncovered buildable
   cells after the requested rooms are solved and expanded.
```

This keeps the existing Z3/entity/system architecture, but makes Z3 output
subject to a deterministic geometry pass that guarantees full utilization of
the buildable first-floor footprint.

## Dynamic Floor-To-Floor Runtime Contract

The pipeline is intended to be dynamic at runtime. It should not hardcode a
single ground-floor shape or a fixed first-floor layout. Each floor generation
run must consume the current runtime files:

- `system_constraints.json`: global floor policy, full-coverage policy,
  staircase continuity, wet-stack policy, and allowed solver behavior.
- `entity_constraints.json`: room-type constraints such as minimum size,
  ventilation, door/window requirements, and type-level relation rules.
- `user_layout_constraints.json`: target floor, requested rooms, and
  user-authored structured rules.
- `z3_solver_config.json`: Z3 runtime options, including
  `require_full_coverage`.
- `data/floor_dna.json`: the accumulated building state, including the plot,
  footprint, structural grid, ground-floor rooms, first-floor rooms, wet-room
  anchors, and constraints.

Runtime behavior:

```text
1. Load current floor DNA and runtime constraint JSON.
2. Build effective rules from user + entity + system constraints.
3. Solve requested rooms with Z3 inside the current reference footprint.
4. Expand solved rooms inside the legal buildable polygon.
5. Add deterministic coverage-fill zones for any uncovered buildable cells.
6. Verify exact full coverage with polygon union/difference.
7. Export DXF and Blender files from the verified geometry.
8. Persist the generated floor back into floor DNA.
9. Use that updated floor DNA as input for the next floor.
```

For the current run:

```text
Ground floor input     -> data/floor_dna.json floors.ground
Generated first floor  -> data/floor_dna.json floors.first
Second-floor handoff   -> data/second_floor_input.json
```

The second-floor generator should use `floors.first` as the immediate lower
reference floor, while still keeping `floors.ground` available for structural,
staircase, and wet-stack continuity checks.

## New Recommended Architecture

The better architecture is:

```text
User natural-language constraints
   ↓
Semantic parser / LLM extracts structured rules
   ↓
Python backend validates structured rules against schema
   ↓
Python backend deterministically compiles rules into Z3 constraints
   ↓
Z3 solver checks satisfiability and returns layout coordinates
   ↓
Python backend verifies final coordinates against the same structured rules
   ↓
Frontend displays verified layout
```

The key change is:

> Agent 1 should not directly generate arbitrary Z3 code from natural language.

Instead:

> Agent 1 or another LLM should only extract structured JSON rules. Python code should compile those rules into Z3 constraints.

## Main Components

### 1. User Constraint Input

The user can still write normal text:

```text
Kitchen should be southeast.
Kitchen must be at least 10x10.
Bathroom should not touch Kitchen.
Living Room should be northeast.
```

This text is not sent directly to Z3.

It first goes through a semantic parsing step.

### 2. Semantic Parser / LLM Rule Extractor

The semantic parser converts natural language into structured JSON.

Example output:

```json
[
  {
    "type": "zone",
    "room": "Kitchen",
    "zone": "southeast"
  },
  {
    "type": "min_size",
    "room": "Kitchen",
    "width": 10,
    "height": 10
  },
  {
    "type": "not_touch",
    "room_a": "Bathroom",
    "room_b": "Kitchen"
  },
  {
    "type": "zone",
    "room": "Living Room",
    "zone": "northeast"
  }
]
```

The LLM is used only to understand language and produce structured data.

It should not be responsible for final mathematical correctness.

### 3. Rule Schema

A schema is the fixed format that structured rules must follow.

It defines:

- allowed rule types
- required fields for each rule type
- allowed values
- expected data types

Example schema ideas:

```text
zone rule:
  type: "zone"
  room: string
  zone: north | south | east | west | northeast | northwest | southeast | southwest

min_size rule:
  type: "min_size"
  room: string
  width: positive number
  height: positive number

touch rule:
  type: "touch"
  room_a: string
  room_b: string

not_touch rule:
  type: "not_touch"
  room_a: string
  room_b: string

relative_position rule:
  type: "relative_position"
  room_a: string
  relation: north_of | south_of | east_of | west_of
  room_b: string
```

Invalid rule example:

```json
{
  "type": "zone",
  "room": "Kitchen",
  "zone": "sky"
}
```

This fails validation because `"sky"` is not an allowed zone.

### 4. Python Rule Validation

The Python backend validates structured rules before Z3 sees them.

Validation checks:

- rule has a valid `type`
- required fields are present
- values are allowed
- dimensions are positive numbers
- room names are not empty
- contradictory rules are detected where possible

Example:

```python
ALLOWED_ZONES = {
    "north", "south", "east", "west",
    "northeast", "northwest", "southeast", "southwest"
}

def validate_rule(rule):
    if "type" not in rule:
        return False, "Missing rule type"

    if rule["type"] == "zone":
        if "room" not in rule:
            return False, "Zone rule missing room"
        if "zone" not in rule:
            return False, "Zone rule missing zone"
        if rule["zone"] not in ALLOWED_ZONES:
            return False, f"Invalid zone: {rule['zone']}"
        return True, "OK"

    if rule["type"] == "min_size":
        if "room" not in rule or "width" not in rule or "height" not in rule:
            return False, "Min-size rule requires room, width, height"
        if rule["width"] <= 0 or rule["height"] <= 0:
            return False, "Room size must be positive"
        return True, "OK"

    return False, f"Unknown rule type: {rule['type']}"
```

This step prevents unclear or invalid rules from reaching the solver.

### 5. Deterministic Z3 Constraint Compiler

After validation, Python converts each structured rule into Z3 constraints.

This should be deterministic code, not generated free-form by Agent 1.

Example:

```python
def compile_rule_to_z3(rule, rooms, solver, context):
    if rule["type"] == "min_size":
        r = rooms[rule["room"]]
        solver.add(r["x2"] - r["x1"] >= rule["width"])
        solver.add(r["y2"] - r["y1"] >= rule["height"])

    elif rule["type"] == "zone":
        r = rooms[rule["room"]]
        mid_x = context["mid_x"]
        mid_y = context["mid_y"]

        if "east" in rule["zone"]:
            solver.add(r["x1"] + r["x2"] >= 2 * mid_x)
        if "west" in rule["zone"]:
            solver.add(r["x1"] + r["x2"] <= 2 * mid_x)
        if "north" in rule["zone"]:
            solver.add(r["y1"] + r["y2"] >= 2 * mid_y)
        if "south" in rule["zone"]:
            solver.add(r["y1"] + r["y2"] <= 2 * mid_y)
```

For:

```json
{
  "type": "zone",
  "room": "Kitchen",
  "zone": "southeast"
}
```

The compiler creates:

```python
solver.add(kitchen_x1 + kitchen_x2 >= 2 * mid_x)
solver.add(kitchen_y1 + kitchen_y2 <= 2 * mid_y)
```

Now "Kitchen should be southeast" is a real mathematical constraint.

### 6. Z3 Solver

Z3 receives only mathematical constraints.

It does not receive natural-language text.

Z3 checks whether all constraints can be true together.

Possible results:

```text
sat      = a valid layout exists
unsat    = no layout can satisfy all constraints
unknown  = solver could not decide within limits
```

If the result is `sat`, Z3 returns a model with coordinate values:

```python
Kitchen = [x1, y1, x2, y2]
Living Room = [x1, y1, x2, y2]
Bathroom = [x1, y1, x2, y2]
```

### 7. Python Post-Verification

After Z3 returns coordinates, Python verifies the final layout against the same structured rules.

This is important because it creates a second safety layer.

Example:

```python
def verify_rule(rule, solved_rooms, context):
    if rule["type"] == "zone":
        room = solved_rooms[rule["room"]]
        cx = room[0] + room[2]
        cy = room[1] + room[3]

        if "east" in rule["zone"] and cx < 2 * context["mid_x"]:
            return False, f"{rule['room']} is not east enough"

        if "south" in rule["zone"] and cy > 2 * context["mid_y"]:
            return False, f"{rule['room']} is not south enough"

    return True, "OK"
```

For "Kitchen should be southeast", Python verifies:

```python
kitchen_center_x >= footprint_mid_x
kitchen_center_y <= footprint_mid_y
```

This means the rule is checked after solving, not only trusted from Agent 1.

### 8. Geometry Verification

The existing geometry checks should remain.

Python should still verify:

- every room has `[x1, y1, x2, y2]`
- `x1 < x2`
- `y1 < y2`
- rooms are inside footprint
- rooms do not overlap
- rooms do not enter exclusion zones
- upper floor does not overlap staircase shaft
- grid snapping is respected if structural grid is enabled

### 9. Agent 2 / Repair Loop

Agent 2 can still be useful, but its role should change.

Instead of only debugging generated Z3 code, Agent 2 can help with:

- ambiguous user constraints
- invalid structured rules
- conflicting rules
- explaining why Z3 returned `unsat`
- suggesting which constraints to relax

Example:

```text
No valid layout exists because:
- Kitchen must be southeast.
- Kitchen must touch Bathroom.
- Bathroom must not touch Kitchen.
These rules conflict.
```

### 10. Recommended File Structure

Suggested new modules:

```text
rule_parser.py       # LLM call that converts natural language to JSON rules
rule_schema.py       # allowed rule models and schema definitions
rule_validator.py    # validates structured rules
rule_compiler.py     # converts structured rules into Z3 constraints
rule_verifier.py     # verifies solved coordinates against structured rules
layout_solver.py     # owns Z3 solver setup and model extraction
```

Current files can then evolve like this:

```text
prompt.py
  old role: ask Agent 1 to write full Z3 code
  new role: ask LLM to produce structured JSON rules only

z3_verifier.py
  old role: hardcoded default checks plus generic geometry
  new role: generic geometry + structured rule verification

main.py
  old role: execute Agent 1 generated Python code
  new role: parse rules, validate rules, compile rules, solve, verify
```

## Final Target Flow

```text
1. User draws plot and enters natural-language constraints.

2. Backend calculates footprint and structural grid.

3. LLM extracts structured rules from user text.

4. Python validates structured rules against schema.

5. Python creates room variables and universal geometry constraints.

6. Python compiles every structured rule into Z3 constraints.

7. Z3 checks satisfiability.

8. If sat, Python extracts layout coordinates.

9. Python verifies:
   - geometry
   - overlap
   - footprint containment
   - exclusion zones
   - all structured user rules

10. If verification passes, layout is shown/exported.

11. If verification fails or Z3 is unsat, Agent 2 explains the issue and suggests constraint relaxation.
```

## Why This Is Better

This approach makes semantic rules reliable.

In the old system:

```text
"Kitchen should be southeast" depends on Agent 1 writing correct Z3 code.
```

In the new system:

```text
"Kitchen should be southeast" becomes a stored JSON rule.
The Python compiler turns it into Z3.
The Python verifier checks it again after solving.
```

This gives:

- more reliable custom constraints
- easier debugging
- clearer user feedback
- less dependence on arbitrary generated code
- safer solver execution
- better separation between language understanding and mathematical solving
