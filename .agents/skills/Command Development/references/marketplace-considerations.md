# Marketplace Considerations for Commands

Guidelines for creating commands designed for distribution and marketplace success.

## Overview

Commands distributed through marketplaces need additional consideration beyond personal use commands. They must work across environments, handle diverse use cases, and provide excellent user experience for unknown users.

## Design for Distribution

### Universal Compatibility

**Cross-platform considerations:**

```markdown
---
description: Cross-platform command
allowed-tools: Bash(*)
---

# Platform-Aware Command

Detecting platform...

case "$(uname)" in
  Darwin*)  PLATFORM="macOS" ;;
  Linux*)   PLATFORM="Linux" ;;
  MINGW*|MSYS*|CYGWIN*) PLATFORM="Windows" ;;
  *)        PLATFORM="Unknown" ;;
esac

Platform: $PLATFORM

<!-- Adjust behavior based on platform -->
if [ "$PLATFORM" = "Windows" ]; then
  # Windows-specific handling
  PATH_SEP="\\"
  NULL_DEVICE="NUL"
else
  # Unix-like handling
  PATH_SEP="/"
  NULL_DEVICE="/dev/null"
fi

[Platform-appropriate implementation...]
```

**Avoid platform-specific commands:**

```markdown
<!-- BAD: macOS-specific -->
!`pbcopy < file.txt`

<!-- GOOD: Platform detection -->
if command -v pbcopy > /dev/null; then
  pbcopy < file.txt
elif command -v xclip > /dev/null; then
  xclip -selection clipboard < file.txt
elif command -v clip.exe > /dev/null; then
  cat file.txt | clip.exe
else
  echo "Clipboard not available on this platform"
fi
```

### Minimal Dependencies

**Check for required tools:**

```markdown
---
description: Dependency-aware command
allowed-tools: Bash(*)
---

# Check Dependencies

Required tools:
- git
- jq
- node

Checking availability...

MISSING_DEPS=""

for tool in git jq node; do
  if ! command -v $tool > /dev/null; then
    MISSING_DEPS="$MISSING_DEPS $tool"
  fi
done

if [ -n "$MISSING_DEPS" ]; then
  ❌ ERROR: Missing required dependencies:$MISSING_DEPS

  INSTALLATION:
  - git: https://git-scm.com/downloads
  - jq: https://stedolan.github.io/jq/download/
  - node: https://nodejs.org/

  Install missing tools and try again.

  Exit.
fi

✓ All dependencies available

[Continue with command...]
```

**Document optional dependencies:**

```markdown
<!--
DEPENDENCIES:
  Required:
  - git 2.0+: Version control
  - jq 1.6+: JSON processing

  Optional:
  - gh: GitHub CLI (for PR operations)
  - docker: Container operations (for containerized tests)

  Feature availability depends on installed tools.
-->
```

### Graceful Degradation

**Handle missing features:**

```markdown
---
description: Feature-aware command
---

# Feature Detection

Detecting available features...

FEATURES=""

if command -v gh > /dev/null; then
  FEATURES="$FEATURES github"
fi

if command -v docker > /dev/null; then
  FEATURES="$FEATURES docker"
fi

Available features: $FEATURES

if echo "$FEATURES" | grep -q "github"; then
  # Full functionality with GitHub integration
  echo "✓ GitHub integration available"
else
  # Reduced functionality without GitHub
  echo "⚠ Limited functionality: GitHub CLI not installed"
  echo "  Install 'gh' for full features"
fi

[Adapt behavior based on available features...]
```

## User Experience for Unknown Users

### Clear Onboarding

**First-run experience:**

```markdown
---
description: Command with onboarding
allowed-tools: Read, Write
---

# First Run Check

if [ ! -f ".claude/command-initialized" ]; then
  **Welcome to Command Name!**

  This appears to be your first time using this command.

  WHAT THIS COMMAND DOES:
  [Brief explanation of purpose and benefits]

  QUICK START:
  1. Basic usage: /command [arg]
  2. For help: /command help
  3. Examples: /command examples

  SETUP:
  No additional setup required. You're ready to go!

  ✓ Initialization complete

  [Create initialization marker]

  Ready to proceed with your request...
fi

[Normal command execution...]
```

**Progressive feature discovery:**

```markdown
---
description: Command with tips
---

# Command Execution

[Main functionality...]

---

💡 TIP: Did you know?

You can speed up this command with the --fast flag:
  /command --fast [args]

For more tips: /command tips
```

