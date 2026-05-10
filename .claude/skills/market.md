# /market

team-market 플러그인 마켓플레이스 관리 스킬.

## 사용법
```
/market list                     ← 마켓 플러그인 목록 보기
/market install <플러그인명>      ← 플러그인 설치
/market remove <플러그인명>       ← 플러그인 제거
/market publish <폴더경로>        ← 새 플러그인 마켓에 등록
/market info <플러그인명>         ← 플러그인 상세 정보
```

## 마켓 경로
```
MARKET_ROOT=~/team-market
REGISTRY=$MARKET_ROOT/registry.json
SKILLS_DIR=~/.claude/skills
CLAUDE_JSON=~/.claude.json
```

---

## 실행 흐름

### /market list
1. `$REGISTRY` 를 읽어 plugins 배열 파싱
2. 각 플러그인에 대해 설치 여부 확인:
   - 스킬이 `~/.claude/skills/` 에 있으면 ✅ 설치됨
   - 없으면 ⬜ 미설치
3. 아래 형식으로 출력:

```
📦 team-market 플러그인 목록

✅ productivity  v1.0.0  —  파일 정리 · 무드보드 수집 자동화 스킬 묶음
   스킬: /moodboard, /sort-files  |  MCP: playwright

총 1개 플러그인 (설치됨 1 / 미설치 0)
```

---

### /market install <플러그인명>

1. `$REGISTRY` 에서 플러그인 항목 찾기. 없으면 "플러그인을 찾을 수 없습니다" 출력 후 중단.
2. 이미 설치된 경우 "이미 설치되어 있습니다. 재설치하려면 /market remove 후 다시 시도하세요." 안내.
3. 플러그인 `path` 로 스킬 파일 목록 확인: `$MARKET_ROOT/{path}/skills/*.md`
4. 스킬 설치: 각 `.md` 파일을 `~/.claude/skills/` 에 **심링크** 생성
   ```bash
   ln -sf $MARKET_ROOT/{path}/skills/{스킬}.md ~/.claude/skills/{스킬}.md
   ```
5. MCP 설치: 플러그인의 `.mcp.json` 이 있으면:
   - `~/.claude.json` 을 읽어 `mcpServers` 에 병합
   - Python으로 JSON 파싱 후 업데이트 후 다시 저장
6. 완료 메시지 출력:

```
✅ productivity 설치 완료!

🔧 등록된 스킬: /moodboard, /sort-files
🌐 등록된 MCP:  playwright

→ Claude Code 재시작 후 스킬을 사용할 수 있습니다.
```

---

### /market remove <플러그인명>

1. `$REGISTRY` 에서 플러그인 항목 찾기.
2. 설치된 스킬 심링크 삭제: `rm ~/.claude/skills/{스킬}.md`
3. MCP 제거: `~/.claude.json` 에서 해당 플러그인의 mcpServers 키 삭제 (Python JSON 파싱)
4. 완료 메시지 출력:

```
🗑 productivity 제거 완료

삭제된 스킬: /moodboard, /sort-files
제거된 MCP:  playwright
```

---

### /market publish <폴더경로>

새 플러그인을 마켓에 등록한다.

1. 인자로 받은 폴더 경로 확인. 없으면 중단.
2. 폴더 구조 검증:
   - `skills/*.md` 파일 최소 1개 이상 있어야 함
   - 없으면 "스킬 파일이 없습니다. skills/ 폴더 안에 .md 파일을 추가하세요." 안내 후 중단
3. 플러그인 메타 정보 수집:
   - `id`: 폴더명
   - `name`: 사용자에게 질문하거나 폴더명으로 기본값
   - `version`: "1.0.0" 기본값
   - `description`: 사용자에게 질문
   - `author`: `git config user.name` 또는 시스템 유저명
   - `skills`: skills/*.md 파일명 목록 (확장자 제외)
   - `mcp`: .mcp.json 에서 mcpServers 키 목록 (없으면 빈 배열)
4. 폴더를 `$MARKET_ROOT/plugins/{id}/` 로 복사 (이미 존재하면 버전 넘버링)
5. `registry.json` 에 새 항목 추가 (Python JSON 파싱)
6. 완료 메시지:

```
📦 my-plugin 등록 완료!

스킬: /my-skill
등록 위치: ~/team-market/plugins/my-plugin/

팀원에게 git push 후 /market install my-plugin 로 설치 가능합니다.
```

---

### /market info <플러그인명>

1. `$REGISTRY` 에서 항목 찾아 상세 출력:

```
📦 productivity  v1.0.0
작성자: taebeomlee
설명: 파일 정리 · 무드보드 수집 자동화 스킬 묶음

스킬:
  • /moodboard  — 키워드로 이미지 수집
  • /sort-files — 폴더 파일 자동 분류

MCP 의존성:
  • playwright (브라우저 자동화)

설치 상태: ✅ 설치됨
```

---

## 안전장치
- 심링크 중복 생성 방지 (이미 있으면 경고)
- MCP 키 충돌 시 기존 값 유지 + 사용자에게 알림
- registry.json 파싱 실패 시 원본 보존 후 중단
- publish 시 id 중복이면 `{id}_2`, `{id}_3` 자동 넘버링
