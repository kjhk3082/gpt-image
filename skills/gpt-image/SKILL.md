---
name: gpt-image
description: "Use when the user wants an actual image file generated — '이미지 생성해줘/만들어줘', 'GPT로/지피티로 이미지', 'gpt image', hero image, illustration, 사진풍 그래픽, 앱 아이콘 시안, thumbnail, poster. Claude Code has no native image generation; this skill delegates to the locally installed OpenAI Codex CLI (ChatGPT login) and its image_generation tool. NOT for inline SVG/CSS graphics, charts, or real-brand logos."
user-invocable: true
---

# gpt-image — ChatGPT(Codex CLI)에 이미지 생성 위임

## Overview

Claude Code는 이미지를 직접 생성할 수 없다. 대신 로컬에 설치된 OpenAI `codex` CLI(ChatGPT 계정 로그인)의 `image_generation` 도구에 `codex exec`로 비대화형 위임한다. 핵심 원칙: **생성 경로를 파싱하지 말고, codex에게 "결과물을 이 절대경로로 복사하라"고 시킨다.**

## Preflight (세션에서 처음 쓸 때 1회)

```bash
command -v codex && codex --version           # 없으면 → 사용자에게 설치 안내: npm install -g @openai/codex
codex login status                            # 미로그인 → 사용자에게 `codex login` 안내 (직접 로그인 금지)
codex features list | grep image_generation   # true가 아니면 → codex features enable image_generation
```

하나라도 실패하면 생성을 시도하기 전에 사용자에게 해결 방법부터 알린다.

## When to Use

- 실제 픽셀 이미지 파일(PNG)이 필요할 때: 일러스트, 히어로/배너, 사진풍, 목업용 그래픽, 썸네일
- **NOT**: 아이콘·다이어그램·차트(인라인 SVG가 낫다), 실존 브랜드 로고(IP 리스크 — 다운로드로 해결), 실존 인물 얼굴/실명

## The Command (검증: codex-cli 0.160.0)

```bash
OUT="/절대/경로/이미지이름.png"
codex exec --skip-git-repo-check \
  -s workspace-write -c 'approval_policy="never"' \
  -C "$(dirname "$OUT")" \
  -o /tmp/gpt-image-last.txt \
  "Use your image_generation tool to create one image.
Prompt: <영어로 쓴 구체적 시각 묘사 — 스타일, 구도, 조명, 컬러 hex, 배경>.
Aspect: <landscape 3:2 | portrait 2:3 | square>.
After generating, copy the resulting PNG to exactly this path: $OUT (mkdir -p if needed).
Do nothing else. Your final message must be only that absolute path." </dev/null
```

- `-s workspace-write`: 사용자 config와 무관하게 동작하도록 샌드박스를 명시한다. 쓰기는 `-C` 디렉토리 안으로만 허용되므로 **`$OUT`은 반드시 `-C` 디렉토리 안**에 둔다(위 명령은 `dirname "$OUT"`이라 자동 충족).
- `-c 'approval_policy="never"'`: exec 모드는 승인 프롬프트에 답할 수 없으므로 명시한다.
- Bash `timeout: 600000`, 1장당 실측 1~3분, exec 1회 ≈ 40k codex 토큰. 2장 이상이면 `run_in_background: true`.
- Aspect 매핑 실측: `landscape 3:2` → 1536×1024, `square` → 1024×1024, `portrait 2:3` → 1024×1536.
- 여러 장은 codex 콜드스타트가 비싸므로 **한 번의 exec에 파일명+프롬프트 목록**으로 묶어서 시킨다(모든 파일은 같은 `-C` 디렉토리 안).
- 참고 이미지 기반 생성/변형: `-i /path/ref.png` 플래그로 첨부.
- 투명 배경: 프롬프트에 `transparent background` 요청 — codex 내장 imagegen이 크로마키 후처리까지 지원.

## Prompt Tips (품질 좌우)

- 영어 한 문단, 구체적으로: 주제 + 스타일(`flat illustration`, `editorial photo`, `3D render`) + 컬러 hex + 배경/조명 + "no text" (글자 왜곡 방지)
- 사용자가 한국어로 요청해도 프롬프트는 영어로 번역해 전달 (품질 ↑). 이미지 안에 한국어 텍스트가 필요하면 예외적으로 명시.

## Verify & Deliver (필수)

1. `test -s "$OUT"` — 파일 존재+크기 확인. 실패 시 `/tmp/gpt-image-last.txt`와 stdout 에러 확인
2. Read 툴로 이미지를 직접 열어 프롬프트 의도와 맞는지 QA
3. 사용자에게 전달 — `SendUserFile` 같은 파일 전달 도구가 있으면 사용하고, 없으면 절대경로를 알린다

## Troubleshooting

| 증상 | 원인/해결 |
|---|---|
| `codex: command not found` | 사용자에게 `npm install -g @openai/codex` 설치 후 `codex login` 안내 |
| `Not inside a trusted directory` | `--skip-git-repo-check` 누락 |
| stdin에서 멈춤 | `</dev/null` 누락 |
| 복사 실패 (`Operation not permitted`, read-only) | `-s workspace-write` 누락 또는 `$OUT`이 `-C` 디렉토리 밖 |
| 이미지 생성 도구가 없다고 응답 | `codex features list`로 `image_generation` 확인 → `codex features enable image_generation`, codex 최신 버전으로 업데이트 |
| 파일이 $OUT에 없음 | `ls -t ~/.codex/generated_images/*/ \| head`에서 최신 PNG 수동 복사 |
| 401/로그인 에러 | 사용자에게 `codex login` 안내 (직접 로그인 금지) |
| 타임아웃 | timeout 600000 + run_in_background로 재시도 |

## Common Mistakes

- `~/.codex/generated_images/` 세션 경로를 직접 추측/파싱 → 세션 ID는 예측 불가. 복사는 codex에게 시킬 것
- 모호한 한 줄 프롬프트 그대로 전달 → 위 Prompt Tips대로 보강 후 전달
- 생성 결과를 안 보고 완료 선언 → 반드시 Read로 QA
