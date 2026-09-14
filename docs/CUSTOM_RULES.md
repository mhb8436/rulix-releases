# Writing custom rules

This document explains how to add new rules by editing YAML alone, with no change to the program.

## Overview

RULIX uses a YAML-based ruleset system. For most pattern types you can add a rule by editing YAML, without touching Go code.

### Supported pattern types

| Pattern type | What it does | YAML only |
|----------|------|----------------|
| `regex` | Regex match, one line at a time | ✅ |
| `regex-multiline` | Regex match over the whole file | ✅ |
| `annotation-missing-attr` | Required annotation attributes | ✅ |
| `ast-method-call` | Method calls, from the AST | ✅ |
| `ast-annotation` | Annotations, from the AST | ✅ |
| `ast-import` | Imports, from the AST | ✅ |
| `ast-variable` | Variables and fields, from the AST | ✅ |
| `ast-try-catch` | Exception handling, from the AST | ✅ |
| `ast-class` | Class shape, from the AST | ✅ |
| `ast-method` | Method shape, from the AST | ✅ |
| `ast-multi-cud` | Several create/update/delete calls together | ✅ |
| `ast-dead-code` | Dead code, from the AST | ✅ |
| `ast-filtered-regex` | Regex with comments filtered out (ANTLR4 token filter) | ✅ |
| `ast-multi-cud` | Multi-row CUD transaction checks | ✅ |
| `cross-file` | Cross-file analysis (for example, unused MyBatis SQL IDs) | ✅ |
| `method-analysis` | Deeper method and AST analysis | ❌ needs Go code |

## Where the ruleset files live

```
configs/
├── profiles.yaml          # Profile definitions & groups
└── rulesets/
    ├── quality.yaml       # Code quality (45+ rules + 8 legacy rules)
    ├── secure.yaml        # Secure coding (80+ rules)
    ├── sql.yaml           # SQL style guide (89 rules)
    ├── modernize.yaml     # Legacy modernization (50 rules)
    ├── spring.yaml        # Spring development standards (25 rules)
    └── egov.yaml          # eGovernment framework standards (33 rules)
```

### Profile groups

```bash
# individual profiles
rulix ./src --profile=quality
rulix ./src --profile=modernize
rulix ./src --profile=spring,egov

# groups (several profiles at once)
rulix ./src --profile=all            # quality + secure + sql + modernize + spring + egov
rulix ./src --profile=essential      # secure + quality
rulix ./src --profile=egov-full      # quality + secure + sql + egov + spring
rulix ./src --profile=migration      # modernize + spring
```

### The rulesets at a glance

| Ruleset | Rules | Applies to | What it covers |
|-------|---------|------|------|
| `quality` | 53+ | Java, JS, HTML, CSS | code quality, naming, performance, legacy APIs |
| `secure` | 80+ | Java, JS | SQL injection, XSS, crypto, authentication and authorization |
| `sql` | 89 | SQL, Java | SQL style, performance, bind variables, DML |
| `modernize` | 50 | Java, XML | eGov 3.x→4.x, javax→jakarta, iBatis→MyBatis |
| `spring` | 25 | Java | DI patterns, @Transactional, controllers, REST APIs |
| `egov` | 33 | Java, XML | eGovernment layer naming, Service/Mapper standards, CRUD prefixes |

## What a rule looks like

### The basic YAML

```yaml
version: "1.0"
profile: "profile-name"

languages:
  - language: java    # java, javascript, html, css, sql, xml
    rules:
      - id: "rule-id"
        name: "Name shown to the user"
        severity: "high"
        category: "category"
        description: "What the rule is about"
        enabled: true
        pattern:
          type: "regex"
          regex: "the-regex"
        custom:
          fix: "how to fix it"
        exclude:
          - "pattern-to-skip"
```

### Required fields

| Field | Meaning | Example |
|------|------|------|
| `id` | A unique rule ID | `"custom-debug-code"` |
| `name` | The rule name shown to the user | `"Debug code found"` |
| `severity` | Severity | `low`, `medium`, `high`, `critical` |
| `category` | Category | `naming`, `security`, `style`, … |
| `description` | The longer explanation | `"Debug code left in production"` |
| `enabled` | Whether it runs | `true` or `false` |
| `pattern.type` | Pattern type | `regex`, `ast-method-call`, … |

### Optional fields

| Field | Meaning |
|------|------|
| `custom.fix` | The suggested fix (shown with 💡 in the report) |
| `custom.example` | An example of the correct code |
| `custom.flags` | Regex flags (`"i"` = case-insensitive) |
| `exclude[]` | Patterns to skip (regex types only) |

### Severity levels

| Severity | Meaning |
|--------|------|
| `critical` | Fix now (a security hole, or a mandatory rule broken) |
| `high` | Should be fixed before release |
| `medium` | Worth improving over time |
| `low` | A recommendation |

---

## Part 1: regex rules

### type: regex

Matches a regex against each line on its own.

```yaml
- id: "custom-system-out"
  name: "System.out prohibited"
  severity: "medium"
  category: "logging"
  description: "Use a logger instead of System.out"
  enabled: true
  pattern:
    type: "regex"
    regex: "System\\.out\\."
  custom:
    fix: "Change it to logger.info() or logger.debug()"
```

### type: regex-multiline

Treats the whole file as one string, so a pattern can span lines.

```yaml
- id: "custom-loop-string-concat"
  name: "String += inside a loop"
  severity: "medium"
  category: "performance"
  description: "String += in a loop is slow"
  enabled: true
  pattern:
    type: "regex-multiline"
    regex: "(for|while)\\s*\\([^)]*\\)\\s*\\{[^}]*\\w+\\s*\\+=\\s*\"[^\"]*\""
  custom:
    fix: "Use a StringBuilder"
```

### type: annotation-missing-attr

Reports a violation when a Java annotation is missing a required attribute.

```yaml
- id: "custom-cacheable-key"
  name: "@Cacheable requires key"
  severity: "high"
  category: "annotation"
  description: "@Cacheable must specify key"
  enabled: true
  pattern:
    type: "annotation-missing-attr"
    annotation: "@Cacheable"
    required_attr: "key"
  custom:
    fix: "Add the key attribute"
    example: "@Cacheable(value = \"users\", key = \"#id\")"
```

---

## Part 2: AST query rules (Java only)

AST query rules read the syntax tree of the Java source to find patterns a regex cannot express. They are **written in YAML alone**, on top of the structure the ANTLR4 parser extracts.

