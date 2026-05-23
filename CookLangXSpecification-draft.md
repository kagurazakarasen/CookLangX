# CookLangX Specification v0.1-draft

CookLangX is a domain specific language for representing cooking recipes as executable procedures.
This specification defines a practical hybrid model: YAML for structure and a compact DSL string for operation lines.

## 1. Goals

CookLangX has the following goals:

1. Human writable and machine readable recipes.
2. Explicit representation of dependencies and parallelizable work.
3. Reliable scaling of ingredient quantities by servings.
4. Interoperability with timers, shopping lists, nutrition analyzers, and future automation.

## 2. Design Principles

1. Recipe text is treated as operations and state transitions, not only prose.
2. A recipe is a directed acyclic graph (DAG) of steps.
3. Natural language notes may coexist with formal structure.
4. Core syntax remains small; advanced features are optional capability levels.

## 3. Conformance Terms

The key words MUST, MUST NOT, SHOULD, SHOULD NOT, and MAY are to be interpreted as described in RFC 2119 style usage.

## 4. File Format

1. Recommended extension: `.ckx`.
2. Encoding: UTF-8.
3. Canonical representation: YAML document.
4. MIME (proposed): `application/vnd.cooklangx+yaml`.

## 5. Document Model

A CookLangX document has these top-level sections:

1. `meta` (required)
2. `ingredients` (required)
3. `tools` (optional)
4. `definitions` (optional)
5. `steps` (required)

### 5.1 meta

`meta` MUST include:

1. `name` (string)
2. `servings` (number > 0)

`meta` MAY include:

1. `author` (string)
2. `lang` (BCP47 language tag, e.g. `ja`, `en-US`)
3. `tags` (string array)
4. `time_total` (duration)

### 5.2 ingredients

`ingredients` is an array of ingredient objects.

Each ingredient MUST include:

1. `id` (identifier)
2. `name` (string)

Each ingredient MAY include:

1. `amount` (number)
2. `unit` (unit string)
3. `scalable` (boolean, default true)
4. `state` (string, e.g. `raw`, `diced`)
5. `substitute` (array of ingredient ids)
6. `nutrition` (object)

### 5.3 tools

`tools` is an array of tool objects.

Each tool MUST include:

1. `id` (identifier)
2. `name` (string)

Each tool MAY include:

1. `capacity` (number or string)
2. `temperature_range` (object with `min` and/or `max`)

### 5.4 definitions

`definitions` is an array of reusable macro operations.

Each definition MUST include:

1. `id` (identifier)
2. `params` (string array, MAY be empty)
3. `body` (array of operation lines or step templates)

### 5.5 steps

`steps` is an ordered array for readability, but execution order is determined by dependencies.

Each step MUST include:

1. `id` (identifier)
2. `run` (DSL operation line)

Each step MAY include:

1. `uses` (array of ingredient refs and/or output refs)
2. `tools` (array of tool ids)
3. `time` (duration)
4. `temperature` (temperature)
5. `depends_on` (array of step ids)
6. `state_in` (object)
7. `state_out` (object)
8. `produces` (output identifier)
9. `yield` (object: amount and unit)
10. `when` (expression string; capability Extended)
11. `repeat` (repeat descriptor; capability Extended)
12. `note` (natural language note)

## 6. Identifier and Reference Rules

### 6.1 Identifier

Identifier regex:

```text
^[a-z][a-z0-9_]*$
```

Identifiers are case-sensitive.

### 6.2 References

1. Ingredient reference: `$<ingredient_id>`
2. Step output reference: `%<step_or_output_id>`
3. Tool reference inside DSL arguments: `&<tool_id>` (optional shorthand)

Examples:

```text
$spaghetti
%boiled_pasta
&frying_pan
```

## 7. Operation DSL

`run` contains one operation line.

### 7.1 Core Form

```text
<action> <arg>* [-> <output_id>]
```

Where:

1. `<action>` is an identifier.
2. `<arg>` is one of:
  - reference (`$id`, `%id`, `&id`)
  - key-value token (`key=value`)
  - quoted text (`"..."`)

Examples:

```text
boil $spaghetti water=2L salt=2g -> boiled_pasta
fry $bacon heat=medium -> crisp_bacon
mix %boiled_pasta %crisp_bacon $egg -> pasta_mix
```

### 7.2 Recommended Primitive Actions

Core action names SHOULD include:

1. `wash`
2. `cut`
3. `mix`
4. `boil`
5. `steam`
6. `fry`
7. `saute`
8. `bake`
9. `simmer`
10. `season`
11. `combine`
12. `rest`
13. `serve`

Implementations MAY support additional action names.

## 8. Time and Temperature

### 8.1 Duration

Duration lexical forms:

1. `30s`
2. `8min`
3. `1h`
4. ISO 8601 duration MAY be supported (`PT8M`).

Canonical normalized unit is seconds.

### 8.2 Temperature

Supported forms:

1. `180C`
2. `356F`
3. qualitative heat (`low`, `medium`, `high`)

If both numeric temperature and qualitative heat are present, numeric value takes precedence.

## 9. Units and Scaling

### 9.1 Unit Set

