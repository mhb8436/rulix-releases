# RULIX quick start — the desktop app and the CLI

RULIX is a desktop app and a command-line tool over the same rule engine. The app
is the faster way to look through a codebase you have not seen before and decide
which rules apply; the CLI is what repeats that decision on every build. One
file joins them: a rule snapshot (section 4).

---

## 1. Install

### What to download

Unzip and you are done. No JVM, no Node.js, no Python.

| Platform | File | Contains |
|---|---|---|
| Windows x64 | `rulix-windows-amd64.zip` | `rulix.exe`, `rulix-gui.exe`, `rulix-report.exe` |
| macOS Apple Silicon | `rulix-darwin-arm64.tar.gz` | `rulix`, `rulix-gui`, `rulix-report` |
| macOS Intel | `rulix-darwin-amd64.tar.gz` | `rulix`, `rulix-gui`, `rulix-report` |
| Linux x64 / ARM64 / x86 | `rulix-linux-*.tar.gz` | `rulix` |
| Windows x86 | `rulix-windows-386.zip` | `rulix.exe` |

If you only want the desktop app, it is published on its own as
`rulix-gui-windows-amd64.exe`, `rulix-gui-darwin-arm64` and
`rulix-gui-darwin-amd64`.

> **There is no desktop build for Linux.** Wails needs GTK and WebKit at build
> time and we do not ship that yet. On Linux, RULIX is the CLI.

### Windows

