# Claude Web Test Skills

> Claude Code 스킬 기반 웹 애플리케이션 자동 테스트 도구 모음

웹 애플리케이션의 네비게이션 구조를 자동으로 분석하고, 테스트 케이스를 생성하며, Chrome 브라우저로 자동 테스트를 수행하는 Claude Code 스킬 세트입니다.

---

## 스킬 목록

| 명령어 | 스킬 | 설명 |
|--------|------|------|
| `/nav-analyze` | web-nav-analyzer | 웹앱의 GNB/LNB 메뉴 구조를 Chrome으로 분석하여 MD 파일 생성 |
| `/web-testcase` | web-testcase-creator | MD 구조 파일 기반으로 테스트 케이스를 Notion DB에 생성 |
| `/web-test` | web-test-runner | Notion DB의 테스트 케이스를 Chrome으로 자동 실행하고 결과 기록 |

---

## 워크플로우

```
+-------------------+       +----------------------+       +------------------+
|  1. nav-analyze   | ----> |  2. web-testcase     | ----> |  3. web-test     |
|  (구조 분석)       |  MD   |  (테스트 케이스 생성)   | Notion|  (테스트 실행)     |
+-------------------+       +----------------------+       +------------------+
  Chrome으로 탐색          사용자 입력 + MD 기반         Notion에서 읽고 실행
  GNB/LNB 파악             Notion DB에 TC 생성          결과를 Notion에 기록
```

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

---

## 설치 방법

각 스킬 폴더를 `~/.claude/skills/`에 복사하면 바로 사용할 수 있습니다.

```bash
cp -r skills/web-nav-analyzer ~/.claude/skills/
cp -r skills/web-testcase-creator ~/.claude/skills/
cp -r skills/web-test-runner ~/.claude/skills/
```

---

## 사전 요구사항

- **Claude Code**: Claude Code CLI 또는 IDE 확장이 설치되어 있어야 합니다
- **Chrome 브라우저**: [Claude in Chrome](https://chromewebstore.google.com/) 확장 프로그램이 설치된 Chrome 브라우저
- **Notion 계정**: 테스트 케이스 관리를 위한 Notion 계정 및 MCP 연결 설정

---

## 사용 예시

### 1단계: 네비게이션 구조 분석

```
/nav-analyze https://your-app.com ./nav-structure.md
```

Chrome 브라우저로 대상 웹앱에 접속하여 GNB(상단 메뉴)와 LNB(좌측 사이드바) 구조를 자동 분석합니다. 로그인이 필요하면 계정 정보를 질문합니다. 결과는 구조화된 MD 파일로 저장됩니다.

### 2단계: 테스트 케이스 생성

```
/web-testcase ./nav-structure.md https://notion.so/your-db-url
```

분석된 MD 파일을 읽어 테스트 대상 화면과 범위를 선택하면, 화면 유형에 맞는 테스트 케이스를 자동 설계하여 Notion DB에 생성합니다.

### 3단계: 자동 테스트 실행

```
/web-test https://notion.so/your-db-url https://your-app.com admin P@ssw0rd
```

Notion DB에서 테스트 케이스를 읽어 사전조건 의존성을 분석한 후, Chrome 브라우저로 순차 실행합니다. 각 테스트 완료 즉시 결과(통과(AI)/실패(AI)/보류(AI))를 Notion에 기록합니다.

---

## 상세 가이드

각 스킬의 상세 동작 방식과 실전 사용 예시는 [docs/guide.md](docs/guide.md)를 참고하세요.

---

## 라이선스

MIT License
