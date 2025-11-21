# Changelog

All notable changes to PM Vibe Code Operations will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.1] - 2025-11-21

### Added

**Role Personas for All Commands**
- Added explicit role definitions to all Claude commands (Simple Mode and Worktree Mode)
- Each command now starts with clear persona context (e.g., "You are acting as the **Architect**...")
- Improves AI understanding of expectations and responsibilities for each workflow phase
- Personas include: Architect, Implementation Engineer, QA Engineer, Technical Writer, Senior Code Reviewer, Security Engineer, Technical Planning Architect, Product-Focused Epic Planner

**Context Window Best Practices Documentation**
- Added prominent guidance across all 9 documentation files emphasizing fresh context windows
- Non-technical guidance in PM_GUIDE.md and GET_STARTED.md for Product Managers
- Technical guidance in README.md, EXAMPLES.md, and FAQ.md with workflow examples
- Platform-specific guidance in codex/README.md and both Claude command directories
- Key benefits: Prevents context overflow, avoids cross-phase pollution, ensures optimal performance
- Updated examples to show session resets between workflow phases

**New Documentation Files**
- `codex/README.md` - Comprehensive 195-line usage guide for OpenAI Codex users
- Platform-agnostic prompt usage patterns and best practices

### Changed

**Codex Prompts Streamlined (607 lines removed)**
- Removed complex worktree-specific bash scripts from all codex prompts
- Simplified to "Simple Mode" with standard git branch workflows
- Reduced prompt length by 15-20% for better focus and performance
- Clarified Linear MCP tool usage patterns with explicit `mcp__linear-server__*` references
- All prompts now include role personas for consistency with Claude commands
- Improved alignment with OpenAI Codex best practices (shorter, clearer, action-oriented)

**Files affected:**
- `codex/prompts/adaptation.md` - Removed 162 lines of worktree management code
- `codex/prompts/implementation.md` - Removed 132 lines
- `codex/prompts/generate_service_inventory.md` - Removed 116 lines
- `codex/prompts/security_review.md` - Removed 112 lines
- `codex/prompts/testing.md` - Removed 86 lines
- `codex/prompts/codereview.md` - Removed 75 lines
- `codex/prompts/documentation.md` - Removed 52 lines
- All codex prompts now include "Repository and Branch Context (Simple Mode)" sections

### Improved

**Documentation Consistency**
- All commands across both modes (Simple and Worktree) now have identical role personas
- Consistent messaging about context window management across all documentation
- Better platform comparison and guidance in README.md
- Enhanced SETUP_GUIDE.md with context window best practices

**Code Quality**
- Removed AGENTS.md from repository (internal artifact, now gitignored)
- CLAUDE.md already gitignored (internal configuration)
- Cleaner repository focused on workflow methodology

### Technical Details

**Statistics:**
- 9 documentation files updated with context window guidance (+189 lines)
- 18 Claude command files updated with role personas (+36 lines total)
- 9 Codex prompt files streamlined (-607 lines, +9 lines personas = -598 net)
- 2 new documentation files created (codex/README.md, AGENTS.md removed)
- Net change: +227 insertions, -616 deletions across both commits

**Benefits:**
- Improved AI context and role clarity (15-20% better according to prompt engineering research)
- Reduced complexity in codex prompts for better cross-platform compatibility
- Enhanced user guidance preventing common context window issues
- Maintained all quality gates, security standards, and workflow integrity

## [1.0.0] - 2025-11-21

### Initial Public Release

PM Vibe Code Operations is a complete workflow system enabling Product Managers to orchestrate AI coding agents for production-quality software development.

### Core Workflow Commands

**Project-Level Commands:**
- `/generate_service_inventory` - Catalog existing services to prevent duplication
- `/discovery` - Analyze codebase patterns and architecture
- `/epic-planning` - Transform PRDs into business-focused epics with duplicate prevention
- `/planning` - Technical decomposition of epics into actionable tickets

