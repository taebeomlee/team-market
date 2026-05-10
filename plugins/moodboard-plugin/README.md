# moodboard-plugin

키워드로 이미지를 수집해 ~/Downloads/ 에 저장하는 스킬.

## 스킬

| 스킬 | 설명 |
|------|------|
| `/moodboard` | 키워드로 이미지 수집 (Pinterest / Unsplash / Google) |

## MCP 의존성

| 서버 | 용도 |
|------|------|
| `playwright` | 브라우저 자동화로 이미지 URL 수집 |

## 사용법

```
/moodboard 웨딩스냅
/moodboard 북유럽 인테리어 --count 50
/moodboard 미니멀 --site unsplash --count 20
/moodboard undo
```
