<!-- 이 파일은 rulix-releases 저장소의 루트 README 로 배포된다.
     릴리스 워크플로가 통째로 복사하므로, 배포 repo에서 직접 고치면
     다음 릴리스에 지워진다. 문구 수정은 여기서 한다.
     링크가 docs/ 로 시작하는 것은 배포 repo 루트 기준이기 때문이다. -->
# RULIX — 데스크톱 앱과 CLI, 같은 엔진

> Java, JavaScript, HTML, CSS, SQL, XML을 **535개 규칙**으로 점검합니다.
> 한 번 내려받으면 데스크톱 앱과 명령행 바이너리가 함께 들어 있습니다.
> JVM도 Node.js도 파이썬도 필요 없고, 인터넷이 끊긴 환경에서 그대로 돕니다.

[![Release](https://img.shields.io/github/v/release/mhb8436/rulix-releases?label=release&color=2f855a)](https://github.com/mhb8436/rulix-releases/releases/latest)
[![Platforms](https://img.shields.io/badge/platforms-Windows%20%7C%20macOS%20%7C%20Linux-lightgrey)](#설치)
[![License: AGPL-3.0](https://img.shields.io/badge/license-AGPL--3.0-blue)](https://www.gnu.org/licenses/agpl-3.0)

> 이 문서는 한국어판입니다. 최신 내용은 [영문 README](README.md)를 기준으로
> 합니다.

[**다운로드**](https://github.com/mhb8436/rulix-releases/releases/latest) ·
[빠른 시작](docs/QUICK_START.ko.md) ·
[English](README.md)

<!-- RULIX-DEMO-KO:START -->
## 데모

### 점검 실행 — 데스크톱 앱

![RULIX 점검](docs/media/scan.gif)

전자정부 표준프레임워크 기반 실제 프로젝트(413파일 / 128,497줄)를 2.56초에
점검한 화면입니다. 경로와 클래스명은 익명화했고 수치는 실제 값입니다.

### 코드를 읽고 그 자리에서 고치기 — 데스크톱 앱

![RULIX 편집기](docs/media/editor.gif)

정적 분석 도구 대부분이 여기서 손을 뗍니다. 파일을 이름으로 찾아 열면 그
파일의 이슈가 여백과 미니맵에 전부 표시되고, 화살표나 `F8` 로 건너뜁니다.
[편집]을 누르고 한 줄을 고쳐 저장하면 **그 파일만** 즉시 다시 검사합니다 —
11건이 10건이 되는 데 전체 점검을 다시 돌리지 않습니다.

편집기가 앱에 들어 있습니다. 반입하는 모든 실행 파일을 승인받아야 하는
현장에서, 승인받을 것이 하나 줄어듭니다.

### 커스텀 규칙 추가 — 데스크톱 앱

![커스텀 규칙](docs/media/custom-rule.gif)

팀 규약을 정규식으로 등록하면 다음 점검부터 바로 잡힙니다. 저장 전에 샘플
코드를 붙여넣으면 실제로 잡히는 자리를 보여 줍니다 — 점검 엔진과 같은 기준으로
판정하므로, 여기서 잡히는 패턴이 점검에서 0건으로 나오는 일은 없습니다.

### 인터넷 없이 만드는 AI 감리 보고서 — 터미널

![폐쇄망 AI 감리 보고서](docs/media/airgap-ai-report.gif)

감리 보고서는 OpenAI 호환 엔드포인트면 무엇으로든 만듭니다. 사내 Ollama나
vLLM 서버가 그대로 붙고, 클라우드 API도 외부 연결도 쓰지 않습니다.

중요한 건 `--offline` 입니다. 목적지가 루프백이 아닌 연결을 전부 거부하는데,
판정을 해석된 IP에서 하므로 호스트명으로 우회할 수 없고 이름 해석 자체도
막힙니다.

녹화를 보면 **먼저** 외부 엔드포인트를 가리킵니다. DNS 조회 단계에서
실패합니다. 그다음 엔드포인트만 로컬 서버로 바꾼 같은 명령이 보고서를
만듭니다. 잘 도는 데모는 아무것도 증명하지 않습니다. 막혀야 할 때 막히는
데모가 증거입니다.

같은 두 단계를 직접 돌려볼 수 있습니다: [폐쇄망 검증](docs/AIRGAP_VERIFICATION.ko.md)

<!-- RULIX-DEMO-KO:END -->

## 입구는 둘, 엔진은 하나

RULIX는 같은 룰 엔진 위에 데스크톱 앱과 CLI를 얹은 도구입니다. 어느 한쪽이
축소판이 아니고, 둘 중 하나만 골라 쓰라는 구조도 아닙니다.

|  | 데스크톱 앱 (`rulix-gui`) | CLI (`rulix`) |
|---|---|---|
| 쓰는 자리 | 코드를 훑고 고치고, 적용할 규칙을 정하고, 넘길 보고서를 만들 때 | CI 파이프라인, 정기 점검, 배치 스크립트 |
| 플랫폼 | Windows x64, macOS (Apple Silicon / Intel) | Windows, macOS, Linux — x64, ARM64, x86 |
| 보고서 | Excel, HTML, JSON, AI 감리, DOCX | 콘솔, Excel, HTML, JSON, AI 감리, DOCX |
| 규칙 조정 | 토글 클릭, 정규식 즉석 테스트, 룰셋 YAML 편집(검증 포함) | YAML 파일, CLI 옵션 |

둘을 잇는 건 파일 하나입니다. **데스크톱 앱에서 저장한 규칙 스냅샷이 CLI가
읽는 바로 그 YAML**이라, 사람이 한 번 내린 판단을 이후 모든 자동 실행이
그대로 따릅니다.

```
데스크톱 앱 · 규칙 탭              ruleset.yaml              CI · 매 빌드
 규칙 켜고 끄고 심각도 조정   ──▶   스냅샷 저장   ──▶   rulix ./src --overrides ruleset.yaml
```

역할 분담이 이렇습니다. 사람은 GUI에서 정하고, 파이프라인은 CLI로 반복하고,
같은 엔진이 같은 파일을 읽으니 결과가 어긋나지 않습니다.

## 설치

[Releases](https://github.com/mhb8436/rulix-releases/releases)에서 내려받습니다.
각 묶음은 그 자체로 완결돼 있어 압축만 풀면 바로 실행됩니다.

| 플랫폼 | 파일 | 들어 있는 것 |
|---|---|---|
| Windows x64 | `rulix-windows-amd64.zip` | `rulix.exe`, `rulix-gui.exe`, `rulix-report.exe` |
| macOS Apple Silicon | `rulix-darwin-arm64.tar.gz` | `rulix`, `rulix-gui`, `rulix-report` |
| macOS Intel | `rulix-darwin-amd64.tar.gz` | `rulix`, `rulix-gui`, `rulix-report` |
| Linux x64 | `rulix-linux-amd64.tar.gz` | `rulix` |
| Linux ARM64 | `rulix-linux-arm64.tar.gz` | `rulix` |
| Windows x86 | `rulix-windows-386.zip` | `rulix.exe` |
| Linux x86 | `rulix-linux-386.tar.gz` | `rulix` |

데스크톱 앱만 필요하면 단독 파일로도 올라갑니다:
`rulix-gui-windows-amd64.exe`, `rulix-gui-darwin-arm64`, `rulix-gui-darwin-amd64`.

**Linux용 데스크톱 앱은 없습니다.** Wails가 빌드 시점에 GTK와 WebKit을
요구하는데 아직 그 빌드를 제공하지 않습니다. Linux에서 RULIX는 CLI입니다.

### Windows

압축을 풀고 `rulix-gui.exe`를 더블클릭하면 됩니다. 데스크톱 앱은 Microsoft
WebView2 런타임으로 화면을 그리는데, Windows 11과 최근 Windows 10에는 기본으로
들어 있습니다. 오래된 장비라면
[WebView2 Runtime](https://developer.microsoft.com/microsoft-edge/webview2/)을
먼저 설치하세요. CLI인 `rulix.exe`는 아무것도 필요 없습니다.

### macOS

```bash
tar xzf rulix-darwin-arm64.tar.gz
cd rulix-darwin-arm64

# 아직 코드 서명을 하지 않아 Gatekeeper가 격리합니다.
xattr -dr com.apple.quarantine .

./rulix-gui          # 데스크톱 앱
./rulix --help       # CLI
```

`xattr` 줄을 건너뛰면 macOS가 "손상된 앱"이라고 막습니다. 서명과 공증은 아직
적용하지 않았습니다.

### Linux

```bash
tar xzf rulix-linux-amd64.tar.gz
cd rulix-linux-amd64
./rulix ./src --profile=essential
```

## 알려진 문제

**macOS 전체화면에서 폴더 선택창이 바로 닫힙니다.** 전체화면 상태로 [찾아보기]
를 누르면 선택창이 떴다가 즉시 사라집니다. 원인을 아직 확인하지 못했습니다.
당분간은 **창 모드에서 폴더를 고른 뒤 전체화면으로 전환**하시면 됩니다. 폴더
경로를 입력란에 직접 붙여넣는 방법도 됩니다.

**macOS 앱이 서명돼 있지 않습니다.** 처음 실행할 때 "손상되었기 때문에 열 수
없습니다"가 뜹니다. 압축을 푼 폴더에서 `xattr -dr com.apple.quarantine .` 를
한 번 실행하면 됩니다.

**Linux용 데스크톱 앱이 없습니다.** Linux 에서 RULIX 는 CLI 입니다.

## 시작하기

### 데스크톱 앱으로

1. **점검** (`Ctrl/Cmd+1`) — [찾아보기]로 소스 폴더를 고르고, 돌릴 프로파일과
   최소 심각도를 정합니다. DDL 파일이나 폴더를 지정하면 교차 분석이 함께
   돕니다. 실행하면 진행률이 나오고 도중에 취소할 수 있습니다.
2. **결과** (`Ctrl/Cmd+3`) — 지표와 등급 대시보드, 그리고 이슈 목록.
   이슈를 누르면 해당 파일의 그 줄이 열립니다.
3. **규칙** (`Ctrl/Cmd+2`) — 이 프로젝트에 안 맞는 규칙을 끄고, 심각도를
   조정하고, 팀 규약을 정규식 커스텀 규칙으로 추가합니다. 저장 전에 샘플로
   매치를 확인할 수 있습니다. **스냅샷을 YAML로 저장해 두세요.** CI가 쓸
   파일이 이겁니다.
4. **코드** (`Ctrl/Cmd+4`) — 프로젝트를 탐색하고 하이라이팅된 소스를 봅니다.
   [편집]으로 고칠 수 있고, 저장하면 그 파일만 즉시 다시 검사합니다.
5. **보고서** (`Ctrl/Cmd+5`) — Excel, HTML, JSON으로 내보냅니다. 개발사에
   넘길 때는 보통 Excel을, AI 감리 보고서의 입력으로는 JSON을 씁니다.
6. **AI 감리** (`Ctrl/Cmd+6`) — OpenAI 호환 엔드포인트로 감리 보고서를
   만듭니다. 폐쇄망 모드를 켤 수 있습니다.
   [AI 감리 보고서](#ai-감리-보고서) 참고.

### 명령행으로

```bash
# 전체 규칙으로 점검, 결과는 콘솔에
rulix ./src --profile=all

# 품질·보안만, 높음 이상
rulix ./src --profile=quality,secure --min-severity=high

# Excel 보고서 — 넘길 때 가장 많이 쓰는 형식
rulix ./src --profile=all -o excel --output-file=report.xlsx

# HTML, JSON
rulix ./src --profile=all -o html --output-file=report.html
rulix ./src --profile=all -o json --output-file=report.json

# 데스크톱 앱에서 정한 규칙 선택을 그대로 재현
rulix ./src --profile=all --overrides=ruleset.yaml -o excel --output-file=report.xlsx

# 명령행에서 조정한 뒤 그 선택을 저장
rulix ./src --profile=all \
  --exclude-rule quality-nc-001,sql-fmt-003 \
  --min-severity=high \
  --save-overrides=ruleset.yaml

# DDL × 쿼리 교차 분석
rulix ./src --profile=sql-ddl --ddl=./ddl/

# 영문 출력
rulix ./src --profile=all --lang=en
```

> 두 입구 모두 기본이 영어입니다. 앱은 **설정**에서 언어를 고르고, CLI 는
> `--lang` (`en` / `ko`) 또는 환경변수 `RULIX_LANG` 을 따릅니다. 시스템 로캘은
> 보지 않습니다 — 돌리는 장비에 따라 언어가 달라지는 도구보다 항상 같은 쪽이
> 낫습니다.

## 프로파일

프로파일은 목적별 규칙 묶음입니다. 쉼표로 여러 개를 함께 씁니다.

| 프로파일 | 점검 내용 | 규칙 수 | 기본 활성 |
|---|---|---|---|
| `quality` | 코드 품질 — 명명, 로깅, 복잡도, 죽은 코드 | 125 | 107 |
| `secure` | 시큐어코딩 — 인젝션, 암호, 인증, 예외 처리 | 131 | 123 |
| `sql` | ANSI SQL 공통, DB 벤더 무관 | 120 | 102 |
| `sql-oracle` | Oracle 전용 — NVL, 힌트, ROWNUM, DECODE | 20 | 18 |
| `sql-format` | SQL 포맷·스타일 | 7 | 7 |
| `modernize` | 레거시 현대화 — eGovFrame 3→4, javax→jakarta, iBatis→MyBatis | 50 | 39 |
| `spring` | Spring / Boot 개발 표준 | 32 | 28 |
| `egov` | 전자정부 표준프레임워크 | 33 | 12 |
| `ddl` | DDL × 쿼리 교차 분석 (`--ddl` 필요) | 17 | 17 |
| | **합계** | **535** | **453** |

일부 규칙은 꺼진 채로 배포합니다. 결함이 아니라 팀 취향에 가까운 항목들이고,
대부분 SQL 포맷과 전자정부 관례입니다. 데스크톱 앱 규칙 탭이나 오버라이드
파일에서 켜면 됩니다.

### 프로파일 그룹

| 그룹 | 포함 |
|---|---|
| `all` | quality, secure, sql, sql-oracle, sql-format, modernize, spring, egov |
| `essential` | quality, secure |
| `sql-all` | sql, sql-oracle, sql-format |
| `sql-ddl` | sql, sql-oracle, ddl |
| `egov-full` | quality, secure, sql, sql-oracle, sql-format, egov, spring |
| `migration` | modernize, spring |

`rulix profiles` 를 치면 지금 쓰는 바이너리 기준으로 목록이 나옵니다.

## 무엇을 잡는가

### 코드 품질 (`quality`)

- 클래스 명명 — Controller, Service, ServiceImpl, Mapper, VO, Const, Util
- `System.out.println` 금지, 로거 사용
- 로거 문자열 접합(`+`) 금지, `{}` 플레이스홀더 사용
- 빈 catch 블록, 죽은 코드, 비대한 클래스
- 메서드 길이, 순환복잡도
- MyBatis XML — `SQL_ID` 주석 필수, `${}` 금지, namespace 필수

### 시큐어코딩 (`secure`)

- SQL 인젝션 — `createStatement()` 금지, `PreparedStatement` 사용
- XSS — `innerHTML`, `document.write`
- 명령어 삽입 — 외부 입력이 들어가는 `Runtime.exec()`, `ProcessBuilder`
- 경로 조작, SSRF, CSRF 토큰 검증
- 취약한 암호 — DES, MD5, SHA-1, RC4 금지, RSA 2048+ / AES 128+
- 하드코딩된 비밀번호·키·DB 접속 정보
- `printStackTrace()`, 응답에 노출되는 `e.getMessage()`

### SQL (`sql`, `sql-oracle`, `sql-format`)

- `SELECT *` 금지, `LIKE` 선행 와일드카드 금지
- 바인드 변수 필수, `UPDATE`/`DELETE` 에 `WHERE` 필수
- 코드 스멜 — `NATURAL JOIN`, `ON 1=1`, `AND`/`OR` 괄호 누락
- Oracle — 힌트, `NVL`·`TO_CHAR` 인덱스 무효화, `DECODE`, `ROWNUM` 페이징

### DDL × 쿼리 교차 분석 (`ddl`)

`--ddl` 로 DDL 파일을 지정하면 스키마와 소스의 SQL을 교차 검증합니다. 대상은
MyBatis XML, 독립 `.sql` 파일, 그리고 Java 소스에 박힌
SQL(`@Select`/`@Insert`/`@Update`/`@Delete`, JPA 네이티브 쿼리,
`JdbcTemplate`, 순수 JDBC, 문자열 리터럴)까지입니다.

- **쿼리 정합성** — 존재하지 않는 테이블·컬럼 참조, 암시적 변환을 부르는 JOIN
  타입 불일치, NOT NULL 컬럼이 빠진 `INSERT`, PK를 건드리는 `UPDATE SET`
- **인덱스 활용** — 인덱스 없는 WHERE/JOIN/GROUP BY/ORDER BY 컬럼, 복합 인덱스
  선두 컬럼 스킵, 인덱스 컬럼을 감싼 함수
- **DDL 개선** — 자주 조회되는데 인덱스가 없는 컬럼, 인덱스 없는 FK,
  미사용·중복 인덱스, PK 없는 테이블

```bash
rulix ./src --profile=ddl --ddl=./ddl/schema.sql
rulix ./src --profile=sql-ddl --ddl=./ddl/
```

데스크톱 앱에서는 점검 탭의 **DDL 크로스 분석** 항목입니다.

## AI 감리 보고서

점검 결과를 감리 보고서 문서로 만듭니다. OpenAI 호환 엔드포인트면 무엇이든
붙으므로 사내 Ollama·vLLM 서버로 충분하고 클라우드 API는 관여하지 않습니다.

**데스크톱 앱** — 점검을 돌린 뒤 **AI 감리** 탭에서 엔드포인트와 모델을 넣고,
[연결 확인]으로 서버가 응답하는지 먼저 보고, 생성합니다. 폐쇄망 모드는
체크박스입니다.

**CLI** — 점검 JSON을 먼저 만드는 두 단계입니다.

```bash
rulix ./src --profile=all -o json --output-file=scan.json

rulix ai-report scan.json --offline \
  --endpoint http://127.0.0.1:11434/v1 \
  --model qwen2.5-coder:7b \
  --project "OO시스템" \
  -o report.html
```

`--offline` 은 다른 무엇보다 먼저 네트워크 차단 장치를 겁니다. 루프백이 아닌
목적지는 해석된 IP에서 거부하고 DNS 해석 자체를 막으므로 호스트명으로
빠져나갈 수 없습니다. 지정한 엔드포인트가 외부 주소면 명령이 그 이유를 밝히며
실패합니다.

말로 믿는 대신 차단 장치가 실제로 살아 있는지 확인하려면
[폐쇄망 검증](docs/AIRGAP_VERIFICATION.ko.md)을 따라 해보세요. 외부 엔드포인트를
가리켜 실패하는 것을 본 다음 로컬로 생성하는 순서입니다.

| 옵션 | 설명 | 기본값 |
|---|---|---|
| `-o`, `--output` | 보고서 경로 | `ai-report.html` |
| `--provider` | `openai`(호환), `claude`, `gemini` | `openai` |
| `--endpoint` | OpenAI 호환 엔드포인트 | — |
| `--model` | 모델 이름 | — |
| `--api-key` | API 키. 로컬 서버는 아무 값이나 됨 | — |
| `--project` | 보고서에 쓸 대상 이름 | 입력 파일명 |
| `--lang` | 보고서 언어 `ko` / `en` | 화면 언어 |
| `--offline` | 폐쇄망 모드. 루프백 밖 연결 차단 | `false` |
| `--timeout` | 요청당 제한시간 | `5m` |

## CLI 레퍼런스

```
rulix [경로] [옵션]
rulix profiles                       프로파일·그룹 목록
rulix report [이름:result.xlsx ...]  Excel 결과로 DOCX 감리 보고서 생성
rulix ai-report [scan.json]          점검 JSON으로 AI 감리 보고서 생성
```

| 옵션 | 축약 | 설명 | 기본값 |
|---|---|---|---|
| `--profile` | `-p` | 사용할 프로파일, 쉼표 구분 | `all` |
| `--profiles-file` | | 프로파일 정의 파일 | `configs/profiles.yaml` |
| `--config` | `-c` | 설정 파일 경로 | — |
| `--overrides` | | 저장해 둔 규칙 스냅샷 적용 | — |
| `--save-overrides` | | 지금 적용된 규칙 집합을 YAML로 저장 | — |
| `--output` | `-o` | `console`, `json`, `html`, `excel` | `console` |
| `--output-file` | | 출력 경로 | stdout |
| `--min-severity` | `-s` | `low`, `medium`, `high`, `critical` | `low` |
| `--rules` | | 점검할 규칙 카테고리 | — |
| `--exclude-rule` | | 제외할 규칙 ID, 쉼표 구분 | — |
| `--ddl` | | DDL 파일·디렉터리. 지정하면 교차 분석 수행 | — |
| `--cross-file-only` | | 크로스파일 분석만 실행 | `false` |
| `--summary` | | 규칙별 요약 테이블. CI에서 유용 | `false` |
| `--no-dedup` | | 중복 검출 유지 | `false` |
| `--lang` | | 출력 언어 `en` / `ko` (또는 `RULIX_LANG`) | `en` |
| `--verbose` | `-v` | 상세 출력 | `false` |

## 데스크톱 앱 레퍼런스

| 탭 | 단축키 | 하는 일 |
|---|---|---|
| 점검 | `Ctrl/Cmd+1` | 소스 폴더, 프로파일, 최소 심각도, DDL 교차 분석 / 실행·취소·진행률 |
| 규칙 | `Ctrl/Cmd+2` | 규칙 켜고 끄기, 심각도 변경, 정규식 커스텀 규칙과 즉석 테스트, 스냅샷 저장·불러오기, **룰셋 YAML 직접 편집** |
| 결과 | `Ctrl/Cmd+3` | 지표·등급 대시보드, 이슈 목록, 해당 줄로 파일 열기 |
| 코드 | `Ctrl/Cmd+4` | **프로젝트 탐색, 하이라이팅된 소스 보기·편집, 고친 파일 즉시 재검사** |
| 보고서 | `Ctrl/Cmd+5` | Excel / HTML / JSON 내보내기, 최근 생성 목록 |
| AI 감리 | `Ctrl/Cmd+6` | 엔드포인트·모델, 연결 확인, 폐쇄망 모드, 진행률, 취소 |
| 설정 | `Ctrl/Cmd+7` | 앱 환경설정 |

### 코드 보기와 편집

편집기가 앱에 들어 있어 현장에 따로 반입할 필요가 없습니다. 이슈에서
[코드에서 열기]를 누르면 그 파일의 그 줄이 열리고, 같은 파일의 이슈가 전부
여백에 표시됩니다. [편집]을 누르고 고친 뒤 `Ctrl/Cmd+S` 로 저장하면 **그 파일만
즉시 다시 검사합니다.** 고쳐졌는지 보려고 대형 프로젝트를 통째로 다시 돌릴
필요가 없습니다.

저장할 때 원본의 줄끝(CRLF/LF)과 마지막 개행, 파일 권한을 그대로 지킵니다.
편집하는 사이에 파일이 외부에서 바뀌었으면 덮어쓰지 않고 알려 줍니다. 한 파일
재검사에서는 크로스파일 규칙이 빠지는데, 프로젝트 전체를 봐야 판정할 수 있기
때문이고 화면에 그 사실을 적습니다.

### 규칙을 바꾸면 결과가 낡는다

규칙을 끄거나 심각도를 바꾸거나 소스를 고치면, 이미 나온 결과는 그 전 상태를
보여줍니다. 결과·보고서·AI 감리 화면 위에 그 사실을 알리는 줄이 뜨고 [다시
점검] 이 함께 나옵니다.

자동으로 다시 계산하지는 않습니다. 대형 프로젝트에서 규칙 하나 끌 때마다 몇
초씩 멈추면 규칙을 조율할 수가 없기 때문입니다. 대신 낡은 결과로 보고서를
만드는 일이 없도록 분명히 표시합니다.

### 화면 사이 이동

화면 안의 버튼으로 들어간 곳에는 **돌아갈 길**이 함께 뜹니다. 이슈에서
[코드에서 열기]로 들어가면 위에 `← 결과로 돌아가기 · secure-plain-001` 이
생기고, 누르면 보던 이슈 목록으로 필터까지 그대로 돌아옵니다.

사이드바로 직접 간 화면에는 뜨지 않습니다. 그건 "돌아가기"가 아니라 "새로
가기"이기 때문입니다. 방문 기록은 따로 쌓이며 `Ctrl/Cmd+[` 와 `Ctrl/Cmd+]` 로
오갑니다.

대시보드에서 심각도나 카테고리를 눌러 이슈 목록으로 들어가면 걸린 필터가
칩으로 보이고 `✕` 로 풉니다.

### 앱 안에서 규칙 고치기

규칙 탭의 폼으로는 단순 정규식 규칙만 만들 수 있습니다. `regex-multiline`,
`ast-*` 계열, 제외 패턴처럼 그 밖의 것은 **[룰셋 파일 편집]** 에서 YAML을 직접
고칩니다.

규칙 목록에서 내장 규칙의 `</>` 를 누르면 그 규칙이 정의된 룰셋 파일이 열리고
해당 줄로 바로 갑니다.

커스텀 규칙을 폼으로 만들 때는 **패턴이 보는 단위**를 고릅니다. [한 줄]은 각
줄을 따로 보고, [여러 줄]은 파일 전체를 이어서 봅니다. 어노테이션과 선언이
떨어져 있는 규칙은 [여러 줄]이라야 잡힙니다. 제외 패턴(주석 줄 건너뛰기 등)과
결과에 함께 보일 수정 안내도 폼에서 넣습니다.

정규식 테스터는 고른 타입 그대로, 점검 엔진과 같은 방식으로 판정합니다. 테스터
에서 맞았는데 점검에서 0건이 나오는 일이 없습니다.

편집기는 저장 전에 검증합니다. CLI가 쓰는 것과 같은 로더, 규칙 엔진이 쓰는 것과
같은 정규식 컴파일러를 씁니다. 중복 규칙 ID, 잘못된 심각도, 컴파일되지 않는
패턴을 줄 번호와 함께 짚어 주고, 검증에 걸린 파일은 저장하지 않습니다. 덮어쓰기
전 내용은 옆에 `.bak` 으로 남습니다.

[커스텀 규칙 내보내기]를 누르면 앱에서 만든 규칙이
`configs/rulesets/custom.yaml` 로 나가고 `custom` 프로파일이 등록됩니다.
그러면 `rulix ./src --profile=custom` 으로 CI에서도 같은 규칙이 돕니다.

## GitHub Action

```yaml
- uses: mhb8436/rulix-ai@v1
  with:
    path: './src'
    profile: 'essential'
    min-severity: 'medium'
    lang: 'ko'
```

데스크톱 앱에서 저장한 스냅샷을 함께 쓰면 팀이 합의한 규칙 그대로 돌아갑니다.

```yaml
- run: rulix ./src --profile=all --overrides=ruleset.yaml --summary --min-severity=high
```

## 문서

- [빠른 시작](docs/QUICK_START.ko.md) — 데스크톱 앱과 CLI, 설치부터 첫 보고서까지
- [사용자 매뉴얼](docs/USER_MANUAL.ko.md) — 화면별·옵션별·규칙별 상세
- [커스텀 규칙 작성](docs/CUSTOM_RULES.ko.md)
- [폐쇄망 검증](docs/AIRGAP_VERIFICATION.ko.md) — 차단 장치가 실제로 도는지 확인

## 라이선스

두 가지로 배포합니다.

- **AGPL-3.0** — 오픈소스 용도
- **상용 라이선스** — 독점 소프트웨어 용도

RULIX는 크래프틱시스템즈 주식회사가 만듭니다. 상용 라이선스나 기술 지원 문의는
이 저장소에 이슈로 남겨 주세요.