### Comprehensive Error Handling

**Anticipate user mistakes:**

```markdown
---
description: Forgiving command
---

# User Input Handling

Argument: "$1"

<!-- Check for common typos -->
if [ "$1" = "hlep" ] || [ "$1" = "hepl" ]; then
  Did you mean: help?

  Showing help instead...
  [Display help]

  Exit.
fi

<!-- Suggest similar commands if not found -->
if [ "$1" != "valid-option1" ] && [ "$1" != "valid-option2" ]; then
  ❌ Unknown option: $1

  Did you mean:
  - valid-option1 (most similar)
  - valid-option2

  For all options: /command help

  Exit.
fi

[Command continues...]
```

**Helpful diagnostics:**

```markdown
---
description: Diagnostic command
---

# Operation Failed

The operation could not complete.

**Diagnostic Information:**

Environment:
- Platform: $(uname)
- Shell: $SHELL
- Working directory: $(pwd)
- Command: /command $@

Checking common issues:
- Git repository: $(git rev-parse --git-dir 2>&1)
- Write permissions: $(test -w . && echo "OK" || echo "DENIED")
- Required files: $(test -f config.yml && echo "Found" || echo "Missing")

This information helps debug the issue.

For support, include the above diagnostics.
```

## Distribution Best Practices

### Namespace Awareness

**Avoid name collisions:**

```markdown
---
description: Namespaced command
---

<!--
COMMAND NAME: plugin-name-command

This command is namespaced with the plugin name to avoid
conflicts with commands from other plugins.

Alternative naming approaches:
- Use plugin prefix: /plugin-command
- Use category: /category-command
- Use verb-noun: /verb-noun

Chosen approach: plugin-name prefix
Reasoning: Clearest ownership, least likely to conflict
-->

# Plugin Name Command

[Implementation...]
```

**Document naming rationale:**

```markdown
<!--
NAMING DECISION:

Command name: /deploy-app

Alternatives considered:
- /deploy: Too generic, likely conflicts
- /app-deploy: Less intuitive ordering
- /my-plugin-deploy: Too verbose

Final choice balances:
- Discoverability (clear purpose)
- Brevity (easy to type)
- Uniqueness (unlikely conflicts)
-->
```

### Configurability

**User preferences:**

```markdown
---
description: Configurable command
allowed-tools: Read
---

# Load User Configuration

Default configuration:
- verbose: false
- color: true
- max_results: 10

Checking for user config: .claude/plugin-name.local.md

if [ -f ".claude/plugin-name.local.md" ]; then
  # Parse YAML frontmatter for settings
  VERBOSE=$(grep "^verbose:" .claude/plugin-name.local.md | cut -d: -f2 | tr -d ' ')
  COLOR=$(grep "^color:" .claude/plugin-name.local.md | cut -d: -f2 | tr -d ' ')
  MAX_RESULTS=$(grep "^max_results:" .claude/plugin-name.local.md | cut -d: -f2 | tr -d ' ')

  echo "✓ Using user configuration"
else
  echo "Using default configuration"
  echo "Create .claude/plugin-name.local.md to customize"
fi

[Use configuration in command...]
```

**Sensible defaults:**

```markdown
---
description: Command with smart defaults
---

# Smart Defaults

Configuration:
- Format: ${FORMAT:-json}  # Defaults to json
- Output: ${OUTPUT:-stdout}  # Defaults to stdout
- Verbose: ${VERBOSE:-false}  # Defaults to false

These defaults work for 80% of use cases.

Override with arguments:
  /command --format yaml --output file.txt --verbose

Or set in .claude/plugin-name.local.md:
\`\`\`yaml
---
format: yaml
output: custom.txt
verbose: true
---
\`\`\`
```

### Version Compatibility

**Version checking:**

```markdown
---
description: Version-aware command
---

<!--
COMMAND VERSION: 2.1.0

COMPATIBILITY:
- Requires plugin version: >= 2.0.0
- Breaking changes from v1.x documented in MIGRATION.md

VERSION HISTORY:
- v2.1.0: Added --new-feature flag
- v2.0.0: BREAKING: Changed argument order
- v1.0.0: Initial release
-->

# Version Check

Command version: 2.1.0
Plugin version: [detect from plugin.json]

if [  "$PLUGIN_VERSION" < "2.0.0" ]; then
  ❌ ERROR: Incompatible plugin version

  This command requires plugin version >= 2.0.0
  Current version: $PLUGIN_VERSION

  Update plugin:
    /plugin update plugin-name

  Exit.
fi

✓ Version compatible

[Command continues...]
```

