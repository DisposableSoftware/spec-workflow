<h1 align="center">Template Project</h1>

<p align="center">A spec-first project, built with Claude Code.</p>

> [!WARNING]
> **Certified, artisanal, vibe-coded slop.**
>
> No human has read this code. An AI wrote all of it while a person said "yeah, looks
> good" and pressed Enter a lot.

<!-- template:begin -->
## Using this template

A stack-neutral starting point for building software with
[Claude Code](https://claude.com/claude-code), spec first. It is the workflow from
[gtk-desktop-template](https://github.com/DisposableSoftware/gtk-desktop-template) with the
GTK, Rust and Nix taken out: the spec, the skills that drive it, the subagents that do the
reading, and the scripts that keep the spec and the code in step. Bring your own language
and toolchain.

You need git, bash, awk, perl and the [GitHub CLI](https://cli.github.com). Everything
works on macOS and Linux.

1. Create your repo from this one: **Use this template** on GitHub, or

   ```sh
   gh repo create my-project --public --template DisposableSoftware/spec-workflow
   ```

   and clone it. With one worktree per branch:

   ```sh
   git clone --bare https://github.com/you/my-project.git my-project/.bare
   cd my-project && echo "gitdir: ./.bare" > .git
   git config remote.origin.fetch "+refs/heads/*:refs/remotes/origin/*"
   git fetch origin && git worktree add main main && cd main
   ```

2. Name it:

   ```sh
   scripts/init.sh "My Project"
   scripts/check.sh
   git commit -am "Name the template My Project"
   ```

   The script removes itself and this section, and enables the pre-push hook.

3. Set up the stack. Add your build, lint and test commands to `scripts/check.sh` and the
   Commands section of `CLAUDE.md`, and the files your stack ignores to `.gitignore`.
4. Edit `spec/product.md` to say what the project is, its non-goals and its stack
   constraints. Then start Claude Code and ask for features with `/spec-feature`, fixes with
   `/spec-bug`, and planned work with `/spec-implement`.

What you get:

- **`spec/`**: every behaviour as a numbered requirement with a status and how it is
  verified. `CLAUDE.md` makes every change go spec first: a `spec:` commit, then code
  with a `Spec: ID` trailer.
- **`/spec-feature`, `/spec-bug` and `/spec-implement`** (`.claude/skills/`):
  `/spec-feature` drafts the spec change and `/spec-bug` reproduces a bug and drafts the
  spec gap; both ask you to approve and implement, or approve the spec only.
  `/spec-implement` builds the roadmap's approved rows in order, one verified commit each.
- **Subagents** (`.claude/agents/`): `spec-researcher` does the read-only code survey and
  bug tracing, so file dumps stay out of the main conversation, and `spec-verifier`
  checks each change against its rows before it is committed, with fresh eyes.
- **One gate** (`scripts/check.sh`): the spec lint, the tooling tests and your project's
  checks. `spec-lint.sh` catches duplicate or misfiled IDs, unknown statuses, `done` rows
  with nothing verifying them, and a roadmap out of step with the spec.
- **Releases**: `release-check.sh` recommends a semver bump from the spec's status
  changes, `release.sh` bumps `VERSION`, tags and drafts notes, the pre-push hook offers
  to do it all when you push main, and `/spec-release` does it from Claude Code.

<!-- template:end -->

## Using the spec workflow

Every change starts in [`spec/`](spec/README.md), as a row with an ID, a status and a way
to verify it. Code follows the spec, never the other way round. You drive it from Claude
Code with three skills:

| You want | Run | What happens |
|----------|-----|--------------|
| New behaviour or a change | `/spec-feature <request>` | Drafts the spec rows (status `planned`) and asks you to approve them. |
| To fix a bug | `/spec-bug <report>` | Reproduces it, finds the root cause, and drafts the spec row that was missing, with a failing check. |
| The approved work built | `/spec-implement` | Works through `spec/roadmap.md`: the verification backlog first, then planned rows in order. |
| To ship | `/spec-release` | Pushes `main` and releases the version `scripts/release-check.sh` recommends. |

### A typical change

1. **Ask.** Run `/spec-feature add CSV export`. Claude researches where the behaviour
   would live, then proposes new requirement rows and any roadmap changes. It writes no
   code.
2. **Approve.** Choose **approve and implement** to build it straight away, or
   **approve the spec only** to commit the rows as `planned` and stop. Edit the wording
   first if a row isn't what you meant.
3. **Implement.** `/spec-implement` takes each outstanding row in roadmap order: code,
   then `scripts/check.sh`, then the `spec-verifier` subagent re-reading the change
   against the row. It sets the row to `done` (or `partial`, with the gap written in)
   and commits with a `Spec: ID` trailer. It asks you when the spec leaves a question
   open, and never pushes.
4. **Release.** `/spec-release` pushes `main` and tags a patch or minor release; it asks
   before a major one.

Use `/spec-implement` on its own to pick up work you approved earlier, and put the rows in
`spec/roadmap.md` in the order you want them built.

### Things to know

- **Status values:** `planned` (agreed, not built), `partial` (built with a known gap),
  `done` (built and verified), `dropped` (no longer wanted). IDs are permanent: never
  renumber or reuse one.
- **Commits:** `spec: <summary>` for spec changes, committed before the code; a
  `Spec: ID, ID` trailer on code commits; `BREAKING` in the body for incompatible changes.
- **Gaps are bugs in the spec.** If code needs behaviour no row describes, go back and
  add the row first. `/spec-bug` exists for the case where a bug slipped past because the
  spec didn't pin the behaviour down.
- **Partial verification is reported, not hidden.** If a manual check couldn't be run,
  the report says so.
- **Add a spec file** (with a new ID prefix) when a feature area doesn't fit the existing
  ones, and list it in `spec/README.md`.

`CLAUDE.md` has the full rules, and the skills themselves are in `.claude/skills/`.

## Development

What the project does is written down in [`spec/`](spec/README.md); see
[`CLAUDE.md`](CLAUDE.md) for how changes are made.

```sh
scripts/check.sh            # spec lint, tooling tests, project checks
scripts/release-check.sh    # what the next release would be, and why
```

## License

MIT, see [LICENSE](LICENSE).