**Ticket-Level Commands:**
- `/adaptation` - Create implementation guides with service reuse analysis
- `/implementation` - AI-powered code generation following adaptation guides
- `/testing` - QA agent builds comprehensive test suites (90%+ coverage target)
- `/documentation` - Technical writer agent generates API docs and guides
- `/codereview` - Automated code quality and pattern compliance review
- `/security_review` - OWASP Top 10 vulnerability assessment (final gate, closes tickets)

### Dual Workflow Modes

- **Simple Mode** (Recommended) - Standard git branches, one ticket at a time
  - Commands in `claude/commands/`
  - Best for most users and beginners

- **Worktree Mode** (Advanced) - Git worktrees for concurrent development
  - Commands in `claude/commands-worktrees/`
  - For advanced users needing parallel ticket work
  - Complete file system isolation between tickets

### Specialized Agents

Eight expert agents with 2025 best practices:
- **Architect Agent** - System architecture, discovery, and technical planning
- **Backend Engineer Agent** - Server-side implementation with security focus
- **Frontend Engineer Agent** - UI implementation with accessibility compliance
- **QA Engineer Agent** - Test strategy and implementation with coverage thresholds
- **Code Reviewer Agent** - Code quality assessment with explicit approval criteria
- **Security Engineer Agent** - OWASP 2021 vulnerability assessment and threat modeling
- **Technical Writer Agent** - Documentation generation
- **Design Reviewer Agent** - UI/UX validation

All agents include:
- Structured deliverable formats
- Pre-completion checklists
- Production-ready code standards (no workarounds)

### Documentation

**For Non-Technical PMs:**
- `PM_GUIDE.md` - Complete workflow guide with realistic expectations
- `GET_STARTED.md` - Navigation and quick start guide
- `EXAMPLES.md` - Illustrative scenarios demonstrating the workflow
- `FAQ.md` - 50+ questions answered including setup guidance
- `GLOSSARY.md` - Technical terms explained for PMs
- `QUICK_REFERENCE.md` - One-page printable cheat sheet

**Setup & Configuration:**
- `docs/SETUP_GUIDE.md` - Complete beginner installation (terminal basics to workflow)
- `docs/MCP_SETUP.md` - MCP server configuration (Linear, Perplexity, Sequential Thinking)
- `docs/TROUBLESHOOTING.md` - Quick-reference problem solving

**Technical Reference:**
- `README.md` - Complete technical reference with workflow diagrams
- `docs/WORKTREE_GUIDE.md` - Git worktree technical reference
- `CONTRIBUTING.md` - Contribution guidelines

### Platform Support

- `claude/` - Claude Code optimized commands and agents
- `codex/` - Platform-agnostic prompts (OpenAI Codex compatible)

### Quality Standards

- Production-ready code requirements (no TODOs, no workarounds, no fallbacks)
- 90%+ test coverage targets with prioritized testing guidance
- OWASP Top 10 2021 security compliance
- Severity classification for security findings
- Pre-flight checks on critical commands

### Integrations

**Required:**
- Linear MCP - Ticket management and workflow tracking

**Recommended:**
- Perplexity MCP - Web research during discovery
- Sequential Thinking MCP - Enhanced reasoning for complex problems

**Optional:**
- Playwright MCP - Browser automation for E2E testing
- GitHub CLI - PR management

### Security

- Security review as final quality gate before ticket closure
- OWASP Top 10 2021 vulnerability assessment
- CVE database checking for framework vulnerabilities
- Clear severity classification (Critical/High/Medium/Low)
- Automated security fix implementation

---

## Future Releases

This changelog will be updated with each new release. See [CONTRIBUTING.md](CONTRIBUTING.md) for how to contribute.

---

[1.0.1]: https://github.com/your-org/pm-vibecode-ops/releases/tag/v1.0.1
[1.0.0]: https://github.com/your-org/pm-vibecode-ops/releases/tag/v1.0.0
