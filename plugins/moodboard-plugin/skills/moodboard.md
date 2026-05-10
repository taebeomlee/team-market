# /moodboard

키워드로 이미지를 수집해 ~/Downloads/ 에 저장하는 스킬.

## 사용법
```
/moodboard 웨딩스냅
/moodboard 웨딩스냅 --count 50
/moodboard 웨딩스냅 --site unsplash
/moodboard 북유럽 인테리어 --count 20 --site pinterest
```

### 파라미터
- `키워드` (필수): 검색할 키워드. 없으면 사용자에게 묻는다.
- `--count N` (선택, 기본 30): 수집할 이미지 수. 예) `--count 50`
- `--site SITE` (선택, 기본 pinterest): 수집할 사이트. 지원 사이트:
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
   - 작업 중 저장된 파일명을 files 배열에 실시간 추가한다.
6. `mcp__playwright__browser_navigate` 로 해당 사이트의 검색 페이지를 연다.
7. `mcp__playwright__browser_evaluate` 로 스크롤하며 이미지 URL을 수집한다.
   - 사이트별 이미지 셀렉터:
     - pinterest: `img[src*="pinimg.com"]` 중 `236x`/`474x`/`736x` 포함된 것, `736x` 로 치환
     - unsplash: `img[src*="images.unsplash.com"]`
     - google: `img[src*="http"]` (data-src 포함)
   - `Set()`으로 중복 제거
   - 목표 count + 예비 5개 이상 확보될 때까지 스크롤 (최대 30회)
   - 검색 결과 0개면 사용자에게 알리고 중단
   - count 미달 시 "XX개만 수집됨" 안내 후 수집된 수로 진행
8. URL 목록을 count+5 개로 확정한다 (앞 count개 = 본 수집, 나머지 = 예비).

### Phase 2 — 병렬 다운로드 (서브에이전트 N개 동시)
9. URL을 Phase 0에서 결정한 서브에이전트 수만큼 균등 분할한다.
10. **Agent 도구**로 서브에이전트를 동시에 실행한다. 각 에이전트 지시:
    - 저장 폴더 경로
    - 담당 URL 목록
    - 파일명 시작 번호
    - 키워드 (파일명에 사용)
    - 아래 다운로드 규칙 준수

    **다운로드 규칙 (각 서브에이전트 공통)**
    - Python `requests` 로 다운로드
    - 파일명: `{키워드}_{번호:02d}.jpg`
    - 실패 시 최대 3회 재시도 (1초 간격)
    - 저장 후 파일 크기 5KB 미만이면 손상 이미지로 판단, 재시도
    - 재시도 후에도 실패하면 해당 번호를 건너뛰고 실패 목록에 기록
    - 성공/실패 목록을 반환

11. 모든 에이전트 완료 후 결과를 취합한다.
    - 성공 개수 확인
    - 부족하면 예비 URL로 보충 다운로드
    - 매니페스트 files 배열 업데이트

### Phase 3 — 결과 보고
12. 아래 형식으로 결과 요약을 출력한다.

```
✅ 무드보드 수집 완료!

📁 저장 위치: ~/Downloads/웨딩스냅/
🖼️  저장 장수: 30장
🌐 수집 사이트: Pinterest
🔍 키워드: 웨딩스냅

📋 파일 목록: 웨딩스냅_01.jpg ~ 웨딩스냅_30.jpg
⚠️  실패: 없음 (또는 실패한 파일명 목록)

↩️  되돌리기: /moodboard undo
```

## 되돌리기 (undo)
`/moodboard undo` 를 입력하면:
1. `~/Downloads/` 에서 가장 최근 `.moodboard_manifest.json` 을 찾는다.
2. manifest의 `folder` 와 `files` 를 읽어 해당 파일들을 삭제한다.
3. 폴더가 비면 폴더도 삭제한다.
4. "되돌리기 완료: {폴더명} 삭제됨" 을 출력한다.

## 안전장치 체크리스트
- [x] 어떤 키워드로도 동작
- [x] --site 로 수집 사이트 변경 가능
- [x] --count 로 수집 장수 변경 가능 (서브에이전트 수 자동 조정)
- [x] undo 로 결과 되돌리기 가능 (manifest 기반)
- [x] 결과 요약 출력 (장수, 저장 위치, 실패 목록)
- [x] 중복 URL 제거 (Set 사용)
- [x] 5KB 미만 손상 이미지 재시도
- [x] 폴더명 충돌 방지 (자동 넘버링)
- [x] 다운로드 3회 재시도
- [x] count 미달 시 예비 URL로 보충
- [x] 검색 결과 0개면 사용자 알림 후 중단
- [x] 최종 개수 검증 후 보고
