# 웹 테스트 자동화 스킬 가이드

## 스킬 전체 구성

```
+-------------------+       +----------------------+       +------------------+
|  1. nav-analyze   | ----> |  2. web-testcase     | ----> |  3. web-test     |
|  (구조 분석)       |  MD   |  (테스트 케이스 생성)   | Notion|  (테스트 실행)     |
+-------------------+       +----------------------+       +------------------+
  Chrome으로 탐색          사용자 입력 + MD 기반         Notion에서 읽고 실행
  GNB/LNB 파악             Notion DB에 TC 생성          결과를 Notion에 기록
```

| 스킬 | 명령어 | 역할 |
|------|--------|------|
| **web-nav-analyzer** | `/nav-analyze` | 웹앱의 GNB/LNB 메뉴 구조를 분석하여 MD 파일 생성 |
| **web-testcase-creator** | `/web-testcase` | MD 구조 파일 기반으로 테스트 케이스를 Notion DB에 생성 |
| **web-test-runner** | `/web-test` | Notion DB의 테스트 케이스를 Chrome으로 자동 실행 |

---

## 전체 프로세스

### Phase 1: 네비게이션 구조 분석

**목적**: 테스트 대상 웹 애플리케이션의 메뉴 구조를 파악합니다.

**실행:**
```
/nav-analyze https://your-app.com ./nav-structure.md
```

**인자:**
- `url` (필수): 대상 웹 애플리케이션 URL
- `output_path` (선택): 결과 MD 파일 경로 (기본값: `nav-structure.md`)

**동작:**
1. Chrome 브라우저로 URL 접속
2. 로그인이 필요하면 계정 정보를 질문
3. GNB(상단 메뉴)와 LNB(좌측 사이드바) 구조를 자동 탐색
4. 각 메뉴의 하위 항목까지 재귀적으로 분석
5. 각 화면의 버튼, 입력 필드, 테이블 등 UI 요소 분석
6. 결과를 구조화된 MD 파일로 저장

**출력 예시 (nav-structure.md):**
```markdown
# MyApp 네비게이션 구조
> 분석 일시: 2026-04-03
> 대상 URL: https://your-app.com

## GNB (상단 네비게이션)
| 메뉴 | 하위 메뉴 | URL 패턴 |
|------|-----------|----------|
| 대시보드 | - | /dashboard |

## LNB (좌측 사이드바)
설정
  ├── 사용자 관리
  └── 권한 관리

## 화면별 상세 분석
### 설정 > 사용자 관리
- URL: /settings/users
- 화면 유형: 목록(List)
- 주요 기능: 사용자 CRUD, 검색, 필터
- UI 요소 힌트: 추가 버튼 "[+]", 검색 placeholder "검색..."
```

---

### Phase 2: 테스트 케이스 생성

**목적**: 분석된 구조를 기반으로 특정 피쳐의 테스트 케이스를 Notion에 생성합니다.

**실행:**
```
/web-testcase ./nav-structure.md https://notion.so/your-db-url
```

**인자:**
- `nav_md_path` (필수): Phase 1에서 생성한 MD 파일 경로
- `notion_db_url` (필수): Notion 테스트 케이스 DB URL

**동작:**
1. MD 파일을 읽어 메뉴/화면 목록을 파악
2. 사용자에게 테스트 대상 화면과 범위를 질문
3. Notion DB 스키마를 확인하고 누락 속성 자동 추가
4. 화면 유형에 따라 적절한 테스트 케이스 자동 설계
5. TC_ID를 자동 채번 (중복 방지)
6. 사용자에게 미리보기를 보여주고 확인
7. 확인 후 Notion에 생성

**테스트 범위 옵션:**
- **전체**: 해당 화면의 모든 기능 (CRUD, UI, 검증) 자동 생성
- **특정 기능**: 사용자가 선택한 기능만 생성
- **사용자 정의**: 사용자가 직접 절차/기대결과를 제공

**Notion DB에 생성되는 속성:**

| 속성 | 설명 | 예시 |
|------|------|------|
| TC_ID | 고유 ID | ADM-SET-001 |
| 모듈 | 1차 메뉴명 | 설정 |
| 화면/메뉴 | 메뉴 경로 | 설정 > 사용자 관리 |
| 테스트 시나리오 | 한 줄 요약 | 사용자 신규 생성 |
| 테스트 유형 | 기능/UI/보안/성능 | 기능 |
| 사전조건 | 선행 조건 | 로그인 상태 |
| 자동화 | Yes/Partial/No | Yes |
| 테스트 결과 | 실행 상태 | 미실행 |
| 대상 URL | 웹앱 URL | https://your-app.com |

