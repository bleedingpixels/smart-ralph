# Contributing to Smart Ralph

*"I'm learnding!"* - You, after reading this guide

First off, thanks for wanting to contribute! This project welcomes contributors of all experience levels. Whether you're fixing a typo or adding a whole new agent, your help is appreciated.

## Table of Contents

- [Code of Conduct](#code-of-conduct)
- [Ways to Contribute](#ways-to-contribute)
- [Development Setup](#development-setup)
- [Making Changes](#making-changes)
- [Pull Request Process](#pull-request-process)
- [Adding Your Fork to the Marketplace](#adding-your-fork-to-the-marketplace)
- [Style Guide](#style-guide)
- [Getting Help](#getting-help)

## Code of Conduct

Be kind. Be respectful. Remember that behind every GitHub handle is a real person. We're all here to build something useful together.

## Ways to Contribute

Not all contributions require code:

- **Report bugs** - Found something broken? Open an issue
- **Suggest features** - Have an idea? We want to hear it
- **Improve docs** - Typos, unclear explanations, missing examples
- **Answer questions** - Help others in issues and discussions
- **Write code** - Bug fixes, new features, refactoring

### Good First Issues

Look for issues labeled `good first issue` or `help wanted`. These are specifically chosen for newer contributors.

## Development Setup

### Prerequisites

- [Claude Code](https://claude.ai/code) installed
- Git
- A project to test with

### Local Development

```bash
# Clone the repo
git clone https://github.com/tzachbon/smart-ralph.git
cd smart-ralph

# Test the plugin locally
claude --plugin-dir ./plugins/ralph-specum

# Make changes, restart Claude Code to reload
```

### Project Structure

```
smart-ralph/
├── plugins/
│   └── ralph-specum/
│       ├── .claude-plugin/
│       │   └── plugin.json      # Plugin manifest
│       ├── agents/              # Sub-agent definitions
│       ├── commands/            # Slash command implementations
│       ├── hooks/               # Stop watcher (logging only)
│       ├── templates/           # Spec file templates
│       └── schemas/             # JSON schemas for validation
└── README.md
```

### Testing Changes

1. Make your changes
2. Restart Claude Code with `--plugin-dir` pointing to your local copy
3. Run through the workflow: `/ralph-specum:start test-feature Some test goal`
4. Verify each phase works as expected

## Making Changes

### Branch Naming

```
feature/description    # New features
fix/description        # Bug fixes
docs/description       # Documentation only
refactor/description   # Code refactoring
```

### Commit Messages

Keep them short and descriptive:

```
Good:
- Add retry logic to spec-executor
- Fix state cleanup on cancel
- Update installation docs

Bad:
- Fixed stuff
- WIP
- asdfasdf
```

## Pull Request Process

1. **Fork** the repo and create your branch from `main`
2. **Make** your changes
3. **Test** locally with Claude Code
4. **Update** documentation if needed
5. **Submit** PR with clear description of changes

### PR Description Template

```markdown
## What

Brief description of changes

## Why

Why this change is needed

## Testing

How you tested it
```

### Review Process

- PRs require at least one approval
- CI checks must pass
- Maintainers may request changes
- Be patient, we review as quickly as we can

## Adding Your Fork to the Marketplace

Want to distribute your own version of Smart Ralph? Here's how to make your fork available via the Claude Code plugin marketplace.

### Prerequisites

Before publishing your fork:

1. **Test thoroughly** - Ensure your changes work as expected
2. **Update documentation** - Document any new features or breaking changes
3. **Choose a version** - Follow semantic versioning for your fork
4. **Create a fork** - Fork the repository to your GitHub account

### Step-by-Step Guide

#### 1. Prepare Your Fork

```bash
# Clone your fork
git clone https://github.com/YOUR_USERNAME/smart-ralph.git
cd smart-ralph

# Make your modifications
# ... edit files ...

# Commit your changes
git add .
git commit -m "feat: describe your changes"
```

#### 2. Update Plugin Metadata

Update the plugin manifest with your fork's information:

**File**: `plugins/ralph-specum/.claude-plugin/plugin.json`

```json
{
  "name": "ralph-specum",
  "version": "3.2.0",  // Your version (use semver)
  "description": "Your description of changes",
  "author": "Your Name <your.email@example.com>",
  "repository": "https://github.com/YOUR_USERNAME/smart-ralph"
}
```

**Version Management Best Practices**:
- **Major version** (X.0.0): Breaking changes, incompatible API changes
- **Minor version** (0.X.0): New features, backwards compatible
- **Patch version** (0.0.X): Bug fixes, backwards compatible

Example: If you fork from v3.2.0 and add new features, use v3.3.0. If you have breaking changes, use v4.0.0.

**Alternative: Independent Versioning for Forks**

If your fork diverges significantly from upstream, consider resetting to v1.0.0:

```json
{
  "name": "ralph-specum-username",
  "version": "1.0.0",
  "description": "Custom fork with XYZ features",
  "upstream_version": "3.2.0"
}
```

This approach:
- Clearly distinguishes your fork from the original
- Avoids version number conflicts
- Makes it easier to track upstream correspondence

#### 3. Configure marketplace.json (Optional)

For marketplace distribution, create or update `marketplace.json`:

**File**: `.claude-plugin/marketplace.json`

```json
{
  "plugins": [
    {
      "name": "ralph-specum-username",
      "version": "1.0.0",
      "owner": {
        "name": "YOUR_USERNAME",
        "type": "user"
      },
      "source": "https://github.com/YOUR_USERNAME/smart-ralph",
      "description": "Custom fork of ralph-specum with [your features]",
      "fork": {
        "original": "ralph-specum",
        "original_owner": "tzachbon",
        "original_source": "https://github.com/tzachbon/smart-ralph"
      },
      "tags": ["fork", "ralph-specum", "custom"],
      "readme": "https://github.com/YOUR_USERNAME/smart-ralph/blob/main/README.md"
    }
  ]
}
```

**Key elements for forks**:
- Update `owner.name` to your GitHub username
- Update `source` to your fork's repository URL
- Add `fork` object with original plugin metadata
- Use a distinct name (e.g., `ralph-specum-username`)
- Add "fork" tag for discoverability

#### 4. Publish Your Fork

Make your fork available as a marketplace source:

```bash
# Push to GitHub
git push origin main

# Tag your release (recommended)
git tag -a v3.3.0 -m "Release v3.3.0: My awesome changes"
git push origin v3.3.0
```

#### 5. Users Install Your Fork

Share these instructions with users:

```bash
# Add your fork as a marketplace source
/plugin marketplace add YOUR_USERNAME/smart-ralph

# Install the plugin from your fork
/plugin install ralph-specum@smart-ralph

# Restart Claude Code
```

**Alternative: Direct installation from GitHub**

```bash
# Install directly without marketplace
/plugin install https://github.com/YOUR_USERNAME/smart-ralph
```

### Maintenance

#### Keeping Your Fork Updated

Periodically sync with upstream to get bug fixes and new features:

```bash
# Add upstream remote (one-time)
git remote add upstream https://github.com/tzachbon/smart-ralph.git

# Fetch upstream changes
git fetch upstream

# Merge upstream changes into your fork
git merge upstream/main

# Push updated fork
git push origin main
```

#### Releasing Updates

When you publish new changes to your fork:

1. Update the version in `plugin.json`
2. Create a new git tag: `git tag -a v3.3.1 -m "Release v3.3.1"`
3. Push the tag: `git push origin v3.3.1`
4. Users can update by re-running `/plugin install ralph-specum@smart-ralph`

### Documentation

Consider documenting your fork-specific changes:

1. **README.md** - Add a section explaining your fork's purpose and changes
2. **CHANGELOG.md** - Track version history and changes
3. **Fork-specific docs** - Add docs/ directory for detailed documentation

### Community & Support

- **Issues** - Enable issues on your fork for bug reports and feature requests
- **Discussions** - Use GitHub Discussions for community Q&A
- **License** - Maintain the original license (or comply with its terms for derivative works)

### Official Documentation

For more details on the plugin marketplace, see:
- [Claude Code Plugin Marketplaces](https://code.claude.com/docs/en/plugin-marketplaces)
- [Plugin Development Guide](https://code.claude.com/docs/en/plugins)

### Attribution

Please credit the original project in your fork's README:

```markdown
This is a fork of [tzachbon/smart-ralph](https://github.com/tzachbon/smart-ralph)
```

## Style Guide

### General Principles

- Keep it simple
- Prefer clarity over cleverness
- Match existing patterns in the codebase

### Agent Definitions

- Use clear, action-oriented descriptions
- Include examples where helpful
- Keep system prompts focused

### Commands

- Command names should be verb-based (`start`, `cancel`, `status`)
- Include help text for all commands
- Handle errors gracefully with useful messages

### File Naming

- Use kebab-case for files: `spec-executor.md`, `task-planner.md`
- Use descriptive names that indicate purpose

## Getting Help

Stuck? Have questions?

- **Issues** - Open an issue with the `question` label
- **Discussions** - Use GitHub Discussions for general questions

## Recognition

Contributors are recognized in release notes. Significant contributors may be invited as collaborators.

---

<div align="center">

*"My cat's breath smells like cat food."*

Thanks for contributing!

</div>
