# RULIX 사용자 매뉴얼

> **RULIX** (Code Quality Analyzer)
> 소스코드 품질 검사 도구 — 데스크톱 앱과 CLI

---

## 1. 개요

RULIX는 Java, JavaScript, HTML, CSS, SQL, XML 소스코드를 개발 표준에 따라
정적 분석합니다. 데스크톱 앱과 명령행 도구 두 가지로 쓸 수 있고, 둘은 같은 룰
엔진 위에 올라가 있습니다.

**주요 특징**

- 535개 검사 규칙 내장 (453개 기본 활성)
- 데스크톱 앱(Windows / macOS)과 CLI(Windows / macOS / Linux)
- 규칙 선택을 YAML 스냅샷으로 저장 — 앱에서 정한 규칙을 CI가 그대로 재현
- 앱 안에 편집기 내장 — 소스와 룰셋 YAML을 보고 고치고 바로 다시 검사
- YAML 설정만으로 규칙 추가·변경 가능 (재빌드 불필요)
- 콘솔, JSON, HTML, Excel 리포트 출력
- DDL × 쿼리 교차 분석으로 인덱스·스키마 정합성 점검
- 폐쇄망에서 로컬 추론 서버로 AI 감리 보고서 생성

**어느 쪽을 쓰는가**

| | 데스크톱 앱 (`rulix-gui`) | CLI (`rulix`) |
|---|---|---|
| 쓰는 자리 | 코드를 훑고, 고치고, 규칙을 정하고, 넘길 보고서를 만들 때 | CI 파이프라인, 정기 점검, 배치 |
| 플랫폼 | Windows x64, macOS | Windows, macOS, Linux |
| 보고서 | Excel, HTML, JSON, AI 감리, DOCX | 콘솔, Excel, HTML, JSON, AI 감리, DOCX |

둘 중 하나만 고르는 구조가 아닙니다. 앱에서 규칙을 정해 스냅샷으로 저장하고,
CI는 그 파일을 `--overrides` 로 읽습니다(3.4, 5장).

---

## 2. 설치

### 2.1 배포 파일 구조

```
rulix-windows-amd64/
  rulix.exe                      ← CLI
  rulix-gui.exe                  ← 데스크톱 앱 (Windows / macOS 만)
  rulix-report.exe               ← DOCX 감리 보고서 생성기 (Windows / macOS 만)
  configs/
    profiles.yaml               ← 프로파일 정의
    rulesets/
      quality.yaml              ← 코드 품질 (125개, 107 활성)
      secure.yaml               ← 시큐어코딩 (131개, 123 활성)
      sql.yaml                  ← SQL 공통 ANSI (120개, 102 활성)
      sql-oracle.yaml           ← SQL Oracle 전용 (20개, 18 활성)
      sql-format.yaml           ← SQL 포맷/스타일 (7개)
      modernize.yaml            ← 레거시 현대화 (50개, 39 활성)
      spring.yaml               ← Spring 표준 (32개, 28 활성)
      egov.yaml                 ← 전자정부 표준 (33개, 12 활성)
      ddl.yaml                  ← DDL × 쿼리 교차 분석 (17개)
```

Linux 번들에는 `rulix` 만 들어 있습니다. 데스크톱 앱은 Wails가 빌드 시점에
GTK·WebKit을 요구해 아직 Linux 빌드를 제공하지 않습니다.

데스크톱 앱만 필요하면 단독 파일로도 배포합니다:
`rulix-gui-windows-amd64.exe`, `rulix-gui-darwin-arm64`, `rulix-gui-darwin-amd64`.

### 2.2 Windows 설치