**Deprecation warnings:**

```markdown
---
description: Command with deprecation warnings
---

# Deprecation Check

if [ "$1" = "--old-flag" ]; then
  ⚠️  DEPRECATION WARNING

  The --old-flag option is deprecated as of v2.0.0
  It will be removed in v3.0.0 (est. June 2025)

  Use instead: --new-flag

  Example:
    Old: /command --old-flag value
    New: /command --new-flag value

  See migration guide: /command migrate

  Continuing with deprecated behavior for now...
fi

[Handle both old and new flags during deprecation period...]
```

## Marketplace Presentation

### Command Discovery

**Descriptive naming:**

```markdown
---
description: Review pull request with security and quality checks
---

<!-- GOOD: Descriptive name and description -->
```

```markdown
---
description: Do the thing
---

<!-- BAD: Vague description -->
```

**Searchable keywords:**

```markdown
<!--
KEYWORDS: security, code-review, quality, validation, audit

These keywords help users discover this command when searching
for related functionality in the marketplace.
-->
```

### Showcase Examples

**Compelling demonstrations:**

```markdown
---
description: Advanced code analysis command
---

# Code Analysis Command

This command performs deep code analysis with actionable insights.

## Demo: Quick Security Audit

Try it now:
\`\`\`
/analyze-code src/ --security
\`\`\`

**What you'll get:**
- Security vulnerability detection
- Code quality metrics
- Performance bottleneck identification
- Actionable recommendations

**Sample output:**
\`\`\`
Security Analysis Results
=========================

🔴 Critical (2):
  - SQL injection risk in users.js:45
  - XSS vulnerability in display.js:23

🟡 Warnings (5):
  - Unvalidated input in api.js:67
  ...

Recommendations:
1. Fix critical issues immediately
2. Review warnings before next release
3. Run /analyze-code --fix for auto-fixes
\`\`\`

---

Ready to analyze your code...

[Command implementation...]
```

### User Reviews and Feedback

**Feedback mechanism:**

```markdown
---
description: Command with feedback
---

# Command Complete

[Command results...]

---

**How was your experience?**

This helps improve the command for everyone.

Rate this command:
- 👍 Helpful
- 👎 Not helpful
- 🐛 Found a bug
- 💡 Have a suggestion

Reply with an emoji or:
- /command feedback

Your feedback matters!
```

**Usage analytics preparation:**

```markdown
<!--
ANALYTICS NOTES:

Track for improvement:
- Most common arguments
- Failure rates
- Average execution time
- User satisfaction scores

Privacy-preserving:
- No personally identifiable information
- Aggregate statistics only
- User opt-out respected
-->
```

## Quality Standards

### Professional Polish

**Consistent branding:**

```markdown
---
description: Branded command
---

# ✨ Command Name

Part of the [Plugin Name] suite

[Command functionality...]

---

**Need Help?**
- Documentation: https://docs.example.com
- Support: support@example.com
- Community: https://community.example.com

Powered by Plugin Name v2.1.0
```

**Attention to detail:**

```markdown
<!-- Details that matter -->

✓ Use proper emoji/symbols consistently
✓ Align output columns neatly
✓ Format numbers with thousands separators
✓ Use color/formatting appropriately
✓ Provide progress indicators
✓ Show estimated time remaining
✓ Confirm successful operations
```

### Reliability

**Idempotency:**

```markdown
---
description: Idempotent command
---

# Safe Repeated Execution

Checking if operation already completed...

if [ -f ".claude/operation-completed.flag" ]; then
  ℹ️  Operation already completed

  Completed at: $(cat .claude/operation-completed.flag)

  To re-run:
  1. Remove flag: rm .claude/operation-completed.flag
  2. Run command again

  Otherwise, no action needed.

  Exit.
fi

Performing operation...

[Safe, repeatable operation...]

Marking complete...
echo "$(date)" > .claude/operation-completed.flag
```

**Atomic operations:**