### Where an AST query beats a regex

| Situation | What a regex cannot do | What the AST query gives you |
|------|-------------|----------------|
| A database call inside a loop | the multiline pattern is complex and misfires often | `context: "inside-loop"` — one line, exact |
| An empty catch block | `catch.*{}` depends on formatting | the AST says whether the statement count is zero |
| Methods per class | not expressible as a regex | `max_methods: 20` and you are done |
| Filtering imports | a line gives no context | `class_annotation` narrows the scope |
| Annotation attribute values | parsing key=value with a regex is painful | `missing_attr_name` plus `attr_value` |

### Shared field: `context`

The context conditions AST query rules share:

| `context` value | Meaning | Usable with |
|------------|------|-----------------|
| `inside-loop` | inside a for/while/do-while loop | ast-method-call, ast-variable |
| `inside-catch` | inside a catch block | ast-method-call, ast-variable |
| `not-inside-catch` | outside any catch block | ast-method-call, ast-variable |
| `method-name:<regex>` | inside a method whose name matches | ast-method-call, ast-annotation |
| `class-field` | a class field, not a local variable | ast-variable |
| *(omitted)* | no condition — anywhere | all types |

### Shared field: the `custom` map

| Key | Type | Meaning |
|----|------|------|
| `fix` | string | the suggested fix (shown with 💡 in the report) |
| `args_min` | int | minimum argument count (ast-method-call) |
| `args_max` | int | maximum argument count (ast-method-call) |
| `max_methods` | int | maximum methods (ast-class) |
| `max_fields` | int | maximum fields (ast-class) |
| `catch_is_empty` | bool | whether the catch block is empty (ast-try-catch) |
| `has_finally` | bool | whether there is a finally block (ast-try-catch) |
| `has_resources` | bool | whether try-with-resources is used (ast-try-catch) |
| `max_overloads` | int | maximum overloads of one name (ast-class) |
| `max_inner_classes` | int | maximum inner classes (ast-class) |
| `max_imports` | int | maximum imports (ast-import) |
| `all_fields_type` | string | a violation when every field has this type (ast-class) |
| `check_type` | string | which dead-code check to run (ast-dead-code) |
| `max_try_lines` | int | maximum lines in a try block (ast-try-catch) |
| `max_if_depth` | int | maximum if nesting depth (ast-method) |
| `max_branches` | int | maximum if-else-if branches (ast-method) |
| `max_lines` | int | maximum lines in a method (ast-method) |
| `max_statements` | int | maximum statements in a method (ast-method) |

---

### 2.1 ast-method-call — method call patterns

Finds particular method calls in Java source. You filter by the object called (qualifier), the method name, the argument count and where the call sits (context).

#### Pattern fields

| Field | Required | Meaning | Example |
|------|------|------|------|
| `method` | ✅ | regex for the method name | `"^(select\|find).*"` |
| `qualifier` | - | regex for the object called | `".*(?i)(dao\|mapper)"` |
| `context` | - | where the call must sit | `"inside-loop"` |

#### Example: a database read inside a loop (the N+1 problem)

```yaml
- id: "quality-ast-001"
  name: "Database read inside a loop"
  severity: "high"
  category: "performance"
  description: "Reading from the database inside a loop causes the N+1 problem"
  enabled: true
  pattern:
    type: "ast-method-call"
    method: "^(select|find|get|query|search|retrieve).*"
    qualifier: ".*(?i)(dao|repository|mapper)"
    context: "inside-loop"
  custom:
    fix: "Use a JOIN, or batch the reads with an IN clause"
```

What it catches:
```java
for (String id : idList) {
    UserVO user = userMapper.selectUserById(id);  // ← violation
    results.add(user);
}
```

#### Example: INSERT/UPDATE inside a loop

```yaml
- id: "quality-ast-002"
  name: "Database write inside a loop"
  severity: "high"
  category: "performance"
  description: "Calling INSERT/UPDATE/DELETE one row at a time inside a loop is slow"
  enabled: true
  pattern:
    type: "ast-method-call"
    method: "^(insert|update|delete|save|merge).*"
    qualifier: ".*(?i)(dao|repository|mapper)"
    context: "inside-loop"
  custom:
    fix: "Use a batch insert/update, or addBatch()/executeBatch()"
```

#### Example: an external API call inside a loop

```yaml
- id: "quality-ast-003"
  name: "Remote API call inside a loop"
  severity: "high"
  category: "performance"
  description: "Calling an external API inside a loop adds up the latency"
  enabled: true
  pattern:
    type: "ast-method-call"
    method: "^(exchange|execute|getForObject|postForObject|send|call).*"
    qualifier: ".*(?i)(restTemplate|webClient|httpClient|feignClient)"
    context: "inside-loop"
  custom:
    fix: "Use a bulk API call, or run them in parallel with CompletableFuture"
```

#### Example: limiting the argument count

```yaml
- id: "custom-too-many-args"
  name: "Too many arguments"
  severity: "medium"
  category: "design"
  description: "More than six arguments. Use a parameter object."
  enabled: true
  pattern:
    type: "ast-method-call"
    method: ".*"
  custom:
    args_min: 7
    fix: "Group the parameters into a DTO or VO"
```

---

### 2.2 ast-annotation — annotation checks

Checks, at the AST level, whether an annotation is present, whether it has an attribute, and what that attribute says.

#### Pattern fields

| Field | Required | Meaning | Example |
|------|------|------|------|
| `annotation` | ✅ | regex for the annotation name | `"^RequestMapping$"` |
| `missing_attr_name` | - | a violation when this attribute is absent | `"method"` |
| `attr_value` | - | a violation when the value does not match this regex | `".*readOnly.*true.*"` |
| `context` | - | context condition | `"method-name:.*"` |

#### Example: @RequestMapping with no method attribute (method level only)

```yaml
- id: "quality-ast-011"
  name: "@RequestMapping has no method attribute"
  severity: "medium"
  category: "design"
  description: "Without a method attribute, @RequestMapping accepts every HTTP method"
  enabled: true
  pattern:
    type: "ast-annotation"
    annotation: "^RequestMapping$"
    missing_attr_name: "method"
    context: "method-name:.*"
  custom:
    fix: "Use a dedicated annotation: @GetMapping, @PostMapping and so on"
```

> **Note**: `context: "method-name:.*"` means "inside any method at all", so a class-level `@RequestMapping` is excluded automatically.

#### Example: checking @Transactional readOnly

