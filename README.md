# gpt-image

Claude Code에서 **ChatGPT 이미지 생성**을 쓰게 해주는 스킬입니다. Claude Code는 이미지를 직접 만들 수 없어서, 내 컴퓨터에 설치된 OpenAI Codex CLI의 `image_generation` 도구에 작업을 넘기고 결과 PNG를 원하는 경로에 저장합니다.

> "랜딩 페이지 히어로 이미지 만들어줘" → Claude가 프롬프트를 영어로 다듬음 → Codex CLI가 생성 → 지정한 경로에 PNG 저장 → Claude가 직접 열어서 확인

## 준비물

| 항목 | 방법 |
|---|---|
| Claude Code | https://claude.com/claude-code |
| OpenAI Codex CLI | `npm install -g @openai/codex` |
| ChatGPT 계정 로그인 | `codex login` (사용량은 로그인한 ChatGPT 계정 기준) |
| 이미지 기능 켜짐 확인 | `codex features list \| grep image_generation` → `true`가 아니면 `codex features enable image_generation` |

## 설치

### 방법 1. 플러그인으로 설치 (권장)

Claude Code 안에서 입력:

```
/plugin marketplace add kjhk3082/gpt-image
/plugin install gpt-image@gpt-image
```

### 방법 2. 스킬 파일만 복사

```bash
mkdir -p ~/.claude/skills/gpt-image
curl -fsSL https://raw.githubusercontent.com/kjhk3082/gpt-image/main/skills/gpt-image/SKILL.md \
  -o ~/.claude/skills/gpt-image/SKILL.md
```

설치 후 Claude Code를 다시 시작하세요.

## 사용법

그냥 말하면 됩니다.

- "블로그 썸네일로 쓸 커피 사진풍 이미지 하나 만들어줘"
- "앱 아이콘 시안 3개, 정사각형으로 만들어줘"
- "이 사진(경로) 분위기로 배너 이미지 만들어줘"

직접 호출할 때는 방법 1로 설치했으면 `/gpt-image:gpt-image <설명>`, 방법 2면 `/gpt-image <설명>`입니다.

## 알아둘 점

- 1장에 1~3분 걸리고, Codex 토큰을 약 4만 개 씁니다(실측 참고치). 여러 장은 한 번의 실행으로 묶어서 만듭니다.
- 크기: 가로 3:2 → 1536×1024 · 정사각형 → 1024×1024 · 세로 2:3 → 1024×1536
- 투명 배경, 참고 이미지 기반 변형도 됩니다.
- 아이콘·차트·다이어그램은 SVG가 더 나아서 이 스킬을 쓰지 않습니다. 실존 브랜드 로고나 실존 인물은 만들지 않습니다.
- Codex는 `workspace-write` 샌드박스로 실행되어, 이미지를 저장할 폴더 안에만 파일을 씁니다.

## 동작 방식

`codex exec`를 비대화형으로 실행해 "이미지를 생성한 뒤 이 절대경로로 복사하라"고 지시합니다. Codex 내부 저장 경로를 추측하지 않고, 결과 파일이 실제로 생겼는지 확인한 다음 Claude가 이미지를 직접 열어 요청 의도와 맞는지 검수합니다. 자세한 규칙은 [`skills/gpt-image/SKILL.md`](skills/gpt-image/SKILL.md)에 있습니다.

## 문제 해결

| 증상 | 해결 |
|---|---|
| `codex: command not found` | `npm install -g @openai/codex` 후 `codex login` |
| 401 / `refresh token has expired` | `codex login`으로 다시 로그인 |
| 이미지 도구가 없다는 응답 | `codex features enable image_generation`, Codex CLI 최신 버전으로 업데이트 |

---

## English

A Claude Code skill that generates real images (PNG) by delegating to the OpenAI Codex CLI's `image_generation` tool (signed in with ChatGPT). Claude refines your request into a detailed prompt, Codex generates the image and copies it to an exact path, and Claude opens the file to check it before handing it to you.

**Requirements:** Claude Code · `npm install -g @openai/codex` · `codex login` · `image_generation` enabled (check with `codex features list`).

**Install as a plugin:**

```
/plugin marketplace add kjhk3082/gpt-image
/plugin install gpt-image@gpt-image
```

**Or copy the skill file:**

```bash
mkdir -p ~/.claude/skills/gpt-image
curl -fsSL https://raw.githubusercontent.com/kjhk3082/gpt-image/main/skills/gpt-image/SKILL.md \
  -o ~/.claude/skills/gpt-image/SKILL.md
```

Then just ask Claude, e.g. "make a hero image for my landing page". Tested with codex-cli 0.160.0.

## License

[MIT](LICENSE)
