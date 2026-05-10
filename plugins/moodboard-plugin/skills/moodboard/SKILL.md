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
/moodboard 웨딩스냅 --count 50
/moodboard 웨딩스냅 --site unsplash
/moodboard 북유럽 인테리어 --count 20 --site pinterest
/moodboard undo
```

### 파라미터
- `키워드` (필수): 검색할 키워드.
- `--count N` (선택, 기본 30): 수집할 이미지 수.
- `--site SITE` (선택, 기본 pinterest): pinterest / unsplash / google

## 실행 흐름

### Phase 0 — 인자 파싱
- count ≤ 30 → 서브에이전트 3개 / ≤ 60 → 5개 / > 60 → 6개

### Phase 1 — URL 수집 (Playwright MCP)
- `~/Downloads/{키워드}/` 폴더 생성 (충돌 시 자동 넘버링)
- `.moodboard_manifest.json` 롤백 파일 생성
- Playwright로 스크롤하며 이미지 URL 수집 (Set으로 중복 제거, 최대 30회)

### Phase 2 — 병렬 다운로드
- Python requests로 다운로드, 파일명: `{키워드}_{번호:02d}.jpg`
- 3회 재시도, 5KB 미만 손상 이미지 재시도

### Phase 3 — 결과 보고
```
✅ 무드보드 수집 완료!
📁 저장 위치: ~/Downloads/웨딩스냅/
🖼️  저장 장수: 30장
↩️  되돌리기: /moodboard undo
```

## undo
`/moodboard undo` → manifest 기반으로 파일 및 폴더 삭제