```yaml
- id: "custom-transactional-readonly"
  name: "Read method without readOnly"
  severity: "medium"
  category: "performance"
  description: "@Transactional on a read method does not set readOnly=true"
  enabled: true
  pattern:
    type: "ast-annotation"
    annotation: "^Transactional$"
    missing_attr_name: "readOnly"
    context: "method-name:^(select|find|get|query|search).*"
  custom:
    fix: "Add @Transactional(readOnly = true)"
```

---

### 2.3 ast-import — import checks

Pulls the Java import statements out of the AST to check for banned or preferred libraries.

#### Pattern fields

| Field | Required | Meaning | Example |
|------|------|------|------|
| `import` | ✅ | regex for the import path | `"^java\\.util\\.Date$"` |
| `class_annotation` | - | only check classes carrying this annotation | `"Controller"` |

#### Example: banning java.util.Date

```yaml
- id: "quality-ast-004"
  name: "java.util.Date import"
  severity: "medium"
  category: "deprecated"
  description: "java.util.Date is not thread-safe. Use the java.time API."
  enabled: true
  pattern:
    type: "ast-import"
    import: "^java\\.util\\.Date$"
  custom:
    fix: "Use java.time.LocalDate, LocalDateTime, ZonedDateTime and friends"
```

#### Example: banning the old commons-lang

```yaml
- id: "quality-ast-005"
  name: "Old commons-lang import"
  severity: "medium"
  category: "deprecated"
  description: "org.apache.commons.lang is end-of-life. Use commons-lang3."
  enabled: true
  pattern:
    type: "ast-import"
    import: "^org\\.apache\\.commons\\.lang\\."
  custom:
    fix: "Migrate to the org.apache.commons.lang3 package"
```

#### Example: banning an import in controllers only

```yaml
- id: "custom-controller-jdbc"
  name: "JDBC used directly in a Controller"
  severity: "high"
  category: "architecture"
  description: "Using JDBC directly in a Controller breaks the layering"
  enabled: true
  pattern:
    type: "ast-import"
    import: "^java\\.sql\\."
    class_annotation: "Controller|RestController"
  custom:
    fix: "Reach the data through the Service layer"
```

---

### 2.4 ast-variable — variable and field checks

Checks the name and type of a local variable or a class field, from the AST.

#### Pattern fields

| Field | Required | Meaning | Example |
|------|------|------|------|
| `name_pattern` | - | regex for the variable name | `"^[a-z]$"` |
| `type_pattern` | - | regex for the type name | `"^Map<String,\\s*Object>$"` |
| `context` | - | `"class-field"`, or a position condition | `"class-field"` |

#### Example: single-letter variable names

```yaml
- id: "custom-single-char-var"
  name: "Single-letter variable name"
  severity: "medium"
  category: "naming"
  description: "Use a name that means something (i, j, k excepted)"
  enabled: true
  pattern:
    type: "ast-variable"
    name_pattern: "^[a-hlo-zA-Z]$"
  custom:
    fix: "Name it after what it holds"
```

#### Example: a Map<String, Object> class field

```yaml
- id: "custom-field-map-object"
  name: "Map<String, Object> as a field type"
  severity: "medium"
  category: "design"
  description: "A Map<String, Object> field gives up type safety"
  enabled: true
  pattern:
    type: "ast-variable"
    type_pattern: "Map<String,Object>"
    context: "class-field"
  custom:
    fix: "Define a proper VO or DTO class"
```

---

### 2.5 ast-try-catch — exception handling checks

Reads the shape of a try-catch from the AST: the exception type caught, whether the catch is empty, whether there is a finally, whether try-with-resources is used.

#### Pattern fields

| Field | Required | Meaning | Example |
|------|------|------|------|
| `exception_type` | - | regex for the exception type | `"IOException\|SQLException"` |

#### The boolean conditions in `custom`

| Key | Meaning |
|----|------|
| `catch_is_empty: true` | a violation when the catch block is empty |
| `catch_is_empty: false` | a violation when the catch block has code in it |
| `has_finally: true` | matches when there is a finally |
| `has_finally: false` | matches when there is no finally |
| `has_resources: true` | matches try-with-resources |
| `has_resources: false` | matches anything that is not try-with-resources |

> **How it runs**: set `exception_type` or `catch_is_empty` and the check runs **per catch clause**. Set neither and it runs **per try block**.

#### Example: an empty catch block

```yaml
- id: "quality-ast-007"
  name: "Empty catch block (AST)"
  severity: "critical"
  category: "exception"
  description: "An empty catch block swallows the exception"
  enabled: true
  pattern:
    type: "ast-try-catch"
  custom:
    catch_is_empty: true
    fix: "Add logger.error(\"message\", e) to the catch block"
```

#### Example: IO/DB exceptions without try-with-resources

```yaml
- id: "quality-ast-008"
  name: "try-with-resources not used"
  severity: "medium"
  category: "resource"
  description: "try-with-resources is what keeps IO and database resources from leaking"
  enabled: true
  pattern:
    type: "ast-try-catch"
    exception_type: "IOException|SQLException|FileNotFoundException"
  custom:
    has_resources: false
    has_finally: false
    fix: "Write it as try (Resource r = new Resource()) { ... }"
```

#### Example: catching RuntimeException

```yaml
- id: "custom-catch-runtime"
  name: "Blanket catch of RuntimeException"
  severity: "high"
  category: "exception"
  description: "Catching RuntimeException can hide a bug"
  enabled: true
  pattern:
    type: "ast-try-catch"
    exception_type: "^(RuntimeException|Exception|Throwable)$"
  custom:
    fix: "Catch the specific exception type"
```

---

### 2.6 ast-class — class shape checks

Reads a class's name, annotations and method and field counts from the AST.

#### Pattern fields

| Field | Required | Meaning | Example |
|------|------|------|------|
| `name_pattern` | - | regex for the class name | `".*ServiceImpl$"` |
| `has_annotation` | - | only classes carrying this annotation | `"Service"` |
| `missing_annotation` | - | a violation when this annotation is absent | `"Transactional"` |

#### The thresholds in `custom`

| Key | Meaning |
|----|------|
| `max_methods` | maximum methods; more than this is a violation |
| `max_fields` | maximum fields; more than this is a violation |

> **Order of checks**: `missing_annotation`, then `max_methods`, then `max_fields`. The first violation produces the finding.

#### Example: an oversized ServiceImpl

