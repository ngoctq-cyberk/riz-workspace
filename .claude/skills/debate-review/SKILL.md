---
name: debate-review
version: 0.1.0
description: |
  Multi-round cross-model code review. Claude và Codex review độc lập,
  rebut findings của nhau, ra final position; main agent tổng hợp thành
  AGREED / CONFIRMED / DISPUTED / RETRACTED.
---

# debate-review

Skill này chạy **một phiên review 3 vòng** giữa Claude và Codex (qua codex
CLI / plugin) để biến mọi finding thành tín hiệu có độ tin cậy rõ ràng:

| Loại | Ý nghĩa |
|---|---|
| **AGREED** | Cả hai model cùng chỉ ra → ưu tiên fix |
| **CONFIRMED** | Một bên raise, bên kia ban đầu phản đối nhưng đã concede |
| **DISPUTED** | Hai bên giữ nguyên quan điểm sau 3 rounds → cần con người quyết |
| **RETRACTED** | Bên raise đã tự rút → bỏ qua |

## Khi nào dùng

- Review những thay đổi quan trọng (security, contract, migration, refactor lớn)
  — nơi 1 model dễ overlook hoặc quá tự tin
- Khi muốn phân biệt finding "thật sự" vs "noise"
- Trước khi merge feature có ảnh hưởng cross-cutting

## Cách dùng

```
/debate-review                    # review working tree diff hiện tại
/debate-review --base main        # review HEAD..main
/debate-review --paths "src/**"   # giới hạn theo glob
```

## Yêu cầu

1. **Codex plugin for Claude Code** đã cài. Docs: https://github.com/openai/codex-plugin-cc
   ```bash
   /plugin marketplace add openai/codex-plugin-cc
   /plugin install codex@openai-codex
   /reload-plugins
   /codex:setup              # xác nhận Codex ready + đã login
   ```
   Skill gọi Codex qua **`/codex:rescue`** (free-form prompt, hỗ trợ
   `--background` / `--resume`) để giữ session Codex xuyên suốt 3 rounds.
2. **ChatGPT subscription** hoặc **OpenAI API key** (yêu cầu của plugin).
3. **Claude Code** phiên bản hỗ trợ Task tool (subagent) + plugin slash commands.
4. Trong git repo (cần `git diff`).

## Cấu trúc skill (3 paths trong `.claude/`)

- `.claude/skills/debate-review/SKILL.md` — manifest (file này)
- `.claude/skills/debate-review/templates/finding-schema.md` — schema bắt buộc cho mọi finding
- `.claude/skills/debate-review/templates/round-prompts.md` — prompt template cho Round 1/2/3
- `.claude/skills/debate-review/templates/final-report.md` — schema báo cáo tổng hợp
- `.claude/skills/debate-review/references/design.md` — vì sao thiết kế thế này, tradeoffs
- `.claude/commands/debate-review.md` — slash command orchestrator ("engine")
- `.claude/agents/debate-reviewer.md` — Claude-side reviewer subagent (dùng cho cả 3 rounds)

## Cài sang project mới

Copy 3 thứ trong `.claude/` của project nguồn sang `.claude/` của project đích:

| Source (project có skill) | Destination (project mới) |
|---|---|
| `.claude/skills/debate-review/` (folder) | `.claude/skills/` |
| `.claude/commands/debate-review.md` | `.claude/commands/` |
| `.claude/agents/debate-reviewer.md` | `.claude/agents/` |

**Cách 1 — Finder** (Cmd+C / Cmd+V):
- Bật hidden files: `⌘ + Shift + .`
- Project đích chưa có `.claude/`? Mở Terminal: `mkdir -p .claude/skills .claude/commands .claude/agents`
- Copy + paste 3 mục vào subfolder tương ứng

**Cách 2 — Terminal**:
```bash
SRC=/path/to/source-project
TGT=/path/to/target-project
mkdir -p "$TGT"/.claude/skills "$TGT"/.claude/commands "$TGT"/.claude/agents
cp -aR "$SRC"/.claude/skills/debate-review "$TGT"/.claude/skills/
cp -a  "$SRC"/.claude/commands/debate-review.md "$TGT"/.claude/commands/
cp -a  "$SRC"/.claude/agents/debate-reviewer.md "$TGT"/.claude/agents/
```

Reload Claude Code session ở project đích → `/debate-review` chạy được.

## Output

Mọi artifact lưu vào `.debate-review/<run-id>/`:

```
.debate-review/2026-04-18-1430-abc123/
├── diff.patch                   # snapshot diff đang review
├── round-1/
│   ├── claude.md                # findings độc lập của Claude
│   └── codex.md                 # findings độc lập của Codex
├── round-2/
│   ├── claude-rebuttals.md      # Claude phản biện findings của Codex
│   └── codex-rebuttals.md       # Codex phản biện findings của Claude
├── round-3/
│   ├── claude-final.md          # Claude: hold / concede / partial
│   └── codex-final.md           # Codex: hold / concede / partial
└── REPORT.md                    # tổng hợp do main agent viết
```

## Quan trọng

- Skill này **chỉ produce report**, không fix code. Việc triage và fix là
  của human hoặc agent khác.
- Không phải mọi DISPUTED finding đều xấu — đó là tín hiệu giá trị nhất:
  hai model thông minh không đồng ý, nghĩa là có gì đó tinh tế đáng review.
- Skill hoàn toàn standalone — chỉ cần git repo + Codex plugin. Không phụ
  thuộc workflow/spec system nào khác.
