# team-market

팀 공유 Claude 플러그인 마켓플레이스.

## 설치 방법

### 1. 레포 클론 (팀원 공유 시)
```bash
git clone <repo-url> ~/team-market
```

### 2. `/market` 스킬 등록
```bash
cp ~/team-market/.claude/skills/market.md ~/.claude/skills/market.md
```

### 3. 플러그인 설치
```
/market list
/market install productivity
```

---

## 플러그인 등록 방법 (신규 추가)

1. `plugins/{이름}/` 폴더 생성
2. `skills/*.md` 스킬 파일 추가
3. `.mcp.json` MCP 설정 추가 (필요 시)
4. `registry.json` 에 항목 추가

---

## 현재 플러그인 목록

| 플러그인 | 스킬 | MCP |
|---------|------|-----|
| productivity | `/moodboard`, `/sort-files` | playwright |

## 폴더 구조

```
team-market/
├── README.md
├── registry.json
├── .claude/
│   └── skills/
│       └── market.md       ← /market 스킬
└── plugins/
    └── productivity/
        ├── README.md
        ├── .mcp.json
        └── skills/
            ├── moodboard.md
            └── sort-files.md
```
