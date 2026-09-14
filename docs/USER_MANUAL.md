# RULIX user manual

> **RULIX** (Code Quality Analyzer)
> Static analysis for source code — the desktop app and the CLI

---

## 1. Overview

RULIX analyses Java, JavaScript, HTML, CSS, SQL and XML source against development
standards. It comes as a desktop app and a command-line tool, both sitting on the
same rule engine.

**What it does**

- 535 rules built in (453 on by default)
- Desktop app (Windows, macOS) and CLI (Windows, macOS, Linux)
- Rule selections save as a YAML snapshot — CI repeats what you decided in the app
- An editor inside the app — read and fix the source and the ruleset YAML, then re-check
- Rules are added and changed in YAML alone, with no rebuild
- Console, JSON, HTML and Excel reports
- DDL × query cross-analysis for index and schema correctness
- AI audit reports from a local inference server, on an air-gapped network

**Which one to use**

| | Desktop app (`rulix-gui`) | CLI (`rulix`) |
|---|---|---|
| Suited to | reading code, fixing it, deciding the rules, producing a report to hand over | CI pipelines, scheduled scans, batches |
| Platforms | Windows x64, macOS | Windows, macOS, Linux |
| Reports | Excel, HTML, JSON, AI audit, DOCX | console, Excel, HTML, JSON, AI audit, DOCX |

You are not meant to pick one forever. Decide the rules in the app, save the snapshot,
and CI reads that file with `--overrides` (sections 3.4 and 5).

---

## 2. Installing

### 2.1 What is in the bundle

```
rulix-windows-amd64/
  rulix.exe                      ← CLI
  rulix-gui.exe                  ← desktop app (Windows and macOS only)
  rulix-report.exe               ← DOCX audit report generator (Windows and macOS only)
  configs/
    profiles.yaml               ← profile definitions
    rulesets/
      quality.yaml              ← code quality (125 rules, 107 on)
      secure.yaml               ← secure coding (131 rules, 123 on)
      sql.yaml                  ← ANSI SQL (120 rules, 102 on)
      sql-oracle.yaml           ← Oracle specifics (20 rules, 18 on)
      sql-format.yaml           ← SQL formatting and style (7 rules)
      modernize.yaml            ← legacy modernization (50 rules, 39 on)
      spring.yaml               ← Spring conventions (32 rules, 28 on)
      egov.yaml                 ← eGovernment standards (33 rules, 12 on)
      ddl.yaml                  ← DDL × query cross-analysis (17 rules)
```

The Linux bundle holds only `rulix`. Wails needs GTK and WebKit at build time, so we
do not ship a Linux build of the desktop app yet.

If you only want the desktop app, it is published on its own:
`rulix-gui-windows-amd64.exe`, `rulix-gui-darwin-arm64`, `rulix-gui-darwin-amd64`.

### 2.2 Installing on Windows

