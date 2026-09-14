# 커스텀 규칙 추가 가이드

이 문서는 프로그램 수정 없이 YAML 파일만으로 새로운 검사 규칙을 추가하는 방법을 설명합니다.

## 개요

RULIX는 YAML 기반의 룰셋 시스템을 사용합니다. 특정 패턴 타입의 규칙은 프로그램 코드 수정 없이 YAML 파일만 편집하여 추가할 수 있습니다.

### 지원 패턴 타입

| 패턴 타입 | 설명 | YAML만으로 추가 |
|----------|------|----------------|
| `regex` | 정규식 패턴 매칭 (한 줄) | ✅ 가능 |
| `regex-multiline` | 멀티라인 정규식 패턴 매칭 | ✅ 가능 |
| `annotation-missing-attr` | 어노테이션 필수 속성 검증 | ✅ 가능 |
| `ast-method-call` | AST 기반 메서드 호출 검사 | ✅ 가능 |
| `ast-annotation` | AST 기반 어노테이션 검사 | ✅ 가능 |
| `ast-import` | AST 기반 import 검사 | ✅ 가능 |
| `ast-variable` | AST 기반 변수/필드 검사 | ✅ 가능 |
| `ast-try-catch` | AST 기반 예외 처리 검사 | ✅ 가능 |
| `ast-class` | AST 기반 클래스 구조 검사 | ✅ 가능 |
| `ast-method` | AST 기반 메서드 구조 분석 | ✅ 가능 |
| `ast-multi-cud` | AST 기반 다중 CUD 호출 검사 | ✅ 가능 |
| `ast-dead-code` | AST 기반 Dead Code 검출 | ✅ 가능 |
| `ast-filtered-regex` | 주석 오탐 제거 정규식 (ANTLR4 토큰 필터) | ✅ 가능 |
| `ast-multi-cud` | 다건 CUD 트랜잭션 검사 | ✅ 가능 |
| `cross-file` | 크로스파일 분석 (MyBatis 미사용 SQL ID) | ✅ 가능 |
| `method-analysis` | 메서드/AST 심화 분석 | ❌ Go 코드 구현 필요 |

## 룰셋 파일 위치

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

### 프로파일 그룹

```bash
# 개별 프로파일
rulix ./src --profile=quality
rulix ./src --profile=modernize
rulix ./src --profile=spring,egov

# 그룹 (여러 프로파일 조합)
rulix ./src --profile=all            # quality + secure + sql + modernize + spring + egov
rulix ./src --profile=essential      # secure + quality
rulix ./src --profile=egov-full      # quality + secure + sql + egov + spring
rulix ./src --profile=migration      # modernize + spring
```

### 룰셋 요약

| 룰셋 | 규칙 수 | 대상 | 설명 |
|-------|---------|------|------|
| `quality` | 53+ | Java, JS, HTML, CSS | 코드 품질, 명명규칙, 성능, 레거시 API |
| `secure` | 80+ | Java, JS | SQL Injection, XSS, 암호화, 인증/인가 |
| `sql` | 89 | SQL, Java | SQL 스타일, 성능, 바인드변수, DML |
| `modernize` | 50 | Java, XML | eGov 3.x→4.x, javax→jakarta, iBatis→MyBatis |
| `spring` | 25 | Java | DI 패턴, @Transactional, Controller, REST API |
| `egov` | 33 | Java, XML | 전자정부 계층 명명, Service/Mapper 표준, CRUD 접두사 |

## 규칙 구조

### 기본 YAML 구조

```yaml
version: "1.0"
profile: "프로파일명"

languages:
  - language: java    # java, javascript, html, css, sql, xml
    rules:
      - id: "규칙-id"
        name: "규칙 표시명"
        severity: "high"
        category: "카테고리"
        description: "규칙 설명"
        enabled: true
        pattern:
          type: "regex"
          regex: "정규식패턴"
        custom:
          fix: "해결방법"
        exclude:
          - "제외패턴"
```

### 필수 필드

| 필드 | 설명 | 예시 |
|------|------|------|
| `id` | 고유한 규칙 ID | `"custom-debug-code"` |
| `name` | 사용자에게 표시되는 규칙명 | `"디버그 코드 감지"` |
| `severity` | 심각도 | `low`, `medium`, `high`, `critical` |
| `category` | 카테고리 | `naming`, `security`, `style` 등 |
| `description` | 상세 설명 | `"프로덕션 코드에 디버그 코드가 있습니다"` |
| `enabled` | 활성화 여부 | `true` 또는 `false` |
| `pattern.type` | 패턴 타입 | `regex`, `ast-method-call` 등 |

### 선택 필드

| 필드 | 설명 |
|------|------|
| `custom.fix` | 해결 방법 제안 (리포트에 💡로 표시) |
| `custom.example` | 올바른 코드 예시 |
| `custom.flags` | 정규식 플래그 (`"i"` = case-insensitive) |
| `exclude[]` | 제외할 패턴 목록 (regex 타입 전용) |

### 심각도 레벨

| 심각도 | 설명 |
|--------|------|
| `critical` | 즉시 수정 필요 (보안 취약점, 필수 규칙 위반) |
| `high` | 릴리즈 전 수정 권장 |
| `medium` | 점진적 개선 필요 |
| `low` | 권장사항 |

---

## Part 1: 정규식 규칙

### type: regex

한 줄 단위로 정규식 패턴을 매칭합니다.

```yaml
- id: "custom-system-out"
  name: "System.out 사용 금지"
  severity: "medium"
  category: "logging"
  description: "System.out 대신 로거를 사용하세요"
  enabled: true
  pattern:
    type: "regex"
    regex: "System\\.out\\."
  custom:
    fix: "logger.info() 또는 logger.debug()로 변경하세요"
```

### type: regex-multiline

파일 전체를 하나의 문자열로 처리하여 여러 줄에 걸친 패턴을 매칭합니다.

```yaml
- id: "custom-loop-string-concat"
  name: "루프 내 String += 사용"
  severity: "medium"
  category: "performance"
  description: "루프에서 String += 사용 시 성능 저하"
  enabled: true
  pattern:
    type: "regex-multiline"
    regex: "(for|while)\\s*\\([^)]*\\)\\s*\\{[^}]*\\w+\\s*\\+=\\s*\"[^\"]*\""
  custom:
    fix: "StringBuilder를 사용하세요"
```

### type: annotation-missing-attr

Java 어노테이션에 필수 속성이 없으면 위반으로 감지합니다.

```yaml
- id: "custom-cacheable-key"
  name: "@Cacheable key 속성 필수"
  severity: "high"
  category: "annotation"
  description: "@Cacheable 어노테이션에 key 속성이 필수입니다"
  enabled: true
  pattern:
    type: "annotation-missing-attr"
    annotation: "@Cacheable"
    required_attr: "key"
  custom:
    fix: "key 속성을 추가하세요"
    example: "@Cacheable(value = \"users\", key = \"#id\")"
```

---

## Part 2: AST 쿼리 규칙 (Java 전용)

AST(Abstract Syntax Tree) 쿼리 규칙은 Java 소스코드의 구문 트리를 분석하여 정규식만으로는 검출하기 어려운 복합 패턴을 찾습니다. **YAML만으로 작성 가능**하며, ANTLR4 파서가 추출한 구조 정보를 활용합니다.

### AST 쿼리가 정규식보다 나은 경우

| 상황 | 정규식의 한계 | AST 쿼리의 장점 |
|------|-------------|----------------|
| 루프 내 DB 호출 | 멀티라인 패턴이 복잡하고 오탐 많음 | `context: "inside-loop"` 한 줄로 정확히 검출 |
| 빈 catch 블록 | `catch.*{}` 패턴이 포맷에 의존적 | AST로 statement 수 0인지 정확히 판별 |
| 클래스 메서드 수 | 정규식으로 불가능 | `max_methods: 20` 으로 즉시 적용 |
| import 경로 필터 | 줄 단위로는 맥락 판단 불가 | `class_annotation` 조건으로 범위 한정 |
| 어노테이션 속성값 | 정규식으로 key=value 파싱 복잡 | `missing_attr_name` + `attr_value`로 정밀 검사 |

