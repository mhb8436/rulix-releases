# RULIX 퀵스타트 — 데스크톱 앱과 CLI

RULIX는 같은 룰 엔진 위에 데스크톱 앱과 명령행 도구를 얹은 구조입니다. 처음
보는 프로젝트를 훑고 적용할 규칙을 정할 때는 앱이 빠르고, 그 결정을 매 빌드마다
반복하는 건 CLI가 합니다. 둘은 규칙 스냅샷 파일 하나로 이어집니다(4장).

---

## 1. 설치

### 받을 파일

압축만 풀면 끝입니다. JVM도 Node.js도 파이썬도 설치할 필요가 없습니다.

| 플랫폼 | 파일 | 들어 있는 것 |
|---|---|---|
| Windows x64 | `rulix-windows-amd64.zip` | `rulix.exe`, `rulix-gui.exe`, `rulix-report.exe` |
| macOS Apple Silicon | `rulix-darwin-arm64.tar.gz` | `rulix`, `rulix-gui`, `rulix-report` |
| macOS Intel | `rulix-darwin-amd64.tar.gz` | `rulix`, `rulix-gui`, `rulix-report` |
| Linux x64 / ARM64 / x86 | `rulix-linux-*.tar.gz` | `rulix` |
| Windows x86 | `rulix-windows-386.zip` | `rulix.exe` |

데스크톱 앱만 필요하면 `rulix-gui-windows-amd64.exe`, `rulix-gui-darwin-arm64`,
`rulix-gui-darwin-amd64` 가 단독 파일로도 올라갑니다.

> **Linux용 데스크톱 앱은 없습니다.** Wails가 빌드 시점에 GTK·WebKit을
> 요구하는데 아직 제공하지 않습니다. Linux에서 RULIX는 CLI입니다.

### Windows