Unzip and double-click `rulix-gui.exe`. The interface renders through the
Microsoft WebView2 runtime, which ships with Windows 11 and with current
Windows 10. On an older machine, install
[WebView2 Runtime](https://developer.microsoft.com/microsoft-edge/webview2/)
first.

### macOS

```bash
tar -xzf rulix-darwin-arm64.tar.gz
cd rulix-darwin-arm64

# The binaries are not code-signed yet, so Gatekeeper quarantines them.
# Skip this line and macOS reports the app as damaged.
xattr -dr com.apple.quarantine .

./rulix-gui
```

### Linux

```bash
tar -xzf rulix-linux-amd64.tar.gz
cd rulix-linux-amd64
./rulix --version
```

### What is in the bundle

```
rulix[.exe]                 ← CLI
rulix-gui[.exe]             ← desktop app (Windows / macOS)
rulix-report[.exe]          ← DOCX audit report generator (Windows / macOS)
configs/
├── profiles.yaml          ← profile definitions
└── rulesets/
    ├── quality.yaml       ← code quality (125 rules)
    ├── secure.yaml        ← secure coding (131 rules)
    ├── sql.yaml           ← ANSI SQL (120 rules)
    ├── sql-oracle.yaml    ← Oracle specifics (20 rules)
    ├── sql-format.yaml    ← SQL formatting (7 rules)
    ├── modernize.yaml     ← legacy modernization (50 rules)
    ├── spring.yaml        ← Spring conventions (32 rules)
    ├── egov.yaml          ← eGovernment Framework standards (33 rules)
    └── ddl.yaml           ← DDL × query cross-analysis (17 rules)
```

The binary and the `configs/` folder next to it are all it needs.

---

## 2. Five minutes with the desktop app

The app opens with seven tabs down the left. `Ctrl/Cmd` + a number moves between
them.

**① Scan** (`Ctrl/Cmd+1`)

Pick the source folder with **Browse**, check the profiles to run, set a minimum
severity, and press Run. Point at a DDL file or folder and SQL cross-analysis
runs too (optional). Progress is shown and the scan can be cancelled.

**② Results** (`Ctrl/Cmd+3`)

A dashboard of metrics and grades first, the issue list below it. Click an issue
and the file opens at that line.

**③ Rules** (`Ctrl/Cmd+2`)

Turn off rules that do not fit this project and adjust severities. You can add a
team convention as a custom regex rule, testing it against sample code before
you save. When you are done, **save the snapshot as YAML** — that is the file
the CLI reads in section 4.

When you build a custom rule you choose **what the pattern looks at**: *per
line* examines each line separately, *whole file* reads the file as one string.
A rule whose annotation and declaration sit on different lines only matches as
whole-file. Exclusions and the fix hint shown with each finding are on the form
too.

Types the form cannot build — the `ast-*` family, for instance — are written
directly in **Edit ruleset files**. The editor validates with the real parser
before saving, so a broken ruleset is never written (section 5). Press `</>` on
a built-in rule to jump to where it is defined.

**④ Code** (`Ctrl/Cmd+4`)

Browse the project and read the source. **Open in code** on an issue opens that
file at that line, with every issue in the file marked in the gutter. Press
**Edit**, fix it, and save with `Ctrl/Cmd+S` — RULIX re-checks **that one file**
immediately. The editor is part of the app, so you do not have to bring one to
the site.

**⑤ Export** (`Ctrl/Cmd+5`)

Excel, HTML or JSON. Excel is the usual format to hand to a client or a vendor;
JSON is the input to the AI audit report.

> Change a rule or edit a source file and the results you are looking at are out
> of date. A line saying so appears above the Results, Export and AI audit
> screens, with **Rescan** next to it. It is there to stop you building a report
> from numbers that no longer hold.

**⑥ AI audit** (`Ctrl/Cmd+6`)

Enter the endpoint and model, press **Test connection** to confirm the server
answers, then generate. Air-gapped mode is a checkbox; with it on, every
connection that leaves loopback is blocked while the report is written. An
in-house Ollama or vLLM server is enough.

**⑦ Settings** (`Ctrl/Cmd+7`)

Interface language, theme, and the defaults used on the AI audit tab. The app is
English by default; Korean is one of the languages you can pick here.

---

## 3. Five minutes with the command line

### Basic runs

```bash
# Everything
./rulix /path/to/project --profile=all

# Code quality and secure coding only
./rulix /path/to/project --profile=essential

# For an eGovernment Framework project
./rulix /path/to/project --profile=egov-full

# Legacy migration assessment
./rulix /path/to/project --profile=migration

# DDL × query cross-analysis
./rulix /path/to/project --profile=sql-ddl --ddl=./ddl/
```

### Profiles

| Profile | What it covers | Rules | On by default |
|---------|----------------|-------|---------------|
| `quality` | Code quality, naming, complexity | 125 | 107 |
| `secure` | SQL injection, XSS, crypto, authentication | 131 | 123 |
| `sql` | ANSI SQL, vendor independent | 120 | 102 |
| `sql-oracle` | Oracle specifics (NVL, hints, ROWNUM) | 20 | 18 |
| `sql-format` | SQL formatting and style | 7 | 7 |
| `modernize` | eGov / javax / iBatis / Spring migration | 50 | 39 |
| `spring` | DI, `@Transactional`, controllers | 32 | 28 |
| `egov` | eGovernment layering, naming, common components | 33 | 12 |
| `ddl` | DDL × query cross-analysis (needs `--ddl`) | 17 | 17 |
| | **Total** | **535** | **453** |

Some rules ship switched off. They encode a team preference rather than a
defect — mostly SQL formatting and eGovernment conventions. Turn them on in the
Rules tab or in an overrides file.

### Profile groups

| Group | Includes | For |
|-------|----------|-----|
| `all` | quality, secure, sql, sql-oracle, sql-format, modernize, spring, egov | everything |
| `essential` | quality, secure | the must-haves |
| `sql-all` | sql, sql-oracle, sql-format | SQL in general |
| `sql-ddl` | sql, sql-oracle, ddl | SQL plus schema cross-checks |
| `egov-full` | quality, secure, sql, sql-oracle, sql-format, egov, spring | eGovernment projects |
| `migration` | modernize, spring | legacy migration assessment |

Run `./rulix profiles` to list them from the binary you have.

### Report formats

```bash
# Console (default)
./rulix /path/to/project --profile=all

# Excel — what usually gets handed over
./rulix /path/to/project --profile=all -o excel --output-file=report.xlsx

# JSON — for CI, and as the input to the AI audit report
./rulix /path/to/project --profile=all -o json --output-file=report.json

# HTML — to share in a browser
./rulix /path/to/project --profile=all -o html --output-file=report.html
```

### Filtering by severity

```bash
# High and above
./rulix /path/to/project --profile=all --min-severity=high

# Critical only
./rulix /path/to/project --profile=all --min-severity=critical
```

### Output language

Both front ends default to English. The CLI takes `--lang` (`en` / `ko`), or the
`RULIX_LANG` environment variable; your system locale is not consulted.

```bash
./rulix /path/to/project --profile=all --lang=ko
```

### AI audit report

Produce the scan JSON first, then build the report from it.

```bash
./rulix /path/to/project --profile=all -o json --output-file=scan.json

./rulix ai-report scan.json --offline \
  --endpoint http://127.0.0.1:11434/v1 \
  --model qwen2.5-coder:7b \
  --project "Example System" \
  -o report.html
```

`--offline` refuses any destination that is not loopback, checked at the
resolved IP, and blocks DNS resolution outright. If the endpoint you gave is
external, the command fails and says why.

---

## 4. Rule snapshots — decide in the app, repeat in CI

The snapshot you save in the desktop app's Rules tab is the very YAML the CLI
reads. The point is that a judgement a person makes once is what every later
automated run follows.

```
Desktop app · Rules tab             ruleset.yaml            CI · every build
 toggle rules, set severities  ──▶   save snapshot   ──▶   --overrides ruleset.yaml
```

**① Save it in the app** — adjust things in the Rules tab, then save the
snapshot. You get `ruleset.yaml`. Commit it.

**② Use it from the CLI**

```bash
./rulix /path/to/project --profile=all --overrides=ruleset.yaml \
  -o excel --output-file=report.xlsx
```

**③ It works the other way too.** Tune on the command line and write the result
to a file the app can load.

```bash
./rulix /path/to/project --profile=all \
  --exclude-rule quality-mb-001,sql-fmt-003 \
  --min-severity=high \
  --save-overrides=ruleset.yaml
```

`--save-overrides` writes the **effective rule set**, filters included. You can
reproduce the same state at any time with `--overrides`.

---

## 5. Adding custom rules

The Rules tab does the same thing: type a regex, confirm the match against
sample code, save, and you never think about where the file lives. What follows
is the YAML route, which is what you use to commit rules to the repository so
the whole team shares them.

### 5.1 Where to put them

```
configs/rulesets/quality.yaml    ← add to an existing file (simplest)
configs/rulesets/custom.yaml     ← a new file (when there are many rules)
```

### 5.2 Choosing a pattern type

```
Can it be found within one line?
├── YES → type: "regex"
│
└── NO → Does it span several lines?
    ├── YES → type: "regex-multiline"
    │
    └── Do you need Java structure?
        ├── imports            → "ast-import"
        ├── calls inside loops → "ast-method-call"
        ├── annotation attrs   → "ast-annotation"
        ├── variable/field type→ "ast-variable"
        ├── try-catch shape    → "ast-try-catch"
        ├── methods per class  → "ast-class"
        └── method complexity  → "ast-method"
```

### 5.3 The five you will write most often

#### (1) Banning a specific API or method

```yaml
- id: "custom-001"
  name: "System.exit() prohibited"
  severity: "critical"
  category: "security"
  description: "System.exit() kills the server process"
  enabled: true
  pattern:
    type: "regex"
    regex: "System\\.exit\\s*\\("
  custom:
    fix: "Throw an exception, or use Spring's shutdown mechanism"
```

#### (2) Banning a library import

```yaml
- id: "custom-002"
  name: "Library not approved for this project"
  severity: "high"
  category: "architecture"
  description: "This library has not been approved"
  enabled: true
  pattern:
    type: "ast-import"
    import: "^com\\.alibaba\\.fastjson\\."
  custom:
    fix: "Use Jackson or Gson"
```

#### (3) An annotation plus a method, on different lines

```yaml
- id: "custom-003"
  name: "DELETE method prefix"
  severity: "medium"
  category: "naming"
  description: "A DELETE handler must start with delete or remove"
  enabled: true
  pattern:
    type: "regex-multiline"
    regex: "@DeleteMapping.*\\n\\s*public\\s+\\w+\\s+(?!(delete|remove)\\w+)([a-z]\\w+)\\s*\\("
  custom:
    fix: "Use a delete* or remove* prefix"
```

#### (4) A risky call inside a loop

```yaml
- id: "custom-004"
  name: "Database read inside a loop"
  severity: "high"
  category: "performance"
  description: "Reading from the database in a loop causes the N+1 problem"
  enabled: true
  pattern:
    type: "ast-method-call"
    method: "^(select|find|get|query).*"
    qualifier: ".*(?i)(mapper|dao|repository)"
    context: "inside-loop"
  custom:
    fix: "Batch the reads with an IN clause, or use a JOIN"
```

#### (5) Limiting class size

```yaml
- id: "custom-005"
  name: "Service class too large"
  severity: "medium"
  category: "design"
  description: "The Service has more than 30 methods"
  enabled: true
  pattern:
    type: "ast-class"
    name_pattern: ".*ServiceImpl$"
  custom:
    max_methods: 30
    fix: "Split the Service along domain lines"
```

### 5.4 Creating a new ruleset file

**① Create the file**: `configs/rulesets/custom.yaml`

```yaml
version: "1.0"
profile: "custom"

languages:
  - language: java
    rules:
      - id: "custom-001"
        name: "rule name"
        severity: "high"
        category: "category"
        description: "description"
        enabled: true
        pattern:
          type: "regex"
          regex: "pattern"
        custom:
          fix: "how to fix it"

  - language: xml
    rules:
      - id: "custom-xml-001"
        name: "XML rule"
        # ...

  - language: sql
    rules:
      - id: "custom-sql-001"
        name: "SQL rule"
        # ...
```

**② Register the profile**: `configs/profiles.yaml`

```yaml
profiles:
  # ... existing profiles ...

  custom:
    name: "Project rules"
    description: "This project's own coding standards"
    ruleset: "rulesets/custom.yaml"
    languages:
      - java
      - xml
      - sql

groups:
  all:
    profiles:
      - quality
      - secure
      - sql
      - modernize
      - spring
      - egov
      - custom          # ← added
```

**③ Run it**

```bash
./rulix /path/to/project --profile=custom           # custom only
./rulix /path/to/project --profile=quality,custom   # quality plus custom
./rulix /path/to/project --profile=all              # everything, custom included
```

> The app can do step ① and ② for you: **Export custom rules** in the ruleset
> editor writes the rules you built in the app to `configs/rulesets/custom.yaml`
> and registers the `custom` profile.

---

## 6. Overriding rules

You can change the severity of an existing rule, or switch it off, per project.
Toggling in the desktop app's Rules tab and saving a snapshot produces a file in
this same format (section 4). What follows is how to write it by hand.

> The field is called `id`. Write `rule_id` and it is silently ignored with no
> error — so if you switched a rule off and it still shows up, check this first.

**① Create the overrides file**: `configs/overrides.yaml`

```yaml
overrides:
  # Change a severity
  - id: "quality-nc-001"
    severity: "low"              # relaxed from high

  # Switch a rule off
  - id: "sql-style-001"
    enabled: false               # ANSI JOIN is fine here

  # Switch several off at once
  - id: "mod-java-001"
    enabled: false               # new Date() allowed (legacy project)
  - id: "mod-java-003"
    enabled: false               # Calendar allowed
```

**② Run it**

```bash
./rulix /path/to/project --profile=all --overrides=configs/overrides.yaml
```

---

## 7. CI/CD

### Jenkins

```groovy
stage('Code Quality') {
    steps {
        sh './rulix ./src --profile=all -o json --output-file=rulix-report.json'
        sh './rulix ./src --profile=all -o excel --output-file=rulix-report.xlsx'
        archiveArtifacts artifacts: 'rulix-report.*'
    }
}
```

### GitLab CI

```yaml
code-quality:
  stage: test
  script:
    - ./rulix ./src --profile=all -o json --output-file=rulix-report.json
    - ./rulix ./src --profile=all -o excel --output-file=rulix-report.xlsx
  artifacts:
    paths:
      - rulix-report.*
```

### A quality gate (fail the build on a critical issue)

```bash
# RULIX exits 1 when it finds issues
./rulix ./src --profile=essential --min-severity=critical
# exit 0 = clean, 1 = there are critical issues
```

---

## 8. Writing the regexes

### Backslashes in YAML

```yaml
# ✅ correct — two backslashes
regex: "System\\.out\\.print"
regex: "\\bSELECT\\b"
regex: "new\\s+Date\\s*\\("

# ❌ wrong — one backslash
regex: "System\.out\.print"
regex: "\bSELECT\b"
```

### The ones you will reach for

| Pattern | Meaning | Example |
|---------|---------|---------|
| `\\.` | a literal dot | `System\\.out` |
| `\\s*` | zero or more spaces | `method\\s*\\(` |
| `\\s+` | one or more spaces | `new\\s+Date` |
| `\\w+` | one or more word characters | `class\\s+\\w+` |
| `\\b` | word boundary | `\\bSELECT\\b` |
| `[^)]*` | anything but `)` | `\\([^)]*\\)` |
| `(?!pattern)` | negative lookahead | `(?!select)\\w+` |
| `(?i)` | case-insensitive | `(?i)select` |
| `[\\s\\S]*?` | anything including newlines, lazy | multiline only |

### regex vs regex-multiline

```yaml
# regex: contained within one line
regex: "@Autowired\\s*$"

# regex-multiline: spans lines (annotation and declaration apart)
type: "regex-multiline"
regex: "@Controller[\\s\\S]*?class\\s+\\w+"
```

> **The rule of thumb**: if your pattern contains `\n` or `[\s\S]`, it must be
> `regex-multiline`. It will never match under `regex`.

---

## 9. Pattern types by language

| Language | regex | regex-multiline | ast-* (7 kinds) | annotation-missing-attr |
|----------|-------|-----------------|-----------------|-------------------------|
| Java | ✅ | ✅ | ✅ | ✅ |
| JavaScript | ✅ | ✅ | ❌ | ❌ |
| HTML | ✅ | ✅ | ❌ | ❌ |
| CSS | ✅ | ✅ | ❌ | ❌ |
| SQL | ✅ | ✅ | ❌ | ❌ |
| XML | ✅ | ✅ | ❌ | ❌ |
| Properties | ✅ | ✅ | ❌ | ❌ |

> AST patterns (`ast-*`) are **Java only**. For other languages use regex or
> regex-multiline.

---

## 10. Troubleshooting

### The desktop app will not start

| Symptom | Cause and fix |
|---------|---------------|
| macOS — "the app is damaged and can't be opened" | Not code-signed yet, so Gatekeeper quarantined it. Run `xattr -dr com.apple.quarantine .` in the folder you extracted. |
| Windows — no window, or a blank one | No WebView2 runtime. Install [WebView2 Runtime](https://developer.microsoft.com/microsoft-edge/webview2/). |
| Linux — there is no `rulix-gui` | We do not build the desktop app for Linux. Use the CLI (`rulix`). |
| The Rules tab is empty | The `configs/` folder must sit next to the executable. Check that you extracted the whole bundle. |

### A rule is not firing

| Check | Fix |
|-------|-----|
| Is `enabled: true`? | `false` means it is skipped |
| Does the pattern contain `\n` under `type: "regex"`? | change it to `regex-multiline` |
| Is the YAML backslash `\.` instead of `\\.`? | write `\\.` |
| Is it an `ast-*` rule on a non-Java file? | `ast-*` is Java only |
| Does the profile include that ruleset? | check `profiles.yaml` |
| Switched it off in an override and it still fires? | the field is `id`, not `rule_id` (section 6) |

### Too many issues

```bash
# Filter by severity
./rulix ./src --profile=all --min-severity=high

# Switch specific rules off in an override
# add enabled: false to configs/overrides.yaml
./rulix ./src --profile=all --overrides=configs/overrides.yaml
```

### Counting issues per rule from the JSON

```bash
./rulix ./src --profile=all -o json --output-file=result.json
python3 -c "
import json
with open('result.json') as f:
    data = json.load(f)
rules = {}
for issue in data.get('issues', []):
    rid = issue.get('rule_id', '')
    rules[rid] = rules.get(rid, 0) + 1
for rid, cnt in sorted(rules.items(), key=lambda x: -x[1]):
    print(f'  {cnt:4d}  {rid}')
print(f'Total: {len(data.get(\"issues\", []))} issues, {len(rules)} rules')
"
```

---

[한국어](QUICK_START.ko.md)