### 공통 필드: `context`

AST 쿼리 규칙에서 공통으로 사용하는 컨텍스트 조건:

| context 값 | 설명 | 사용 가능한 타입 |
|------------|------|-----------------|
| `inside-loop` | for/while/do-while 루프 내부 | ast-method-call, ast-variable |
| `inside-catch` | catch 블록 내부 | ast-method-call, ast-variable |
| `not-inside-catch` | catch 블록 외부 | ast-method-call, ast-variable |
| `method-name:<regex>` | 특정 이름의 메서드 내부 | ast-method-call, ast-annotation |
| `class-field` | 클래스 필드 (로컬 변수 대신) | ast-variable |
| *(생략)* | 조건 없음 (모든 위치) | 모든 타입 |

### 공통 필드: `custom` 맵

| 키 | 타입 | 설명 |
|----|------|------|
| `fix` | string | 수정 제안 (리포트에 💡로 표시) |
| `args_min` | int | 최소 인자 수 (ast-method-call) |
| `args_max` | int | 최대 인자 수 (ast-method-call) |
| `max_methods` | int | 최대 메서드 수 (ast-class) |
| `max_fields` | int | 최대 필드 수 (ast-class) |
| `catch_is_empty` | bool | catch 블록이 비어있는지 (ast-try-catch) |
| `has_finally` | bool | finally 블록 존재 여부 (ast-try-catch) |
| `has_resources` | bool | try-with-resources 사용 여부 (ast-try-catch) |
| `max_overloads` | int | 동명 메서드 최대 개수 (ast-class) |
| `max_inner_classes` | int | 내부 클래스 최대 개수 (ast-class) |
| `max_imports` | int | 최대 import 수 (ast-import) |
| `all_fields_type` | string | 모든 필드가 이 타입이면 위반 (ast-class) |
| `check_type` | string | Dead code 체크 유형 (ast-dead-code) |
| `max_try_lines` | int | try 블록 최대 라인 수 (ast-try-catch) |
| `max_if_depth` | int | if 최대 중첩 깊이 (ast-method) |
| `max_branches` | int | if-else-if 최대 분기 수 (ast-method) |
| `max_lines` | int | 메서드 최대 라인 수 (ast-method) |
| `max_statements` | int | 메서드 최대 statement 수 (ast-method) |

---

### 2.1 ast-method-call — 메서드 호출 패턴

Java 소스코드에서 특정 메서드 호출을 검색합니다. 호출 대상(qualifier), 메서드명, 인자 수, 호출 위치(context)를 조합하여 필터링합니다.

#### 패턴 필드

| 필드 | 필수 | 설명 | 예시 |
|------|------|------|------|
| `method` | ✅ | 메서드명 정규식 | `"^(select\|find).*"` |
| `qualifier` | - | 호출 대상 객체 정규식 | `".*(?i)(dao\|mapper)"` |
| `context` | - | 호출 위치 조건 | `"inside-loop"` |

#### 예시: 루프 내 DB 조회 (N+1 문제)

```yaml
- id: "quality-ast-001"
  name: "루프 내 DB 조회 호출"
  severity: "high"
  category: "performance"
  description: "루프 안에서 DB 조회 메서드를 호출하면 N+1 문제가 발생합니다"
  enabled: true
  pattern:
    type: "ast-method-call"
    method: "^(select|find|get|query|search|retrieve).*"
    qualifier: ".*(?i)(dao|repository|mapper)"
    context: "inside-loop"
  custom:
    fix: "JOIN이나 IN절 배치 조회로 변경하세요"
```

검출 대상 예:
```java
for (String id : idList) {
    UserVO user = userMapper.selectUserById(id);  // ← 위반!
    results.add(user);
}
```

#### 예시: 루프 내 DB INSERT/UPDATE

```yaml
- id: "quality-ast-002"
  name: "루프 내 DB 변경 호출"
  severity: "high"
  category: "performance"
  description: "루프 안에서 INSERT/UPDATE/DELETE를 개별 호출하면 성능이 저하됩니다"
  enabled: true
  pattern:
    type: "ast-method-call"
    method: "^(insert|update|delete|save|merge).*"
    qualifier: ".*(?i)(dao|repository|mapper)"
    context: "inside-loop"
  custom:
    fix: "Batch Insert/Update 또는 addBatch()/executeBatch()를 사용하세요"
```

#### 예시: 루프 내 외부 API 호출

```yaml
- id: "quality-ast-003"
  name: "루프 내 원격 API 호출"
  severity: "high"
  category: "performance"
  description: "루프 안에서 외부 API를 호출하면 응답시간이 누적됩니다"
  enabled: true
  pattern:
    type: "ast-method-call"
    method: "^(exchange|execute|getForObject|postForObject|send|call).*"
    qualifier: ".*(?i)(restTemplate|webClient|httpClient|feignClient)"
    context: "inside-loop"
  custom:
    fix: "벌크 API 호출 또는 CompletableFuture 병렬 처리를 사용하세요"
```

#### 예시: 인자 수 제한

```yaml
- id: "custom-too-many-args"
  name: "메서드 호출 인자 과다"
  severity: "medium"
  category: "design"
  description: "메서드 호출 인자가 6개를 초과합니다. 파라미터 객체를 사용하세요."
  enabled: true
  pattern:
    type: "ast-method-call"
    method: ".*"
  custom:
    args_min: 7
    fix: "DTO/VO 객체로 파라미터를 그룹화하세요"
```

---

### 2.2 ast-annotation — 어노테이션 검사

Java 어노테이션의 존재, 속성 유무, 속성값을 AST 레벨에서 검사합니다.

#### 패턴 필드

| 필드 | 필수 | 설명 | 예시 |
|------|------|------|------|
| `annotation` | ✅ | 어노테이션명 정규식 | `"^RequestMapping$"` |
| `missing_attr_name` | - | 이 속성이 없으면 위반 | `"method"` |
| `attr_value` | - | 속성값이 이 정규식과 불일치하면 위반 | `".*readOnly.*true.*"` |
| `context` | - | 컨텍스트 조건 | `"method-name:.*"` |

#### 예시: @RequestMapping에 method 속성 누락 (메서드 레벨만)

```yaml
- id: "quality-ast-011"
  name: "@RequestMapping method 속성 누락"
  severity: "medium"
  category: "design"
  description: "@RequestMapping에 method 속성이 없으면 모든 HTTP 메서드를 허용합니다"
  enabled: true
  pattern:
    type: "ast-annotation"
    annotation: "^RequestMapping$"
    missing_attr_name: "method"
    context: "method-name:.*"
  custom:
    fix: "@GetMapping, @PostMapping 등 전용 어노테이션을 사용하세요"
```

> **참고**: `context: "method-name:.*"`는 "어떤 메서드 내부에든 있으면"이라는 조건이므로, 클래스 레벨 `@RequestMapping`은 자동 제외됩니다.

#### 예시: @Transactional readOnly 속성값 검사

```yaml
- id: "custom-transactional-readonly"
  name: "조회 메서드 readOnly 누락"
  severity: "medium"
  category: "performance"
  description: "조회 메서드의 @Transactional에 readOnly=true가 없습니다"
  enabled: true
  pattern:
    type: "ast-annotation"
    annotation: "^Transactional$"
    missing_attr_name: "readOnly"
    context: "method-name:^(select|find|get|query|search).*"
  custom:
    fix: "@Transactional(readOnly = true)를 추가하세요"
```

---

