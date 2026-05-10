---
name: sort-files
description: >
  Automatically organizes files in a folder (default: ~/Downloads) into categorized subfolders by file type.
  Use this skill whenever the user asks to organize, sort, clean up, or tidy a folder — especially Downloads.
  Also triggers on "폴더 정리", "파일 정리", "다운로드 정리" and similar Korean phrasing.
  Handles: dry-run preview before moving, parallel execution via subagents, duplicate detection,
  ambiguous file flagging, and macOS system file cleanup.
  Invoke when the user says things like: "sort my downloads", "clean up my files", "organize my folder",
  "정리해줘", "파일 분류해줘" — even if they don't explicitly say "/sort-files".
---

# sort-files

Organizes files in a target folder into categorized subfolders. The workflow mirrors how a careful person
would do it: look first, flag anything unclear, ask about duplicates, then move everything in parallel.

## Target folder

Default: `~/Downloads`  
Override: the user can pass any path as an argument (e.g., `/sort-files ~/Desktop`).

## Step 1 — Explore

Spawn an **Explore subagent** to list all files and subdirectories in the target folder (maxdepth 2).
Collect: filename, extension, size, date modified.

## Step 2 — Classify

For each item apply the category rules below. Identify **ambiguous files** separately (do not silently assign them).

### Category rules (by extension)

| Category | Extensions |
|---|---|
| 문서 | pdf, docx, doc, txt, md |
| 프레젠테이션 | pptx, ppt, key |
| 스프레드시트 | xlsx, xls, csv |
| 이미지 | jpg, jpeg, png, gif, webp, heic, svg |
| 오디오 | wav, mp3, flac, aac, m4a |
| 비디오 | mp4, mov, avi, mkv |
| 설치파일 | zip, dmg, pkg, exe, app (and installer-related folders) |
| 폰트 | ttf, otf, woff, woff2 (and font-related folders) |

### What counts as ambiguous

Flag a file as ambiguous — don't silently categorize it — when:
- **Extension conflicts with apparent content**: e.g., `.html` file that reads like a saved document rather than a webpage
- **No extension**: the file type is genuinely unclear
- **Folder paired with an installer**: a folder whose name matches a nearby `.zip`/`.dmg` (it's probably the extracted version — belongs in 설치파일, but worth flagging)
- **Font-named folder**: a folder whose name suggests it contains fonts

## Step 3 — Dry-run output

Before moving anything, print a clear summary:

```
=== 생성할 폴더 ===
mkdir 문서 프레젠테이션 ...

=== 이동할 파일 ===
[문서/]
  ✓ report.pdf
  ✓ notes.txt
...

=== 삭제 대상 ===
  🗑 .DS_Store
  🗑 .localized

=== 모호한 파일 (나중에 확인 필요) ===
  ⚠ index.html — HTML이지만 문서 성격
  ⚠ MyApp/ — 설치파일 폴더로 보이나 내용 불명확
```

Ask the user to confirm before proceeding.

## Step 4 — Duplicate check

If two files have identical names (ignoring ` (1)`, ` (2)` suffixes) and the same file size,
ask the user whether to delete the duplicate or keep both.

## Step 5 — Execute with parallel subagents

Spawn one subagent **per category** in the same message (parallel). Each subagent:
1. Creates its target folder (`mkdir -p`)
2. Moves its assigned files (`mv`)
3. Reports what it did

Delete macOS system files (`.DS_Store`, `.localized`) in a separate subagent running in parallel.

Handle special filenames carefully — files with quotes, parentheses, or Unicode characters
in their names may require Python `shutil.move` instead of `mv`.

## Step 6 — Verify

After all subagents complete, run `ls -la <target>` to confirm the root folder contains only
the new category subfolders (plus any ambiguous files not yet moved).

## Step 7 — Report ambiguous files

Show the ambiguous files list and ask the user to decide where each one goes:

```
아래 파일은 분류가 애매해서 그대로 뒀습니다. 어디로 옮길까요?

⚠ index.html
   → 문서/ 로 옮기기 / 웹/ 폴더 새로 만들기 / 그냥 두기

⚠ MyApp/
   → 설치파일/ 로 옮기기 / 그냥 두기
```

Move them according to the user's answer.
