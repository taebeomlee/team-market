---
name: "moodboard"
description: >
  키워드로 이미지를 수집해 ~/Downloads/{키워드}/ 폴더에 저장하는 무드보드 스킬.
  사용자가 "무드보드 만들어줘", "이미지 수집해줘", "핀터레스트에서 사진 모아줘",
  "웨딩스냅 이미지 30장 다운로드해줘", "인테리어 레퍼런스 모아줘" 라고 하면 실행한다.
  "/moodboard 키워드", "/moodboard 키워드 --count N", "/moodboard --site unsplash" 형태로 호출.
  "/moodboard undo" 로 마지막 수집 결과를 롤백한다.
  지원 사이트: pinterest (기본), unsplash, google.
metadata:
  version: 1.0.0
  visibility: public
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
- `키워드` (필수): 검색할 키워드. 없으면 사용자에게 묻는다.
- `--count N` (선택, 기본 30): 수집할 이미지 수.
- `--site SITE` (선택, 기본 pinterest): 수집할 사이트.
  - `pinterest` → `https://kr.pinterest.com/search/pins/?q={키워드}&rs=typed`
  - `unsplash` → `https://unsplash.com/s/photos/{키워드}`
  - `google` → `https://www.google.com/search?q={키워드}&tbm=isch`

## 실행 흐름

### Phase 0 — 인자 파싱
1. 인자에서 키워드, `--count`, `--site` 를 파싱한다.
2. `--count` 가 없으면 30, `--site` 가 없으면 pinterest 로 기본값 설정.
3. 서브에이전트 수를 count 기준으로 결정한다.
   - count ≤ 30 → 서브에이전트 3개 (count/3 씩)
   - count ≤ 60 → 서브에이전트 5개 (count/5 씩)
   - count > 60 → 서브에이전트 6개 (count/6 씩)

### Phase 1 — URL 수집 (Playwright MCP, 순차)
4. 저장 폴더명을 결정한다.
   - 기본: `~/Downloads/{키워드}/`
   - 폴더가 이미 존재하면 `{키워드}_2`, `{키워드}_3` 순으로 자동 넘버링.
5. **롤백용 매니페스트** 파일을 생성한다.
   - 경로: `{저장폴더}/.moodboard_manifest.json`
   - 내용: `{ "keyword": "...", "site": "...", "count": N, "folder": "...", "files": [], "timestamp": "..." }`
6. `mcp__playwright__browser_navigate` 로 해당 사이트의 검색 페이지를 연다.
7. `mcp__playwright__browser_evaluate` 로 스크롤하며 이미지 URL을 수집한다.
   - 사이트별 이미지 셀렉터:
     - pinterest: `img[src*="pinimg.com"]` 중 `736x` 포함된 것
     - unsplash: `img[src*="images.unsplash.com"]`
     - google: `img[src*="http"]` (data-src 포함)
   - `Set()`으로 중복 제거
   - 목표 count + 예비 5개 이상 확보될 때까지 스크롤 (최대 30회)
   - 검색 결과 0개면 사용자에게 알리고 중단

### Phase 2 — 병렬 다운로드 (서브에이전트 N개 동시)
8. URL을 서브에이전트 수만큼 균등 분할한다.
9. Agent 도구로 서브에이전트를 동시에 실행한다.
   - Python `requests` 로 다운로드
   - 파일명: `{키워드}_{번호:02d}.jpg`
   - 실패 시 최대 3회 재시도, 5KB 미만이면 손상 이미지로 판단 재시도
10. 완료 후 결과 취합, 부족하면 예비 URL로 보충.

### Phase 3 — 결과 보고
```
✅ 무드보드 수집 완료!
📁 저장 위치: ~/Downloads/웨딩스냅/
🖼️  저장 장수: 30장
🌐 수집 사이트: Pinterest
🔍 키워드: 웨딩스냅
↩️  되돌리기: /moodboard undo
```

## 되돌리기 (undo)
`/moodboard undo` 를 입력하면:
1. `~/Downloads/` 에서 가장 최근 `.moodboard_manifest.json` 을 찾는다.
2. manifest의 `files` 를 읽어 해당 파일들을 삭제한다.
3. 폴더가 비면 폴더도 삭제한다.