### 2.3 ast-import — import 경로 검사

Java import 문을 AST에서 추출하여 금지/권장 라이브러리를 검사합니다.

#### 패턴 필드

| 필드 | 필수 | 설명 | 예시 |
|------|------|------|------|
| `import` | ✅ | import 경로 정규식 | `"^java\\.util\\.Date$"` |
| `class_annotation` | - | 이 어노테이션이 있는 클래스에서만 검사 | `"Controller"` |

#### 예시: java.util.Date 사용 금지

```yaml
- id: "quality-ast-004"
  name: "java.util.Date import"
  severity: "medium"
  category: "deprecated"
  description: "java.util.Date는 thread-unsafe합니다. java.time API를 사용하세요."
  enabled: true
  pattern:
    type: "ast-import"
    import: "^java\\.util\\.Date$"
  custom:
    fix: "java.time.LocalDate, LocalDateTime, ZonedDateTime 등을 사용하세요"
```

#### 예시: commons-lang 구버전 사용 금지

```yaml
- id: "quality-ast-005"
  name: "commons-lang 구버전 import"
  severity: "medium"
  category: "deprecated"
  description: "org.apache.commons.lang은 EOL입니다. commons-lang3를 사용하세요."
  enabled: true
  pattern:
    type: "ast-import"
    import: "^org\\.apache\\.commons\\.lang\\."
  custom:
    fix: "org.apache.commons.lang3 패키지로 마이그레이션하세요"
```

#### 예시: Controller에서 특정 import 금지

```yaml
- id: "custom-controller-jdbc"
  name: "Controller에서 JDBC 직접 사용"
  severity: "high"
  category: "architecture"
  description: "Controller에서 JDBC를 직접 사용하면 계층 구조를 위반합니다"
  enabled: true
  pattern:
    type: "ast-import"
    import: "^java\\.sql\\."
    class_annotation: "Controller|RestController"
  custom:
    fix: "Service 레이어를 통해 데이터에 접근하세요"
```

---

### 2.4 ast-variable — 변수/필드 검사

로컬 변수 또는 클래스 필드의 이름과 타입을 AST에서 검사합니다.

#### 패턴 필드

| 필드 | 필수 | 설명 | 예시 |
|------|------|------|------|
| `name_pattern` | - | 변수명 정규식 | `"^[a-z]$"` |
| `type_pattern` | - | 타입명 정규식 | `"^Map<String,\\s*Object>$"` |
| `context` | - | `"class-field"` 또는 위치 조건 | `"class-field"` |

#### 예시: 단일 문자 변수명 검출

```yaml
- id: "custom-single-char-var"
  name: "단일 문자 변수명"
  severity: "medium"
  category: "naming"
  description: "의미 있는 변수명을 사용하세요 (i, j, k 제외)"
  enabled: true
  pattern:
    type: "ast-variable"
    name_pattern: "^[a-hlo-zA-Z]$"
  custom:
    fix: "변수의 역할을 나타내는 이름을 사용하세요"
```

#### 예시: 클래스 필드 중 Map<String, Object> 타입 검출

```yaml
- id: "custom-field-map-object"
  name: "필드에 Map<String, Object> 사용"
  severity: "medium"
  category: "design"
  description: "필드 타입으로 Map<String, Object>를 사용하면 타입 안전성이 없습니다"
  enabled: true
  pattern:
    type: "ast-variable"
    type_pattern: "Map<String,Object>"
    context: "class-field"
  custom:
    fix: "전용 VO/DTO 클래스를 정의하세요"
```

---

### 2.5 ast-try-catch — 예외 처리 검사

try-catch 블록의 구조를 AST에서 분석합니다: catch 예외 타입, 빈 catch, finally 유무, try-with-resources 사용 여부.

#### 패턴 필드

| 필드 | 필수 | 설명 | 예시 |
|------|------|------|------|
| `exception_type` | - | 예외 타입 정규식 | `"IOException\|SQLException"` |

#### custom 불리언 조건

| 키 | 설명 |
|----|------|
| `catch_is_empty: true` | catch 블록이 비어있으면 위반 |
| `catch_is_empty: false` | catch 블록에 코드가 있으면 위반 |
| `has_finally: true` | finally가 있으면 매치 |
| `has_finally: false` | finally가 없으면 매치 |
| `has_resources: true` | try-with-resources면 매치 |
| `has_resources: false` | try-with-resources가 아니면 매치 |

> **동작 방식**: `exception_type`이나 `catch_is_empty`가 설정되면 **catch 절 단위**로 검사합니다. 아무것도 설정하지 않으면 **try 블록 단위**로 검사합니다.

#### 예시: 빈 catch 블록

```yaml
- id: "quality-ast-007"
  name: "빈 catch 블록 (AST)"
  severity: "critical"
  category: "exception"
  description: "catch 블록이 비어있으면 예외가 무시됩니다"
  enabled: true
  pattern:
    type: "ast-try-catch"
  custom:
    catch_is_empty: true
    fix: "catch 블록에 logger.error(\"message\", e)를 추가하세요"
```

#### 예시: IO/DB 예외에 try-with-resources 미사용

```yaml
- id: "quality-ast-008"
  name: "try-with-resources 미사용"
  severity: "medium"
  category: "resource"
  description: "IO/DB 예외 처리 시 try-with-resources를 사용하면 리소스 누수를 방지합니다"
  enabled: true
  pattern:
    type: "ast-try-catch"
    exception_type: "IOException|SQLException|FileNotFoundException"
  custom:
    has_resources: false
    has_finally: false
    fix: "try (Resource r = new Resource()) { ... } 구문을 사용하세요"
```

#### 예시: RuntimeException catch 경고

```yaml
- id: "custom-catch-runtime"
  name: "RuntimeException 포괄 catch"
  severity: "high"
  category: "exception"
  description: "RuntimeException을 catch하면 버그가 숨겨질 수 있습니다"
  enabled: true
  pattern:
    type: "ast-try-catch"
    exception_type: "^(RuntimeException|Exception|Throwable)$"
  custom:
    fix: "구체적인 예외 타입을 catch하세요"
```

---

### 2.6 ast-class — 클래스 구조 검사

클래스의 이름, 어노테이션, 메서드/필드 수를 AST에서 분석합니다.

#### 패턴 필드

| 필드 | 필수 | 설명 | 예시 |
|------|------|------|------|
| `name_pattern` | - | 클래스명 정규식 | `".*ServiceImpl$"` |
| `has_annotation` | - | 이 어노테이션이 있는 클래스만 대상 | `"Service"` |
| `missing_annotation` | - | 이 어노테이션이 없으면 위반 | `"Transactional"` |

#### custom 임계값

| 키 | 설명 |
|----|------|
| `max_methods` | 최대 메서드 수 (초과 시 위반) |
| `max_fields` | 최대 필드 수 (초과 시 위반) |

> **동작 우선순위**: `missing_annotation` → `max_methods` → `max_fields` 순서로 체크하며, 첫 번째 위반에서 이슈를 생성합니다.

#### 예시: ServiceImpl 과대 클래스

```yaml
- id: "quality-ast-009"
  name: "ServiceImpl 과대 클래스"
  severity: "medium"
  category: "design"
  description: "ServiceImpl 클래스의 메서드가 너무 많습니다. 책임을 분리하세요."
  enabled: true
  pattern:
    type: "ast-class"
    name_pattern: ".*ServiceImpl$"
  custom:
    max_methods: 20
    fix: "도메인별로 Service 클래스를 분리하세요"
```

#### 예시: Controller 과대 클래스

```yaml
- id: "quality-ast-010"
  name: "Controller 과대 클래스"
  severity: "medium"
  category: "design"
  description: "Controller 클래스의 메서드가 너무 많습니다"
  enabled: true
  pattern:
    type: "ast-class"
    name_pattern: ".*Controller$"
  custom:
    max_methods: 15
    fix: "기능 단위로 Controller를 분리하세요"
```