```markdown
---
description: Atomic command
---

# Atomic Operation

This operation is atomic - either fully succeeds or fully fails.

Creating temporary workspace...
TEMP_DIR=$(mktemp -d)

Performing changes in isolated environment...
[Make changes in $TEMP_DIR]

if [ $? -eq 0 ]; then
  ✓ Changes validated

  Applying changes atomically...
  mv $TEMP_DIR/* ./target/

  ✓ Operation complete
else
  ❌ Changes failed validation

  Rolling back...
  rm -rf $TEMP_DIR

  No changes applied. Safe to retry.
fi
```

## Testing for Distribution

### Pre-Release Checklist

```markdown
<!--
PRE-RELEASE CHECKLIST:

Functionality:
- [ ] Works on macOS
- [ ] Works on Linux
- [ ] Works on Windows (WSL)
- [ ] All arguments tested
- [ ] Error cases handled
- [ ] Edge cases covered

User Experience:
- [ ] Clear description
- [ ] Helpful error messages
- [ ] Examples provided
- [ ] First-run experience good
- [ ] Documentation complete

Distribution:
- [ ] No hardcoded paths
- [ ] Dependencies documented
- [ ] Configuration options clear
- [ ] Version number set
- [ ] Changelog updated

Quality:
- [ ] No TODO comments
- [ ] No debug code
- [ ] Performance acceptable
- [ ] Security reviewed
- [ ] Privacy considered

Support:
- [ ] README complete
- [ ] Troubleshooting guide
- [ ] Support contact provided
- [ ] Feedback mechanism
- [ ] License specified
-->
```

### Beta Testing

**Beta release approach:**

```markdown
---
description: Beta command (v0.9.0)
---

# 🧪 Beta Command

**This is a beta release**

Features may change based on feedback.

BETA STATUS:
- Version: 0.9.0
- Stability: Experimental
- Support: Limited
- Feedback: Encouraged

Known limitations:
- Performance not optimized
- Some edge cases not handled
- Documentation incomplete

Help improve this command:
- Report issues: /command report-issue
- Suggest features: /command suggest
- Join beta testers: /command join-beta

---

[Command implementation...]

---

**Thank you for beta testing!**

Your feedback helps make this command better.
```

## Maintenance and Updates

### Update Strategy

**Versioned commands:**

```markdown
<!--
VERSION STRATEGY:

Major (X.0.0): Breaking changes
- Document all breaking changes
- Provide migration guide
- Support old version briefly

Minor (x.Y.0): New features
- Backward compatible
- Announce new features
- Update examples

Patch (x.y.Z): Bug fixes
- No user-facing changes
- Update changelog
- Security fixes prioritized

Release schedule:
- Patches: As needed
- Minors: Monthly
- Majors: Annually or as needed
-->
```

**Update notifications:**

```markdown
---
description: Update-aware command
---

# Check for Updates

Current version: 2.1.0
Latest version: [check if available]

if [ "$CURRENT_VERSION" != "$LATEST_VERSION" ]; then
  📢 UPDATE AVAILABLE

  New version: $LATEST_VERSION
  Current: $CURRENT_VERSION

  What's new:
  - Feature improvements
  - Bug fixes
  - Performance enhancements

  Update with:
    /plugin update plugin-name

  Release notes: https://releases.example.com/v$LATEST_VERSION
fi

[Command continues...]
```

## Best Practices Summary

### Distribution Design

1. **Universal**: Works across platforms and environments
2. **Self-contained**: Minimal dependencies, clear requirements
3. **Graceful**: Degrades gracefully when features unavailable
4. **Forgiving**: Anticipates and handles user mistakes
5. **Helpful**: Clear errors, good defaults, excellent docs

### Marketplace Success

1. **Discoverable**: Clear name, good description, searchable keywords
2. **Professional**: Polished presentation, consistent branding
3. **Reliable**: Tested thoroughly, handles edge cases
4. **Maintainable**: Versioned, updated regularly, supported
5. **User-focused**: Great UX, responsive to feedback

### Quality Standards

1. **Complete**: Fully documented, all features working
2. **Tested**: Works in real environments, edge cases handled
3. **Secure**: No vulnerabilities, safe operations
4. **Performant**: Reasonable speed, resource-efficient
5. **Ethical**: Privacy-respecting, user consent

With these considerations, commands become marketplace-ready and delight users across diverse environments and use cases.

## Forking for Distribution

### When to Fork vs Contribute Upstream

