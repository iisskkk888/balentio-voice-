# Git Guidelines

## Conventional Commits Format

```
<type>[optional scope]: <description>
```

### Commit Types

- `feat`: New features
- `fix`: Bug fixes
- `docs`: Documentation changes
- `refactor`: Code changes that neither fix bugs nor add features
- `perf`: Performance improvements
- `test`: Adding or modifying tests
- `chore`: Maintenance tasks, dependency updates
- `style`: Formatting changes
- `build`: Build system or dependencies

### Commit Message Rules

- **Start with lowercase** after the colon: `feat: add feature` (not `feat: Add feature`)
- **Use imperative mood**: "add" not "added" or "adds"
- **No period at the end**
- **Keep under 50-72 characters**
- **Scope is optional**: use only when it adds clarity (`feat(auth): add login`)

### Examples

- `feat(transcription): add model selection for OpenAI providers`
- `fix(audio): resolve buffer overflow on long recordings`
- `refactor(dialog): simplify state management`
- `docs: update API documentation`
- `chore: update dependencies to latest versions`

### Breaking Changes

Add `!` after type/scope: `feat(api)!: change endpoint structure`

### Multi-line Commits

Use HEREDOC for longer messages:

```bash
git commit -m "$(cat <<'EOF'
feat(auth): add JWT authentication

- Add access and refresh token generation
- Implement token validation middleware
- Store tokens in secure storage
EOF
)"
```

## Commit Best Practices

- NEVER include AI watermarks or attribution
- Each commit = one logical change
- Write for your future self
- If you need multiple paragraphs, consider splitting the commit

## Pull Request Guidelines

- NEVER include AI watermarks or attribution
- PR title follows conventional commit format
- Focus on "why" and "what", not "how it was created"
- Keep descriptions short and clear (2-3 sentences for most changes)
- Link to issues if relevant

### PR Description Format

**Paragraph 1**: What changed and why
**Paragraph 2**: How it works (if needed)

Example:
```
This adds JWT authentication to the backend API. Users can now login and
receive access/refresh tokens that are validated on protected routes.

The implementation uses separate secrets for access (15min) and refresh
(30 days) tokens, with middleware that validates tokens on each request.
```

### Basic Git Commands

```bash
# Create feature branch
git checkout -b feature-name

# Make changes, then commit
git add .
git commit -m "feat: add feature"

# Push to GitHub
git push origin feature-name

# Create PR on GitHub UI or:
gh pr create --title "feat: add feature" --body "Description here"

# Merge (use GitHub UI or):
git checkout main
git merge feature-name
git push
```

## What NOT to Include

- Any AI watermarks or attribution
- "Co-Authored-By" lines
- Tool attribution
- Long explanations of obvious changes