#### 예시: @Service 클래스에 @Transactional 누락

```yaml
- id: "custom-service-transactional"
  name: "Service에 @Transactional 누락"
  severity: "medium"
  category: "design"
  description: "Service 클래스에는 @Transactional이 필요합니다"
  enabled: true
  pattern:
    type: "ast-class"
    name_pattern: ".*ServiceImpl$"
    has_annotation: "Service"
    missing_annotation: "Transactional"
  custom:
    fix: "클래스 또는 메서드에 @Transactional을 추가하세요"
```

#### 예시: VO 클래스 필드 수 제한

```yaml
- id: "custom-vo-too-many-fields"
  name: "VO 필드 과다"
  severity: "low"
  category: "design"
  description: "VO 클래스의 필드가 너무 많습니다. 분리를 고려하세요."
  enabled: true
  pattern:
    type: "ast-class"
    name_pattern: ".*(?i)(vo|dto)$"
  custom:
    max_fields: 30
    fix: "관련 필드를 하위 VO/DTO로 그룹화하세요"
```

### 2.7 ast-method — 메서드 구조 분석

메서드의 if-else 중첩 깊이, 분기 수, 라인 수, statement 수를 AST에서 분석합니다.

#### 패턴 필드

| 필드 | 필수 | 설명 | 예시 |
|------|------|------|------|
| `name_pattern` | - | 메서드명 정규식 (미지정 시 전체) | `"process.*"` |

#### custom 임계값

| 키 | 설명 | 기본값 |
|----|------|--------|
| `max_if_depth` | 최대 if 중첩 깊이 (초과 시 위반) | 미사용 |
| `max_branches` | 최대 if-else-if 분기 수 (초과 시 위반) | 미사용 |
| `max_lines` | 최대 메서드 라인 수 (초과 시 위반) | 미사용 |
| `max_statements` | 최대 statement 수 (초과 시 위반) | 미사용 |

> **동작**: `max_if_depth` → `max_branches` → `max_lines` → `max_statements` 순서로 체크하며, 첫 번째 위반에서 이슈를 생성합니다.
> **참고**: else-if 체인은 depth가 아닌 branch로 카운트됩니다. `if-else if-else if-else`는 depth=1, branches=4입니다.

#### 예시: 깊은 if-else 중첩 (Arrow Anti-Pattern)

```yaml
- id: "quality-mth-001"
  name: "깊은 if-else 중첩"
  severity: "high"
  category: "complexity"
  description: "if-else 중첩이 3단계를 초과합니다"
  enabled: true
  pattern:
    type: "ast-method"
  custom:
    max_if_depth: 3
    fix: "Guard Clause(early return) 패턴으로 중첩을 줄이세요"
```

#### 예시: 과도한 if-else-if 분기

```yaml
- id: "quality-mth-002"
  name: "만능 if-else 분기"
  severity: "medium"
  category: "complexity"
  description: "if-else-if 분기가 5개를 초과합니다"
  enabled: true
  pattern:
    type: "ast-method"
  custom:
    max_branches: 5
    fix: "전략 패턴(Strategy), Map 디스패치, 또는 enum으로 전환하세요"
```

#### 예시: Long Method

```yaml
- id: "quality-mth-003"
  name: "만능 메서드 (Long Method)"
  severity: "high"
  category: "complexity"
  description: "메서드가 100라인을 초과합니다"
  enabled: true
  pattern:
    type: "ast-method"
  custom:
    max_lines: 100
    fix: "메서드를 조회/검증/처리/저장 등 기능 단위로 분리하세요"
```

#### 예시: Service 메서드 statement 수 제한

```yaml
- id: "custom-svc-statements"
  name: "Service 메서드 과대"
  severity: "medium"
  category: "complexity"
  description: "Service 메서드의 statement가 50개를 초과합니다"
  enabled: true
  pattern:
    type: "ast-method"
    name_pattern: "(process|execute|handle).*"
  custom:
    max_statements: 50
    fix: "private 메서드로 추출하여 가독성을 높이세요"
```

---

### 2.8 ast-dead-code — Dead Code 검출

미사용 private 메서드 또는 미사용 private 필드를 감지합니다. 같은 파일 내에서 선언 이외의 참조가 없으면 위반으로 보고합니다.

#### custom 설정

| 키 | 값 | 설명 |
|----|-----|------|
| `check_type` | `"private-method"` | 미사용 private 메서드 검출 |
| `check_type` | `"private-field"` | 미사용 private 필드 검출 |

> **참고**: `static final` 상수 필드는 자동 제외됩니다. `public`/`protected` 멤버는 외부에서 사용될 수 있으므로 검사 대상이 아닙니다.

#### 예시: 미사용 private 메서드

```yaml
- id: "quality-dead-method-001"
  name: "미사용 private 메서드"
  severity: "medium"
  category: "dead-code"
  description: "private 메서드가 같은 파일 내에서 호출되지 않습니다"
  enabled: true
  pattern:
    type: "ast-dead-code"
  custom:
    check_type: "private-method"
    fix: "사용하지 않는 private 메서드를 삭제하세요"
```

#### 예시: 미사용 private 필드

```yaml
- id: "quality-dead-field-001"
  name: "미사용 private 필드"
  severity: "medium"
  category: "dead-code"
  description: "private 필드가 선언 외에 사용되지 않습니다"
  enabled: true
  pattern:
    type: "ast-dead-code"
  custom:
    check_type: "private-field"
    fix: "사용하지 않는 private 필드를 삭제하세요"
```

---

### 2.9 ast-filtered-regex — 주석 오탐 제거 정규식

`regex` 타입과 동일하게 줄 단위 정규식 매칭을 수행하지만, ANTLR4 토큰 스트림을 활용하여 **주석으로만 구성된 라인을 자동 제외**합니다. `// logger.info(...)` 같은 주석 내 패턴에서 오탐이 발생하는 것을 방지합니다.

> **Java 전용**: Java AST가 없는 파일에서는 일반 `regex`와 동일하게 동작합니다.

#### 패턴 필드

| 필드 | 필수 | 설명 | 예시 |
|------|------|------|------|
| `regex` | ✅ | 정규식 패턴 (regexp2 지원) | `"logger\\.(debug\|info)\\s*\\([^)]*\\+"` |

#### 추가 필드

| 필드 | 설명 |
|------|------|
| `exclude[]` | 매칭된 라인 중 제외할 패턴 목록 |
| `custom.flags` | `"i"` = 대소문자 무시 |

#### `regex`와의 핵심 차이

| | `regex` | `ast-filtered-regex` |
|---|---------|---------------------|
| 주석 라인 | 매칭됨 (오탐 발생) | 자동 제외 |
| AST 필요 | ❌ | ✅ (없으면 regex 폴백) |
| 성능 | 빠름 | 약간 느림 (토큰 분석) |
| 사용 시점 | 오탐 무관한 패턴 | 주석에서 오탐이 잦은 패턴 |

#### 예시: Logger 문자열 연결 (주석 내 오탐 제거)

```yaml
- id: "quality-lg-005"
  name: "Logger 문자열 연결 사용"
  severity: "high"
  category: "logging"
  description: "Logger에서 '+' 연산자로 문자열 연결은 금지됩니다. {} 플레이스홀더를 사용하세요."
  enabled: true
  pattern:
    type: "ast-filtered-regex"
    regex: "logger\\.(debug|info|warn|error)\\s*\\([^)]*\\+[^)]*\\)"
  custom:
    rule_id: "LG-005"
    fix: "{} 플레이스홀더를 사용하세요: logger.info(\"message: {}\", value)"
```

검출 대상:
```java
logger.info("사용자: " + userId);      // ← 위반! + 연산자 사용
// logger.info("test" + value);        // ← ast-filtered-regex는 주석이므로 제외
```