1. `rulix-windows-amd64.zip` 압축 해제
2. 원하는 경로에 폴더 배치 (예: `C:\tools\rulix\`)
3. 데스크톱 앱은 `rulix-gui.exe` 더블클릭
4. (선택) CLI를 아무 데서나 쓰려면 환경 변수 PATH에 추가

```
[시스템 속성] → [환경 변수] → Path → 편집 → C:\tools\rulix 추가
```

> 데스크톱 앱은 Microsoft WebView2 런타임으로 화면을 그립니다. Windows 11과
> 최근 Windows 10에는 기본 포함이고, 없으면 창이 뜨지 않거나 흰 화면이
> 나옵니다. 그럴 때
> [WebView2 Runtime](https://developer.microsoft.com/microsoft-edge/webview2/)을
> 설치하세요.

### 2.3 macOS 설치

```bash
tar -xzf rulix-darwin-arm64.tar.gz      # Intel 은 rulix-darwin-amd64.tar.gz
cd rulix-darwin-arm64

# 코드 서명·공증을 아직 적용하지 않아 Gatekeeper가 격리합니다.
# 이 줄을 건너뛰면 "손상되었기 때문에 열 수 없습니다" 가 뜹니다.
xattr -dr com.apple.quarantine .

./rulix-gui        # 데스크톱 앱
./rulix --help     # CLI
```

### 2.4 Linux 설치

```bash
tar -xzf rulix-linux-amd64.tar.gz
cd rulix-linux-amd64
./rulix --help
```

### 2.5 설치 확인

```bash
./rulix profiles
```

프로파일 목록이 출력되면 정상입니다. 데스크톱 앱은 실행 후 규칙 탭에 규칙이
채워져 있으면 정상이고, 비어 있다면 실행 파일 옆에 `configs/` 폴더가 함께
풀렸는지 확인하세요.

---

## 3. 데스크톱 앱으로 점검하기

### 3.1 화면 구성

왼쪽 사이드바에 탭 여섯 개가 있습니다. `Ctrl/Cmd` + 숫자로도 이동합니다.

| 탭 | 단축키 | 하는 일 |
|----|--------|--------|
| 점검 | `Ctrl/Cmd+1` | 소스 폴더, 프로파일, 최소 심각도, DDL 교차 분석 / 실행·취소·진행률 |
| 규칙 | `Ctrl/Cmd+2` | 규칙 켜고 끄기, 심각도 변경, 커스텀 규칙, 스냅샷 저장·불러오기, 룰셋 YAML 편집 |
| 결과 | `Ctrl/Cmd+3` | 지표·등급 대시보드, 이슈 목록 |
| 코드 | `Ctrl/Cmd+4` | 프로젝트 탐색, 소스 보기·편집, 고친 파일 즉시 재검사 |
| 보고서 | `Ctrl/Cmd+5` | Excel / HTML / JSON 내보내기 |
| AI 감리 | `Ctrl/Cmd+6` | 로컬 추론 서버로 감리 보고서 생성 |
| 설정 | `Ctrl/Cmd+7` | 앱 환경설정 |

> 앱 화면은 현재 한국어만 지원합니다. CLI는 `--lang=en|ko` 를 따릅니다.

### 3.2 점검 실행

점검 탭에서 순서대로 채웁니다.

1. **대상 폴더** — [찾아보기]로 소스 루트를 고릅니다. 경로를 직접 입력해도
   됩니다.
2. **프로파일** — 돌릴 프로파일을 체크합니다. 여러 개를 함께 고를 수
   있습니다. 무엇을 고를지는 4.2의 표를 참고하세요.
3. **최소 심각도** — 낮음 이상 / 보통 이상 / 높음 이상 / 심각만. 결과에 담을
   최소 등급입니다.
4. **DDL 크로스 분석** (선택) — DDL 파일이나 폴더를 지정하면 SQL의
   테이블·컬럼 존재성과 인덱스 사용까지 함께 점검합니다.

실행하면 진행률이 표시되고, 오래 걸리는 대형 프로젝트는 도중에 취소할 수
있습니다. 끝나면 결과 탭으로 넘어갑니다.

### 3.3 결과 보기

결과 탭 위쪽은 지표와 등급 대시보드입니다. 규모(라인 수, 주석 비율), 순환
복잡도, 중복도, 기술부채와 A~E 등급이 나옵니다. 규칙 위반 건수만으로는 보이지
않는 "규모 대비 품질"이 여기서 드러납니다.

아래쪽은 이슈 목록입니다. 심각도와 규칙으로 걸러 볼 수 있고, 이슈를 누르면
해당 파일이 그 줄에서 열립니다.

### 3.4 규칙 조정과 스냅샷

규칙 탭에서 이 프로젝트에 맞게 규칙을 손봅니다.

- **끄기 / 켜기** — 토글 하나로 바뀝니다. 프로파일 단위 일괄 토글도 됩니다.
- **심각도 변경** — 기본 심각도가 현장과 안 맞으면 규칙별로 바꿉니다.
- **커스텀 규칙** — 팀 규약을 정규식으로 등록합니다. 저장 전에 샘플 코드에
  붙여 실제로 매치되는지 확인할 수 있어, 패턴을 틀린 채로 넣는 일이 줄어듭니다.

조정이 끝나면 **스냅샷 저장**으로 `ruleset.yaml` 을 남깁니다. 이 파일이 CLI의
`--overrides` 가 읽는 바로 그 형식이라, 앱에서 켜고 끈 규칙과 바꾼 심각도를
CI가 매 빌드 그대로 재현합니다.

**커스텀 규칙의 패턴 타입** — 폼에서 패턴이 보는 단위를 고릅니다. [한 줄]은
각 줄을 따로 보고, [여러 줄]은 파일 전체를 이어서 봅니다. 어노테이션과 선언이
다른 줄에 있는 규칙(전자정부 명명 규칙 같은 것)은 [여러 줄]이라야 잡힙니다.
정규식 테스터도 고른 타입 그대로 판정하므로, 테스터에서 맞았는데 점검에서
0건이 나오는 일이 없습니다. 제외 패턴과 수정 안내도 폼에서 넣습니다.

**내장 규칙 정의 보기** — 규칙 목록에서 내장 규칙의 `</>` 를 누르면 그 규칙이
정의된 룰셋 파일이 열리고 해당 줄로 바로 갑니다.

**룰셋 파일 편집** — 폼으로 만들 수 없는 타입(`ast-*` 계열 등)은 YAML을 직접 고칩니다.
`regex-multiline` 이나 `ast-*` 계열, 제외 패턴처럼 그 밖의 것은 [탐색] 아래
[룰셋 파일 편집]에서 YAML을 직접 고칩니다. 저장 전에 CLI가 쓰는 로더와 규칙
엔진이 쓰는 정규식 컴파일러로 검증하고, 중복 ID·잘못된 심각도·컴파일되지 않는
패턴을 줄 번호와 함께 짚어 줍니다. 검증에 걸린 파일은 저장하지 않고, 덮어쓰기
전 내용은 옆에 `.bak` 으로 남습니다.

**커스텀 규칙 내보내기** — 앱이 만든 커스텀 규칙은 `~/.rulix/custom_rules.yaml`
에 들어가는데 CLI 는 그 파일을 읽지 않습니다. [커스텀 규칙 내보내기]를 누르면
`configs/rulesets/custom.yaml` 로 나가고 `custom` 프로파일이 등록되어,
`rulix <경로> --profile=custom` 으로 CI에서도 같은 규칙이 돕니다.

```bash
rulix /path/to/project --profile=all --overrides=ruleset.yaml \
  -o excel --output-file=report.xlsx
```

반대 방향도 됩니다. CLI의 `--save-overrides` 로 만든 파일을 앱에서 불러오면
같은 상태가 화면에 그대로 뜹니다. 파일 형식은 5장에 있습니다.

### 3.5 보고서 내보내기

보고서 탭에서 Excel, HTML, JSON을 만듭니다.

| 형식 | 쓰는 자리 |
|------|----------|
| Excel | 기관·개발사 제출용. 시트별 전체 데이터 |
| HTML | 브라우저로 바로 여는 단일 파일 |
| JSON | CI 연동, 외부 도구 import, AI 감리 보고서 입력 |

만든 파일은 [최근 생성] 목록에 남아 바로 열어볼 수 있습니다.

### 3.6 코드 보기와 편집

코드 탭(`Ctrl/Cmd+4`)에서 프로젝트를 탐색하고 소스를 봅니다. 편집기가 앱에
들어 있어 현장에 별도 편집기를 반입할 필요가 없습니다.

- **파일 찾기** — 트리 위 상자에 이름을 넣으면 걸러집니다. 파일이 수천 개인
  프로젝트에서 폴더를 하나씩 펼쳐 내려가는 것이 컨설팅 중에 가장 자주 하는
  헛일입니다.
- **열기** — 왼쪽 트리에서 파일을 고르거나, 이슈 상세의 [코드에서 열기]를
  누르면 그 파일의 그 줄이 열립니다. 같은 파일의 이슈가 전부 여백에 표시됩니다.
- **이슈 사이 이동** — 편집기 위 화살표로 건너뜁니다. `F8` · `Shift+F8` 도
  같습니다. 표시가 `4곳 중 3번째` 처럼 나와 지금 어디인지 알 수 있고, 같은
  줄에 여러 건이어도 한 번만 멈춥니다.
- **탐색** — 점검 대상이 아닌 파일도 트리에 보입니다. 컨설팅에서는 규칙이 보는
  파일만이 아니라 설정과 빌드 스크립트도 같이 읽어야 하기 때문입니다.
  `node_modules` 같은 의존성 폴더는 제외합니다.
- **들여쓰기** — 새로 입력하는 것은 열어 둔 파일을 따라갑니다. RULIX 가 탭과
  스페이스를 규칙으로 검사하므로, 편집기가 반대쪽을 넣으면 고치러 들어와서
  위반을 만드는 셈이 됩니다. 기존 내용은 다시 포맷하지 않습니다.
- **편집** — 기본은 읽기 전용입니다. [편집]을 눌러야 고칠 수 있습니다. 저장은
  `Ctrl/Cmd+S` 입니다.
- **재검사** — 저장하면 **그 파일만** 즉시 다시 검사합니다. 고쳐졌는지 보려고
  대형 프로젝트를 통째로 다시 돌릴 필요가 없습니다. 다만 크로스파일 규칙은
  프로젝트 전체를 봐야 판정할 수 있어 한 파일 재검사에서는 빠집니다.

저장할 때 원본의 줄끝(CRLF/LF)과 마지막 개행, 파일 권한을 그대로 지킵니다.
CRLF 파일을 LF로 저장해 버리면 한 줄만 고쳐도 파일 전체가 바뀐 것으로 잡히기
때문입니다. 편집하는 사이에 파일이 외부에서 바뀌었으면 덮어쓰지 않고
알려 줍니다.

> 2MB 를 넘는 파일과 바이너리는 열지 않습니다. 열어 둔 프로젝트 폴더 밖의
> 파일도 읽거나 쓰지 않습니다.

### 3.7 결과가 낡았을 때

규칙을 끄거나 심각도를 바꾸거나 소스를 고치면, 이미 나온 결과는 그 전 상태를
보여줍니다. 결과·보고서·AI 감리 화면 위에 그 사실을 알리는 줄이 뜨고
[다시 점검] 이 함께 나옵니다.

자동으로 다시 계산하지는 않습니다. 대형 프로젝트에서 규칙 하나 끌 때마다 몇
초씩 멈추면 규칙을 조율할 수가 없기 때문입니다. 대신 **낡은 결과로 보고서를
만드는 일**이 없도록 분명히 표시합니다.

### 3.8 화면 사이 이동

화면 안의 버튼으로 들어간 곳에는 돌아갈 길이 함께 뜹니다. 이슈에서
[코드에서 열기]로 들어가면 `← 결과로 돌아가기 · secure-plain-001` 이 생기고,
누르면 보던 이슈 목록으로 필터까지 그대로 돌아옵니다.

사이드바로 직접 간 화면에는 뜨지 않습니다. 그건 "돌아가기"가 아니라 "새로
가기"이기 때문입니다. 방문 기록은 따로 쌓이며 `Ctrl/Cmd+[` 와 `Ctrl/Cmd+]` 로
오갑니다.

대시보드에서 심각도나 카테고리를 눌러 이슈 목록으로 들어가면 걸린 필터가
칩으로 보이고 `✕` 로 풉니다.

### 3.9 AI 감리 보고서

AI 감리 탭에서 점검 결과를 감리 보고서로 만듭니다. 엔드포인트와 모델을 넣고
[연결 확인]으로 서버가 응답하는지 먼저 본 뒤 생성하면 됩니다. 폐쇄망 모드는
체크박스입니다. 같은 탭에서 기관 제출용 DOCX 보고서도 만들 수 있습니다.
자세한 내용은 10장에 있습니다.

---

## 4. 명령행으로 점검하기

### 4.1 소스코드 검사

```cmd
REM 기본 검사 (전체 규칙)
rulix.exe C:\projects\my-app\src

REM 특정 프로파일로 검사
rulix.exe C:\projects\my-app\src --profile=quality
rulix.exe C:\projects\my-app\src --profile=sql
rulix.exe C:\projects\my-app\src --profile=secure

REM 여러 프로파일 조합
rulix.exe C:\projects\my-app\src --profile=quality,secure,sql
```

### 4.2 사용 가능한 프로파일

| 프로파일 | 규칙 수 | 기본 활성 | 대상 언어 | 설명 |
|----------|---------|-----------|-----------|------|
| `quality` | 125 | 107 | Java, JS, HTML, CSS | 코드 품질, 명명규칙, 복잡도 |
| `secure` | 131 | 123 | Java, JS | 시큐어코딩 가이드 |
| `sql` | 120 | 102 | SQL, XML, Java | ANSI SQL 공통 규칙 (DB 불문) |
| `sql-oracle` | 20 | 18 | SQL, XML, Java | Oracle 전용 (NVL, ROWNUM, hints 등) |
| `sql-format` | 7 | 7 | SQL, XML, Java | SQL 포맷 강제 (대문자, 80자 등) |
| `modernize` | 50 | 39 | Java, XML | eGov 3.x→4.x, javax→jakarta |
| `spring` | 32 | 28 | Java | Spring/Boot 개발 표준 |
| `egov` | 33 | 12 | Java, XML | 전자정부 프레임워크 표준 |
| `ddl` | 17 | 17 | SQL, XML, Java | DDL × 쿼리 교차 분석 (`--ddl` 필요) |

**SQL 프로파일 조합 가이드**

| DB 환경 | 권장 프로파일 | 명령어 |
|---------|-------------|--------|
| MySQL / PostgreSQL / CUBRID | `sql` | `--profile=sql` |
| Oracle | `sql,sql-oracle` | `--profile=sql,sql-oracle` |
| Oracle + 포맷 강제 | `sql-all` (그룹) | `--profile=sql-all` |
| 아무 DB + 포맷 강제 | `sql,sql-format` | `--profile=sql,sql-format` |

**프로파일 그룹**

| 그룹 | 포함 프로파일 | 설명 |
|------|-------------|------|
| `all` | 전체 8개 (ddl 제외) | 모든 규칙 검사 (Oracle+포맷 포함) |
| `essential` | quality, secure | 필수 품질+보안 |
| `sql-all` | sql, sql-oracle, sql-format | SQL 전체 (Oracle+포맷) |
| `sql-ddl` | sql, sql-oracle, ddl | SQL + 스키마 교차 검증 |
| `egov-full` | quality, secure, sql, sql-oracle, sql-format, egov, spring | 전자정부 전체 |
| `migration` | modernize, spring | 마이그레이션 점검 |

### 4.3 출력 형식

```cmd
REM 콘솔 출력 (기본)
rulix.exe C:\src --profile=sql

REM HTML 리포트
rulix.exe C:\src --profile=all -o html --output-file=report.html

REM Excel 리포트
rulix.exe C:\src --profile=all -o excel --output-file=report.xlsx

REM JSON 출력
rulix.exe C:\src --profile=all -o json --output-file=report.json
```

**Excel 리포트 시트 구성**

| 시트명 | 내용 |
|--------|------|
| 요약 | 전체 통계 (파일 수, 이슈 수, 심각도별 분포) |
| 이슈목록 | 전체 이슈 상세 (필터, 정렬 가능) |
| 파일별통계 | 파일별 이슈 수 |
| 적용규칙 | 적용된 규칙 목록과 위반 건수 |

### 4.4 심각도 필터

```cmd
REM high 이상만 표시
rulix.exe C:\src --profile=all --min-severity=high

REM critical만 표시
rulix.exe C:\src --profile=all --min-severity=critical
```

심각도 단계: `low` < `medium` < `high` < `critical`

### 4.5 상세 모드

```cmd
rulix.exe C:\src --profile=sql -v
```

로드된 프로파일, 처리 파일 수, 총 이슈 수 등 상세 정보를 표시합니다.

### 4.6 규칙별 요약 테이블 (CI/CD)

```cmd
rulix.exe C:\src --profile=all --summary
```

규칙 ID별 위반 건수를 테이블로 출력합니다. CI/CD 파이프라인에서 빠르게 확인할 때 유용합니다.

### 4.7 특정 규칙 제외

```cmd
REM 특정 규칙 ID를 제외하고 검사 (오버라이드 파일 없이)
rulix.exe C:\src --profile=all --exclude-rule=quality-lg-005,secure-sql-002
```

### 4.8 크로스파일 분석만 실행

```cmd
REM MyBatis 미사용 SQL ID, Mapper-VO 불일치 등 아키텍처 이슈만 확인
rulix.exe C:\src --profile=all --cross-file-only
```

### 4.9 중복 규칙 병합 제어

여러 프로파일을 조합하면 동일한 이슈가 여러 규칙에서 중복 검출될 수 있습니다. 기본적으로 RULIX는 같은 파일+라인에서 중복 검출된 이슈를 자동 병합합니다.

```cmd
REM 기본: 자동 병합 (동일 위치 중복 이슈는 최고 심각도 1건만 유지)
rulix.exe C:\src --profile=all

REM 병합 비활성화: 모든 이슈를 개별 표시 (디버깅/규칙 확인 용)
rulix.exe C:\src --profile=all --no-dedup
```

---

## 5. 규칙 오버라이드

현장 환경에 맞게 규칙의 심각도를 변경하거나 특정 규칙을 끌 수 있습니다.

이 파일은 데스크톱 앱 규칙 탭에서 [스냅샷 저장]으로 만든 것과 같은 형식입니다
(3.4). 앱에서 토글로 조정한 결과를 CLI가 그대로 읽고, CLI의
`--save-overrides` 로 만든 파일을 앱이 다시 불러옵니다.

> 규칙 식별 필드는 `id` 입니다. `rule_id` 로 적으면 오류 없이 조용히
> 무시되므로, 껐는데도 계속 검출된다면 이 항목부터 확인하세요.

### 5.1 오버라이드 파일 작성

`overrides.yaml` 파일 생성:

```yaml
overrides:
  # 규칙 비활성화
  - id: "sql-fmt-001"
    enabled: false

  # 심각도 변경 (high → low)
  - id: "sql-smell-017"
    severity: "low"

  # 여러 규칙 한번에 비활성화
  - id: "sql-from-001"
    enabled: false
  - id: "sql-dml-001"
    enabled: false
```

### 5.2 오버라이드 적용

```cmd
rulix.exe C:\src --profile=sql --overrides=overrides.yaml
```

---

## 6. 규칙 상세

### 6.1 SQL 프로파일 구조

SQL 규칙은 DB 환경에 따라 3개 프로파일로 분리되어 있습니다:

#### sql (ANSI SQL 공통 — 88개)

DB 벤더에 무관하게 적용되는 공통 SQL 규칙입니다.
MySQL, PostgreSQL, CUBRID, Oracle 등 모든 DB에서 사용할 수 있습니다.

- SELECT *, INSERT 컬럼 누락, UPDATE/DELETE WHERE 누락
- 서브쿼리, 별칭, 조인 조건 검사
- SQL 냄새 (Code Smell): NATURAL JOIN, ON 1=1, AND/OR 괄호 등
- MyBatis: ${} SQL Injection, SELECT *, WHERE 누락

#### sql-oracle (Oracle 전용 — 19개)

Oracle DB 환경에서만 의미 있는 규칙입니다.

| 분류 | 규칙 예시 |
|------|----------|
| Oracle 힌트 | `/*+ HINT */` 무단 사용, LEADING 외 힌트 금지 |
| Oracle 함수 | NVL, TO_CHAR, DECODE, DBMS_RANDOM, INSTR, REGEXP_LIKE |
| Oracle 구문 | ROWNUM 페이징, EXECUTE IMMEDIATE, FOR UPDATE WAIT |

#### sql-format (포맷/스타일 — 7개)

팀별 코딩 스타일에 따라 선택적으로 적용하는 포맷 규칙입니다.

| ID | 규칙명 | 설명 |
|----|--------|------|
| sql-fmt-001 | SQL 키워드 대문자 강제 | SELECT, FROM 등 대문자 |
| sql-fmt-002 | 라인 80자 제한 | 한 줄 80자 초과 감지 |
| sql-fmt-003 | SQL 내 빈줄 금지 | SQL 중간 빈 라인 감지 |
| sql-sel-004 | 한줄 한컬럼 | 컬럼을 한 줄에 하나만 기술 |
| sql-from-005 | 한줄 한테이블 | 테이블을 한 줄에 하나만 기술 |
| sql-where-007 | 한줄 한조건 | 조건을 한 줄에 하나만 기술 |
| sql-sel-006 | 주석 형식 강제 | `/* */` 형식만 허용 |

### 6.2 SQL 냄새(Smell) 규칙

#### 공통 Smell (sql.yaml 내)

| ID | 규칙명 | 심각도 | 설명 |
|----|--------|--------|------|
| sql-smell-010 | NULL = 비교 | high | `col = NULL`은 항상 UNKNOWN. IS NULL 사용 |
| sql-smell-011 | NATURAL JOIN | high | 컬럼 추가 시 조인 조건이 암묵적 변경 |
| sql-smell-012 | ON 1=1 가짜 조인 | high | Cartesian Product를 조인으로 위장 |
| sql-smell-013 | 자기 자신 별칭 | low | `COL AS COL` 무의미한 별칭 |
| sql-smell-017 | COUNT(DISTINCT) | low | 대량 데이터 정렬/해시 비용 |
| sql-smell-018 | WHERE 1=1 | low | 동적 SQL 흔적, 코드 정리 필요 |
| sql-smell-020 | DISTINCT + GROUP BY | high | GROUP BY만으로 유일성 보장 |
| sql-smell-021 | INSERT INTO ... ORDER BY | high | INSERT에 ORDER BY는 무의미 |
| sql-smell-023 | DISTINCT + JOIN | high | 잘못된 JOIN 중복을 DISTINCT로 숨김 |
| sql-smell-024 | LEFT JOIN + COUNT(*) | high | NULL 행까지 카운트됨 |
| sql-smell-025 | 동일 컬럼 다중 OR | low | IN 절로 변환 권장 |
| sql-smell-026 | 중첩 CTE | high | WITH 안에 WITH, 순차 CTE로 분리 |
| sql-smell-027 | AND/OR 괄호 누락 | high | 우선순위 실수 유발 |

#### Oracle 전용 Smell (sql-oracle.yaml 내)

| ID | 규칙명 | 심각도 | 설명 |
|----|--------|--------|------|
| sql-smell-003 | WHERE NVL/COALESCE 감싸기 | high | 인덱스 무효화 |
| sql-smell-014 | ORDER BY DBMS_RANDOM | high | 전체 테이블 스캔+정렬 유발 |
| sql-smell-015 | WHERE절 INSTR | high | 문자열 검색 함수로 인덱스 무효화 |
| sql-smell-016 | REGEXP_LIKE | low | 단순 패턴은 LIKE가 효율적 |
| sql-smell-022 | JOIN ON절 NVL/TO_CHAR | high | ON 조건 함수로 인덱스 무효화 |

#### AST 구조 분석 (6개, 공통)

| ID | 규칙명 | 심각도 | 설명 |
|----|--------|--------|------|
| sql-smell-030 | LEFT JOIN WHERE 필터 | high | LEFT JOIN이 사실상 INNER JOIN |
| sql-smell-031 | LEFT JOIN 뒤 INNER JOIN | high | INNER JOIN이 LEFT JOIN 무효화 |
| sql-smell-032 | HAVING 비집계 조건 | high | WHERE로 이동해야 성능 향상 |
| sql-smell-033 | 미사용 CTE | high | WITH 정의 후 미참조 |
| sql-smell-034 | 서브쿼리 ORDER BY | high | 페이징 없는 정렬은 무의미 |
| sql-smell-035 | 인라인뷰 별칭 누락 | high | 별칭 없으면 참조 불가 |

### 6.3 규칙 타입 요약

| 타입 | 설명 | 수정 범위 |
|------|------|-----------|
| `regex` | 한 줄 패턴 매칭 | YAML만 수정 |
| `regex-multiline` | 여러 줄 패턴 매칭 | YAML만 수정 |
| `ast-filtered-regex` | 주석 오탐 제거 정규식 (ANTLR4 토큰 필터) | YAML만 수정 |
| `ast-method-call` | Java AST 메서드 호출 검사 | YAML만 수정 |
| `ast-annotation` | Java AST 어노테이션 검사 | YAML만 수정 |
| `ast-import` | Java AST import 경로 검사 | YAML만 수정 |
| `ast-variable` | Java AST 변수/필드 검사 | YAML만 수정 |
| `ast-try-catch` | Java AST 예외 처리 검사 | YAML만 수정 |
| `ast-class` | Java AST 클래스 구조 검사 | YAML만 수정 |
| `ast-method` | Java AST 메서드 구조 분석 | YAML만 수정 |
| `ast-dead-code` | Java AST Dead Code 검출 | YAML만 수정 |
| `ast-multi-cud` | 다건 CUD 트랜잭션 검사 | YAML만 수정 |
| `cross-file` | 크로스파일 분석 (MyBatis 미사용 SQL ID 등) | YAML만 수정 |
| `annotation-missing-attr` | Java 어노테이션 속성 검사 | YAML만 수정 |
| `line-length` | 라인 길이 검사 | YAML만 수정 |
| `ast-sql-*` | SQL AST 구조 분석 | Go 코드 + YAML |
| `method-analysis` | Go 내장 분석 로직 | Go 코드 필요 |

### 6.4 규칙 엔진 동작 방식 비교

RULIX의 규칙은 입력 데이터와 매칭 방식에 따라 크게 4가지로 분류됩니다.

#### regex (단일 라인 매칭)

```
입력: file.Lines → 한 줄씩 순회
처리: pattern.MatchString(line)
```

- 파일의 각 줄을 **개별적으로** regex 매칭
- `.`는 줄바꿈(`\n`)을 매칭하지 않음
- 가장 빠르고 단순한 방식
- **한계**: 여러 줄에 걸친 패턴 감지 불가, 주석 내 오탐 가능

#### regex-multiline (전체 파일 매칭)

```
입력: strings.Join(file.Lines, "\n") → 파일 전체를 하나의 문자열로 합침
처리: pattern.FindStringMatch(content) → FindNextMatch 반복
```

- 파일의 **모든 줄을 `\n`으로 join**하여 하나의 문자열로 처리
- `.`가 `\n`도 매칭 (Singleline 모드)
- `SELECT\s+\*\s+FROM` 같은 **여러 줄에 걸친 패턴** 감지 가능
- 라인 번호는 매치 위치에서 역산
- **한계**: 주석 내 오탐 가능, 큰 파일에서 상대적으로 느림

#### ast-filtered-regex (주석 오탐 제거 regex)

```
입력: file.Lines → 한 줄씩 순회 (regex와 동일)
추가: ANTLR4 파서로 주석 라인 맵 생성 → 주석 줄 자동 스킵
```

- 기본 동작은 regex와 동일 (한 줄씩 매칭)
- **차이점**: ANTLR4 토큰 스트림으로 **주석 전용 라인을 자동 필터링**
- 예: `// System.out.println 금지` 같은 주석에서 오탐하지 않음
- Java 파일이 아니거나 AST가 없으면 일반 regex와 동일하게 동작

#### AST 쿼리 (ast-method-call, ast-class, ast-annotation 등)

```
입력: ANTLR4 파스 트리 (구조화된 AST)
처리: FindMethodInvocations(), FindAnnotations(), FindClassDeclarationsInfo() 등
     → 구조화된 데이터에서 속성 조건 매칭
```

- **텍스트가 아닌 파싱된 구조체**를 순회
- 메서드 호출, 어노테이션, 클래스, 변수, try-catch 등을 구조적으로 식별
- 컨텍스트 조건 지원: `inside-loop`, `inside-catch`, `method-name:<regex>` 등
- 주석/문자열 내 오탐이 구조적으로 불가능
- **한계**: Java만 지원 (ANTLR4 파서 필요)

#### 비교 요약표

| 항목 | regex | regex-multiline | ast-filtered-regex | AST 쿼리 |
|------|-------|-----------------|-------------------|----------|
| **입력 단위** | 라인 1개씩 | 파일 전체 (join) | 라인 1개씩 | ANTLR4 파스 트리 |
| **매칭 방식** | 텍스트 정규식 | 텍스트 정규식 | 정규식 + 주석 필터 | 구조체 속성 비교 |
| **멀티라인 패턴** | 불가 | **가능** | 불가 | 해당없음 (구조) |
| **주석 오탐 제거** | 없음 | 없음 | **자동 제거** | **구조적 제거** |
| **컨텍스트 조건** | 없음 | 없음 | 없음 | inside-loop, inside-catch 등 |
| **타임아웃** | 100ms | 500ms | 100ms | 없음 (트리 순회) |
| **지원 언어** | 모든 언어 | 모든 언어 | Java (fallback: 전체) | Java 전용 |
| **YAML 설정** | `type: "regex"` | `type: "regex-multiline"` | `type: "ast-filtered-regex"` | `type: "ast-method-call"` 등 |
| **적합한 용도** | 단순 키워드 감지 | 여러 줄 패턴 | 주석 오탐 제거 필요 시 | 메서드/클래스/구조 분석 |

#### 동일 규칙의 타입별 구현 예시

`System.out.println` 사용을 감지하는 규칙을 3가지 방식으로 작성한 예시:

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

| 소스코드 | regex | ast-filtered-regex | ast-method-call |
|----------|-------|-------------------|-----------------|
| `System.out.println("hello");` | 검출 | 검출 | 검출 |
| `// System.out.println 금지` | 오탐 | 스킵 | 스킵 |
| `/* System.out.println */` | 오탐 | 스킵 | 스킵 |
| `String s = "System.out.println";` | 오탐 | 오탐 | 스킵 |

> **선택 가이드**: 단순 키워드는 `regex`, 주석 오탐이 문제되면 `ast-filtered-regex`, 구조적 정확성이 필요하면 `ast-*` 타입을 사용하세요. 여러 줄 패턴은 `regex-multiline`만 가능합니다.

---

## 7. 규칙 추가 가이드

### 7.1 Regex 규칙 추가 (가장 간단)

`configs/rulesets/sql.yaml`에 아래 형식으로 추가합니다:

```yaml
- id: "sql-custom-001"
  name: "사용자 정의 규칙"
  severity: "high"           # low, medium, high, critical
  category: "performance"
  description: "이 규칙의 설명"
  enabled: true
  pattern:
    type: "regex"
    regex: "\\bFORBIDDEN_KEYWORD\\b"
    flags: "i"               # i = 대소문자 무시
  custom:
    rule_id: "SQL-CUSTOM-001"
    fix: "수정 방법 안내"
```

**regex 작성 시 주의사항**

- YAML에서 `\`는 `\\`로 이스케이프
- `\b` = 단어 경계, `\s` = 공백, `\w` = 단어 문자
- `(?:...)` = 비캡처 그룹
- `flags: "i"` 를 `custom:` 아래에 넣으면 대소문자 무시

### 7.2 Regex-multiline 규칙 추가

여러 줄에 걸친 패턴을 검사합니다:

```yaml
- id: "sql-custom-002"
  name: "SELECT 후 불필요한 패턴"
  severity: "high"
  category: "performance"
  description: "SELECT 문에서 특정 패턴이 감지되었습니다"
  enabled: true
  pattern:
    type: "regex-multiline"
    regex: "\\bSELECT\\b[\\s\\S]*?\\bFORBIDDEN\\b"
    flags: "i"
  custom:
    rule_id: "SQL-CUSTOM-002"
    fix: "수정 방법 안내"
```

**멀티라인 패턴 핵심**

- `[\\s\\S]*?` = 줄바꿈 포함 모든 문자 (비탐욕적)
- `[\\s\\S]*` = 줄바꿈 포함 모든 문자 (탐욕적)
- 패턴이 너무 넓으면 오탐 발생, `*?` (비탐욕적) 사용 권장

### 7.3 규칙 비활성화

YAML에서 `enabled: false`로 변경하거나 오버라이드 파일 사용:

```yaml
# 방법 1: sql.yaml에서 직접 수정
- id: "sql-smell-016"
  enabled: false       # ← 비활성화

# 방법 2: overrides.yaml 사용 (원본 수정 없이)
overrides:
  - id: "sql-smell-016"
    enabled: false
```

### 7.4 새 프로파일 추가

`configs/profiles.yaml`에 추가:

```yaml
profiles:
  # 기존 프로파일...

  my-rules:
    name: "우리 프로젝트 규칙"
    description: "프로젝트 전용 규칙셋"
    ruleset: "rulesets/my-rules.yaml"
    languages:
      - java
      - sql

groups:
  # 기존 그룹...

  my-project:
    name: "우리 프로젝트 전체"
    description: "우리 프로젝트용 전체 검사"
    profiles:
      - quality
      - secure
      - my-rules
```

그런 다음 `configs/rulesets/my-rules.yaml` 파일을 생성합니다.

### 7.5 새 YAML 규칙셋 파일 생성

```yaml
version: "1.0"
profile: "my-rules"

languages:
  - language: java
    rules:
      - id: "my-java-001"
        name: "System.exit 사용 금지"
        severity: "critical"
        category: "restriction"
        description: "System.exit() 호출은 금지됩니다."
        enabled: true
        pattern:
          type: "regex"
          regex: "\\bSystem\\.exit\\s*\\("
        custom:
          rule_id: "MY-JAVA-001"
          fix: "System.exit() 대신 예외를 throw하세요"

  - language: sql
    rules:
      - id: "my-sql-001"
        name: "TRUNCATE 사용 금지"
        severity: "critical"
        category: "restriction"
        description: "TRUNCATE TABLE은 금지됩니다."
        enabled: true
        pattern:
          type: "regex"
          regex: "\\bTRUNCATE\\s+TABLE\\b"
          flags: "i"
        custom:
          rule_id: "MY-SQL-001"
          fix: "DBA 승인 없이 TRUNCATE를 사용할 수 없습니다"
```

---

## 8. 실전 활용 예시

### 8.1 SQL 파일만 검사

```cmd
rulix.exe C:\projects\sql-files --profile=sql -o excel --output-file=sql_report.xlsx
```

### 8.2 CI/CD 파이프라인 연동

```cmd
REM critical 이슈가 있으면 빌드 실패 (종료 코드 1)
rulix.exe C:\src --profile=essential --min-severity=critical
if %ERRORLEVEL% NEQ 0 (
    echo "Critical 이슈 발견! 빌드 실패"
    exit /b 1
)
```

### 8.3 배치 파일 예시

`run_rulix.bat` 파일 생성:

```bat
@echo off
SET RULIX_HOME=C:\tools\rulix
SET TARGET=%1

if "%TARGET%"=="" (
    echo 사용법: run_rulix.bat [검사대상경로]
    exit /b 1
)

echo === RULIX 소스코드 품질 검사 ===
echo 대상: %TARGET%
echo.

%RULIX_HOME%\rulix.exe %TARGET% ^
    --profile=all ^
    --profiles-file=%RULIX_HOME%\configs\profiles.yaml ^
    -o excel ^
    --output-file=rulix_report.xlsx ^
    --min-severity=low ^
    -v

echo.
echo 리포트 생성 완료: rulix_report.xlsx
pause
```

실행:
```cmd
run_rulix.bat C:\projects\my-app\src
```

### 8.4 MyBatis XML 포함 검사

SQL 프로파일은 `.sql` 파일뿐 아니라 MyBatis XML (`*Mapper.xml`) 내부의 SQL도 함께 검사합니다.

```cmd
REM src 아래 .sql 파일 + .xml(MyBatis) 모두 검사
rulix.exe C:\projects\my-app\src --profile=sql
```

---

## 9. 옵션 전체 목록

| 옵션 | 축약 | 기본값 | 설명 |
|------|------|--------|------|
| `--profile` | `-p` | `all` | 프로파일 지정 (쉼표 구분) |
| `--config` | `-c` | - | 설정 파일 직접 지정 (--profile과 동시 사용 불가) |
| `--profiles-file` | - | `configs/profiles.yaml` | 프로파일 정의 파일 경로 |
| `--overrides` | - | - | 규칙 오버라이드 파일 (데스크톱 앱 스냅샷과 같은 형식) |
| `--save-overrides` | - | - | 지금 적용된 규칙 집합을 YAML로 저장 |
| `--ddl` | - | - | DDL 파일·디렉터리. 지정하면 교차 분석 수행 |
| `--lang` | - | 시스템 로캘 | 출력 언어 (en/ko) |
| `--output` | `-o` | `console` | 출력 형식 (console/json/html/excel) |
| `--output-file` | - | stdout | 출력 파일 경로 |
| `--min-severity` | `-s` | `low` | 최소 심각도 필터 |
| `--rules` | - | - | 검사할 카테고리 필터 (쉼표 구분) |
| `--exclude-rule` | - | - | 제외할 규칙 ID (쉼표 구분) |
| `--summary` | - | false | 규칙별 요약 테이블 출력 (CI/CD 용) |
| `--cross-file-only` | - | false | 크로스파일 이슈만 표시 (아키텍처 리뷰 모드) |
| `--no-dedup` | - | false | 중복 규칙 자동 병합 비활성화 |
| `--verbose` | `-v` | false | 상세 출력 |

**서브커맨드**

| 커맨드 | 설명 |
|--------|------|
| `profiles` | 사용 가능한 프로파일 목록 표시 |
| `profiles -v` | 프로파일 상세 정보 표시 |
| `ai-report` | 점검 결과로 AI 감리 보고서 생성 (아래 10장) |
| `report` | 엑셀 점검 결과를 감리 보고서로 병합 |

---

## 10. AI 감리 보고서 (폐쇄망 지원)

점검 결과 JSON을 받아 감리 보고서 HTML을 만든다. OpenAI 호환 엔드포인트면
무엇이든 붙으므로 Ollama·vLLM·LM Studio 같은 **사내 추론 서버를 그대로 쓸 수
있고, 인터넷 연결이 필요 없다.**

```bash
# 1) 점검 결과를 JSON으로 저장
rulix /path/to/src --profile=all -o json --output-file=scan.json

# 2) 사내 추론 서버로 보고서 생성
rulix ai-report scan.json --offline \
    --endpoint http://127.0.0.1:11434/v1 \
    --model qwen2.5-coder:7b \
    --project "OO시스템" \
    -o report.html
```

### 옵션

| 옵션 | 기본값 | 설명 |
|------|--------|------|
| `--output` / `-o` | `ai-report.html` | 결과 HTML 경로 |
| `--provider` | `openai` | `openai`(호환 엔드포인트) / `claude` / `gemini` |
| `--endpoint` | OpenAI 공식 | OpenAI 호환 엔드포인트 주소 |
| `--model` | 프로바이더 기본값 | 모델 이름 |
| `--api-key` | - | API 키. 로컬 서버는 아무 값이나 된다 |
| `--project` | 입력 파일명 | 보고서에 쓸 대상 시스템 이름 |
| `--lang` | `ko` | `ko` / `en` |
| `--offline` | false | **폐쇄망 모드.** 루프백 밖 연결을 차단 |
| `--timeout` | `5m` | 요청당 제한시간 |

### `--offline` — 폐쇄망 모드

`--offline` 을 켜면 RULIX는 **루프백(127.0.0.1, ::1) 밖으로 나가는 모든 연결을
거부한다.** 목적지 IP를 보고 막으므로 호스트명을 바꿔 우회할 수 없고, 이름
해석(DNS)도 함께 막힌다. 프록시 설정으로도 빠져나가지 못한다.

외부 주소를 지정하면 보고서가 만들어지지 않고 차단 사유와 함께 실패한다.

```
$ rulix ai-report scan.json --offline --endpoint https://api.openai.com/v1
🔒 폐쇄망 모드: 루프백(127.0.0.1) 밖으로 나가는 연결을 차단합니다
Error: ... 폐쇄망 모드: 외부 연결이 차단되었습니다. 루프백 주소만 허용됩니다
```

이 동작을 직접 확인하는 절차는 [폐쇄망 동작 검증](AIRGAP_VERIFICATION.ko.md)에
정리되어 있다.

`claude`, `gemini` 프로바이더는 외부 API이므로 `--offline` 과 함께 쓸 수 없다.

### 보고서 내용

| 항목 | 만드는 주체 |
|------|------------|
| 대표 발견사항 선정 | RULIX (심각도·분류 기준, 항상 같은 입력에 같은 결과) |
| 파일명·라인·규칙 ID | RULIX (점검 결과 그대로. 모델이 바꾸지 못한다) |
| 품질 점수·A~E 등급 | RULIX (부채비율·보안 심각도에서 계산) |
| 문제 설명·개선 코드 | 모델 |
| 총평·위험·로드맵·결론 | 모델 |

모델이 존재하지 않는 파일명이나 규칙 ID를 지어내면 그 항목은 기각되고 규칙에
등록된 설명으로 대체된다. 보고서 하단의 생성 주체 표기에는 실제 사용한
엔드포인트 성격과 모델명이 남는다 (예: `local:qwen2.5-coder:7b`).

### 권장 모델

폐쇄망 환경에서 실측한 결과는 [로컬 LLM 평가](LOCAL_LLM_EVALUATION.md) 참고.
7B급에서는 `qwen2.5-coder:7b` 가 가장 안정적이었다.

### GUI에서 만들기

데스크톱 앱의 **[AI 감리]** 탭(`⌘5` / `Ctrl+5`)에서 같은 일을 한다.
점검을 먼저 실행해야 탭이 열린다.

| 항목 | 설명 |
|------|------|
| 기관·시스템명 | 보고서 제목에 쓰인다 |
| 생성 방식 | `사내 추론 서버 (OpenAI 호환)` / `Claude` / `Gemini` |
| 엔드포인트 | 사내 추론 서버 주소. 기본값 `http://127.0.0.1:11434/v1` |
| 모델 | 모델 이름. [연결 확인] 후에는 서버가 알려준 목록에서 고른다 |
| 폐쇄망 모드 | 기본 켜짐. 보고서 생성 중 모델 서버 연결을 루프백으로 제한한다 |

**[서버에 연결해 보기]** 를 먼저 누르면 주소와 모델 이름을 미리 검증한다.
폐쇄망 설치에서 가장 흔한 실패가 주소·모델명 오타인데, 2분을 기다린 끝에
실패를 보는 대신 바로 확인할 수 있다. 모델 목록을 받아오면 입력란이 선택
목록으로 바뀌고, 지금 적힌 모델이 목록에 없으면 알려준다.

생성 중에는 단계별 진행 상황이 표시되고 취소할 수 있다. 저장 위치는 생성을
시작하기 전에 묻는다.

외부 API(`Claude`, `Gemini`)를 고르면 폐쇄망 모드는 자동으로 꺼진다. 둘은
함께 쓸 수 없다.

GUI의 폐쇄망 모드는 **보고서 생성 경로**에 적용된다. 프로세스 전체의 모든
연결을 막는 것은 CLI의 `--offline` 쪽이다. 검증 절차는 CLI 기준으로
[폐쇄망 동작 검증](AIRGAP_VERIFICATION.ko.md)에 정리되어 있다.

> **DOCX 보고서는 별개다.** 같은 화면 아래쪽의 [DOCX 보고서]는 별도 생성기를
> 통해 Word 문서를 만들며, AI 본문은 Anthropic API만 지원한다. 폐쇄망에서는
> [기본 텍스트]만 쓸 수 있다.

---

## 11. 문제 해결

### 데스크톱 앱이 안 뜰 때

| 증상 | 원인과 해결 |
|------|------------|
| macOS — "손상되었기 때문에 열 수 없습니다" | 코드 서명·공증이 아직 없어 Gatekeeper가 격리한 것입니다. 압축 푼 폴더에서 `xattr -dr com.apple.quarantine .` 실행 후 다시 여세요 |
| Windows — 창이 안 뜨거나 흰 화면 | Microsoft WebView2 런타임이 없습니다. [WebView2 Runtime](https://developer.microsoft.com/microsoft-edge/webview2/) 설치 |
| Linux — `rulix-gui` 파일이 없음 | Linux용 데스크톱 앱은 빌드하지 않습니다. CLI(`rulix`)를 사용하세요 |
| 규칙 탭이 비어 있음 | 실행 파일과 같은 위치에 `configs/` 폴더가 있어야 합니다. 번들을 통째로 풀었는지 확인하세요 |
| macOS 전체화면에서 폴더 선택창이 바로 닫힘 | 원인 확인 중입니다. 창 모드에서 폴더를 고른 뒤 전체화면으로 바꾸거나, 경로를 입력란에 직접 붙여넣으세요 |
| AI 감리 탭에서 생성이 실패함 | [연결 확인]을 먼저 눌러 엔드포인트와 모델 이름을 검증하세요. 폐쇄망 모드에서 외부 주소를 지정하면 차단됩니다(10장) |

### 프로파일 로드 실패

```
프로파일 로드 실패: open configs/profiles.yaml: no such file or directory
```

`--profiles-file` 옵션으로 정확한 경로를 지정하세요:

```cmd
rulix.exe C:\src --profiles-file=C:\tools\rulix\configs\profiles.yaml --profile=sql
```

### 한글 깨짐 (콘솔 출력)

```cmd
chcp 65001
rulix.exe C:\src --profile=sql
```

또는 HTML/Excel 출력을 사용하면 한글이 정상 표시됩니다.

### 검사 대상 파일이 없음

RULIX는 확장자 기반으로 파일을 수집합니다:

| 프로파일 | 검사 확장자 |
|----------|------------|
| quality | `.java`, `.js`, `.html`, `.css` |
| secure | `.java` |
| sql | `.sql`, `.xml` |
| modernize | `.java`, `.xml` |

---

## 부록: 전체 규칙 수 현황

| 프로파일 | 전체 | 활성 | 주요 검사 항목 |
|----------|------|------|---------------|
| quality | 125 | 107 | 명명규칙, 복잡도, 코드스멜, 레이어 구조, 주석 |
| secure | 131 | 123 | SQL Injection, XSS, 암호화, 인증/인가, 에러처리 |
| sql | 120 | 102 | ANSI SQL 공통: SELECT, JOIN, WHERE, DML, 냄새 |
| sql-oracle | 20 | 18 | Oracle 전용: NVL, TO_CHAR, hints, ROWNUM, DECODE |
| sql-format | 7 | 7 | SQL 포맷: 키워드 대문자, 라인 길이, 한줄한컬럼 |
| modernize | 50 | 39 | eGov 마이그레이션, javax→jakarta, deprecated API |
| spring | 32 | 28 | DI 패턴, @Transactional, REST API, 보안 |
| egov | 33 | 12 | 레이어 명명, Service/Mapper, 공통컴포넌트 |
| ddl | 17 | 17 | 테이블·컬럼 존재성, 인덱스 활용, DDL 개선 |
| **합계** | **535** | **453** | |

> **활성 규칙**: `enabled: true`인 규칙만 검사에 적용됩니다. 비활성 규칙은 다른 프로파일과 중복되어 `enabled: false`로 설정된 것입니다.

---

[English](USER_MANUAL.md)