**Fork when:**
- You need custom features not aligned with upstream goals
- Upstream is unmaintained or inactive
- You require different release cadence or priorities
- You want to experiment with significant changes
- You need to maintain a stable version while upstream evolves rapidly
- You're creating a specialized variant for a specific use case

**Contribute upstream when:**
- Features benefit the broader community
- Bug fixes or improvements applicable to all users
- Documentation enhancements
- Performance optimizations
- Security patches

**Decision framework:**

```markdown
<!--
FORK DECISION CHECKLIST:

Before forking, consider:
- [ ] Have I discussed the feature with upstream maintainers?
- [ ] Is this change specific to my use case or generally useful?
- [ ] Am I willing to maintain the fork long-term?
- [ ] Have I checked if upstream is accepting contributions?
- [ ] Can the functionality be achieved through configuration instead?

If yes to most above, contribute upstream.
If no to most above, forking may be appropriate.
-->
```

### Configuring marketplace.json for Forks

When forking a plugin, update `marketplace.json` to reflect your ownership while maintaining traceability to the original:

```json
{
  "plugins": [
    {
      "name": "your-fork-plugin-name",
      "version": "1.0.0",
      "owner": {
        "name": "your-username",
        "type": "user"
      },
      "source": "https://github.com/your-username/your-fork-repo",
      "description": "Your fork description - mention original plugin",
      "fork": {
        "original": "original-plugin-name",
        "original_owner": "original-owner-name",
        "original_source": "https://github.com/original-owner/original-repo"
      },
      "tags": ["fork", "custom-feature"],
      "readme": "https://github.com/your-username/your-fork-repo/blob/main/README.md"
    }
  ]
}
```

**Key modifications for forks:**

1. **Update owner.name**: Change to your GitHub username
2. **Update source**: Point to your fork's repository
3. **Add fork metadata**: Include `fork` object with original plugin info
4. **Update description**: Clearly indicate this is a fork and why
5. **Add fork tag**: Helps users identify forked plugins

**Example before and after:**

```json
// Original marketplace.json
{
  "plugins": [{
    "name": "ralph-specum",
    "version": "3.0.0",
    "owner": {
      "name": "tzachbon",
      "type": "user"
    },
    "source": "https://github.com/tzachbon/smart-ralph"
  }]
}

// Forked marketplace.json
{
  "plugins": [{
    "name": "ralph-specum-custom",
    "version": "1.0.0",
    "owner": {
      "name": "your-username",
      "type": "user"
    },
    "source": "https://github.com/your-username/smart-ralph-fork",
    "description": "Custom fork of ralph-specum with additional features",
    "fork": {
      "original": "ralph-specum",
      "original_owner": "tzachbon",
      "original_source": "https://github.com/tzachbon/smart-ralph"
    }
  }]
}
```

### Version Naming Conventions for Forks

**Recommended approaches:**

1. **Independent versioning**: Reset to 1.0.0 and maintain your own version sequence
   ```json
   {
     "name": "plugin-fork",
     "version": "1.0.0"
   }
   ```

2. **Owner prefix**: Add your username to the plugin name
   ```json
   {
     "name": "username-plugin-name",
     "version": "1.0.0"
   }
   ```

3. **Variant suffix**: Add a descriptive suffix to the original name
   ```json
   {
     "name": "plugin-name-enhanced",
     "version": "1.0.0"
   }
   ```

4. **Downstream version reference**: Track upstream version in description
   ```json
   {
     "name": "plugin-name",
     "version": "1.2.0",
     "description": "Fork based on upstream v3.0.0",
     "upstream_version": "3.0.0"
   }
   ```

**Best practices:**

- Always use semantic versioning (MAJOR.MINOR.PATCH)
- Document version correspondence with upstream
- Include changelog with fork-specific changes
- Avoid using original version numbers to prevent confusion

**Version compatibility tracking:**

```markdown
<!--
VERSION TRACKING:

Fork version: 1.2.0
Based on upstream: 3.0.0
Divergence date: 2025-01-15

Fork-specific changes:
- v1.2.0: Added custom feature X
- v1.1.0: Modified behavior of Y
- v1.0.0: Initial fork from upstream v3.0.0

Upstream changes not yet merged:
- [ ] Upstream PR #123
- [ ] Custom feature X (submitted for review)
-->
```

### How Users Discover and Install Forked Plugins

**Discovery methods:**