#### 예시: 메서드 체인 과다 (exclude 활용)

```yaml
- id: "quality-chain-004"
  name: "과도한 메서드 체인"
  severity: "low"
  category: "maintainability"
  description: "메서드 체인이 4단계 이상입니다. 가독성을 위해 중간 변수를 사용하세요."
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
    fix: "중간 변수를 사용하여 체인을 분리하세요"
```

---

### 2.10 ast-multi-cud — 다건 CUD 트랜잭션 검사

Service 메서드 내에서 **2개 이상의 CUD(Create/Update/Delete) 호출**이 있지만 `@Transactional` 어노테이션이 없는 경우를 감지합니다. 트랜잭션 없이 다건 CUD를 수행하면 중간 실패 시 데이터 정합성이 깨질 수 있습니다.

#### 패턴 필드

| 필드 | 필수 | 설명 | 예시 |
|------|------|------|------|
| `method` | ✅ | CUD 메서드명 정규식 | `"insert\|update\|delete\|save"` |
| `qualifier` | - | 호출 대상 객체 정규식 | `".*Mapper\|.*Repository"` |

#### custom 설정

| 키 | 설명 | 기본값 |
|----|------|--------|
| `min_cud_calls` | 최소 CUD 호출 수 (이상이면 위반) | `2` |

#### 동작 방식

1. 메서드별로 `@Transactional` 어노테이션 유무 확인
2. `@Transactional`이 없는 메서드에서 `method` + `qualifier` 패턴에 매칭되는 호출 수 카운트
3. 호출 수 >= `min_cud_calls`이면 위반 보고

#### 예시: 다건 CUD에 @Transactional 누락

```yaml
- id: "spring-tx-005"
  name: "@Transactional 누락 (다건 CUD)"
  severity: "high"
  category: "transaction"
  description: "2개 이상의 CUD 작업을 호출하지만 @Transactional이 없어 데이터 정합성이 보장되지 않습니다."
  enabled: true
  pattern:
    type: "ast-multi-cud"
    method: "insert|update|delete|save|remove|create|modify"
    qualifier: ".*Dao|.*Repository|.*Mapper|.*Service"
  custom:
    min_cud_calls: 2
    rule_id: "SPRING-TX-005"
    fix: "@Transactional을 추가하거나, 트랜잭션이 있는 메서드로 위임하세요"
```

검출 대상:
```java
// @Transactional 없음!
public void transferMoney(String from, String to, int amount) {
    accountMapper.updateBalance(from, -amount);  // CUD 1
    accountMapper.updateBalance(to, amount);      // CUD 2 → 위반!
}
```

정상 코드:
```java
@Transactional
public void transferMoney(String from, String to, int amount) {
    accountMapper.updateBalance(from, -amount);
    accountMapper.updateBalance(to, amount);     // @Transactional 있으므로 통과
}
```

---

### 2.11 cross-file — 크로스파일 분석

여러 파일을 교차 분석하여 파일 간 불일치를 감지합니다. 현재 **MyBatis 미사용 SQL ID** 검출을 지원합니다.

#### 동작 방식

1. XML Mapper 파일에서 `<select>`, `<insert>`, `<update>`, `<delete>` 태그의 `id` 속성 수집
2. 같은 디렉토리(또는 프로젝트) 내 Java 파일에서 해당 SQL ID가 참조되는지 확인
3. Java 코드에서 메서드 호출(`mapper.sqlId(`) 또는 문자열 참조(`"namespace.sqlId"`)가 없으면 위반

> **주의**: 이 규칙은 Analyzer 레벨에서 동작합니다. `pattern.type: "cross-file"`로 선언하면 자동으로 크로스파일 분석이 활성화됩니다.

#### 예시: 미사용 MyBatis SQL ID

```yaml
- id: "quality-mybatis-unused-001"
  name: "미사용 MyBatis SQL ID"
  severity: "medium"
  category: "dead-code"
  description: "MyBatis XML의 SQL ID가 Java 코드에서 사용되지 않습니다"
  enabled: true
  pattern:
    type: "cross-file"
  custom:
    fix: "사용하지 않는 SQL 매핑을 제거하거나 Mapper 인터페이스 메서드를 추가하세요"
```

검출 대상:
```xml
<!-- UserMapper.xml -->
<select id="selectDeletedUsers" resultType="UserVo">  <!-- Java에서 미사용 → 위반! -->
    SELECT * FROM TB_USER WHERE del_yn = 'Y'
</select>
```

#### 예시: Mapper Interface → XML 역방향 불일치

Mapper 인터페이스 메서드에 대응하는 XML SQL ID가 없으면 런타임 BindingException 발생:

```yaml
- id: "quality-mapper-no-xml-001"
  name: "Mapper 메서드에 대응 XML SQL ID 없음"
  severity: "high"
  category: "cross-file"
  description: "Mapper 인터페이스 메서드에 대응하는 XML SQL ID가 없습니다"
  enabled: true
  pattern:
    type: "cross-file"
  custom:
    fix: "XML Mapper에 대응하는 SQL ID를 추가하세요"
```

#### 예시: resultMap property ↔ VO 필드 불일치

XML resultMap의 property가 VO 클래스에 존재하지 않으면 런타임 매핑 실패:

```yaml
- id: "quality-resultmap-vo-001"
  name: "resultMap property와 VO 필드 불일치"
  severity: "high"
  category: "cross-file"
  description: "resultMap의 property가 VO 클래스에 존재하지 않습니다"
  enabled: true
  pattern:
    type: "cross-file"
  custom:
    fix: "VO 클래스에 필드를 추가하거나 property명을 수정하세요"
```

#### 예시: 미사용 Service 메서드

Service 메서드가 Controller나 다른 Service에서 호출되지 않으면 dead code:

```yaml
- id: "quality-unused-service-001"
  name: "미사용 Service 메서드"
  severity: "medium"
  category: "dead-code"
  description: "Service 메서드가 다른 파일에서 호출되지 않습니다"
  enabled: true
  pattern:
    type: "cross-file"
  custom:
    fix: "사용하지 않는 Service 메서드를 제거하세요"
```

#### 예시: 미사용 VO/DTO 파일

VO/DTO 클래스가 다른 파일에서 전혀 참조되지 않으면 삭제 대상:

```yaml
- id: "quality-unused-vo-001"
  name: "미사용 VO/DTO 파일"
  severity: "medium"
  category: "dead-code"
  description: "VO/DTO 클래스가 다른 파일에서 전혀 참조되지 않습니다"
  enabled: true
  pattern:
    type: "cross-file"
  custom:
    fix: "사용하지 않는 VO/DTO 파일을 삭제하세요"
```

---

## 정규식 작성 시 주의사항

### 1. 백슬래시 이스케이핑

YAML에서 백슬래시는 `\\`로 작성해야 합니다:

```yaml
# ✅ 올바른 예
regex: "System\\.out\\.print"
regex: "\\bSELECT\\b"

# ❌ 잘못된 예
regex: "System\.out\.print"
regex: "\bSELECT\b"
```

### 2. 특수문자 이스케이핑

| 문자 | 이스케이프 | 의미 |
|------|-----------|------|
| `.` | `\\.` | 리터럴 점 |
| `*` | `\\*` | 리터럴 별표 |
| `(` | `\\(` | 리터럴 괄호 |
| `{` | `\\{` | 리터럴 중괄호 |
| `$` | `\\$` | 리터럴 달러 |
| `|` | `\|` | OR (이스케이프 불필요) |

### 3. 자주 사용하는 정규식 패턴

