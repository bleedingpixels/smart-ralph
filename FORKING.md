# FORKING.md

This guide explains how to fork Smart Ralph and configure it as a plugin marketplace for your team or personal use.

## Why Fork Smart Ralph?

Forking Smart Ralph allows you to:

- **Customize agents**: Modify research, requirements, design, or task planning agents
- **Customize templates**: Adapt spec templates to your team's documentation style
- **Private development**: Test changes before contributing upstream
- **Team customization**: Configure specific workflows for your organization
- **Version pinning**: Lock to specific versions for stability

## Prerequisites

- A GitHub account
- Basic familiarity with git
- Claude Code installed and configured

## Step 1: Fork the Repository

1. Visit https://github.com/tzachbon/smart-ralph
2. Click the "Fork" button in the top-right corner
3. Choose your account as the destination
4. Wait for the fork to complete

## Step 2: Understand the Marketplace Structure

Smart Ralph uses Claude Code's marketplace system. The marketplace configuration is in `.claude-plugin/marketplace.json`:

```json
{
  "name": "smart-ralph",
  "owner": {
    "name": "your-username"
  },
  "metadata": {
    "description": "Spec-driven development with task-by-task execution"
  },
  "plugins": [
    {
      "name": "ralph-specum",
      "description": "Spec-driven development with research, requirements, design, tasks, and autonomous execution. Fresh context per task.",
      "version": "3.2.0",
      "author": {
        "name": "your-username"
      },
      "source": "./plugins/ralph-specum",
      "category": "development",
      "tags": ["ralph", "spec-driven", "autonomous", "research", "tasks"]
    }
  ]
}
```

### Key Fields to Update

- `name`: Marketplace identifier (keep as `smart-ralph` or customize)
- `owner.name`: Your GitHub username or team name
- `plugins[].version`: Update when you make changes
- `plugins[].author.name`: Your username or team name

## Step 3: Configure Your Fork's Marketplace

### Option A: Keep marketplace.json Simple (Recommended)

If you only want to use your fork without customization, you don't need to modify `marketplace.json` at all. Claude Code will use the marketplace as-is.

### Option B: Customize Marketplace Metadata

Update `.claude-plugin/marketplace.json` to reflect your fork:

```json
{
  "name": "smart-ralph",
  "owner": {
    "name": "bleedingpixels"
  },
  "metadata": {
    "description": "Customized Smart Ralph for bleedingpixels team"
  },
  "plugins": [
    {
      "name": "ralph-specum",
      "description": "Customized version with team-specific workflows",
      "version": "3.2.0-bleedingpixels.1",
      "author": {
        "name": "bleedingpixels"
      },
      "source": "./plugins/ralph-specum",
      "category": "development",
      "tags": ["ralph", "spec-driven", "autonomous", "research", "tasks", "custom"]
    }
  ]
}
```

## Step 4: Add Your Fork to Claude Code

Once you've configured your marketplace, add it to Claude Code:

```bash
# Add your fork as a marketplace (replace with your username)
/plugin marketplace add bleedingpixels/smart-ralph

# Install the plugin from your fork
/plugin install ralph-specum@smart-ralph

# Restart Claude Code
```

### Verify Installation

```bash
# List installed plugins
/plugin list

# Check marketplace sources
/plugin marketplace list
```

## Step 5: Customize Your Fork (Optional)

Now that your fork is installed, you can customize it:

### Customize Agents

Edit agent prompts in `plugins/ralph-specum/agents/`:

- `research-analyst.md` - Research phase behavior
- `product-manager.md` - Requirements generation
- `architect-reviewer.md` - Technical design
- `task-planner.md` - Task breakdown logic
- `spec-executor.md` - Task execution behavior

### Customize Templates

Modify templates in `plugins/ralph-specum/templates/`:

- Research, requirements, design, and task templates
- Add new sections or modify existing ones

### Customize Commands

Edit slash commands in `plugins/ralph-specum/commands/`:

- Modify command behavior
- Add new commands
- Change agent delegation logic

## Version Management for Forks

When you customize your fork, follow semantic versioning:

### Version Format

```
<original-version>-<fork-name>.<fork-version>
```

Examples:
- `3.2.0-bleedingpixels.1` - First customization based on v3.2.0
- `3.2.0-bleedingpixels.2` - Second customization
- `3.3.0-bleedingpixels.1` - Customization based on v3.3.0

### When to Bump Versions

Update the version in BOTH files when making changes:

1. **`.claude-plugin/marketplace.json`** - Update the plugin version
2. **`plugins/ralph-specum/.claude-plugin/plugin.json`** - Update the plugin version