```yaml
- id: "quality-ast-009"
  name: "ServiceImpl is too large"
  severity: "medium"
  category: "design"
  description: "This ServiceImpl has too many methods. Split its responsibilities."
  enabled: true
  pattern:
    type: "ast-class"
    name_pattern: ".*ServiceImpl$"
  custom:
    max_methods: 20
    fix: "Split the Service along domain lines"
```

#### Example: an oversized Controller

```yaml
- id: "quality-ast-010"
  name: "Controller is too large"
  severity: "medium"
  category: "design"
  description: "This Controller has too many methods"
  enabled: true
  pattern:
    type: "ast-class"
    name_pattern: ".*Controller$"
  custom:
    max_methods: 15
    fix: "Split the Controller by feature"
```

#### Example: a @Service without @Transactional

```yaml
- id: "custom-service-transactional"
  name: "Service has no @Transactional"
  severity: "medium"
  category: "design"
  description: "A Service class needs @Transactional"
  enabled: true
  pattern:
    type: "ast-class"
    name_pattern: ".*ServiceImpl$"
    has_annotation: "Service"
    missing_annotation: "Transactional"
  custom:
    fix: "Add @Transactional to the class or the method"
```

#### Example: limiting fields on a VO

```yaml
- id: "custom-vo-too-many-fields"
  name: "Too many fields on a VO"
  severity: "low"
  category: "design"
  description: "This VO has too many fields. Consider splitting it."
  enabled: true
  pattern:
    type: "ast-class"
    name_pattern: ".*(?i)(vo|dto)$"
  custom:
    max_fields: 30
    fix: "Group the related fields into a nested VO or DTO"
```

### 2.7 ast-method — method shape analysis

Reads a method's if-else nesting depth, branch count, line count and statement count from the AST.

#### Pattern fields

| Field | Required | Meaning | Example |
|------|------|------|------|
| `name_pattern` | - | regex for the method name; omit it for all methods | `"process.*"` |

#### The thresholds in `custom`

| Key | Meaning | Default |
|----|------|--------|
| `max_if_depth` | maximum if nesting depth; more is a violation | off |
| `max_branches` | maximum if-else-if branches; more is a violation | off |
| `max_lines` | maximum lines in the method; more is a violation | off |
| `max_statements` | maximum statements; more is a violation | off |

> **Order of checks**: `max_if_depth`, `max_branches`, `max_lines`, `max_statements`. The first violation produces the finding.
> **Note**: an else-if chain counts as branches, not depth. `if-else if-else if-else` is depth 1, branches 4.

#### Example: deep if-else nesting (the arrow anti-pattern)

```yaml
- id: "quality-mth-001"
  name: "Deep if-else nesting"
  severity: "high"
  category: "complexity"
  description: "if-else nesting goes deeper than three levels"
  enabled: true
  pattern:
    type: "ast-method"
  custom:
    max_if_depth: 3
    fix: "Flatten it with guard clauses (early returns)"
```

#### Example: too many if-else-if branches

```yaml
- id: "quality-mth-002"
  name: "Catch-all if-else chain"
  severity: "medium"
  category: "complexity"
  description: "More than five if-else-if branches"
  enabled: true
  pattern:
    type: "ast-method"
  custom:
    max_branches: 5
    fix: "Move to the strategy pattern, a Map dispatch, or an enum"
```

#### Example: a long method

```yaml
- id: "quality-mth-003"
  name: "Catch-all method (long method)"
  severity: "high"
  category: "complexity"
  description: "The method is longer than 100 lines"
  enabled: true
  pattern:
    type: "ast-method"
  custom:
    max_lines: 100
    fix: "Split it by what it does — read, validate, process, store"
```

#### Example: limiting statements in a Service method

```yaml
- id: "custom-svc-statements"
  name: "Service method too large"
  severity: "medium"
  category: "complexity"
  description: "More than 50 statements in this Service method"
  enabled: true
  pattern:
    type: "ast-method"
    name_pattern: "(process|execute|handle).*"
  custom:
    max_statements: 50
    fix: "Extract private methods so it reads better"
```

---

### 2.8 ast-dead-code — dead code

Finds private methods and private fields that are never used. If nothing in the same file references it beyond the declaration, it is reported.

#### The `custom` settings

| Key | Value | Meaning |
|----|-----|------|
| `check_type` | `"private-method"` | find unused private methods |
| `check_type` | `"private-field"` | find unused private fields |

> **Note**: `static final` constants are excluded automatically. `public` and `protected` members may be used from elsewhere, so they are not checked.

#### Example: an unused private method

```yaml
- id: "quality-dead-method-001"
  name: "Unused private method"
  severity: "medium"
  category: "dead-code"
  description: "This private method is never called in the file that declares it"
  enabled: true
  pattern:
    type: "ast-dead-code"
  custom:
    check_type: "private-method"
    fix: "Delete the unused private method"
```

#### Example: an unused private field

```yaml
- id: "quality-dead-field-001"
  name: "Unused private field"
  severity: "medium"
  category: "dead-code"
  description: "This private field is never used beyond its declaration"
  enabled: true
  pattern:
    type: "ast-dead-code"
  custom:
    check_type: "private-field"
    fix: "Delete the unused private field"
```

---

### 2.9 ast-filtered-regex — regex with comments filtered out

Matches line by line exactly as `regex` does, but uses the ANTLR4 token stream to **skip lines that are only a comment**. It keeps a pattern from firing on something like `// logger.info(...)`.

> **Java only**: on a file with no Java AST it behaves exactly like plain `regex`.

#### Pattern fields

| Field | Required | Meaning | Example |
|------|------|------|------|
| `regex` | ✅ | the pattern (regexp2 syntax) | `"logger\\.(debug\|info)\\s*\\([^)]*\\+"` |

#### Extra fields

| Field | Meaning |
|------|------|
| `exclude[]` | patterns that drop a line that otherwise matched |
| `custom.flags` | `"i"` = case-insensitive |

#### How it differs from `regex`

| | `regex` | `ast-filtered-regex` |
|---|---------|---------------------|
| A comment line | matches, and misfires | skipped automatically |
| Needs an AST | ❌ | ✅ (falls back to regex without one) |
| Speed | fast | slightly slower (it tokenizes) |
| When to use it | patterns that do not misfire | patterns that keep firing on comments |

#### Example: logger string concatenation, without the comment misfires