Recommended built-in units:

1. Mass: `mg`, `g`, `kg`
2. Volume: `ml`, `l`, `tsp`, `tbsp`, `cup`
3. Count: `piece`, `clove`, `leaf`

### 9.2 Scaling Rule

Given:

1. original servings $S_o$
2. target servings $S_t$

For each ingredient with `scalable=true`, scaled amount is:

$$
amount' = amount \times \frac{S_t}{S_o}
$$

If `scalable=false`, amount MUST remain unchanged.

Rounding policy SHOULD be configurable; default is half-up to 2 decimal places for non-count units.

## 10. Dependency and Execution Semantics

1. A step depends on explicit `depends_on`.
2. A step also has implicit dependencies from `%output` references in `run` or `uses`.
3. Final dependency graph MUST be acyclic.
4. Any two steps without path constraints MAY run in parallel.
5. A scheduler SHOULD prioritize critical path reduction while respecting tool conflicts.

### 10.1 Tool Conflict Rule

If two ready steps require the same exclusive tool instance, they MUST NOT execute simultaneously.

## 11. State Transition Model

Each step MAY define `state_in` and `state_out`.

Example:

```yaml
state_in:
  onion: raw
state_out:
  onion: translucent
```

When state constraints are present:

1. `state_in` SHOULD be validated against prior producing steps.
2. unresolved required states SHOULD raise warnings or errors by strictness mode.

## 12. Natural Language Coexistence

`note` allows retaining human-friendly hints.

Implementations:

1. MUST ignore `note` for dependency semantics.
2. MAY show `note` in UI, voice guidance, and print output.

## 13. Capability Levels

### 13.1 Core

A Core implementation MUST support:

1. Sections in Chapter 5.
2. DSL in 7.1.
3. Time in 8.1.
4. Scaling in Chapter 9.
5. DAG validation in Chapter 10.

### 13.2 Extended

Extended adds:

1. `when` conditional step activation.
2. `repeat` loops.
3. `definitions` macro expansion.
4. nutrition and shopping-list integration metadata.

## 14. Validation Rules

A conforming validator SHOULD report machine-readable error codes.

Recommended error set:

1. `CKX001`: missing required section.
2. `CKX002`: invalid identifier format.
3. `CKX003`: duplicate identifier.
4. `CKX004`: unknown ingredient/tool/output reference.
5. `CKX005`: dependency cycle detected.
6. `CKX006`: invalid duration format.
7. `CKX007`: invalid temperature format.
8. `CKX008`: non-positive servings.
9. `CKX009`: unsupported unit.
10. `CKX010`: macro expansion failure.

## 15. Canonical Example (Carbonara)

```yaml
meta:
  name: Carbonara
  servings: 2
  lang: ja
  spec_version: 0.1

ingredients:
  - id: spaghetti
    name: Spaghetti
    amount: 200
    unit: g
  - id: egg
    name: Egg
    amount: 2
    unit: piece
  - id: bacon
    name: Bacon
    amount: 80
    unit: g
  - id: salt
    name: Salt
    amount: 2
    unit: g

tools:
  - id: pot
    name: Pot
  - id: frying_pan
    name: Frying Pan

steps:
  - id: boil_pasta
    run: boil $spaghetti water=2L salt=2g -> boiled_pasta
    tools: [pot]
    time: 8min
    temperature: 100C

  - id: cook_bacon
    run: fry $bacon heat=medium -> crisp_bacon
    tools: [frying_pan]
    time: 4min

  - id: make_egg_mix
    run: mix $egg -> egg_mix
    time: 1min

  - id: combine
    run: combine %boiled_pasta %crisp_bacon %egg_mix -> carbonara
    depends_on: [boil_pasta, cook_bacon, make_egg_mix]
    note: Keep heat low to avoid scrambling egg.

  - id: plate
    run: serve %carbonara
    depends_on: [combine]
```

## 16. Informative Internal Pipeline

Typical processing pipeline:

1. Parse YAML.
2. Parse DSL lines into AST nodes.
3. Resolve symbol table (ingredients, tools, outputs, step ids).
4. Build dependency DAG.
5. Validate constraints.
6. Emit execution plan, timers, and derived artifacts.

## 17. JSON Interchange Mapping (Informative)

Implementations MAY expose normalized JSON with the same semantic fields.

Recommended minimal fields:

1. `meta`
2. `ingredients`
3. `tools`
4. `steps`
5. `graph` (`nodes`, `edges`)

## 18. Versioning and Forward Compatibility

1. Document SHOULD include `meta.spec_version` (e.g. `0.1`).
2. Unknown fields MUST be ignored unless strict mode is enabled.
3. Breaking grammar changes MUST increment major version.

## 19. Security and Safety Considerations

1. This format does not guarantee food safety.
2. Implementations SHOULD allow warnings for undercooking risk patterns.
3. Tool and temperature instructions SHOULD be validated before automatic device control.

## 20. Open Items for v0.2

1. Formal EBNF for DSL.
2. Standard action ontology and multilingual aliases.
3. Allergen and nutrition profile schema.
4. Deterministic scheduler policy profiles.
5. IoT command binding profile for smart appliances.

