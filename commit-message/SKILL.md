---
name: commit-message
description: >
  Generate conventional commit messages from staged git changes. Use this skill when the user
  asks to generate a commit message, write a commit message, or commit staged changes with a
  descriptive message. Triggers on requests like "generate commit message", "write commit msg",
  "commit this", "/commit", or any request involving creating git commit messages from current
  staged code.
---

# Commit Message Generator

Generate structured commit messages from staged git changes using the Conventional Commits format.

## Workflow

1. Run `git diff --cached --stat` to get an overview of changed files
2. Run `git diff --cached` to get the full staged diff
3. Run `git log --oneline -10` to understand recent commit style
4. Analyze the diff and classify the change type
5. Generate the commit message following the format below
6. Present the message to the user for confirmation
7. Run `git commit -m "<message>"` after user approval

## Commit Message Format

```
<type>(<scope>): <short description>

- <change point 1>
- <change point 2>
- ...
```

### Type

Determine the primary type from the diff content:

| Type | When to Use |
|------|-------------|
| `feat` | New feature or capability added |
| `fix` | Bug fix |
| `refactor` | Code restructuring without behavior change |
| `style` | Formatting, whitespace, semicolons (no logic change) |
| `docs` | Documentation only changes |
| `test` | Adding or updating tests |
| `chore` | Build process, dependencies, tooling |
| `perf` | Performance improvement |
| `ci` | CI/CD configuration changes |

If a commit spans multiple types, use the most dominant one. If truly equal, prefer `feat` > `fix` > `refactor`.

### Scope

Derive scope from the changed file paths or module names. Use the most specific relevant module/component name.

- Single file in `src/auth/login.ts` → scope: `auth`
- Multiple files in `src/components/` → scope: `components`
- Project-wide config change → scope: `config`
- If scope is unclear or too broad, omit it: `feat: description`

### Short Description

- Use imperative mood: "add", "fix", "update" (not "added", "fixes", "updated")
- Lowercase first letter, no period at end
- Max 50 characters
- Describe WHAT changed, not HOW

### Change Points

List specific changes as bullet points:

- Each point starts with a verb in imperative mood
- Be concrete: reference file names, function names, or components
- Group related changes into a single point
- Typically 2-5 points, skip if only one trivial change
- Focus on WHAT and WHY, not line-by-line diffs

## Examples

**Example 1: New feature**

Diff: Added a new `UserAvatar` component with image upload support.

```
feat(components): add user avatar with image upload

- add UserAvatar component with drag-and-drop support
- integrate S3 presigned URL for secure image upload
- add avatar size variants (sm, md, lg) via props
```

**Example 2: Bug fix**

Diff: Fixed timezone handling in date formatter that caused incorrect display.

```
fix(utils): correct timezone offset in date formatting

- use UTC base time before applying local timezone conversion
- handle DST transition edge cases in formatDate()
```

**Example 3: Refactoring**

Diff: Extracted repeated validation logic into shared utility.

```
refactor(validation): extract shared form validation helpers

- move email/phone validators from LoginForm and SignupForm to validators.ts
- replace inline regex patterns with reusable validate() calls
- remove duplicated error message constants
```

**Example 4: Multiple scopes**

Diff: Updated API endpoint and corresponding frontend call.

```
feat(api): add pagination support for user list endpoint

- add page and pageSize query params to GET /api/users
- implement cursor-based pagination in UserRepository
- update UserList component to handle paginated responses
```

## Rules

- Always read the full staged diff before generating the message
- Never fabricate changes not present in the diff
- Write the commit message in English regardless of conversation language
- If no files are staged, inform the user and suggest `git add`
- If the diff is too large (>500 lines), summarize by file/module rather than line-by-line
- Use a HEREDOC when executing `git commit` to preserve formatting:
  ```bash
  git commit -m "$(cat <<'EOF'
  type(scope): description

  - change point 1
  - change point 2
  EOF
  )"
  ```
