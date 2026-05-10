---
name: moodboard
description: Use this skill when the user asks to collect images, create a moodboard, download Pinterest images, or says "무드보드 만들어줘", "이미지 수집해줘", "핀터레스트에서 사진 모아줘", "웨딩스냅 이미지 30장 다운로드해줘". Invoke as /moodboard <keyword> [--count N] [--site pinterest|unsplash|google]. Use /moodboard undo to rollback.
version: 1.0.0
---

# /moodboard

키워드로 이미지를 수집해 ~/Downloads/ 에 저장하는 스킬.

## 사용법
```
/moodboard 웨딩스냅
/moodboard 웨딩스냅 --count 50 --site unsplash
/moodboard undo
```

## 실행 흐름

### Phase 0 — 인자 파싱
- count ≤ 30 → 서브에이전트 3개 / ≤ 60 → 5개 / > 60 → 6개

### Phase 1 — URL 수집 (Playwright MCP)
- `~/Downloads/{키워드}/` 폴더 생성, `.moodboard_manifest.json` 생성
- Playwright로 스크롤하며 URL 수집 (최대 30회, Set으로 중복 제거)

### Phase 2 — 병렬 다운로드
- 파일명: `{키워드}_{번호:02d}.jpg`, 3회 재시도, 5KB 미만 재시도

### Phase 3 — 결과 보고 및 undo 지원
