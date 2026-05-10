# productivity plugin

파일 정리 · 무드보드 수집 자동화 스킬 묶음.

## 포함된 스킬

| 스킬 | 설명 |
|------|------|
| `/moodboard` | 키워드로 이미지 수집 → ~/Downloads/{키워드}/ 저장 |
| `/sort-files` | 폴더 파일 자동 분류 (기본: ~/Downloads) |

## MCP 의존성

| 서버 | 용도 |
|------|------|
| `playwright` | /moodboard 이미지 URL 수집 (브라우저 자동화) |

> Playwright MCP는 `~/.claude.json` 에 전역 등록되어 있음.  
> 이 폴더의 `.mcp.json` 은 프로젝트 레벨 사용 시 참조.

## 파일 구조

```
~/.claude/plugins/productivity/
├── README.md
├── .mcp.json          ← Playwright MCP 설정
└── skills/
    ├── moodboard.md   ← /moodboard 스킬 원본
    └── sort-files.md  ← /sort-files 스킬 원본
```

## 활성화 (심링크)

```bash
ln -sf ~/.claude/plugins/productivity/skills/moodboard.md ~/.claude/skills/moodboard.md
ln -sf ~/.claude/plugins/productivity/skills/sort-files.md ~/.claude/skills/sort-files.md
```

## 사용법

```
/moodboard 웨딩스냅
/moodboard 북유럽 인테리어 --count 50 --site unsplash
/moodboard undo

/sort-files
/sort-files ~/Desktop
```