1. Unzip `rulix-windows-amd64.zip`
2. Put the folder where you want it (for example `C:\tools\rulix\`)
3. Double-click `rulix-gui.exe` for the desktop app
4. Optionally add the folder to PATH so the CLI runs from anywhere

```
System Properties → Environment Variables → Path → Edit → add C:\tools\rulix
```

> The desktop app renders through the Microsoft WebView2 runtime. It ships with
> Windows 11 and with current Windows 10. Without it you get no window, or a blank
> one. In that case install
> [WebView2 Runtime](https://developer.microsoft.com/microsoft-edge/webview2/)
> first.

### 2.3 Installing on macOS

```bash
tar -xzf rulix-darwin-arm64.tar.gz      # on Intel, rulix-darwin-amd64.tar.gz
cd rulix-darwin-arm64

# Not code-signed or notarized yet, so Gatekeeper quarantines it.
# Skip this line and macOS says the app is damaged.
xattr -dr com.apple.quarantine .

./rulix-gui        # desktop app
./rulix --help     # CLI
```

### 2.4 Installing on Linux

```bash
tar -xzf rulix-linux-amd64.tar.gz
cd rulix-linux-amd64
./rulix --help
```

### 2.5 Checking the install

```bash
./rulix profiles
```

A list of profiles means it works. For the desktop app, the Rules tab should be
populated; if it is empty, check that the `configs/` folder was extracted next to
the executable.

---

## 3. Scanning with the desktop app

### 3.1 The screens

Seven tabs run down the left sidebar. `Ctrl/Cmd` and a number moves between them.

| Tab | Shortcut | What it does |
|----|--------|--------|
| Scan | `Ctrl/Cmd+1` | source folder, profiles, minimum severity, DDL cross-analysis; run, cancel, progress |
| Rules | `Ctrl/Cmd+2` | turn rules on and off, change severity, custom rules, save and load snapshots, edit ruleset YAML |
| Results | `Ctrl/Cmd+3` | metrics and grades dashboard, issue list |
| Code | `Ctrl/Cmd+4` | browse the project, read and edit the source, re-check the file you just fixed |
| Export | `Ctrl/Cmd+5` | export Excel, HTML or JSON |
| AI audit | `Ctrl/Cmd+6` | generate an audit report from a local inference server |
| Settings | `Ctrl/Cmd+7` | interface language, theme, AI endpoint defaults |

> Both front ends are English by default. The app takes its language from Settings; the CLI takes `--lang` (`en` / `ko`) or `RULIX_LANG`.

### 3.2 Running a scan

Fill the Scan tab in order.

1. **Source folder** — pick the source root with **Browse**. You can also type the
   path.
2. **Profiles** — check the profiles to run. You can pick several. Section 4.2 has
   the table of what each one covers.
3. **Minimum severity** — low and up, medium and up, high and up, or critical only.
   Anything below it stays out of the results.
4. **DDL cross-analysis** (optional) — point at a DDL file or folder and RULIX also
   checks table and column existence and index usage in your SQL.

Progress is shown while it runs, and a long scan on a large project can be
cancelled. When it finishes you land on the Results tab.

### 3.3 Reading the results

The top of the Results tab is the metrics and grades dashboard: size (lines,
comment ratio), cyclomatic complexity, duplication, technical debt and an A–E grade.
This is where quality relative to size shows up — something a violation count alone does not tell you.

Below it is the issue list. Filter by severity and rule; click an issue and the file
opens at that line.

### 3.4 Tuning rules, and snapshots

The Rules tab is where you fit the rules to this project.

- **On and off** — one toggle. You can also switch a whole profile at once.
- **Severity** — change it per rule when the default does not match your site.
- **Custom rules** — register a team convention as a regex. Test it against sample
  code before saving, so a wrong pattern does not get in.

When you are done, **save the snapshot** as `ruleset.yaml`. That file is exactly what
the CLI's `--overrides` reads, so the rules you switched and the severities you
changed are reproduced on every build.

**What the pattern looks at** — the form lets you choose. *Per line* examines each
line on its own; *whole file* reads the file as one string. A rule whose annotation
and declaration sit on different lines (eGovernment naming rules, for instance) only
matches as whole-file. The tester judges with the type you picked, so a pattern that
matches there never comes back empty in the scan. Exclusions and the fix hint are on the form too.

**Seeing a built-in rule's definition** — press `</>` on a built-in rule and the
ruleset file that defines it opens at that line.

**Editing ruleset files** — types the form cannot build (the `ast-*` family and so on) are written directly in YAML.
For `regex-multiline`, the `ast-*` family and anything else the form does not cover,
open **Edit ruleset files** under Browse. Before saving it validates with the loader
the CLI uses and the regex compiler the rule engine uses, marking duplicate IDs,
invalid severities and patterns that will not compile with line numbers. A file that
fails validation is not written, and the previous content is kept as a `.bak` beside it.

**Exporting custom rules** — rules you build in the app live in `~/.rulix/custom_rules.yaml`,
which the CLI does not read. **Export custom rules** writes them to
`configs/rulesets/custom.yaml` and registers a `custom` profile, so
`rulix <path> --profile=custom` runs the same rules in CI.

```bash
rulix /path/to/project --profile=all --overrides=ruleset.yaml \
  -o excel --output-file=report.xlsx
```

It works the other way too: load a file made with the CLI's `--save-overrides` into
the app and the same state appears on screen. Section 5 has the file format.

### 3.5 Exporting reports

The Export tab produces Excel, HTML and JSON.

| Format | What it is for |
|------|----------|
| Excel | handing over to a client or a vendor; full data across sheets |
| HTML | a single file that opens in a browser |
| JSON | CI, importing into another tool, and the input to the AI audit report |

What you produce stays in the **Recently saved** list so you can open it again.

### 3.6 Reading and editing code

The Code tab (`Ctrl/Cmd+4`) is where you browse the project and read the source. The
editor is part of the app, so you do not have to bring one to the site.

- **Finding a file** — the box above the tree filters by name. On a project with
  thousands of files, opening folders one at a time is the most common waste of
  time in a consulting engagement.
- **Opening** — pick a file in the tree on the left, or press **Open in code** on an
  issue and that file opens at that line. Every finding in the file is marked in the gutter and the minimap.
- **Moving between findings** — the arrows above the editor step through them, and
  `F8` / `Shift+F8` do the same. The counter reads `3 of 4 lines`, so you know
  where you are. Several findings on one line count as one stop.
- **Browsing** — files the rules do not look at appear in the tree too. In consulting
  you read the configuration and the build scripts as well, not only what the rules
  cover. Dependency folders like `node_modules` are left out.
- **Indentation** — new input follows the file you have open. RULIX checks tabs
  and spaces as a rule, so an editor that inserted the wrong one would create the
  very violation you came to fix. Existing content is never reformatted.
- **Editing** — read-only until you press **Edit**. Save with
  `Ctrl/Cmd+S`.
- **Re-check** — saving re-checks **that one file** immediately. You do not re-scan
  a large project just to see whether the fix took. Cross-file rules are the
  exception: they need the whole project, so they sit out a single-file re-check.

Saving preserves the file's line endings (CRLF/LF), its trailing newline and its
permissions. Write a CRLF file back as LF and a one-line change shows up as the
whole file having changed. If something else changed the file while you were editing,
it is not overwritten — you are told.

> Files over 2MB and binaries are not opened. Nothing outside the project folder you
> have open is read or written.

### 3.7 When the results go stale

Turn a rule off, change a severity or edit the source, and the results on screen are
from before that. A line saying so appears above the Results, Export and AI audit
screens, with **Rescan** next to it.

Nothing is recomputed automatically. On a large project, pausing for seconds every
time you toggle a rule makes the rules impossible to tune. What it does instead is
make sure **you never build a report from numbers that no longer hold**.

### 3.8 Moving between screens

A screen you entered by pressing a button carries a way back. Open an issue with
**Open in code** and `← Back to results · secure-plain-001` appears at the top;
pressing it returns you to the issue list with your filters intact.

Screens you reach from the sidebar do not show it — that is going somewhere, not
coming back. Visited screens are tracked separately: `Ctrl/Cmd+[` and `Ctrl/Cmd+]`
move through them.

Drilling into the issue list from the dashboard shows the active filter as a chip
you can clear with `✕`.

### 3.9 AI audit reports

The AI audit tab turns a scan into a written report. Enter the endpoint and model,
press **Test connection** to confirm the server answers, then generate. Air-gapped
mode is a checkbox. The same tab also produces the DOCX report for submission.
Section 10 has the details.

---

## 4. Scanning from the command line

### 4.1 Scanning source

```cmd
REM default scan (every rule)
rulix.exe C:\projects\my-app\src

REM one profile
rulix.exe C:\projects\my-app\src --profile=quality
rulix.exe C:\projects\my-app\src --profile=sql
rulix.exe C:\projects\my-app\src --profile=secure

REM several profiles
rulix.exe C:\projects\my-app\src --profile=quality,secure,sql
```

### 4.2 The profiles

| Profile | Rules | On by default | Languages | What it covers |
|----------|---------|-----------|-----------|------|
| `quality` | 125 | 107 | Java, JS, HTML, CSS | code quality, naming, complexity |
| `secure` | 131 | 123 | Java, JS | secure coding |
| `sql` | 120 | 102 | SQL, XML, Java | ANSI SQL, whatever the database |
| `sql-oracle` | 20 | 18 | SQL, XML, Java | Oracle specifics (NVL, ROWNUM, hints) |
| `sql-format` | 7 | 7 | SQL, XML, Java | SQL formatting (uppercase, 80 columns) |
| `modernize` | 50 | 39 | Java, XML | eGov 3.x→4.x, javax→jakarta |
| `spring` | 32 | 28 | Java | Spring and Boot conventions |
| `egov` | 33 | 12 | Java, XML | eGovernment Framework standards |
| `ddl` | 17 | 17 | SQL, XML, Java | DDL × query cross-analysis (needs `--ddl`) |

**Picking the SQL profiles**

| Your database | Use | Command |
|---------|-------------|--------|
| MySQL / PostgreSQL / CUBRID | `sql` | `--profile=sql` |
| Oracle | `sql,sql-oracle` | `--profile=sql,sql-oracle` |
| Oracle, with formatting enforced | `sql-all` (group) | `--profile=sql-all` |
| Any database, with formatting enforced | `sql,sql-format` | `--profile=sql,sql-format` |

**Profile groups**

| Group | Profiles | What it is for |
|------|-------------|------|
| `all` | all eight (not ddl) | everything, Oracle and formatting included |
| `essential` | quality, secure | the must-haves |
| `sql-all` | sql, sql-oracle, sql-format | all of SQL, Oracle and formatting included |
| `sql-ddl` | sql, sql-oracle, ddl | SQL plus schema cross-checks |
| `egov-full` | quality, secure, sql, sql-oracle, sql-format, egov, spring | the full eGovernment set |
| `migration` | modernize, spring | migration assessment |

### 4.3 Output formats

```cmd
REM console (default)
rulix.exe C:\src --profile=sql

REM HTML report
rulix.exe C:\src --profile=all -o html --output-file=report.html

REM Excel report
rulix.exe C:\src --profile=all -o excel --output-file=report.xlsx

REM JSON output
rulix.exe C:\src --profile=all -o json --output-file=report.json
```

**The Excel report's sheets**

| Sheet | Contents |
|--------|------|
| Summary | overall figures: files, issues, severity breakdown |
| Issues | every issue in full, filterable and sortable |
| Files | issues per file |
| Rules | the rules that ran, and how often each fired |

### 4.4 Filtering by severity

```cmd
REM high and above
rulix.exe C:\src --profile=all --min-severity=high

REM critical only
rulix.exe C:\src --profile=all --min-severity=critical
```

The order: `low` < `medium` < `high` < `critical`

### 4.5 Verbose mode

```cmd
rulix.exe C:\src --profile=sql -v
```

Shows the profiles loaded, the number of files processed, the total issue count and more.

### 4.6 A per-rule summary table (for CI)

```cmd
rulix.exe C:\src --profile=all --summary
```

Prints a table of violations per rule ID. Useful for a quick read in a CI pipeline.

### 4.7 Excluding specific rules

```cmd
REM exclude rule IDs without an overrides file
rulix.exe C:\src --profile=all --exclude-rule=quality-lg-005,secure-sql-002
```

### 4.8 Cross-file analysis only

```cmd
REM only the architectural findings: unused MyBatis SQL IDs, Mapper/VO mismatches and so on
rulix.exe C:\src --profile=all --cross-file-only
```

### 4.9 Merging duplicate findings

Combining profiles can make several rules report the same thing. By default RULIX merges findings that land on the same file and line.

```cmd
REM default: merged — one finding per location, at the highest severity
rulix.exe C:\src --profile=all

REM no merging: every finding shown separately, for debugging a rule
rulix.exe C:\src --profile=all --no-dedup
```

---

## 5. Overriding rules

You can change a rule's severity, or switch it off, to fit the site.

This is the same format the desktop app's Rules tab writes with **Save snapshot**
(section 3.4). The CLI reads what you toggled in the app, and the app loads a file
made with the CLI's `--save-overrides`.

> The field that names the rule is `id`. Write `rule_id` and it is silently ignored
> with no error — so if you switched a rule off and it still fires, check this first.

### 5.1 Writing the overrides file

Create `overrides.yaml`:

```yaml
overrides:
  # switch a rule off
  - id: "sql-fmt-001"
    enabled: false

  # change a severity (high → low)
  - id: "sql-smell-017"
    severity: "low"

  # switch several off at once
  - id: "sql-from-001"
    enabled: false
  - id: "sql-dml-001"
    enabled: false
```

### 5.2 Applying the overrides

```cmd
rulix.exe C:\src --profile=sql --overrides=overrides.yaml
```

---

## 6. The rules in detail

### 6.1 How the SQL profiles are split

The SQL rules are split across three profiles, by what your database is:

#### sql (ANSI SQL, vendor independent — 88 rules)

Rules that hold whatever the database vendor is.
They apply on MySQL, PostgreSQL, CUBRID, Oracle — all of them.

- SELECT *, INSERT with no column list, UPDATE/DELETE with no WHERE
- subqueries, aliases, join conditions
- code smells: NATURAL JOIN, ON 1=1, missing parentheses around AND/OR
- MyBatis: ${} SQL injection, SELECT *, missing WHERE

#### sql-oracle (Oracle only — 19 rules)

Rules that only mean something on Oracle.

| Area | Examples |
|------|----------|
| Oracle hints | `/*+ HINT */` without sign-off; only LEADING allowed |
| Oracle functions | NVL, TO_CHAR, DECODE, DBMS_RANDOM, INSTR, REGEXP_LIKE |
| Oracle syntax | ROWNUM paging, EXECUTE IMMEDIATE, FOR UPDATE WAIT |

#### sql-format (formatting and style — 7 rules)

Formatting rules you switch on if your team wants them.

| ID | Rule | What it checks |
|----|--------|------|
| sql-fmt-001 | SQL keywords uppercase | SELECT, FROM and the rest in uppercase |
| sql-fmt-002 | 80-column limit | lines longer than 80 characters |
| sql-fmt-003 | no blank lines in SQL | a blank line in the middle of a statement |
| sql-sel-004 | one column per line | list one column per line |
| sql-from-005 | one table per line | list one table per line |
| sql-where-007 | one condition per line | write one condition per line |
| sql-sel-006 | comment style | only `/* */` allowed |

### 6.2 SQL code smells

#### Common smells (in sql.yaml)

| ID | Rule | Severity | Why |
|----|--------|--------|------|
| sql-smell-010 | comparing to NULL with = | high | `col = NULL` is always UNKNOWN; use IS NULL |
| sql-smell-011 | NATURAL JOIN | high | adding a column silently changes the join |
| sql-smell-012 | fake join, ON 1=1 | high | a cartesian product dressed up as a join |
| sql-smell-013 | aliasing to itself | low | `COL AS COL` says nothing |
| sql-smell-017 | COUNT(DISTINCT) | low | costs a sort or hash over a lot of rows |
| sql-smell-018 | WHERE 1=1 | low | a leftover of dynamic SQL; clean it up |
| sql-smell-020 | DISTINCT with GROUP BY | high | GROUP BY already guarantees uniqueness |
| sql-smell-021 | INSERT INTO … ORDER BY | high | ORDER BY on an INSERT does nothing |
| sql-smell-023 | DISTINCT with JOIN | high | hides duplicates caused by a wrong join |
| sql-smell-024 | LEFT JOIN with COUNT(*) | high | it counts the NULL rows too |
| sql-smell-025 | repeated OR on one column | low | use an IN clause instead |
| sql-smell-026 | nested CTE | high | a WITH inside a WITH; split them into sequential CTEs |
| sql-smell-027 | AND/OR without parentheses | high | precedence mistakes waiting to happen |

#### Oracle-only smells (in sql-oracle.yaml)

| ID | Rule | Severity | Why |
|----|--------|--------|------|
| sql-smell-003 | NVL/COALESCE around a WHERE column | high | the index cannot be used |
| sql-smell-014 | ORDER BY DBMS_RANDOM | high | forces a full table scan and a sort |
| sql-smell-015 | INSTR in WHERE | high | a string function on the column defeats the index |
| sql-smell-016 | REGEXP_LIKE | low | for a simple pattern, LIKE is cheaper |
| sql-smell-022 | NVL/TO_CHAR in a JOIN ON | high | a function in the join condition defeats the index |

#### From the AST (6 rules, all databases)

| ID | Rule | Severity | Why |
|----|--------|--------|------|
| sql-smell-030 | WHERE filter on a LEFT JOIN | high | makes the LEFT JOIN an INNER JOIN in practice |
| sql-smell-031 | INNER JOIN after a LEFT JOIN | high | the INNER JOIN cancels the LEFT JOIN |
| sql-smell-032 | non-aggregate condition in HAVING | high | move it to WHERE and it runs faster |
| sql-smell-033 | unused CTE | high | defined in WITH and never referenced |
| sql-smell-034 | ORDER BY in a subquery | high | sorting without paging buys nothing |
| sql-smell-035 | inline view with no alias | high | without an alias you cannot reference it |

### 6.3 The rule types

| Type | What it does | What you have to change |
|------|------|-----------|
| `regex` | match a pattern line by line | YAML only |
| `regex-multiline` | match across lines | YAML only |
| `ast-filtered-regex` | regex with comment lines filtered out (ANTLR4 tokens) | YAML only |
| `ast-method-call` | method calls, from the Java AST | YAML only |
| `ast-annotation` | annotations, from the Java AST | YAML only |
| `ast-import` | import paths, from the Java AST | YAML only |
| `ast-variable` | variables and fields, from the Java AST | YAML only |
| `ast-try-catch` | exception handling, from the Java AST | YAML only |
| `ast-class` | class shape, from the Java AST | YAML only |
| `ast-method` | method shape, from the Java AST | YAML only |
| `ast-dead-code` | dead code, from the Java AST | YAML only |
| `ast-multi-cud` | several CUD calls without a transaction | YAML only |
| `cross-file` | cross-file analysis (unused MyBatis SQL IDs and the like) | YAML only |
| `annotation-missing-attr` | required annotation attributes | YAML only |
| `line-length` | line length | YAML only |
| `ast-sql-*` | SQL structure, from the SQL AST | Go code plus YAML |
| `method-analysis` | analysis written in Go | Go code |

### 6.4 How the rule engines differ

RULIX's rules fall into four kinds, by what they are given and how they match.

#### regex (one line at a time)

```
input: file.Lines → walked line by line
match: pattern.MatchString(line)
```

- each line of the file is matched **on its own**
- `.` does not match a newline (`\n`)
- the fastest and simplest of the four
- **limits**: cannot see a pattern that spans lines, and can misfire inside comments

#### regex-multiline (the whole file)

```
input: strings.Join(file.Lines, "\n") → the whole file as one string
match: pattern.FindStringMatch(content), then FindNextMatch
```

- every line is **joined with `\n`** into a single string
- `.` matches `\n` too (singleline mode)
- can see a **pattern that spans lines**, such as `SELECT\s+\*\s+FROM`
- the line number is worked back from the match offset
- **limits**: can misfire inside comments, and is slower on a large file

#### ast-filtered-regex (regex, comments removed)

```
input: file.Lines → line by line, as with regex
plus: an ANTLR4 pass maps the comment lines, and those lines are skipped
```

- behaves like regex otherwise (line by line)
- **the difference**: the ANTLR4 token stream **filters out comment-only lines**
- so a comment like `// no System.out.println` does not fire the rule
- on a non-Java file, or one with no AST, it behaves exactly like plain regex

#### AST queries (ast-method-call, ast-class, ast-annotation and the rest)

```
input: the ANTLR4 parse tree
match: FindMethodInvocations(), FindAnnotations(), FindClassDeclarationsInfo() and so on
     → conditions are checked against structured data
```

- walks **parsed structures, not text**
- identifies method calls, annotations, classes, variables and try-catch structurally
- supports context conditions: `inside-loop`, `inside-catch`, `method-name:<regex>`
- misfiring inside a comment or a string is structurally impossible
- **limit**: Java only (it needs the ANTLR4 parser)

#### Side by side

| | regex | regex-multiline | ast-filtered-regex | AST query |
|------|-------|-----------------|-------------------|----------|
| **Input unit** | one line at a time | the whole file (joined) | one line at a time | the ANTLR4 parse tree |
| **How it matches** | text regex | text regex | regex plus a comment filter | comparing structural properties |
| **Multiline patterns** | no | **yes** | no | n/a (structural) |
| **Comment misfires** | not handled | not handled | **removed automatically** | **removed structurally** |
| **Context conditions** | none | none | none | inside-loop, inside-catch and so on |
| **Timeout** | 100 ms | 500 ms | 100 ms | none (it walks the tree) |
| **Languages** | all | all | Java (falls back to regex elsewhere) | Java only |
| **In YAML** | `type: "regex"` | `type: "regex-multiline"` | `type: "ast-filtered-regex"` | `type: "ast-method-call"` and so on |
| **Best for** | simple keyword detection | multiline patterns | when comments cause misfires | method, class and structural analysis |

#### The same rule written three ways

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

| Source | regex | ast-filtered-regex | ast-method-call |
|----------|-------|-------------------|-----------------|
| `System.out.println("hello");` | found | found | found |
| `// System.out.println is banned` | misfire | skipped | skipped |
| `/* System.out.println */` | misfire | skipped | skipped |
| `String s = "System.out.println";` | misfire | misfire | skipped |

> **Which to use**: a simple keyword → `regex`; misfires on comments → `ast-filtered-regex`; structural accuracy → an `ast-*` type. A pattern that spans lines can only be `regex-multiline`.

---

## 7. Adding rules

### 7.1 A regex rule (the simplest)

Add it to `configs/rulesets/sql.yaml` in this shape:

```yaml
- id: "sql-custom-001"
  name: "A rule of your own"
  severity: "high"           # low, medium, high, critical
  category: "performance"
  description: "What this rule is about"
  enabled: true
  pattern:
    type: "regex"
    regex: "\\bFORBIDDEN_KEYWORD\\b"
    flags: "i"               # i = case-insensitive
  custom:
    rule_id: "SQL-CUSTOM-001"
    fix: "How to fix it"
```

**Things to watch**

- in YAML a `\` must be written `\\`
- `\b` word boundary, `\s` whitespace, `\w` word character
- `(?:...)` is a non-capturing group
- put `flags: "i"` under `custom:` for case-insensitive matching

### 7.2 A regex-multiline rule

For a pattern that spans lines:

```yaml
- id: "sql-custom-002"
  name: "Unnecessary pattern after SELECT"
  severity: "high"
  category: "performance"
  description: "A particular pattern was found in a SELECT"
  enabled: true
  pattern:
    type: "regex-multiline"
    regex: "\\bSELECT\\b[\\s\\S]*?\\bFORBIDDEN\\b"
    flags: "i"
  custom:
    rule_id: "SQL-CUSTOM-002"
    fix: "How to fix it"
```

**The key pieces**

- `[\\s\\S]*?` — any character including newlines, lazy
- `[\\s\\S]*` — any character including newlines, greedy
- too broad a pattern misfires; prefer the lazy `*?`

### 7.3 Switching a rule off

Either set `enabled: false` in the YAML, or use an overrides file:

```yaml
# option 1: edit sql.yaml directly
- id: "sql-smell-016"
  enabled: false       # ← off

# option 2: overrides.yaml, leaving the original alone
overrides:
  - id: "sql-smell-016"
    enabled: false
```

### 7.4 Adding a profile

Add it to `configs/profiles.yaml`:

```yaml
profiles:
  # the existing profiles …

  my-rules:
    name: "Our project rules"
    description: "A ruleset for this project alone"
    ruleset: "rulesets/my-rules.yaml"
    languages:
      - java
      - sql

groups:
  # the existing groups …

  my-project:
    name: "Everything for our project"
    description: "The full scan for this project"
    profiles:
      - quality
      - secure
      - my-rules
```

Then create `configs/rulesets/my-rules.yaml`.

### 7.5 Creating a ruleset file

```yaml
version: "1.0"
profile: "my-rules"

languages:
  - language: java
    rules:
      - id: "my-java-001"
        name: "System.exit() prohibited"
        severity: "critical"
        category: "restriction"
        description: "Do not call System.exit()."
        enabled: true
        pattern:
          type: "regex"
          regex: "\\bSystem\\.exit\\s*\\("
        custom:
          rule_id: "MY-JAVA-001"
          fix: "Throw an exception instead of calling System.exit()"

  - language: sql
    rules:
      - id: "my-sql-001"
        name: "TRUNCATE prohibited"
        severity: "critical"
        category: "restriction"
        description: "TRUNCATE TABLE is prohibited."
        enabled: true
        pattern:
          type: "regex"
          regex: "\\bTRUNCATE\\s+TABLE\\b"
          flags: "i"
        custom:
          rule_id: "MY-SQL-001"
          fix: "TRUNCATE needs DBA sign-off"
```

---

## 8. Worked examples

### 8.1 Scanning only the SQL

```cmd
rulix.exe C:\projects\sql-files --profile=sql -o excel --output-file=sql_report.xlsx
```

### 8.2 In a CI pipeline

```cmd
REM fail the build when there is a critical issue (exit code 1)
rulix.exe C:\src --profile=essential --min-severity=critical
if %ERRORLEVEL% NEQ 0 (
    echo "Critical issues found — failing the build"
    exit /b 1
)
```

### 8.3 A batch file

Create `run_rulix.bat`:

```bat
@echo off
SET RULIX_HOME=C:\tools\rulix
SET TARGET=%1

if "%TARGET%"=="" (
    echo Usage: run_rulix.bat [path-to-scan]
    exit /b 1
)

echo === RULIX source code scan ===
echo Target: %TARGET%
echo.

%RULIX_HOME%\rulix.exe %TARGET% ^
    --profile=all ^
    --profiles-file=%RULIX_HOME%\configs\profiles.yaml ^
    -o excel ^
    --output-file=rulix_report.xlsx ^
    --min-severity=low ^
    -v

echo.
echo Report written: rulix_report.xlsx
pause
```

Run it:
```cmd
run_rulix.bat C:\projects\my-app\src
```

### 8.4 Including MyBatis XML

The SQL profiles check the SQL inside MyBatis XML (`*Mapper.xml`) as well as `.sql` files.

```cmd
REM scan both the .sql files and the MyBatis .xml under src
rulix.exe C:\projects\my-app\src --profile=sql
```

---

## 9. Every option

| Option | Short | Default | What it does |
|------|------|--------|------|
| `--profile` | `-p` | `all` | profiles to run, comma-separated |
| `--config` | `-c` | - | a config file directly (cannot be combined with --profile) |
| `--profiles-file` | - | `configs/profiles.yaml` | where the profile definitions live |
| `--overrides` | - | - | a rule overrides file (the desktop app's snapshot format) |
| `--save-overrides` | - | - | write the effective rule set to a YAML file |
| `--ddl` | - | - | a DDL file or directory; giving it switches cross-analysis on |
| `--lang` | - | `en` | output language, `en` or `ko` (or `RULIX_LANG`) |
| `--output` | `-o` | `console` | output format: console, json, html, excel |
| `--output-file` | - | stdout | where to write it |
| `--min-severity` | `-s` | `low` | the lowest severity to report |
| `--rules` | - | - | rule categories to check, comma-separated |
| `--exclude-rule` | - | - | rule IDs to leave out, comma-separated |
| `--summary` | - | false | print a per-rule summary table, for CI |
| `--cross-file-only` | - | false | only the cross-file findings, for an architecture review |
| `--no-dedup` | - | false | keep duplicate findings instead of merging them |
| `--verbose` | `-v` | false | verbose output |

**Subcommands**

| Command | What it does |
|--------|------|
| `profiles` | list the available profiles |
| `profiles -v` | list them with details |
| `ai-report` | build an AI audit report from a scan result (section 10) |
| `report` | merge Excel scan results into an audit report |

---

## 10. AI audit reports (air-gap capable)

Takes a scan result in JSON and writes an audit report as HTML. Any OpenAI-compatible
endpoint works, so an **in-house inference server — Ollama, vLLM, LM Studio — can be
used as it is, with no internet connection.**

```bash
# 1) save the scan result as JSON
rulix /path/to/src --profile=all -o json --output-file=scan.json

# 2) build the report through your own server
rulix ai-report scan.json --offline \
    --endpoint http://127.0.0.1:11434/v1 \
    --model qwen2.5-coder:7b \
    --project "Example System" \
    -o report.html
```

### Options

| Option | Default | What it does |
|------|--------|------|
| `--output` / `-o` | `ai-report.html` | where to write the HTML |
| `--provider` | `openai` | `openai` (any compatible endpoint), `claude`, `gemini` |
| `--endpoint` | OpenAI's own | the OpenAI-compatible endpoint to use |
| `--model` | the provider's default | model name |
| `--api-key` | - | API key; a local server accepts anything |
| `--project` | the input file name | the system's name, as it appears in the report |
| `--lang` | `ko` | `ko` / `en` |
| `--offline` | false | **air-gapped mode**; blocks every connection outside loopback |
| `--timeout` | `5m` | timeout per request |

### `--offline` — air-gapped mode

With `--offline`, RULIX **refuses every connection that leaves loopback (127.0.0.1,
::1).** It judges by the destination IP, so renaming the host does not get around it,
and DNS resolution is blocked too. A proxy setting is no way out either.

Point it at an external address and no report is produced: it fails, and says why.

```
$ rulix ai-report scan.json --offline --endpoint https://api.openai.com/v1
🔒 Air-gapped mode: every connection that leaves loopback (127.0.0.1) is blocked
Error: ... Air-gapped mode blocked an outbound connection. Only loopback addresses are allowed.
```

There is a procedure for checking this yourself:
[Verifying air-gapped operation](AIRGAP_VERIFICATION.md).

The `claude` and `gemini` providers are external APIs, so they cannot be combined with `--offline`.

### What goes into the report

| Part | Who produces it |
|------|------------|
| Choosing the key findings | RULIX — by severity and category; the same input always gives the same set |
| File names, line numbers, rule IDs | RULIX — straight from the scan; the model cannot change them |
| Quality score and A–E grades | RULIX — computed from the debt ratio and the worst security severity |
| The explanation and the improved code | the model |
| Overview, risks, roadmap, conclusion | the model |

If the model invents a file name or a rule ID that does not exist, that item is
rejected and replaced with the rule's registered description. The footer records what
actually generated it — the kind of endpoint and the model (`local:qwen2.5-coder:7b`, say).

### Which model

Measurements from an air-gapped environment are in [Local LLM evaluation](LOCAL_LLM_EVALUATION.md).
At the 7B size, `qwen2.5-coder:7b` held up best.

### Doing it in the app

The desktop app's **AI audit** tab (`Ctrl/Cmd+6`) does the same thing.
Run a scan first; the tab needs a result.

| Field | What it is |
|------|------|
| Organization / system | used in the report title |
| Generated by | on-premise inference server (OpenAI-compatible), Claude, or Gemini |
| Endpoint | your server's address; defaults to `http://127.0.0.1:11434/v1` |
| Model | the model name; after a connection check you pick from what the server reports |
| Air-gapped mode | on by default; limits connections to loopback while the report is written |

Press **Test connection** first and the address and model name are checked up front.
The commonest failure in an air-gapped install is a typo in one of those two, and this
way you find out immediately instead of after a two-minute wait. If the server returns
a model list, the field becomes a picker, and you are told if what you typed is not on it.

Progress is shown step by step while it runs, and you can cancel. You are asked where
to save before generation starts.

Choosing an external API (Claude, Gemini) switches air-gapped mode off — the two
cannot be combined.

The app's air-gapped mode applies to **the report generation path**. Blocking every
connection the process makes is what the CLI's `--offline` does. The verification
procedure is written against the CLI: [Verifying air-gapped operation](AIRGAP_VERIFICATION.md).

> **The DOCX report is a separate path.** The DOCX section lower down the same screen
> builds a Word document through a separate generator, and its AI body supports only
> the Anthropic API. On an air-gapped network only the plain-text option works.

---

## 11. Troubleshooting

### The desktop app will not start

| Symptom | Cause and fix |
|------|------------|
| macOS — "the app is damaged and can't be opened" | Not code-signed or notarized yet, so Gatekeeper quarantined it. Run `xattr -dr com.apple.quarantine .` in the folder you extracted, then open it again. |
| Windows — no window, or a blank one | No Microsoft WebView2 runtime. Install [WebView2 Runtime](https://developer.microsoft.com/microsoft-edge/webview2/). |
| Linux — there is no `rulix-gui` | We do not build the desktop app for Linux. Use the CLI (`rulix`). |
| The Rules tab is empty | The `configs/` folder must sit next to the executable. Check you extracted the whole bundle. |
| macOS — the folder picker closes at once in fullscreen | We have not identified the cause yet. Pick the folder in windowed mode and then go fullscreen, or paste the path into the field. |
| Generation fails on the AI audit tab | Press Test connection first to check the endpoint and model name. In air-gapped mode an external address is blocked (section 10). |

### Profiles will not load

```
Could not load the profiles: open configs/profiles.yaml: no such file or directory
```

Point `--profiles-file` at the right path:

```cmd
rulix.exe C:\src --profiles-file=C:\tools\rulix\configs\profiles.yaml --profile=sql
```

### Mangled characters in the console

```cmd
chcp 65001
rulix.exe C:\src --profile=sql
```

Or use the HTML or Excel output, where the characters always come out right.

### It found no files to scan

RULIX collects files by extension:

| Profile | Extensions |
|----------|------------|
| quality | `.java`, `.js`, `.html`, `.css` |
| secure | `.java` |
| sql | `.sql`, `.xml` |
| modernize | `.java`, `.xml` |

---

## Appendix: the rule counts

| Profile | Total | On | Main checks |
|----------|------|------|---------------|
| quality | 125 | 107 | naming, complexity, code smells, layering, comments |
| secure | 131 | 123 | SQL injection, XSS, crypto, authentication and authorization, error handling |
| sql | 120 | 102 | ANSI SQL: SELECT, JOIN, WHERE, DML, code smells |
| sql-oracle | 20 | 18 | Oracle: NVL, TO_CHAR, hints, ROWNUM, DECODE |
| sql-format | 7 | 7 | SQL formatting: uppercase keywords, line length, one column per line |
| modernize | 50 | 39 | eGov migration, javax → jakarta, deprecated APIs |
| spring | 32 | 28 | DI patterns, @Transactional, REST APIs, security |
| egov | 33 | 12 | layer naming, Service/Mapper, common components |
| ddl | 17 | 17 | table and column existence, index usage, DDL improvements |
| **Total** | **535** | **453** | |

> **On** means `enabled: true` — only those run. The rest are switched off because they overlap with another profile, or because they encode a preference rather than a defect.

---

[한국어](USER_MANUAL.ko.md)