```yaml
- id: "quality-lg-005"
  name: "String concatenation in a logger call"
  severity: "high"
  category: "logging"
  description: "Do not build a log message with '+'. Use {} placeholders."
  enabled: true
  pattern:
    type: "ast-filtered-regex"
    regex: "logger\\.(debug|info|warn|error)\\s*\\([^)]*\\+[^)]*\\)"
  custom:
    rule_id: "LG-005"
    fix: "Use {} placeholders: logger.info(\"message: {}\", value)"
```

What it catches:
```java
logger.info("user: " + userId);        // ← violation, uses +
// logger.info("test" + value);        // ← a comment, so ast-filtered-regex skips it
```

#### Example: long method chains (using exclude)

```yaml
- id: "quality-chain-004"
  name: "Method chain too long"
  severity: "low"
  category: "maintainability"
  description: "The chain is four calls or longer. Break it with an intermediate variable."
  enabled: true
  pattern:
    type: "ast-filtered-regex"
    regex: "\\w+\\.\\w+\\([^)]*\\)\\.\\w+\\([^)]*\\)\\.\\w+\\([^)]*\\)\\.\\w+\\("
  exclude:
    - ".stream()"
    - ".builder()"
    - "StringBuilder"
    - "StringBuffer"
  custom:
    fix: "Split the chain with an intermediate variable"
```

---

### 2.10 ast-multi-cud — multi-row CUD without a transaction

Finds a Service method that makes **two or more create/update/delete calls** without `@Transactional`. Without a transaction, a failure halfway through leaves the data inconsistent.

#### Pattern fields

| Field | Required | Meaning | Example |
|------|------|------|------|
| `method` | ✅ | regex for the CUD method names | `"insert\|update\|delete\|save"` |
| `qualifier` | - | regex for the object called | `".*Mapper\|.*Repository"` |

#### The `custom` settings

| Key | Meaning | Default |
|----|------|--------|
| `min_cud_calls` | how many CUD calls make it a violation | `2` |

#### How it runs

1. For each method, check whether it has `@Transactional`
2. In methods without it, count the calls matching `method` and `qualifier`
3. Report a violation when the count reaches `min_cud_calls`

#### Example: several CUD calls with no @Transactional

```yaml
- id: "spring-tx-005"
  name: "@Transactional missing (several CUD calls)"
  severity: "high"
  category: "transaction"
  description: "Two or more CUD calls with no @Transactional, so the data can be left inconsistent."
  enabled: true
  pattern:
    type: "ast-multi-cud"
    method: "insert|update|delete|save|remove|create|modify"
    qualifier: ".*Dao|.*Repository|.*Mapper|.*Service"
  custom:
    min_cud_calls: 2
    rule_id: "SPRING-TX-005"
    fix: "Add @Transactional, or delegate to a method that has one"
```

What it catches:
```java
// no @Transactional
public void transferMoney(String from, String to, int amount) {
    accountMapper.updateBalance(from, -amount);  // CUD 1
    accountMapper.updateBalance(to, amount);      // second CUD → violation
}
```

Code that passes:
```java
@Transactional
public void transferMoney(String from, String to, int amount) {
    accountMapper.updateBalance(from, -amount);
    accountMapper.updateBalance(to, amount);     // @Transactional is present, so it passes
}
```

---

### 2.11 cross-file — cross-file analysis

Cross-checks several files against each other to find mismatches between them. Today it covers **unused MyBatis SQL IDs**.

#### How it runs

1. Collect the `id` attributes of `<select>`, `<insert>`, `<update>` and `<delete>` in the XML mappers
2. Look for references to those SQL IDs in the Java files of the same directory (or project)
3. Report a violation when the Java code has neither a call (`mapper.sqlId(`) nor a string reference (`"namespace.sqlId"`)

> **Note**: this runs at the analyzer level. Declaring `pattern.type: "cross-file"` switches cross-file analysis on by itself.

#### Example: an unused MyBatis SQL ID

```yaml
- id: "quality-mybatis-unused-001"
  name: "Unused MyBatis SQL ID"
  severity: "medium"
  category: "dead-code"
  description: "This SQL ID in the MyBatis XML is never used from Java"
  enabled: true
  pattern:
    type: "cross-file"
  custom:
    fix: "Remove the unused mapping, or add the matching Mapper interface method"
```

What it catches:
```xml
<!-- UserMapper.xml -->
<select id="selectDeletedUsers" resultType="UserVo">  <!-- never used from Java → violation -->
    SELECT * FROM TB_USER WHERE del_yn = 'Y'
</select>
```

#### Example: the other direction — Mapper interface to XML

A Mapper interface method with no matching XML SQL ID throws a BindingException at runtime:

```yaml
- id: "quality-mapper-no-xml-001"
  name: "Mapper method has no matching XML SQL ID"
  severity: "high"
  category: "cross-file"
  description: "This Mapper interface method has no matching SQL ID in the XML"
  enabled: true
  pattern:
    type: "cross-file"
  custom:
    fix: "Add the matching SQL ID to the XML mapper"
```

#### Example: a resultMap property with no matching VO field

If a resultMap property does not exist on the VO class, the mapping fails at runtime:

```yaml
- id: "quality-resultmap-vo-001"
  name: "resultMap property does not match the VO"
  severity: "high"
  category: "cross-file"
  description: "This resultMap property does not exist on the VO class"
  enabled: true
  pattern:
    type: "cross-file"
  custom:
    fix: "Add the field to the VO, or correct the property name"
```

#### Example: an unused Service method

A Service method that no Controller and no other Service calls is dead code:

```yaml
- id: "quality-unused-service-001"
  name: "Unused Service method"
  severity: "medium"
  category: "dead-code"
  description: "This Service method is not called from any other file"
  enabled: true
  pattern:
    type: "cross-file"
  custom:
    fix: "Remove the unused Service method"
```

#### Example: an unused VO/DTO file

A VO/DTO class that no other file references at all can go:

```yaml
- id: "quality-unused-vo-001"
  name: "Unused VO/DTO file"
  severity: "medium"
  category: "dead-code"
  description: "This VO/DTO class is not referenced from any other file"
  enabled: true
  pattern:
    type: "cross-file"
  custom:
    fix: "Delete the unused VO/DTO file"
```

---

## Things to watch when writing the regex

### 1. Escaping backslashes

In YAML a backslash must be written `\\`:

```yaml
# ✅ correct
regex: "System\\.out\\.print"
regex: "\\bSELECT\\b"

# ❌ wrong
regex: "System\.out\.print"
regex: "\bSELECT\b"
```

### 2. Escaping special characters