---

### Phase 3: 테스트 실행

**목적**: Notion에 등록된 테스트 케이스를 Chrome 브라우저로 자동 실행합니다.

**실행:**
```
/web-test https://notion.so/your-db-url https://your-app.com user password
```

**인자:**
- `notion_db_url` (필수): Notion 테스트 케이스 DB URL
- `server_url` (선택): 테스트 대상 URL (미지정 시 TC의 "대상 URL" 사용)
- `username` (선택): 로그인 계정
- `password` (선택): 로그인 비밀번호

**동작:**
1. Notion DB에서 모든 테스트 케이스 수집
2. 사전조건 의존성 분석 후 실행 순서 결정
3. 실행 계획을 사용자에게 보여주고 확인
4. Chrome 브라우저로 순차 실행
5. 각 TC 완료 즉시 Notion에 결과 기록

**결과 태그:**
- **통과(AI)**: 모든 기대결과 충족
- **실패(AI)**: 기대결과 미충족 또는 에러 발생
- **보류(AI)**: 사전조건 미충족, 환경 제약, 수동 확인 필요

---

## 실전 사용 예시

### 예시 1: 새 웹앱 전체 테스트 구축

```bash
# 1단계: 구조 분석
/nav-analyze https://cms.company.com ./cms-nav.md

# 2단계: 설정 > 사용자 관리 화면의 전체 테스트 케이스 생성
/web-testcase ./cms-nav.md https://notion.so/your-test-db

# 3단계: 테스트 실행
/web-test https://notion.so/your-test-db https://cms.company.com admin P@ssw0rd
```

### 예시 2: 특정 피쳐만 테스트 케이스 추가

```bash
# MD 파일이 이미 있는 경우, 바로 특정 피쳐의 TC 추가
/web-testcase ./cms-nav.md https://notion.so/your-test-db
# → "어떤 화면?" → "사용자 관리"
# → "어떤 범위?" → "특정 기능: 사용자 삭제"
```

### 예시 3: 사용자 정의 테스트 케이스 추가

```bash
/web-testcase ./cms-nav.md https://notion.so/your-test-db
# → "어떤 화면?" → "설정 > 알림 설정"
# → "어떤 범위?" → "사용자 정의"
# → 절차: "1. 알림 설정 메뉴 진입 2. 이메일 알림 토글 ON 3. 저장"
# → 사전조건: "관리자 로그인"
# → UI 힌트: "이메일 알림 토글은 '이메일 알림' 라벨 옆"
```

---

## ADMIN 포털 전용 스킬 (기존)

범용 스킬과 별도로, ADMIN 포털에 특화된 스킬도 사용 가능합니다:

| 스킬 | 명령어 | 설명 |
|------|--------|------|
| **admin-portal-testcase-creator** | `/admin-testcase` | 사용자 입력 기반 ADMIN 포털 TC 생성 |
| **admin-portal-test** | `/admin-test` | ADMIN 포털 전용 자동 테스트 실행 |

ADMIN 포털 전용 스킬은 사이드바 메뉴 구조, 기본 로그인 정보, TC_ID 채번 규칙이 미리 설정되어 있어 별도 구조 분석 없이 바로 사용 가능합니다.

---

## 스킬 간 데이터 흐름

```
[웹 애플리케이션]
       |
       | Chrome 브라우저 탐색
       v
[nav-analyze] ──> nav-structure.md (GNB/LNB 구조 + 화면별 UI 분석)
                         |
                         | 구조 파일 읽기 + 사용자 입력
                         v
               [web-testcase] ──> Notion DB (TC_ID, 절차, 기대결과, UI 힌트)
                                        |
                                        | 테스트 케이스 읽기
                                        v
                              [web-test] ──> Chrome 자동 실행 ──> Notion 결과 기록
```

## 참고사항

- **Notion DB는 공유 가능**: 한 번 생성한 테스트 DB를 여러 사람이 함께 사용 가능
- **증분 추가 지원**: 이미 TC가 있는 DB에 새로운 피쳐의 TC만 추가 가능 (중복 방지)
- **재실행 가능**: 테스트 결과가 기록된 후에도 재실행하여 결과 갱신 가능
- **SSL 인증서 자동 우회**: 개발/스테이징 환경의 자체 서명 인증서 지원
- **세션 만료 자동 처리**: 테스트 중 세션 만료 시 자동 재로그인