1. **Marketplace search**: Users can search for your fork by name or tags
   ```
   Search: "username-plugin-name" or "plugin fork custom"
   ```

2. **Direct installation**: Users install using your fork's source URL
   ```bash
   /plugin install https://github.com/your-username/your-fork-repo
   ```

3. **README links**: Document your fork in the original plugin's discussions
   ```markdown
   ## Community Forks

   - [Your Fork](https://github.com/your-username/fork) - Description of custom features
   ```

4. **Social proof**: Encourage users who benefit from your fork to star it

**Installation instructions for fork README:**

```markdown
## Installation

This is a fork of [original-plugin](https://github.com/original-owner/original-repo).

### Install via Claude Code Marketplace

\`\`\`bash
/plugin install your-username/your-fork-name
\`\`\`

### Install Directly from GitHub

\`\`\`bash
/plugin install https://github.com/your-username/your-fork-repo
\`\`\`

### Why This Fork?

This fork includes:
- Custom feature X not in upstream
- Modified behavior of Y for specific use case
- Additional configuration options
- Bug fixes not yet merged upstream

See [FORK_CHANGES.md](FORK_CHANGES.md) for complete details.
```

**Upgrade path for users:**

```markdown
## Upgrading from Original Plugin

If you're already using the original plugin:

1. Uninstall original: `/plugin uninstall original-plugin`
2. Install fork: `/plugin install your-username/your-fork-name`
3. Migrate configuration (if needed)

**Note**: Configuration files are typically compatible, but check [MIGRATION.md](MIGRATION.md) for any specific changes.
```

### Maintenance Responsibilities for Fork Owners

**Essential maintenance tasks:**

1. **Track upstream changes**
   ```bash
   # Add original repo as upstream remote
   git remote add upstream https://github.com/original-owner/original-repo

   # Fetch upstream changes regularly
   git fetch upstream

   # Review upstream changes
   git log main..upstream/main
   ```

2. **Security updates**
   - Monitor upstream for security patches
   - Promptly merge critical fixes
   - Test and release updates quickly

3. **Bug fixes**
   - Fix fork-specific bugs
   - Consider contributing fixes upstream
   - Document known issues

4. **Feature management**
   - Clearly document fork-specific features
   - Avoid unnecessary divergence
   - Submit valuable features upstream

5. **User support**
   - Respond to issues in your fork
   - Provide upgrade guidance
   - Document breaking changes

6. **Version management**
   ```json
   {
     "version": "1.2.0",
     "upstream_version": "3.0.0",
     "last_sync": "2025-01-15",
     "pending_upstream_changes": 5
   }
   ```

**Maintenance checklist:**

```markdown
<!--
FORK MAINTENANCE CHECKLIST:

Weekly:
- [ ] Check upstream for new releases
- [ ] Review upstream issues/PRs
- [ ] Respond to fork issues

Monthly:
- [ ] Test upstream changes for compatibility
- [ ] Merge upstream fixes if compatible
- [ ] Update documentation
- [ ] Review fork-specific features for upstream potential

Quarterly:
- [ ] Consider rebasing on latest upstream
- [ ] Review fork purpose and value
- [ ] Update version tracking
- [ ] Communicate with upstream maintainers

As Needed:
- [ ] Security updates (immediate)
- [ ] Critical bug fixes (prompt)
- [ ] User support requests (timely)
-->
```

**Communication with upstream:**

- Be transparent about your fork
- Credit original authors
- Share valuable improvements
- Coordinate on significant changes
- Respect original licensing

**Fork lifecycle considerations:**

```markdown
## Fork Lifecycle

**Phase 1: Creation** (Days 1-7)
- Set up fork infrastructure
- Document rationale
- Establish version baseline

**Phase 2: Active Development** (Weeks 1-12)
- Implement fork-specific features
- Build user community
- Establish maintenance routines

**Phase 3: Maintenance** (Ongoing)
- Track upstream changes
- Support users
- Consider upstream contributions

**Phase 4: Reintegration or Continuation** (Long-term)
- Evaluate if features should go upstream
- Decide whether to maintain or deprecate
- Plan user migration if sunsetting
```

**When to sunset a fork:**

- Upstream adopts your features
- Your use case is no longer relevant
- Unable to maintain adequately
- User base has migrated away

If sunsetting, provide users with:
- Advance notice (at least 3 months)
- Migration path back to upstream
- Archive of fork-specific features
- Reason for deprecation
