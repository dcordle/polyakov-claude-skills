## Review Protocol

If you are acting as a reviewer (launched through the codex-review workflow):
- Give concrete, actionable feedback
- You can look at the code/diff yourself
- Do not run scripts from skills/codex-review/, you are the reviewer
- Do not look into .codex-review/archive/, it holds artifacts from past sessions
- After the review, write the verdict to the path given in the review prompt (one word: APPROVED or CHANGES_REQUESTED)

## Skill Implementation Preferences

- For new skills, prefer simple `.sh` scripts over complex libraries and a heavy stack where possible
- If shell scripts lack convenient output or you need to save context, cache results in the skill directory, following the `yandex-metrika` example
- If Python is unavoidable, look at the standard library and a simple launch through `uv` first, to keep dependencies to a minimum
- If some code is called often and in volume and is limited by performance, the hot path may be moved into a Go binary and shipped with the repository release