압축을 풀고 `rulix-gui.exe` 를 더블클릭합니다. 화면은 Microsoft WebView2
런타임으로 그리는데 Windows 11과 최근 Windows 10에는 기본으로 들어 있습니다.
오래된 장비라면 [WebView2 Runtime](https://developer.microsoft.com/microsoft-edge/webview2/)을
먼저 설치하세요.

### macOS

```bash
tar -xzf rulix-darwin-arm64.tar.gz
cd rulix-darwin-arm64

# 코드 서명을 아직 하지 않아 Gatekeeper가 격리합니다.
# 이 줄을 건너뛰면 "손상된 앱"이라고 뜹니다.
xattr -dr com.apple.quarantine .

./rulix-gui
```

### Linux

```bash
tar -xzf rulix-linux-amd64.tar.gz
cd rulix-linux-amd64
./rulix --version
```

### 배포 구조

```
rulix[.exe]                 ← CLI
rulix-gui[.exe]             ← 데스크톱 앱 (Windows / macOS)
rulix-report[.exe]          ← DOCX 감리 보고서 생성기 (Windows / macOS)
configs/
├── profiles.yaml          ← 프로파일 정의
└── rulesets/
    ├── quality.yaml       ← 코드 품질 (125 규칙)
    ├── secure.yaml        ← 시큐어코딩 (131 규칙)
    ├── sql.yaml           ← ANSI SQL (120 규칙)
    ├── sql-oracle.yaml    ← Oracle 전용 (20 규칙)
    ├── sql-format.yaml    ← SQL 포맷 (7 규칙)
    ├── modernize.yaml     ← 레거시 현대화 (50 규칙)
    ├── spring.yaml        ← Spring 표준 (32 규칙)
    ├── egov.yaml          ← 전자정부 표준 (33 규칙)
    └── ddl.yaml           ← DDL × 쿼리 교차 분석 (17 규칙)
```

바이너리와 `configs/` 폴더만 있으면 동작합니다.

---

## 2. 데스크톱 앱으로 5분

앱을 띄우면 왼쪽에 탭 여섯 개가 있습니다. `Ctrl/Cmd` + 숫자로도 이동합니다.

**① 점검** (`Ctrl/Cmd+1`)

[찾아보기]로 소스 폴더를 고르고, 돌릴 프로파일을 체크하고, 최소 심각도를
정한 다음 실행합니다. DDL 파일이나 폴더를 지정하면 SQL 교차 분석이 함께
돕니다(선택). 진행률이 나오고 도중에 취소할 수 있습니다.

**② 결과** (`Ctrl/Cmd+3`)

지표와 등급 대시보드가 먼저 보이고 그 아래가 이슈 목록입니다. 이슈를 누르면
해당 파일의 그 줄이 열립니다.

**③ 규칙** (`Ctrl/Cmd+2`)

이 프로젝트에 안 맞는 규칙을 끄고, 심각도를 조정합니다. 팀 규약을 정규식
커스텀 규칙으로 추가할 수도 있는데, 저장 전에 샘플 코드로 매치를 확인할 수
있습니다. 조정이 끝나면 **스냅샷을 YAML로 저장해 두세요.** 4장에서 CLI가 쓸
파일이 이겁니다.

커스텀 규칙을 만들 때 **패턴이 보는 단위**를 고릅니다 — [한 줄]은 각 줄을
따로, [여러 줄]은 파일 전체를 이어서 봅니다. 어노테이션과 선언이 떨어져 있는
규칙은 [여러 줄]이라야 잡힙니다. 제외 패턴과 수정 안내도 폼에 있습니다.

`ast-*` 계열처럼 폼으로 안 되는 타입은 [룰셋 파일 편집]에서 YAML을 직접
고칩니다. 저장 전에 실제 파서로 검증하므로 깨진 룰셋이 저장되지 않습니다(5장).
내장 규칙은 `</>` 를 누르면 정의된 자리로 바로 갑니다.

**④ 코드** (`Ctrl/Cmd+4`)

프로젝트를 탐색하고 소스를 봅니다. 이슈 상세의 [코드에서 열기]를 누르면 그
파일의 그 줄이 열리고, 같은 파일의 이슈가 여백에 표시됩니다. [편집]을 누르고
고친 뒤 `Ctrl/Cmd+S` 로 저장하면 **그 파일만** 즉시 다시 검사합니다. 편집기가
앱에 들어 있어 현장에 별도 편집기를 반입하지 않아도 됩니다.

**⑤ 보고서** (`Ctrl/Cmd+5`)

Excel, HTML, JSON으로 내보냅니다. 기관이나 개발사에 넘길 때는 보통 Excel을,
AI 감리 보고서의 입력으로는 JSON을 씁니다.

> 규칙이나 소스를 바꾸면 이미 나온 결과는 낡습니다. 결과·보고서·AI 감리 화면
> 위에 그 사실이 표시되고 [다시 점검] 이 함께 나옵니다. 낡은 결과로 보고서를
> 만드는 일을 막기 위한 것입니다.

**⑥ AI 감리** (`Ctrl/Cmd+6`)

엔드포인트와 모델을 넣고 [연결 확인]으로 서버가 응답하는지 본 뒤 생성합니다.
폐쇄망 모드는 체크박스이고, 켜면 루프백 밖으로 나가는 연결을 전부 차단한 채로
만듭니다. 사내 Ollama·vLLM 서버면 충분합니다.

---

## 3. 명령행으로 5분

### 기본 실행

```bash
# 전체 검사
./rulix /path/to/project --profile=all

# 코드 품질 + 시큐어코딩만
./rulix /path/to/project --profile=essential

# 전자정부프레임워크 프로젝트 전용
./rulix /path/to/project --profile=egov-full

# 레거시 마이그레이션 진단
./rulix /path/to/project --profile=migration

# DDL × 쿼리 교차 분석
./rulix /path/to/project --profile=sql-ddl --ddl=./ddl/
```

### 프로파일 목록

| 프로파일 | 설명 | 규칙 수 | 기본 활성 |
|----------|------|---------|-----------|
| `quality` | 코드 품질, 명명규칙, 복잡도 | 125 | 107 |
| `secure` | SQL Injection, XSS, 암호화, 인증 | 131 | 123 |
| `sql` | ANSI SQL 공통 (DB 벤더 무관) | 120 | 102 |
| `sql-oracle` | Oracle 전용 (NVL, 힌트, ROWNUM) | 20 | 18 |
| `sql-format` | SQL 포맷·스타일 | 7 | 7 |
| `modernize` | eGov/javax/iBatis/Spring 전환 | 50 | 39 |
| `spring` | DI, @Transactional, Controller | 32 | 28 |
| `egov` | 전자정부 계층/명명/공통 컴포넌트 | 33 | 12 |
| `ddl` | DDL × 쿼리 교차 분석 (`--ddl` 필요) | 17 | 17 |
| | **합계** | **535** | **453** |

일부 규칙은 꺼진 채로 배포합니다. 결함이 아니라 팀 취향에 가까운 항목이라
그렇고, 대부분 SQL 포맷과 전자정부 관례입니다. 규칙 탭이나 오버라이드 파일에서
켜면 됩니다.

### 프로파일 그룹

| 그룹 | 포함 | 용도 |
|------|------|------|
| `all` | quality, secure, sql, sql-oracle, sql-format, modernize, spring, egov | 전체 검사 |
| `essential` | quality, secure | 필수 점검 |
| `sql-all` | sql, sql-oracle, sql-format | SQL 전반 |
| `sql-ddl` | sql, sql-oracle, ddl | SQL + 스키마 교차 |
| `egov-full` | quality, secure, sql, sql-oracle, sql-format, egov, spring | 전자정부 프로젝트 |
| `migration` | modernize, spring | 레거시 전환 진단 |

`./rulix profiles` 를 치면 지금 쓰는 바이너리 기준 목록이 나옵니다.

### 리포트 출력

```bash
# 콘솔 (기본)
./rulix /path/to/project --profile=all

# Excel — 기관 제출용
./rulix /path/to/project --profile=all -o excel --output-file=report.xlsx

# JSON — CI/자동화 연동, AI 감리 보고서 입력
./rulix /path/to/project --profile=all -o json --output-file=report.json

# HTML — 웹 공유
./rulix /path/to/project --profile=all -o html --output-file=report.html
```

### 심각도 필터

```bash
# high 이상만 출력
./rulix /path/to/project --profile=all --min-severity=high

# critical만 출력
./rulix /path/to/project --profile=all --min-severity=critical
```

### AI 감리 보고서

점검 JSON을 먼저 만든 뒤 두 번째 명령으로 보고서를 만듭니다.

```bash
./rulix /path/to/project --profile=all -o json --output-file=scan.json

./rulix ai-report scan.json --offline \
  --endpoint http://127.0.0.1:11434/v1 \
  --model qwen2.5-coder:7b \
  --project "OO시스템" \
  -o report.html
```

`--offline` 은 루프백이 아닌 목적지를 해석된 IP에서 거부하고 DNS 해석 자체를
막습니다. 지정한 엔드포인트가 외부 주소면 명령이 그 이유를 밝히며 실패합니다.

---

## 4. 규칙 스냅샷 — 앱에서 정하고 CLI로 반복

데스크톱 앱 규칙 탭에서 저장한 스냅샷은 CLI가 읽는 바로 그 YAML입니다. 사람이
한 번 내린 판단을 이후 모든 자동 실행이 그대로 따르게 하는 게 목적입니다.

```
데스크톱 앱 · 규칙 탭            ruleset.yaml            CI · 매 빌드
 규칙 켜고 끄고 심각도 조정  ──▶   스냅샷 저장   ──▶   --overrides ruleset.yaml
```

**① 앱에서 저장** — 규칙 탭에서 조정한 뒤 스냅샷 저장. `ruleset.yaml` 이
나옵니다. 저장소에 커밋해 두세요.

**② CLI에서 사용**

```bash
./rulix /path/to/project --profile=all --overrides=ruleset.yaml \
  -o excel --output-file=report.xlsx
```

**③ 반대 방향도 됩니다.** 명령행에서 조정한 결과를 파일로 떨어뜨려 앱에서
불러올 수 있습니다.

```bash
./rulix /path/to/project --profile=all \
  --exclude-rule quality-mb-001,sql-fmt-003 \
  --min-severity=high \
  --save-overrides=ruleset.yaml
```

`--save-overrides` 는 필터까지 적용된 **최종 규칙 집합**을 씁니다. 같은 상태를
`--overrides` 로 언제든 재현할 수 있습니다.

---

## 5. 커스텀 규칙 추가

데스크톱 앱 규칙 탭에서도 같은 일을 합니다. 정규식을 넣고 샘플 코드로 매치를
확인한 뒤 저장하면 끝이고, 파일 위치를 신경 쓸 필요가 없습니다. 아래는 YAML을
직접 쓰는 방법으로, 규칙을 저장소에 커밋해 팀 전체가 공유할 때 씁니다.

### 5.1 어디에 추가?

```
configs/rulesets/quality.yaml    ← 기존 파일에 추가 (간단)
configs/rulesets/custom.yaml     ← 새 파일 생성 (규칙이 많을 때)
```

### 5.2 패턴 타입 선택 (의사결정 트리)

```
한 줄에서 찾을 수 있는가?
├── YES → type: "regex"
│
└── NO → 여러 줄에 걸치는가?
    ├── YES → type: "regex-multiline"
    │
    └── Java 구조 분석이 필요한가?
        ├── import 검사 → "ast-import"
        ├── 루프 내 메서드 호출 → "ast-method-call"
        ├── 어노테이션 속성 → "ast-annotation"
        ├── 변수/필드 타입 → "ast-variable"
        ├── try-catch 구조 → "ast-try-catch"
        ├── 클래스 메서드 수 → "ast-class"
        └── 메서드 복잡도/길이 → "ast-method"
```

### 5.3 현장에서 가장 많이 쓰는 5가지

#### (1) 특정 API/메서드 금지

```yaml
- id: "custom-001"
  name: "System.exit() 사용 금지"
  severity: "critical"
  category: "security"
  description: "System.exit()는 서버 프로세스를 종료시킵니다"
  enabled: true
  pattern:
    type: "regex"
    regex: "System\\.exit\\s*\\("
  custom:
    fix: "예외를 throw하거나 Spring의 정상 종료 메커니즘을 사용하세요"
```

#### (2) 금지 라이브러리 import

```yaml
- id: "custom-002"
  name: "프로젝트 금지 라이브러리"
  severity: "high"
  category: "architecture"
  description: "승인되지 않은 라이브러리입니다"
  enabled: true
  pattern:
    type: "ast-import"
    import: "^com\\.alibaba\\.fastjson\\."
  custom:
    fix: "Jackson 또는 Gson을 사용하세요"
```

#### (3) 어노테이션 + 메서드 조합 (다른 줄)

```yaml
- id: "custom-003"
  name: "DELETE 메서드 접두사"
  severity: "medium"
  category: "naming"
  description: "DELETE 메서드는 delete/remove로 시작해야 합니다"
  enabled: true
  pattern:
    type: "regex-multiline"
    regex: "@DeleteMapping.*\\n\\s*public\\s+\\w+\\s+(?!(delete|remove)\\w+)([a-z]\\w+)\\s*\\("
  custom:
    fix: "delete* 또는 remove* 접두사를 사용하세요"
```

#### (4) 루프 내 위험 호출

```yaml
- id: "custom-004"
  name: "루프 내 DB 조회"
  severity: "high"
  category: "performance"
  description: "루프에서 DB 조회 시 N+1 문제 발생"
  enabled: true
  pattern:
    type: "ast-method-call"
    method: "^(select|find|get|query).*"
    qualifier: ".*(?i)(mapper|dao|repository)"
    context: "inside-loop"
  custom:
    fix: "IN절 배치 조회 또는 JOIN으로 변경하세요"
```

#### (5) 클래스 크기 제한

```yaml
- id: "custom-005"
  name: "Service 클래스 과대"
  severity: "medium"
  category: "design"
  description: "Service 메서드가 30개를 초과합니다"
  enabled: true
  pattern:
    type: "ast-class"
    name_pattern: ".*ServiceImpl$"
  custom:
    max_methods: 30
    fix: "도메인별로 Service를 분리하세요"
```

### 5.4 새 룰셋 파일로 만들기

**① 파일 생성**: `configs/rulesets/custom.yaml`

```yaml
version: "1.0"
profile: "custom"

languages:
  - language: java
    rules:
      - id: "custom-001"
        name: "규칙 이름"
        severity: "high"
        category: "카테고리"
        description: "설명"
        enabled: true
        pattern:
          type: "regex"
          regex: "패턴"
        custom:
          fix: "해결방법"

  - language: xml
    rules:
      - id: "custom-xml-001"
        name: "XML 규칙"
        # ...

  - language: sql
    rules:
      - id: "custom-sql-001"
        name: "SQL 규칙"
        # ...
```

**② 프로파일 등록**: `configs/profiles.yaml`

```yaml
profiles:
  # ... 기존 프로파일 ...

  custom:
    name: "프로젝트 맞춤 규칙"
    description: "프로젝트 고유 코딩 표준"
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
      - custom          # ← 추가
```

**③ 실행**

```bash
./rulix /path/to/project --profile=custom          # 커스텀만
./rulix /path/to/project --profile=quality,custom   # 품질 + 커스텀
./rulix /path/to/project --profile=all              # 전체 (custom 포함)
```

---

## 6. 규칙 오버라이드

프로젝트별로 기존 규칙의 심각도를 변경하거나 비활성화할 수 있습니다. 데스크톱
앱 규칙 탭에서 토글로 조정한 뒤 스냅샷을 저장해도 같은 형식의 파일이 나옵니다
(4장). 아래는 그 파일을 손으로 쓰는 방법입니다.

> 필드 이름은 `id` 입니다. `rule_id` 로 쓰면 오류 없이 조용히 무시되므로,
> 규칙을 껐는데 그대로 검출된다면 이 항목부터 확인하세요.

**① 오버라이드 파일 생성**: `configs/overrides.yaml`

```yaml
overrides:
  # 심각도 변경
  - id: "quality-nc-001"
    severity: "low"              # high → low로 완화

  # 규칙 비활성화
  - id: "sql-style-001"
    enabled: false               # ANSI JOIN 허용

  # 여러 규칙 일괄 비활성화
  - id: "mod-java-001"
    enabled: false               # new Date() 허용 (레거시 프로젝트)
  - id: "mod-java-003"
    enabled: false               # Calendar 허용
```

**② 실행**

```bash
./rulix /path/to/project --profile=all --overrides=configs/overrides.yaml
```

---

## 7. CI/CD 연동

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

### 품질 게이트 (critical 이슈 시 빌드 실패)

```bash
# RULIX는 이슈 발견 시 exit code 1 반환
./rulix ./src --profile=essential --min-severity=critical
# exit code 0 = 통과, 1 = critical 이슈 있음
```

---

## 8. 정규식 작성 팁

### YAML 백슬래시 규칙

```yaml
# ✅ 올바른 예 (백슬래시 2개)
regex: "System\\.out\\.print"
regex: "\\bSELECT\\b"
regex: "new\\s+Date\\s*\\("

# ❌ 잘못된 예 (백슬래시 1개)
regex: "System\.out\.print"
regex: "\bSELECT\b"
```

### 자주 쓰는 정규식

| 패턴 | 의미 | 예시 |
|------|------|------|
| `\\.` | 리터럴 점 | `System\\.out` |
| `\\s*` | 공백 0개 이상 | `메서드\\s*\\(` |
| `\\s+` | 공백 1개 이상 | `new\\s+Date` |
| `\\w+` | 단어 문자 1개 이상 | `class\\s+\\w+` |
| `\\b` | 단어 경계 | `\\bSELECT\\b` |
| `[^)]*` | `)` 아닌 모든 문자 | `\\([^)]*\\)` |
| `(?!pattern)` | 부정 전방탐색 | `(?!select)\\w+` |
| `(?i)` | 대소문자 무시 | `(?i)select` |
| `[\\s\\S]*?` | 줄바꿈 포함 (비탐욕) | multiline 전용 |

### regex vs regex-multiline

```yaml
# regex: 한 줄에서 완결
regex: "@Autowired\\s*$"

# regex-multiline: 여러 줄 걸침 (어노테이션 + 선언이 다른 줄)
type: "regex-multiline"
regex: "@Controller[\\s\\S]*?class\\s+\\w+"
```

> **핵심**: `\n` 또는 `[\s\S]`가 패턴에 있으면 반드시 `regex-multiline` 사용. `regex`에서는 절대 매칭 안 됨.

---

## 9. 지원 언어별 패턴 타입

| 언어 | regex | regex-multiline | ast-* (7종) | annotation-missing-attr |
|------|-------|-----------------|-------------|------------------------|
| Java | ✅ | ✅ | ✅ | ✅ |
| JavaScript | ✅ | ✅ | ❌ | ❌ |
| HTML | ✅ | ✅ | ❌ | ❌ |
| CSS | ✅ | ✅ | ❌ | ❌ |
| SQL | ✅ | ✅ | ❌ | ❌ |
| XML | ✅ | ✅ | ❌ | ❌ |
| Properties | ✅ | ✅ | ❌ | ❌ |

> AST 패턴(`ast-*`)은 **Java 전용**입니다. 다른 언어는 regex/regex-multiline을 사용하세요.

---

## 10. 트러블슈팅

### 데스크톱 앱이 안 뜰 때

| 증상 | 원인과 해결 |
|------|------------|
| macOS — "손상되었기 때문에 열 수 없습니다" | 코드 서명이 아직 없어 Gatekeeper가 격리한 것. 압축 푼 폴더에서 `xattr -dr com.apple.quarantine .` |
| Windows — 창이 안 뜨거나 흰 화면 | WebView2 런타임 없음. [WebView2 Runtime](https://developer.microsoft.com/microsoft-edge/webview2/) 설치 |
| Linux — `rulix-gui` 가 없음 | Linux용 데스크톱 앱은 빌드하지 않습니다. CLI(`rulix`)를 쓰세요 |
| 규칙 탭이 비어 있음 | 실행 파일 옆에 `configs/` 폴더가 있어야 합니다. 번들을 통째로 풀었는지 확인 |

### 규칙이 동작하지 않을 때

| 확인 사항 | 해결 |
|-----------|------|
| `enabled: true` 인가? | false면 무시됨 |
| 패턴에 `\n`이 있는데 `type: "regex"` 인가? | `regex-multiline`로 변경 |
| YAML 백슬래시가 `\\.` 가 아니라 `\.` 인가? | `\\.`로 수정 |
| ast-* 규칙인데 Java 파일이 아닌가? | ast-*는 Java 전용 |
| 프로파일에 해당 룰셋이 포함되어 있는가? | profiles.yaml 확인 |
| 오버라이드로 껐는데 계속 나오는가? | 필드명이 `rule_id`가 아니라 `id` 인지 확인 (6장) |

### 너무 많은 이슈가 나올 때

```bash
# 심각도 필터
./rulix ./src --profile=all --min-severity=high

# 오버라이드로 특정 규칙 끄기
# configs/overrides.yaml에 enabled: false 추가
./rulix ./src --profile=all --overrides=configs/overrides.yaml
```

### JSON으로 규칙별 카운트 확인

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

[English](QUICK_START.md)