| Character | Escaped | Meaning |
|------|-----------|------|
| `.` | `\\.` | a literal dot |
| `*` | `\\*` | a literal asterisk |
| `(` | `\\(` | a literal parenthesis |
| `{` | `\\{` | a literal brace |
| `$` | `\\$` | a literal dollar sign |
| `|` | `\|` | alternation (no escape needed) |

### 3. Patterns you will reach for

| Pattern | Meaning |
|------|------|
| `\\b` | word boundary |
| `\\s+` | one or more spaces |
| `\\w+` | one or more word characters |
| `[^)]*` | anything but `)` |
| `(?!pattern)` | Negative lookahead |
| `(?i)` | case-insensitive, inline |
| `^prefix` | starts with prefix |
| `suffix$` | ends with suffix |

### 4. Testing the regex

Test the pattern before you add the rule:

- [regex101.com](https://regex101.com/) — choose PCRE mode
- [regexr.com](https://regexr.com/)

---

## Adding a new rule, step by step

### 1. Add it to an existing ruleset

Pick the ruleset file it belongs in and add the rule:

```bash
vi configs/rulesets/quality.yaml
```

### 2. Create a new ruleset file

If you have many rules in a new category, give them their own file:

```yaml
# configs/rulesets/custom.yaml
version: "1.0"
profile: "custom"

languages:
  - language: java
    rules:
      - id: "custom-rule-001"
        # ... the rule definitions
```

Then register the profile in `profiles.yaml`:

```yaml
# configs/profiles.yaml
profiles:
  custom:
    name: "Custom rules"
    description: "Rules specific to this project"
    source: "rulesets/custom.yaml"
    languages:
      - java
```

### 3. Test it

```bash
# build
make dev

# run the profile that includes the new rule
./build/rulix /path/to/source --profile=custom

# check one category only
./build/rulix /path/to/source --profile=quality --categories=performance

# JSON output, to look closer
./build/rulix /path/to/source --profile=quality -o json --output-file=report.json
```

---

## Rules per language

You can define rules for each language. **AST query rules (`ast-*`) are Java only.**

```yaml
languages:
  - language: java
    rules:
      - id: "java-rule"        # regex and ast-* both work

  - language: javascript
    rules:
      - id: "js-rule"          # regex only

  - language: sql
    rules:
      - id: "sql-rule"         # regex only
```

Supported languages: `java`, `javascript`, `html`, `css`, `sql`, `xml`, `properties`

---

## The categories already in use

Reuse one of these rather than inventing a category:

| Category | Meaning | Rulesets |
|----------|------|------|
| `naming` | naming conventions | quality, egov |
| `security` | security | secure, egov |
| `logging` | logging | quality, egov |
| `exception` | exception handling | quality, spring, egov |
| `style` | coding style | quality |
| `performance` | performance | quality |
| `design` | design and structure | quality, spring |
| `architecture` | architecture | spring |
| `deprecated` | use of a deprecated API | quality |
| `resource` | resource handling | quality |
| `documentation` | documentation | quality |
| `maintainability` | maintainability | quality |
| `code-quality` | code quality | quality |
| `annotation` | annotations | quality |
| `egov-migration` | eGovFrame migration | modernize |
| `jakarta-migration` | javax → jakarta | modernize |
| `spring-migration` | replacing deprecated Spring APIs | modernize |
| `ibatis-migration` | iBatis → MyBatis | modernize |
| `java-modernize` | modernizing legacy Java APIs | modernize |
| `dependency-injection` | DI patterns | spring |
| `transaction` | @Transactional | spring |
| `rest-api` | REST API patterns | spring |
| `thread-safety` | thread safety | spring |
| `testing` | testing patterns | spring |
| `egov-standard` | eGovernment standards | egov |
| `egov-common` | using the common components | egov |
| `method-naming` | CRUD method prefixes | egov |
| `mybatis` | MyBatis XML standards | egov |

---

## Troubleshooting

### The rule is not firing

1. Check `enabled: true`
2. Check the regex syntax (regex101.com)
3. Check the backslash escaping (`\\`)
4. For an AST rule, check the file is Java (`ast-*` is Java only)
5. Run it verbose: `./build/rulix -v`

### An AST rule finds less than you expected

1. Check the `qualifier` regex is not too strict
2. Check the `context` condition is the one you meant (`inside-loop` vs `inside-catch`)
3. Check whether the `method` pattern needs `^` or `$` anchors

### It finds far too much

1. Add `exclude` patterns (regex types)
2. Make the pattern more specific
3. Narrow the scope with `context`
4. Narrow the classes with `class_annotation` or `has_annotation`
5. Adjust `severity` and filter with `--min-severity`

### Regex performance

- Regex matching has a 100 ms timeout
- A complex pattern costs time; keep it as simple as it can be
- A specific pattern like `[^)]*` is faster than `.*`

---

## Part 3: the newer rulesets in practice

### 3.1 modernize — legacy modernization

For migrating legacy code onto a current stack. All 50 rules are written in YAML alone, combining the **regex** and **ast-import** types.

#### What is in it

| Category | Rules | Pattern types | What it covers |
|----------|---------|-----------|------|
| `egov-migration` | 5 | ast-import, regex | eGovFrame 3.x → 4.x package and path changes |
| `jakarta-migration` | 10 | ast-import | javax.* → jakarta.* namespace change |
| `spring-migration` | 10 | regex | replacing deprecated Spring classes |
| `ibatis-migration` | 8 | regex, ast-import | iBatis → MyBatis API changes |
| `java-modernize` | 12 | regex, ast-import | modernizing legacy Java APIs |
| (XML) | 5 | regex | iBatis XML tags, Spring bean XML |

#### Examples by pattern type

**ast-import** — package migration, the most precise way:

```yaml
- id: "mod-jakarta-001"
  name: "javax.servlet in use"
  severity: "high"
  category: "jakarta-migration"
  description: "javax.servlet became jakarta.servlet in Spring Boot 3.x and Jakarta EE 9+."
  enabled: true
  pattern:
    type: "ast-import"
    import: "^javax\\.servlet\\."
  custom:
    rule_id: "MOD-JAKARTA-001"
    fix: "Change it to the jakarta.servlet package"
    since: "Jakarta EE 9 / Spring Boot 3.0"
```

> **ast-import vs regex**: for imports, `ast-import` is the accurate one. A `regex` search for `import javax.servlet` can fire on a comment or a string literal; `ast-import` only looks at the import statements the AST actually holds.

**regex** — method calls and class references:

```yaml
- id: "mod-ibatis-003"
  name: "queryForObject() in use (iBatis)"
  severity: "high"
  category: "ibatis-migration"
  description: "queryForObject() is an iBatis method. Use MyBatis selectOne()."
  enabled: true
  pattern:
    type: "regex"
    regex: "\\.queryForObject\\s*\\("
  custom:
    fix: "Use selectOne(), or a Mapper interface method"
```

**XML rules** — iBatis and Spring XML tags:

```yaml
- id: "mod-xml-002"
  name: "iBatis isNotNull/isEqual tags"
  severity: "high"
  category: "ibatis-migration"
  description: "<isNotNull>, <isEqual> and friends are iBatis dynamic SQL tags."
  enabled: true
  pattern:
    type: "regex"
    regex: "<(isNotNull|isNotEmpty|isNull|isEmpty|isEqual|isNotEqual)\\b"
  custom:
    fix: "Change them to <if test=\"...\">, or <choose>/<when>"
```

#### Adding your own modernize rule

To add a rule for legacy code specific to your project:

```yaml
# add it to configs/rulesets/modernize.yaml
- id: "mod-custom-001"
  name: "Old in-house utility package in use"
  severity: "medium"
  category: "java-modernize"
  description: "The com.mycompany.legacy.util package has moved"
  enabled: true
  pattern:
    type: "ast-import"
    import: "^com\\.mycompany\\.legacy\\.util\\."
  custom:
    fix: "Change it to com.mycompany.core.util"
```

---

### 3.2 spring — Spring development standards

Checks the architecture and coding standards of a Spring or Spring Boot project, combining the **regex**, **regex-multiline** and **ast-class** types.

#### What is in it

| Category | Rules | Pattern types | What it covers |
|----------|---------|-----------|------|
| `dependency-injection` | 3 | regex | field injection via @Autowired/@Inject/@Resource |
| `transaction` | 3 | regex-multiline | using @Transactional correctly |
| `architecture` | 3 | regex-multiline, ast-class | breaking the Controller → Service → Repository layering |
| `rest-api` | 4 | regex, regex-multiline | REST API response patterns |
| `thread-safety` | 2 | regex, regex-multiline | state on a singleton bean |
| `exception` | 3 | regex, regex-multiline | exception handling standards |
| `testing` | 3 | regex, regex-multiline | @SpringBootTest, Thread.sleep |
| `security` | 2 | regex | CORS and CSRF configuration |

#### The key choice: regex or regex-multiline

**regex** (line by line) — when the pattern fits on one line:

```yaml
# @Autowired alone at the end of a line means field injection
- id: "spring-di-001"
  pattern:
    type: "regex"
    regex: "@Autowired\\s*$"
```

**regex-multiline** (whole file) — when the annotation and the code are on different lines:

```yaml
# JdbcTemplate inside a @Controller class (spans lines)
- id: "spring-ctrl-002"
  pattern:
    type: "regex-multiline"
    regex: "@(Controller|RestController)[\\s\\S]*?(JdbcTemplate|DataSource)\\s+\\w+"
```

> **Careful**: `regex` matches line by line. If your pattern contains `\n` or `[\s\S]` — anything that spans lines — it must be `regex-multiline`. Get this wrong and the rule never matches at all.

**ast-class** — the shape of a Controller or Service class:

```yaml
# limit the methods on a Controller
- id: "spring-ctrl-005"
  pattern:
    type: "ast-class"
    name_pattern: ".*Controller$"
  custom:
    max_methods: 15
    fix: "Move the business logic into the Service layer"
```

**ast-method** — method shape (if nesting, branch count, long methods):

```yaml
# deep if-else nesting
- id: "quality-mth-001"
  pattern:
    type: "ast-method"
  custom:
    max_if_depth: 3
    fix: "Flatten it with guard clauses (early returns)"

# too many if-else-if branches
- id: "quality-mth-002"
  pattern:
    type: "ast-method"
  custom:
    max_branches: 5
    fix: "Move to the strategy pattern, or a Map dispatch"

# long methods
- id: "quality-mth-003"
  pattern:
    type: "ast-method"
  custom:
    max_lines: 100
    fix: "Split the method by what it does"
```

#### Adding your own spring rule

```yaml
# check the return type of an @Async method
- id: "spring-custom-001"
  name: "@Async returning void"
  severity: "medium"
  category: "design"
  description: "An @Async void method cannot report its exception back to the caller"
  enabled: true
  pattern:
    type: "regex-multiline"
    regex: "@Async.*\\n\\s*public\\s+void\\s+"
  custom:
    fix: "Return Future<T> or CompletableFuture<T>"

# prefer @ConfigurationProperties over many @Value fields
- id: "spring-custom-002"
  name: "Too many @Value fields"
  severity: "low"
  category: "design"
  description: "With this many @Value fields, group them with @ConfigurationProperties"
  enabled: true
  pattern:
    type: "regex"
    regex: "@Value\\s*\\(\\s*\"\\$\\{"
  custom:
    fix: "Group them under @ConfigurationProperties(prefix=\"...\")"
```

---

### 3.3 egov — eGovernment Framework standards

Checks the architecture, naming and use of common components in the Korean eGovernment Standard Framework, combining the **regex**, **regex-multiline** and **ast-class** types.

#### What is in it

| Category | Rules | Pattern types | What it covers |
|----------|---------|-----------|------|
| `naming` | 6 | regex, regex-multiline | Controller/Service/Mapper/VO suffixes, package layout |
| `egov-standard` | 4 | regex, regex-multiline, ast-class | ServiceImpl inheritance, broken layering, oversized classes |
| `egov-common` | 5 | regex, regex-multiline | using the common components (file upload, paging, ID generation) |
| `method-naming` | 4 | regex-multiline | CRUD method prefixes (select*/insert*/update*/delete*) |
| `exception` | 3 | regex, regex-multiline | EgovBizException and throws conventions |
| `logging` | 3 | regex | System.out, logger string concatenation, printStackTrace |
| `security` | 2 | regex | XSS filtering, SQL injection |
| `mybatis` (XML) | 3 | regex | namespace, ${}, SELECT * |

#### The key pattern: checking method names (regex-multiline)

The annotation and the method declaration sit on different lines, so this needs `regex-multiline`:

```yaml
# a GET handler whose name does not start with select/get/find
- id: "egov-method-001"
  name: "Read method prefix not followed"
  severity: "medium"
  category: "method-naming"
  pattern:
    type: "regex-multiline"
    regex: "@(GetMapping|RequestMapping).*\\n\\s*public\\s+\\w+\\s+(?!(select|get|find|search|retrieve|view|download|check|is)\\w+)([a-z]\\w+)\\s*\\("
  custom:
    fix: "Read methods take a select*, get* or find* prefix"
```

What it catches:
```java
@GetMapping("/process")
public String processData(ModelMap model) {  // ← violation: not select*/get*/find*
```

#### The key pattern: layer naming (regex-multiline plus negative lookahead)

```yaml
# has @Controller but the class name lacks the Controller suffix
- id: "egov-naming-001"
  pattern:
    type: "regex-multiline"
    regex: "@Controller[\\s\\S]*?class\\s+(?!\\w*Controller\\b)\\w+"

# has @Mapper but the interface name lacks the Mapper/DAO suffix
- id: "egov-naming-004"
  pattern:
    type: "regex-multiline"
    regex: "@Mapper[\\s\\S]*?interface\\s+(?!\\w*(Mapper|DAO)\\b)\\w+"
```

> **`[\\s\\S]*?`**: under regex-multiline this means "any character including newlines, lazily". It still matches when several lines sit between the annotation and the class declaration.

#### Adding your own egov rule

```yaml
# no constructing a Service inside a Controller
- id: "egov-custom-001"
  name: "Service constructed inside a Controller"
  severity: "high"
  category: "egov-standard"
  description: "new Service() in a Controller takes it out of dependency injection"
  enabled: true
  pattern:
    type: "regex-multiline"
    regex: "@Controller[\\s\\S]*?new\\s+\\w+Service(Impl)?\\s*\\("
  custom:
    fix: "Inject the Service through Spring DI"

# avoid resultType="hashmap" in mapper XML
- id: "egov-custom-002"
  name: "MyBatis resultType=hashmap"
  severity: "medium"
  category: "mybatis"
  description: "Returning a hashmap gives up type safety. Define a VO or DTO."
  enabled: true
  pattern:
    type: "regex"
    regex: "resultType\\s*=\\s*\"(hashmap|HashMap|map)\""
  custom:
    fix: "Define a VO class and name it in resultType"
```

---

## Part 4: choosing a pattern type

### Which type, when

```
Does the pattern fit on one line?
├── YES → does it misfire on comments?
│   ├── YES → type: "ast-filtered-regex"
│   │         e.g. logger.info(... +), System.out, security patterns
│   └── NO  → type: "regex"
│             e.g. new Date(), @Autowired$
└── NO → does it span several lines?
    ├── YES → type: "regex-multiline"
    │         e.g. @Controller...class, @GetMapping...public void
    └── do you need the Java AST?
        ├── import paths           → type: "ast-import"
        ├── a call, and where it sits → type: "ast-method-call"
        ├── annotation attributes  → type: "ast-annotation"
        ├── variable/field name or type → type: "ast-variable"
        ├── try-catch shape        → type: "ast-try-catch"
        ├── methods/fields per class → type: "ast-class"
        └── several CUD calls without @Transactional → type: "ast-multi-cud"
```

### regex vs regex-multiline, the difference that matters

| | `regex` | `regex-multiline` |
|---|---------|-------------------|
| What it matches against | each line of the file | the whole file |
| Matching `\n` | ❌ never | ✅ yes |
| `[\s\S]` | ❌ within a line only | ✅ newlines included |
| `^` / `$` | start and end of a line | start and end of the file |
| Speed | fast | can be slow, depending on the pattern |
| When to use it | single-line patterns | annotation plus declaration, block structure |

> **The mistake everyone makes**: `\n` or `[\s\S]` under `regex` **never matches**. A pattern that spans lines must be `regex-multiline`.

### Where an ast-* type beats a regex

| What you are checking | the trouble with regex | the ast-* answer |
|-----------|-------------|-------------|
| Import paths | misfires on comments and strings | `ast-import` looks only at real imports |
| A call inside a loop | the multiline pattern gets complex | `ast-method-call` plus `context: inside-loop` |
| Methods per class | not expressible | `ast-class` plus `max_methods: N` |
| Annotation attributes | parsing key=value is painful | `ast-annotation` plus `missing_attr_name` |
| Catch blocks | nested braces are hard to parse | `ast-try-catch` plus `catch_is_empty` |
| Misfiring on comments | cannot tell comment from code | `ast-filtered-regex` skips comment lines |
| Multi-row CUD transactions | a regex cannot reason per method | `ast-multi-cud` plus `min_cud_calls: N` |

### How the engines differ

| | regex | regex-multiline | ast-filtered-regex | AST query |
|------|-------|-----------------|-------------------|----------|
| **Input unit** | one line at a time | the whole file (joined with `\n`) | one line at a time | the ANTLR4 parse tree |
| **How it matches** | text regex | text regex | regex plus a comment filter | comparing structural properties |
| **Multiline patterns** | no | **yes** | no | n/a (structural) |
| **Comment misfires** | not handled | not handled | **removed automatically** | **removed structurally** |
| **Context conditions** | none | none | none | inside-loop, inside-catch and so on |
| **Timeout** | 100 ms | 500 ms | 100 ms | none (it walks the tree) |
| **Languages** | all | all | Java (falls back to regex elsewhere) | Java only |
| **Best for** | simple keyword detection | multiline patterns | when comments cause misfires | method, class and structural analysis |

### The same rule written three ways

Detecting `System.out.println`, three ways:

```yaml
# 1) regex — can misfire on comments
- id: "example-regex"
  pattern:
    type: "regex"
    regex: "System\\.out\\.println"

# 2) ast-filtered-regex — skips comment lines
- id: "example-ast-filtered"
  pattern:
    type: "ast-filtered-regex"
    regex: "System\\.out\\.println"

# 3) ast-method-call — only real call nodes, the most accurate
- id: "example-ast-call"
  pattern:
    type: "ast-method-call"
    method: "println"
    qualifier: "System\\.out"
```

**What each one does:**

| Source | regex | ast-filtered-regex | ast-method-call |
|----------|-------|-------------------|-----------------|
| `System.out.println("hello");` | found | found | found |
| `// System.out.println is banned` | misfire | skipped | skipped |
| `/* System.out.println */` | misfire | skipped | skipped |
| `String s = "System.out.println";` | misfire | misfire | skipped |

> **In short**: a simple keyword → `regex`; misfires on comments → `ast-filtered-regex`; structural accuracy → `ast-*`; a pattern that spans lines → `regex-multiline`

---

[한국어](CUSTOM_RULES.ko.md)
