---
name: setup-matt-pocock-skills
description: "Bind this repository to Opran for the engineering skills: write the `<!-- opran:begin -->` block in AGENTS.md naming its Opran project, repository, default branch, gate and conventions. Run once per repository, and again to refresh the block."
disable-model-invocation: true
---

# Set up the skills for Opran

The engineering skills read the issue tracker, the triage labels and the domain docs from the repository's context. In this fork that context is **Opran**: the project's issues and merge requests through the `opran` MCP server, and the team's conventions in the wiki pages of the conventions project the project imports (`CONV`). The repository itself holds only one block in `AGENTS.md`, between `<!-- opran:begin -->` and `<!-- opran:end -->`. This skill writes that block. It writes no `docs/agents/*.md`: what those held lives in the conventions project's pages.

This is a prompt-driven skill, not a script. Explore, show the user the block, confirm, then write.

## Process

### 1. Explore

Read what exists; don't assume:

- **The repository:** `git remote -v`. The Opran repository is the remote whose URL is `https://<install>/<KEY>/<name>.git`; its handle is `<KEY>/<name>`. With no such remote, ask the user for the repository's handle.
- **The project:** call `get_project` with `<KEY>` through the `opran` MCP server. It gives the repository's default branch (`repositories[].defaultBranch`), the projects it imports (`imports`), and its agent instructions, each entry naming its project. The conventions project is the one it imports whose entry tells how the team works (normally `CONV`); if it imports several, or none, or the repository is in the conventions project itself, ask the user which. If the `opran` server isn't connected, stop and tell the user to add it to `.mcp.json` and enable it, then run this skill again.
- **The gate and the queue:** whether `.opran/gate.yaml` exists on the default branch's tip in Opran (`git fetch origin`, then `git cat-file -e origin/<branch>:.opran/gate.yaml`), and whether a protection of the repository has the merge queue on (`get_project`'s `protections[].mergeQueue`). The block names both either way, as the queue's gate reads that file; when either is missing, say so in step 2.
- **The files:** `AGENTS.md` and `CLAUDE.md` at the root, and whether either already holds the block (its two marker lines). Also note an older `## Agent skills` section without markers, and `docs/agents/`: they are upstream's setup, which the block replaces.

### 2. Show the block and confirm

Show the user the block, filled in from what you found, and where it goes (step 3). Let them correct the key, the repository, the default branch or the conventions project before writing. A default branch other than the one `get_project` gives is a mismatch to name: the block should say the branch Opran has, unless the user is changing it there too.

The block, exactly (the same text Opran's onboarding writes in its setup merge request, so either can rewrite the other's; its apostrophes are the typographic `’`, copy them as they are):

```markdown
<!-- opran:begin -->
## Agent skills

This repository is `<KEY>/<name>`, of the Opran project `<KEY>`; its default branch is `<branch>`.

- The gate’s steps are in `.opran/gate.yaml`.
- The conventions: call `get_project`, then read `<CONV>`’s pages for the parts that apply.
- Work reaches `<branch>` only by merge request, through the merge queue, as `<CONV>`’s Merge requests page describes.
<!-- opran:end -->
```

### 3. Write

**The file:** `AGENTS.md`. Put the block in `CLAUDE.md` only when there is no `AGENTS.md`; otherwise leave `CLAUDE.md` alone. With neither file, create `AGENTS.md`.

**Where in it:**

- A marker counts only as a line of its own (indented by at most three spaces, spaces or tabs after it allowed), outside fenced code: a fence opens with three or more backticks or tildes (a backtick line whose rest holds a backtick opens none) and closes with the same character, at least as many, and nothing but spaces or tabs after them. A marker in a fence, in inline code or within a sentence is text, not a marker.
- The file has the block: replace everything from its begin line to its end line, both included, with the new block. Leave everything around it as written.
- The file has no block: add it at the end, after the file's text with its trailing whitespace dropped and one blank line, the block ending in a line ending. The block is the file's whole text when the file is new, empty or holds only whitespace.
- The file begins the block more than once, or has a begin with no end (or an end with no begin): write nothing. Tell the user which lines, and let them settle it first.
- Keep the file's line endings (CRLF if it uses them), as git stores the file: write in the checkout's working copy, and let git's own line-ending settings do what they do on commit.

Running the skill again rewrites the block in place; when it already says the same, change nothing.

**Upstream's setup:** an older `## Agent skills` section that no markers enclose (the block's own heading is inside them), and `docs/agents/*.md`, are what upstream's setup wrote. Leave them while the project still uses them; offer to remove them once its conventions project's pages carry what they said. Never remove them without the user's yes.

### 4. Done

Tell the user the block is written and where, that the skills now find the tracker through `get_project` and the conventions project's pages, and that the change reaches the default branch by merge request like any other.
