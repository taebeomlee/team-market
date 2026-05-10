---
name: "sort-files"
description: >
  폴더 파일을 종류별로 자동 분류하는 스킬. 기본 대상: ~/Downloads.
  사용자가 "파일 정리해줘", "다운로드 정리해줘", "폴더 분류해줘", "sort my downloads",
  "clean up my files", "organize my folder", "정리해줘", "파일 분류해줘" 라고 하면 실행한다.
  "/sort-files" 또는 "/sort-files ~/Desktop" 처럼 경로 지정도 가능.
  실행 전 dry-run 미리보기를 보여주고 확인 후 이동한다.
  모호한 파일은 임의 배치하지 않고 완료 후 사용자에게 목록을 제시한다.
metadata:
  version: 1.0.0
  visibility: public
---

# sort-files

폴더 안의 파일을 카테고리별 서브폴더로 자동 정리한다.

## 대상 폴더

기본: `~/Downloads`
오버라이드: 인자로 경로 지정 (예: `/sort-files ~/Desktop`)

## Step 1 — 탐색

Explore 서브에이전트로 대상 폴더의 파일 목록 수집 (maxdepth 2).
수집 항목: 파일명, 확장자, 크기, 수정일.

## Step 2 — 분류

### 카테고리 규칙

| 카테고리 | 확장자 |
|---------|--------|
| 문서 | pdf, docx, doc, txt, md |
| 프레젠테이션 | pptx, ppt, key |
| 스프레드시트 | xlsx, xls, csv |
| 이미지 | jpg, jpeg, png, gif, webp, heic, svg |
| 오디오 | wav, mp3, flac, aac, m4a |
| 비디오 | mp4, mov, avi, mkv |
| 설치파일 | zip, dmg, pkg, exe, app |
| 폰트 | ttf, otf, woff, woff2 |

### 모호한 파일 조건
- 확장자가 없는 파일
- 확장자와 내용이 불일치하는 파일
- 설치파일과 이름이 같은 폴더 (압축 해제본으로 추정)
- 폰트 이름의 폴더

→ 모호한 파일은 임의 배치 금지. 완료 후 사용자에게 목록 제시.

## Step 3 — Dry-run 미리보기

이동 전 아래 형식으로 출력 후 확인 요청:
```
=== 생성할 폴더 ===
mkdir 문서 이미지 ...

=== 이동할 파일 ===
[문서/] report.pdf, notes.txt
[이미지/] photo.jpg ...

=== 삭제 대상 ===
🗑 .DS_Store, .localized

=== 모호한 파일 ===
⚠ index.html — HTML이지만 문서 성격
```

## Step 4 — 중복 확인

이름과 크기가 동일한 파일이 있으면 삭제 또는 유지 여부를 사용자에게 질문한다.

## Step 5 — 병렬 실행

카테고리별로 서브에이전트를 동시에 실행한다. 각 에이전트:
1. `mkdir -p` 로 폴더 생성
2. `mv` 로 파일 이동 (특수문자 파일명은 Python shutil.move 사용)
3. 결과 보고

## Step 6 — 검증

`ls -la <대상>` 으로 루트에 카테고리 폴더만 남았는지 확인.

## Step 7 — 모호한 파일 처리

```
아래 파일은 분류가 애매해서 그대로 뒀습니다. 어디로 옮길까요?

⚠ index.html → 문서/ / 웹/ 새로 만들기 / 그냥 두기
⚠ MyApp/     → 설치파일/ / 그냥 두기
```