**Version bump types:**
- **Patch** (x.x.X): Bug fixes, small tweaks
- **Minor** (x.X.x): New features, agent improvements
- **Major** (X.x.x): Breaking changes, major rewrites

### Example: Bumping Version After Customization

```bash
# 1. Make your changes
# Edit agents/research-analyst.md

# 2. Update versions in both files
# .claude-plugin/marketplace.json: "version": "3.2.0-bleedingpixels.1"
# plugins/ralph-specum/.claude-plugin/plugin.json: "version": "3.2.0-bleedingpixels.1"

# 3. Commit and push
git add .
git commit -m "feat: customize research analyst for team workflow"
git push origin main

# 4. Update in Claude Code
/plugin marketplace update smart-ralph
```

## Step 6: Submitting Updates Upstream

If you've made improvements that could benefit the broader community, consider contributing them back!

### Contributing Guidelines

1. **Keep changes general**: Avoid team-specific customizations
2. **Test thoroughly**: Ensure changes work in various contexts
3. **Document changes**: Update README and agent descriptions
4. **Follow conventions**: Match the existing code style

### Pull Request Workflow

```bash
# 1. Ensure your fork is up to date
git remote add upstream https://github.com/tzachbon/smart-ralph
git fetch upstream
git rebase upstream/main

# 2. Create a feature branch
git checkout -b feature/my-improvement

# 3. Make and commit your changes
git add .
git commit -m "feat: add improvement to research phase"

# 4. Push to your fork
git push origin feature/my-improvement

# 5. Open a pull request on GitHub
# Visit: https://github.com/tzachbon/smart-ralph/compare
```

### What to Contribute

Good contributions include:
- Bug fixes
- Performance improvements
- Documentation enhancements
- General feature additions
- Agent prompt improvements (when broadly applicable)

### What to Keep in Your Fork

Keep these in your fork:
- Team-specific workflows
- Private configurations
- Experimental features
- Custom branding

## Troubleshooting

### Marketplace Not Loading

**Problem**: Can't add your fork as a marketplace

**Solutions**:
- Verify your fork is public (or set up authentication for private repos)
- Check that `.claude-plugin/marketplace.json` exists in your fork
- Validate JSON syntax: `claude plugin validate .`

### Version Conflicts

**Problem**: Claude Code installs an older version

**Solutions**:
- Update both `marketplace.json` and `plugin.json` with the same version
- Run `/plugin marketplace update smart-ralph`
- Uninstall and reinstall: `/plugin uninstall ralph-specum@smart-ralph`

### Customizations Not Applied

**Problem**: Changes to agents aren't showing up

**Solutions**:
- Verify you edited files in `plugins/ralph-specum/` (not in `.claude` cache)
- Reinstall the plugin: `/plugin install ralph-specum@smart-ralph --force`
- Restart Claude Code completely

### Private Repository Authentication

For private forks, set up authentication:

```bash
# Set GitHub token (for auto-updates)
export GITHUB_TOKEN=ghp_xxxxxxxxxxxxxxxxxxxx

# Add to shell config for persistence
echo 'export GITHUB_TOKEN=ghp_xxxxxxxxxxxxxxxxxxxx' >> ~/.bashrc
source ~/.bashrc
```

See the [official marketplace documentation](https://code.claude.com/docs/en/plugin-marketplaces#private-repositories) for details.

## Advanced: Multiple Marketplaces

You can maintain multiple marketplaces for different purposes:

```bash
# Official upstream (stable)
/plugin marketplace add tzachbon/smart-ralph

# Your fork (development/custom)
/plugin marketplace add bleedingpixels/smart-ralph

# Install from your fork
/plugin install ralph-specum@smart-ralph
```

When multiple marketplaces have the same plugin name, Claude Code uses the most recently added one. To use a specific marketplace:

```bash
# Remove and re-add to switch
/plugin marketplace remove smart-ralph
/plugin marketplace add tzachbon/smart-ralph
```

## Resources

- [Claude Code Marketplace Documentation](https://code.claude.com/docs/en/plugin-marketplaces)
- [Smart Ralph Documentation](README.md)
- [Contributing Guidelines](CONTRIBUTING.md)
- [Troubleshooting Guide](TROUBLESHOOTING.md)

## Support

- **Issues**: https://github.com/tzachbon/smart-ralph/issues
- **Discussions**: https://github.com/tzachbon/smart-ralph/discussions
- **Claude Code Docs**: https://code.claude.com/docs

---

**Happy forking!** Remember to keep your fork updated with upstream changes and contribute back improvements that benefit the community.