| 패턴 | 설명 |
|------|------|
| `\\b` | 단어 경계 |
| `\\s+` | 하나 이상의 공백 |
| `\\w+` | 하나 이상의 단어 문자 |
| `[^)]*` | `)` 가 아닌 모든 문자 |
| `(?!pattern)` | Negative lookahead |
| `(?i)` | Case-insensitive 플래그 (정규식 내) |
| `^prefix` | 문자열 시작이 prefix |
| `suffix$` | 문자열 끝이 suffix |

### 4. 정규식 테스트

규칙을 추가하기 전에 정규식을 테스트하는 것이 좋습니다:

- [regex101.com](https://regex101.com/) — PCRE 모드 선택
- [regexr.com](https://regexr.com/)

---

## 새 규칙 추가 절차

### 1. 기존 룰셋에 추가

적절한 룰셋 파일을 선택하여 규칙을 추가합니다:

```bash
vi configs/rulesets/quality.yaml
```

### 2. 새 룰셋 파일 생성

새로운 카테고리의 규칙이 많다면 별도 파일을 생성할 수 있습니다:

```yaml
# configs/rulesets/custom.yaml
version: "1.0"
profile: "custom"

languages:
  - language: java
    rules:
      - id: "custom-rule-001"
        # ... 규칙 정의
```

그리고 `profiles.yaml`에 프로파일을 추가합니다:

```yaml
# configs/profiles.yaml
profiles:
  custom:
    name: "커스텀 규칙"
    description: "프로젝트 맞춤 규칙"
    source: "rulesets/custom.yaml"
    languages:
      - java
```

### 3. 테스트

```bash
# 빌드
make dev

# 새 규칙이 포함된 프로파일로 테스트
./build/rulix /path/to/source --profile=custom

# 특정 카테고리만 검사
./build/rulix /path/to/source --profile=quality --categories=performance

# JSON 출력으로 상세 분석
./build/rulix /path/to/source --profile=quality -o json --output-file=report.json
```

---

## 언어별 규칙 적용

각 언어에 맞는 규칙을 정의할 수 있습니다. **AST 쿼리 규칙(`ast-*`)은 Java 전용**입니다.

```yaml
languages:
  - language: java
    rules:
      - id: "java-rule"        # regex, ast-* 모두 사용 가능

  - language: javascript
    rules:
      - id: "js-rule"          # regex만 사용 가능

  - language: sql
    rules:
      - id: "sql-rule"         # regex만 사용 가능
```

지원 언어: `java`, `javascript`, `html`, `css`, `sql`, `xml`, `properties`

---

## 기존 카테고리 목록

일관성을 위해 기존 카테고리를 사용하는 것이 좋습니다:

| 카테고리 | 설명 | 룰셋 |
|----------|------|------|
| `naming` | 명명규칙 | quality, egov |
| `security` | 보안 | secure, egov |
| `logging` | 로깅 | quality, egov |
| `exception` | 예외처리 | quality, spring, egov |
| `style` | 코딩 스타일 | quality |
| `performance` | 성능 | quality |
| `design` | 설계/구조 | quality, spring |
| `architecture` | 아키텍처 | spring |
| `deprecated` | 폐기된 API 사용 | quality |
| `resource` | 리소스 관리 | quality |
| `documentation` | 문서화 | quality |
| `maintainability` | 유지보수성 | quality |
| `code-quality` | 코드 품질 | quality |
| `annotation` | 어노테이션 | quality |
| `egov-migration` | eGovFrame 마이그레이션 | modernize |
| `jakarta-migration` | javax→jakarta 전환 | modernize |
| `spring-migration` | Spring deprecated 교체 | modernize |
| `ibatis-migration` | iBatis→MyBatis 전환 | modernize |
| `java-modernize` | Java 레거시 API 현대화 | modernize |
| `dependency-injection` | DI 패턴 | spring |
| `transaction` | @Transactional 관련 | spring |
| `rest-api` | REST API 패턴 | spring |
| `thread-safety` | 스레드 안전성 | spring |
| `testing` | 테스트 패턴 | spring |
| `egov-standard` | 전자정부 표준 | egov |
| `egov-common` | 공통 컴포넌트 활용 | egov |
| `method-naming` | CRUD 메서드 접두사 | egov |
| `mybatis` | MyBatis XML 표준 | egov |

---

## 문제 해결

### 규칙이 동작하지 않을 때

1. `enabled: true` 확인
2. 정규식 문법 검증 (regex101.com)
3. 백슬래시 이스케이핑 확인 (`\\`)
4. AST 규칙의 경우 Java 파일인지 확인 (ast-* 타입은 Java 전용)
5. verbose 모드로 실행: `./build/rulix -v`

### AST 규칙에서 예상보다 적게 검출될 때

1. `qualifier` 정규식이 너무 엄격하지 않은지 확인
2. `context` 조건이 올바른지 확인 (예: `inside-loop` vs `inside-catch`)
3. `method` 패턴에 `^`/`$` 앵커가 필요한지 확인

### 너무 많은 이슈가 감지될 때

1. `exclude` 패턴 추가 (regex 타입)
2. 정규식/패턴을 더 구체적으로 수정
3. `context` 필드로 범위 한정
4. `class_annotation` 이나 `has_annotation`으로 대상 클래스 한정
5. `severity` 조정 후 `--min-severity` 옵션 활용

### 정규식 성능 문제

- 정규식 매칭에는 100ms 타임아웃이 설정되어 있습니다
- 복잡한 정규식은 성능에 영향을 줄 수 있으니 가능한 단순하게 작성하세요
- `.*`보다 `[^)]*` 같은 구체적 패턴이 더 빠릅니다

---

## Part 3: 신규 룰셋 실전 가이드

### 3.1 modernize — 레거시 현대화 규칙

레거시 코드를 현대 기술 스택으로 마이그레이션할 때 사용합니다. **regex**, **ast-import** 타입을 조합하여 50개 규칙을 YAML만으로 구현했습니다.

#### 카테고리 구성

| 카테고리 | 규칙 수 | 패턴 타입 | 설명 |
|----------|---------|-----------|------|
| `egov-migration` | 5 | ast-import, regex | eGovFrame 3.x→4.x 패키지/경로 변경 |
| `jakarta-migration` | 10 | ast-import | javax.*→jakarta.* 네임스페이스 변경 |
| `spring-migration` | 10 | regex | Spring deprecated 클래스 교체 |
| `ibatis-migration` | 8 | regex, ast-import | iBatis→MyBatis API 전환 |
| `java-modernize` | 12 | regex, ast-import | Java 레거시 API 현대화 |
| (XML) | 5 | regex | iBatis XML 태그, Spring bean XML |

#### 규칙 작성 패턴별 예시

**ast-import** — 패키지 마이그레이션 (가장 정확한 방식):

```yaml
- id: "mod-jakarta-001"
  name: "javax.servlet 사용"
  severity: "high"
  category: "jakarta-migration"
  description: "javax.servlet은 Spring Boot 3.x/Jakarta EE 9+에서 jakarta.servlet으로 변경되었습니다."
  enabled: true
  pattern:
    type: "ast-import"
    import: "^javax\\.servlet\\."
  custom:
    rule_id: "MOD-JAKARTA-001"
    fix: "jakarta.servlet 패키지로 변경하세요"
    since: "Jakarta EE 9 / Spring Boot 3.0"
```

> **ast-import vs regex**: import 검사는 `ast-import`가 정확합니다. `regex`로 `import javax.servlet`을 검색하면 주석이나 문자열 리터럴에서 오탐이 발생할 수 있지만, `ast-import`는 AST에서 추출한 실제 import문만 검사합니다.

**regex** — 메서드 호출/클래스 참조 검사:

```yaml
- id: "mod-ibatis-003"
  name: "queryForObject() 사용 (iBatis)"
  severity: "high"
  category: "ibatis-migration"
  description: "queryForObject()는 iBatis 메서드입니다. MyBatis의 selectOne()을 사용하세요."
  enabled: true
  pattern:
    type: "regex"
    regex: "\\.queryForObject\\s*\\("
  custom:
    fix: "selectOne() 또는 Mapper 인터페이스 메서드를 사용하세요"
```

**XML 규칙** — iBatis/Spring XML 태그 검사:

```yaml
- id: "mod-xml-002"
  name: "iBatis isNotNull/isEqual 태그"
  severity: "high"
  category: "ibatis-migration"
  description: "<isNotNull>, <isEqual> 등은 iBatis 동적 SQL 태그입니다."
  enabled: true
  pattern:
    type: "regex"
    regex: "<(isNotNull|isNotEmpty|isNull|isEmpty|isEqual|isNotEqual)\\b"
  custom:
    fix: "<if test=\"...\"> 또는 <choose>/<when> 으로 변경하세요"
```

#### 커스텀 modernize 규칙 추가 예시

프로젝트에 특화된 레거시 검출 규칙을 추가하려면:

```yaml
# configs/rulesets/modernize.yaml 에 추가
- id: "mod-custom-001"
  name: "프로젝트 구버전 유틸리티 사용"
  severity: "medium"
  category: "java-modernize"
  description: "com.mycompany.legacy.util 패키지는 신규 패키지로 이전되었습니다"
  enabled: true
  pattern:
    type: "ast-import"
    import: "^com\\.mycompany\\.legacy\\.util\\."
  custom:
    fix: "com.mycompany.core.util 패키지로 변경하세요"
```

---

### 3.2 spring — Spring 개발 표준 규칙

Spring/Spring Boot 프로젝트의 아키텍처 및 코딩 표준을 검사합니다. **regex**, **regex-multiline**, **ast-class** 타입을 조합합니다.

#### 카테고리 구성

| 카테고리 | 규칙 수 | 패턴 타입 | 설명 |
|----------|---------|-----------|------|
| `dependency-injection` | 3 | regex | @Autowired/@Inject/@Resource 필드 주입 경고 |
| `transaction` | 3 | regex-multiline | @Transactional 올바른 사용 |
| `architecture` | 3 | regex-multiline, ast-class | Controller→Service→Repository 계층 위반 |
| `rest-api` | 4 | regex, regex-multiline | REST API 응답 패턴 |
| `thread-safety` | 2 | regex, regex-multiline | 싱글톤 빈 상태 관리 |
| `exception` | 3 | regex, regex-multiline | 예외 처리 표준 |
| `testing` | 3 | regex, regex-multiline | @SpringBootTest, Thread.sleep |
| `security` | 2 | regex | CORS, CSRF 설정 |

#### 핵심 패턴: regex vs regex-multiline 선택

**regex** (한 줄 매칭) — 패턴이 한 줄에 완결되는 경우:

```yaml
# @Autowired가 줄 끝에 단독으로 있으면 필드 주입
- id: "spring-di-001"
  pattern:
    type: "regex"
    regex: "@Autowired\\s*$"
```

**regex-multiline** (멀티라인 매칭) — 어노테이션과 코드가 다른 줄에 있는 경우:

```yaml
# @Controller 클래스 안에서 JdbcTemplate 사용 (여러 줄에 걸침)
- id: "spring-ctrl-002"
  pattern:
    type: "regex-multiline"
    regex: "@(Controller|RestController)[\\s\\S]*?(JdbcTemplate|DataSource)\\s+\\w+"
```

> **주의**: `regex` 타입은 줄 단위로 매칭합니다. 패턴에 `\n`, `[\s\S]` 등 여러 줄에 걸치는 표현이 있으면 반드시 `regex-multiline`을 사용하세요. 이것을 틀리면 규칙이 절대 매칭되지 않습니다.

**ast-class** — Controller/Service 클래스 구조 검사:

```yaml
# Controller 메서드 수 제한
- id: "spring-ctrl-005"
  pattern:
    type: "ast-class"
    name_pattern: ".*Controller$"
  custom:
    max_methods: 15
    fix: "비즈니스 로직을 Service 레이어로 이동하세요"
```

**ast-method** — 메서드 구조 분석 (if 중첩, 분기 수, Long Method):

```yaml
# 깊은 if-else 중첩 검출
- id: "quality-mth-001"
  pattern:
    type: "ast-method"
  custom:
    max_if_depth: 3
    fix: "Guard Clause(early return) 패턴으로 중첩을 줄이세요"

# 과도한 if-else-if 분기 검출
- id: "quality-mth-002"
  pattern:
    type: "ast-method"
  custom:
    max_branches: 5
    fix: "전략 패턴이나 Map 디스패치로 전환하세요"

# Long Method 검출
- id: "quality-mth-003"
  pattern:
    type: "ast-method"
  custom:
    max_lines: 100
    fix: "기능 단위로 메서드를 분리하세요"
```

#### 커스텀 spring 규칙 추가 예시

```yaml
# @Async 메서드에 반환타입 검사
- id: "spring-custom-001"
  name: "@Async void 반환"
  severity: "medium"
  category: "design"
  description: "@Async void 메서드는 예외를 호출자에게 전달할 수 없습니다"
  enabled: true
  pattern:
    type: "regex-multiline"
    regex: "@Async.*\\n\\s*public\\s+void\\s+"
  custom:
    fix: "Future<T> 또는 CompletableFuture<T>를 반환하세요"

# @Value 직접 사용 대신 @ConfigurationProperties 권장
- id: "spring-custom-002"
  name: "@Value 과다 사용"
  severity: "low"
  category: "design"
  description: "@Value가 많으면 @ConfigurationProperties로 그룹화하세요"
  enabled: true
  pattern:
    type: "regex"
    regex: "@Value\\s*\\(\\s*\"\\$\\{"
  custom:
    fix: "@ConfigurationProperties(prefix=\"...\")로 그룹화하세요"
```

---

### 3.3 egov — 전자정부프레임워크 표준 규칙

전자정부 표준프레임워크의 아키텍처, 명명규칙, 공통 컴포넌트 활용을 검사합니다. **regex**, **regex-multiline**, **ast-class** 타입을 조합합니다.

#### 카테고리 구성

| 카테고리 | 규칙 수 | 패턴 타입 | 설명 |
|----------|---------|-----------|------|
| `naming` | 6 | regex, regex-multiline | Controller/Service/Mapper/VO 접미사, 패키지 구조 |
| `egov-standard` | 4 | regex, regex-multiline, ast-class | ServiceImpl 상속, 계층 위반, 과대 클래스 |
| `egov-common` | 5 | regex, regex-multiline | 공통 컴포넌트(파일업로드, 페이징, ID생성) 활용 |
| `method-naming` | 4 | regex-multiline | CRUD 메서드 접두사 (select*/insert*/update*/delete*) |
| `exception` | 3 | regex, regex-multiline | EgovBizException, throws 표준 |
| `logging` | 3 | regex | System.out, Logger 문자열 연결, printStackTrace |
| `security` | 2 | regex | XSS 필터, SQL Injection |
| `mybatis` (XML) | 3 | regex | namespace, ${}, SELECT * |

#### 핵심 패턴: 메서드 명명규칙 검사 (regex-multiline)

메서드 명명규칙은 어노테이션과 메서드 선언이 다른 줄에 있으므로 `regex-multiline`을 사용합니다:

```yaml
# GET 요청인데 select/get/find로 시작하지 않는 메서드
- id: "egov-method-001"
  name: "조회 메서드 접두사 위반"
  severity: "medium"
  category: "method-naming"
  pattern:
    type: "regex-multiline"
    regex: "@(GetMapping|RequestMapping).*\\n\\s*public\\s+\\w+\\s+(?!(select|get|find|search|retrieve|view|download|check|is)\\w+)([a-z]\\w+)\\s*\\("
  custom:
    fix: "조회 메서드: select*, get*, find* 접두사 사용"
```

검출 대상:
```java
@GetMapping("/process")
public String processData(ModelMap model) {  // ← 위반! select*/get*/find* 아님
```

#### 핵심 패턴: 계층 명명규칙 (regex-multiline + negative lookahead)

```yaml
# @Controller 있지만 클래스명에 Controller 접미사 없음
- id: "egov-naming-001"
  pattern:
    type: "regex-multiline"
    regex: "@Controller[\\s\\S]*?class\\s+(?!\\w*Controller\\b)\\w+"

# @Mapper 있지만 인터페이스명에 Mapper/DAO 접미사 없음
- id: "egov-naming-004"
  pattern:
    type: "regex-multiline"
    regex: "@Mapper[\\s\\S]*?interface\\s+(?!\\w*(Mapper|DAO)\\b)\\w+"
```

> **`[\\s\\S]*?`**: regex-multiline에서 "줄 바꿈을 포함한 모든 문자를 비탐욕적(non-greedy)으로 매칭"합니다. 어노테이션과 class 선언 사이에 여러 줄이 있어도 정확히 매칭됩니다.

#### 커스텀 egov 규칙 추가 예시

```yaml
# Controller에서 Service 직접 생성 금지
- id: "egov-custom-001"
  name: "Controller에서 Service 직접 생성"
  severity: "high"
  category: "egov-standard"
  description: "Controller에서 new Service()를 직접 생성하면 DI가 작동하지 않습니다"
  enabled: true
  pattern:
    type: "regex-multiline"
    regex: "@Controller[\\s\\S]*?new\\s+\\w+Service(Impl)?\\s*\\("
  custom:
    fix: "Spring DI를 통해 Service를 주입하세요"

# Mapper XML에 resultType="hashmap" 지양
- id: "egov-custom-002"
  name: "MyBatis resultType=hashmap 사용"
  severity: "medium"
  category: "mybatis"
  description: "hashmap 반환은 타입 안전하지 않습니다. VO/DTO를 정의하세요."
  enabled: true
  pattern:
    type: "regex"
    regex: "resultType\\s*=\\s*\"(hashmap|HashMap|map)\""
  custom:
    fix: "전용 VO 클래스를 정의하여 resultType에 지정하세요"
```

---

## Part 4: 패턴 타입 선택 가이드

### 언제 어떤 타입을 사용할까?

```
패턴이 한 줄에 완결되는가?
├── YES → 주석에서 오탐이 발생하는가?
│   ├── YES → type: "ast-filtered-regex"
│   │         예: logger.info(... +), System.out, 보안 패턴
│   └── NO  → type: "regex"
│             예: new Date(), @Autowired$
└── NO → 패턴이 여러 줄에 걸치는가?
    ├── YES → type: "regex-multiline"
    │         예: @Controller...class, @GetMapping...public void
    └── Java AST 구조 분석이 필요한가?
        ├── import 경로 → type: "ast-import"
        ├── 메서드 호출 + 위치(루프/catch) → type: "ast-method-call"
        ├── 어노테이션 속성 → type: "ast-annotation"
        ├── 변수/필드 타입·이름 → type: "ast-variable"
        ├── try-catch 구조 → type: "ast-try-catch"
        ├── 클래스 메서드/필드 수 → type: "ast-class"
        └── 다건 CUD + @Transactional 검사 → type: "ast-multi-cud"
```

### regex vs regex-multiline 핵심 차이

| | `regex` | `regex-multiline` |
|---|---------|-------------------|
| 매칭 단위 | 파일의 각 줄(line) | 파일 전체(entire file) |
| `\n` 매칭 | ❌ 불가 | ✅ 가능 |
| `[\s\S]` | ❌ 줄 내에서만 | ✅ 줄 바꿈 포함 |
| `^` / `$` | 줄의 시작/끝 | 파일의 시작/끝 |
| 성능 | 빠름 | 패턴에 따라 느릴 수 있음 |
| 사용 시점 | 한 줄 패턴 | 어노테이션+선언, 블록 구조 |

> **가장 흔한 실수**: `regex` 타입에 `\n` 이나 `[\s\S]`를 사용하면 **절대 매칭되지 않습니다**. 여러 줄에 걸치는 패턴은 반드시 `regex-multiline`을 사용하세요.

### ast-* 타입이 regex보다 나은 경우

| 검사 대상 | regex의 문제 | ast-* 해결책 |
|-----------|-------------|-------------|
| import 경로 | 주석/문자열 오탐 | `ast-import`: 실제 import문만 검사 |
| 루프 내 메서드 호출 | 멀티라인 패턴 복잡 | `ast-method-call` + `context: inside-loop` |
| 클래스 메서드 수 | regex로 불가능 | `ast-class` + `max_methods: N` |
| 어노테이션 속성 | key=value 파싱 복잡 | `ast-annotation` + `missing_attr_name` |
| catch 블록 분석 | 중첩 중괄호 파싱 어려움 | `ast-try-catch` + `catch_is_empty` |
| 주석 내 패턴 오탐 | 주석/코드 구분 불가 | `ast-filtered-regex`: 주석 라인 자동 제외 |
| 다건 CUD 트랜잭션 | regex로 메서드 단위 분석 불가 | `ast-multi-cud` + `min_cud_calls: N` |

### 엔진 동작 방식 비교

| 항목 | regex | regex-multiline | ast-filtered-regex | AST 쿼리 |
|------|-------|-----------------|-------------------|----------|
| **입력 단위** | 라인 1개씩 | 파일 전체 (`\n` join) | 라인 1개씩 | ANTLR4 파스 트리 |
| **매칭 방식** | 텍스트 정규식 | 텍스트 정규식 | 정규식 + 주석 필터 | 구조체 속성 비교 |
| **멀티라인 패턴** | 불가 | **가능** | 불가 | 해당없음 (구조) |
| **주석 오탐 제거** | 없음 | 없음 | **자동 제거** | **구조적 제거** |
| **컨텍스트 조건** | 없음 | 없음 | 없음 | inside-loop, inside-catch 등 |
| **타임아웃** | 100ms | 500ms | 100ms | 없음 (트리 순회) |
| **지원 언어** | 모든 언어 | 모든 언어 | Java (fallback: 전체) | Java 전용 |
| **적합한 용도** | 단순 키워드 감지 | 여러 줄 패턴 | 주석 오탐 제거 필요 시 | 메서드/클래스/구조 분석 |

### 동일 규칙의 타입별 구현 예시

`System.out.println` 사용 감지를 3가지 방식으로 작성:

```yaml
# 1) regex — 주석에서도 오탐 가능
- id: "example-regex"
  pattern:
    type: "regex"
    regex: "System\\.out\\.println"

# 2) ast-filtered-regex — 주석 라인 자동 스킵
- id: "example-ast-filtered"
  pattern:
    type: "ast-filtered-regex"
    regex: "System\\.out\\.println"

# 3) ast-method-call — 실제 메서드 호출 노드만 감지 (가장 정확)
- id: "example-ast-call"
  pattern:
    type: "ast-method-call"
    method: "println"
    qualifier: "System\\.out"
```

**오탐/정탐 비교:**

| 소스코드 | regex | ast-filtered-regex | ast-method-call |
|----------|-------|-------------------|-----------------|
| `System.out.println("hello");` | 검출 | 검출 | 검출 |
| `// System.out.println 금지` | 오탐 | 스킵 | 스킵 |
| `/* System.out.println */` | 오탐 | 스킵 | 스킵 |
| `String s = "System.out.println";` | 오탐 | 오탐 | 스킵 |

> **요약**: 단순 키워드 → `regex`, 주석 오탐 문제 → `ast-filtered-regex`, 구조적 정확성 → `ast-*`, 여러 줄 패턴 → `regex-multiline`

---

[English](CUSTOM_RULES.md)
