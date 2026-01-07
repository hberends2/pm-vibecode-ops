# Educational Walkthroughs: Using the PM Vibe Code Workflow

## About These Walkthroughs

These are **educational demonstrations** showing how to use the workflow step-by-step. They illustrate the process, commands, and expected outputs—not real-world deployments or case studies.

**What you'll learn:**
- How to progress from PRD to production
- What each command does and when to use it
- How to review outputs at each phase
- Common patterns and best practices

**These are demonstrations of:**
- Command usage and workflow sequence
- Quality gate patterns
- Decision points for PMs
- How AI agents collaborate

---

## Table of Contents

1. [Walkthrough 1: Adding a CSV Export Feature](#walkthrough-1-adding-a-csv-export-feature)
2. [Walkthrough 2: Building User Profile Customization](#walkthrough-2-building-user-profile-customization)
3. [Walkthrough 3: Implementing Real-Time Notifications](#walkthrough-3-implementing-real-time-notifications)
4. [Common Patterns Across Walkthroughs](#common-patterns-across-walkthroughs)
5. [Learning from These Examples](#learning-from-these-examples)

---

## Walkthrough 1: Adding a CSV Export Feature

### Scenario Setup

**Feature**: Data export to CSV format
**Complexity**: Low-to-medium (good first feature)
**PM Background**: Non-technical PM
**Codebase**: Existing analytics platform with reports

### The PRD

```markdown
# CSV Export Functionality - PRD

## Problem Statement

Analytics users need to export dashboard data to perform custom analysis
in Excel/Google Sheets. Currently, they screenshot dashboards or manually
copy data, which is time-consuming and error-prone.

## User Personas

**Primary**: Data Analysts at enterprise customers
- Need: Export datasets for custom analysis
- Frequency: Weekly exports
- Pain: Manual data entry is time-consuming

**Secondary**: Executives creating presentations
- Need: Clean data exports for reports
- Frequency: Monthly
- Pain: Can't verify accuracy of manual transcription

## Success Criteria

**User Metrics**:
- Users can export datasets successfully
- Export includes all visible columns with headers
- Export completes in reasonable time

**Technical Metrics**:
- 90%+ test coverage
- No critical security issues
- Performant for typical dataset sizes

## User Stories

### Story 1: Basic Export
As a data analyst, I want to export dashboard data to CSV
so that I can perform custom analysis in Excel.

**Acceptance Criteria**:
- Export button visible on all dashboard views
- CSV includes all visible columns with proper headers
- File downloads with descriptive name: dashboard-{name}-{date}.csv
- Exports respect current filters and date ranges
- Loading indicator shows progress for larger exports

### Story 2: Large Dataset Handling
As a power user, I want to export large datasets
so that I can analyze complete data sets.

**Acceptance Criteria**:
- Exports handle datasets up to 10,000 rows
- Progress indicator shows completion percentage
- Cancellable for long-running exports
- Clear messaging if dataset is too large

### Story 3: Error Handling
As a user, I want clear error messages if export fails
so that I know what went wrong and how to fix it.

**Acceptance Criteria**:
- Network errors display helpful message
- Size limit errors explain how to reduce data
- Permission errors direct user to administrator
- All errors logged for debugging

## Scope

**In Scope for V1**:
- Export visible dashboard data to CSV
- Support for datasets up to 10,000 rows
- Error handling and progress indicators
- Proper file naming

**Out of Scope** (Future):
- Export to Excel (.xlsx) format
- Scheduled/automated exports
- Export to Google Sheets directly
- Custom column selection

## Non-Functional Requirements

**Performance**:
- Generate CSV efficiently for typical datasets
- Support multiple concurrent exports
- Reasonable memory usage

**Security**:
- Respect existing data permissions
- Audit log for exports
- No data leakage across tenants

**Compatibility**:
- CSV format compatible with Excel, Google Sheets
- UTF-8 encoding for international characters
- Proper escaping of special characters
```

### Workflow Execution

**Best Practice Note**: In this walkthrough (and all workflows), run each command in a fresh Claude Code session. After completing each step, close Claude Code and open a new session for the next command. This prevents context overflow and ensures optimal performance.

#### Step 1: Service Inventory (5 minutes)

**Session 1:**
```bash
# Open Claude Code
claude

# Run service inventory
/generate-service-inventory ./src services-inventory.md

# Review output, then close Claude Code
```

**What to Look For**:
- Existing report/data services
- Authorization/permission services
- Audit logging capabilities
- File generation utilities

**Example Findings**:
```markdown
# Service Inventory

## Data Services
- ReportService: Handles data fetching and filtering
- QueryService: Database query execution
- DataTransformService: Format conversion

## Security Services
- AuthorizationService: Permission checks
- AuditLogService: Activity tracking

## Utility Services
- FileService: File generation and storage
```

**PM Decision Point**: Review what exists. In this example, most infrastructure exists—we just need CSV formatting logic.

#### Step 2: Discovery (10 minutes)

**Session 2 (fresh context):**
```bash
# Open new Claude Code session
claude

# Run discovery
/discovery csv-export-prd.md Analytics ./src "reporting, data export"

# Review findings, then close Claude Code
```

**What Discovery Finds**:
- Technology stack and frameworks
- Existing patterns for file downloads
- Similar features (e.g., PDF export)
- Architectural conventions

**Example Discovery Output** (in Linear ticket):
```markdown
## Discovery Findings - CSV Export

**Tech Stack**:
- Backend: Node.js, Express
- Database: PostgreSQL
- Frontend: React

**Relevant Patterns**:
- PDF export follows controller → service → formatter pattern
- File downloads use streaming for large files
- Permission checks at controller level

**Reuse Opportunities**:
- ReportService: Already fetches filtered data
- AuthorizationService: Handles permission checks
- AuditLogService: Can track export events

**Recommendations**:
- Follow PDF export architectural pattern
- Add CSV formatter service
- Reuse existing download mechanism
```

**PM Review**: Verify AI understands your architecture. Check that suggested patterns make sense.

#### Step 3: Epic Planning (15 minutes)

**Session 3:**
```bash
/epic-planning csv-export-prd.md DISC-001 "sales-blocker" "feature-parity"
```

**What AI Proposes**:

The AI creates a structured plan showing proposed epics before creating anything:

```markdown
## Proposed Epic Creation Plan

### Epic 1: CSV Export Core Functionality
**Business Value**: Enable data export capability
**Scope**: Basic export with proper formatting
**Success Criteria**: Users can export dashboard data to CSV

### Epic 2: Large Dataset Handling & Performance
**Business Value**: Support enterprise use cases
**Scope**: Chunked processing, progress indicators
**Success Criteria**: Handle 10K+ row datasets efficiently

### Epic 3: Error Handling & User Feedback
**Business Value**: Reduce support burden, improve UX
**Scope**: Comprehensive error handling
**Success Criteria**: Clear error messages, <5 support tickets per 1000 exports

Proceed with creating these epics? [Requires approval]
```

**PM Approval Step**:
- Review each proposed epic
- Verify business value aligns with goals
- Check success criteria are measurable
- Approve or request modifications

**Example PM Decision**:
```
✅ Approve Epic 1 (Priority 1 - core capability)
✅ Approve Epic 2 (Priority 1 - enterprise requirement)
⏸️  Defer Epic 3 (Priority 2 - can add later if needed)
```

#### Step 4: Technical Planning (20 minutes)

**Session 4:**
```bash
/planning EPIC-001,EPIC-002 --discovery DISC-001
```

**What AI Generates**:

Sub-tickets with technical breakdown:

**Epic 1: Core Functionality** (example tickets):
- Create CSV formatter utility service
- Extend ReportService with CSV export method
- Add export API endpoint
- Create export button UI component
- Integrate frontend with export API
- Add permission checks
- Implement audit logging
- Add export progress indicator

**Epic 2: Large Datasets** (example tickets):
- Implement chunked CSV generation
- Add cancellation support
- Create progress websocket
- Add pagination UX for large datasets

**PM Review Checklist**:
- ✅ All acceptance criteria from PRD covered?
- ✅ Tickets follow discovered patterns?
- ✅ Dependencies make logical sense?
- ✅ Reusing existing services where possible?

#### Step 5-9: Implementation Through Security Review

**The Ticket-Level Workflow**:

For each ticket, run these commands in sequence (fresh session for each):

```bash
# Session 5: Adaptation
/adaptation TICKET-101

# Session 6: Implementation
/implementation TICKET-101

# Session 7: Testing
/testing TICKET-101 unit,integration,e2e 90

# Session 8: Documentation
/documentation TICKET-101

# Session 9: Code Review
/codereview TICKET-101

# Session 10: Security Review (final gate)
/security-review TICKET-101
```

**What Happens at Each Phase**:

**Adaptation**: Creates implementation guide
- Analyzes service inventory for reuse
- Maps existing patterns to new feature
- Documents which services to use
- Creates detailed implementation plan

**Implementation**: AI writes code
- Follows adaptation guide
- Reuses existing services
- Creates feature branch and PR
- Commits working code

**Testing**: Builds comprehensive test suite
- Generates unit tests
- Adds integration tests
- Creates E2E tests if needed
- Runs tests, fixes failures automatically
- Achieves target coverage (90%)

**Documentation**: Generates docs
- API documentation
- Inline code comments
- User guides
- README updates

**Code Review**: Quality checks
- Pattern compliance
- Anti-pattern detection
- Performance analysis
- Documentation completeness

**Security Review**: OWASP assessment (final gate)
- Vulnerability scanning
- Permission checks
- Input validation review
- **Closes ticket as "Done" when passing**

**PM Role**: Review outputs at each phase, don't read code directly.

### What Gets Delivered

After completing the workflow, you'll have:

**Code**:
- CSV export functionality
- Comprehensive test suite (90%+ coverage)
- Security-reviewed code
- Well-documented APIs

**Documentation**:
- API documentation
- User guides
- Inline code comments
- Technical decisions documented

**Quality Assurance**:
- All tests passing
- Code review approved
- Security review passed (no critical/high issues)
- All acceptance criteria met

### Key Learning Points

**What Worked Well**:
1. **Service Inventory First**: Discovering existing services prevented rebuilding functionality
2. **Following Patterns**: Discovery found PDF export pattern to follow
3. **Clear Success Criteria**: Made validation straightforward
4. **Quality Gates**: Caught issues before production

**Common Surprises**:
1. **AI finds edge cases**: Test generation often includes scenarios PM didn't consider
2. **Comprehensive coverage**: 90%+ test coverage is achievable without manual effort
3. **Pattern consistency**: AI follows existing patterns better than some junior developers

**PM Insights**:
- Focus on user value in PRD, not technical implementation
- Service inventory saves significant time
- Trust quality gates—they catch issues reliably
- Review outputs, not code

---

## Walkthrough 2: Building User Profile Customization

### Scenario Setup

**Feature**: User profile editing with photo upload
**Complexity**: Medium (multiple services integration)
**PM Background**: Technical PM (former developer)
**Codebase**: Existing social platform

### The PRD (Abbreviated)

```markdown
# User Profile Customization - PRD

## Problem Statement
Low profile completion rates correlate with poor retention.
Users need an easy way to customize their profiles.

## Success Criteria
- Users can upload profile photos
- Profile information is editable
- Changes save automatically
- Profile updates reflect immediately across the app

## Key Features
1. Profile photo upload with crop/resize
2. Bio editor with formatting
3. Custom profile URL
4. Social links integration
5. Profile preview before publishing
```

### Workflow Highlights

**Service Inventory Findings**:
- Existing image upload service (used for posts)
- User management service
- Authentication patterns
- Storage integration

**Discovery Insights**:
- Image upload patterns already established
- User update patterns exist
- Real-time update architecture in place

**Epic Planning**:
- Epic 1: Profile Photo & Basic Info
- Epic 2: Rich Profile Content
- Epic 3: Social Integration

**Technical Planning**:
Generated 12 sub-tickets across 3 epics, each 2-4 hours of work.

### Unique Learning: Technical PM Perspective

**Key Insight**: Even technical PMs should resist specifying implementation.

**What Happened**:
- PM initially tried to specify which image processing library to use
- PM specified database schema changes
- PM dictated API endpoint structure

**What Changed**:
- Learned to describe WHAT users need, not HOW to build it
- AI chose better libraries with current security patches
- AI designed cleaner API structure
- Result: Better implementation than PM would have specified

**Lesson**: Technical knowledge helps with reviews, not specifications.

---

## Walkthrough 3: Implementing Real-Time Notifications

### Scenario Setup

**Feature**: Real-time in-app notification system
**Complexity**: High (new infrastructure required)
**PM Background**: Non-technical PM
**Codebase**: Project management SaaS

### The PRD (Abbreviated)

```markdown
# Real-Time Notifications System - PRD

## Problem
Users miss critical project updates because email notifications
are delayed and often ignored.

## Solution
In-app real-time notifications with smart prioritization.

## Success Criteria
- Notifications appear within seconds of event
- Users can manage notification preferences
- Critical notifications are highlighted
- Notification history is accessible

## Features
1. Real-time notification center
2. Desktop notifications (opt-in)
3. Notification prioritization
4. Per-project preferences
5. Read/unread tracking
```

### Workflow Highlights

**Service Inventory Surprise**:
No real-time infrastructure existed. This revealed a significant gap.

**Discovery Findings**:
- Event-driven architecture already in use
- Existing notification preferences table
- User session management in place

**Epic Planning Decision Point**:

AI proposed **Epic 1: Real-Time Infrastructure** (WebSocket server) as prerequisite.

**PM Challenge**: Approving infrastructure epic with no immediate user value.

**AI Explanation**:
```
Epic 1 (Infrastructure) is required before Epic 2 (Notification UI).
Real-time infrastructure enables:
- WebSocket connections
- Event streaming
- Connection management

Without this, notification UI cannot function in real-time.

Estimated effort: Infrastructure epic similar in size to other epics.
User-facing value delivered in Epic 2 (builds on Epic 1).
```

**PM Decision**: Approved infrastructure epic based on AI explanation.

**Key Learning**: Sometimes necessary technical prerequisites have no direct user value. Trust AI to explain why infrastructure is needed.

### Execution

**Implementation**:
- Epic 1: WebSocket infrastructure
- Epic 2: Notification UI
- Epic 3: Preferences management
- Epic 4: Desktop notifications

**Quality Gates**:
- All security reviews passed
- Comprehensive testing included connection handling
- Documentation covered WebSocket API

### PM Insights

**Biggest Challenge**: Understanding when infrastructure work is necessary vs. over-engineering.

**How AI Helped**:
- Clear explanation of technical prerequisites
- Sizing estimates for infrastructure work
- Showed how infrastructure enables user-facing features

**Lesson**: Non-technical PMs can approve technical decisions when AI explains the "why" clearly.

---

## Common Patterns Across Walkthroughs

### Success Factors

**Clear Requirements Win**:
- Specific acceptance criteria lead to accurate implementations
- Business context helps AI prioritize correctly
- Success metrics provide objective validation

**Service Inventory Prevents Duplication**:
- All three walkthroughs found 60-75% reuse opportunities
- Prevented rebuilding existing functionality
- Accelerated development significantly

**Discovery Ensures Consistency**:
- New code matches existing patterns
- Architectural decisions align with codebase
- Reduces technical debt

**Quality Gates Catch Issues**:
- Security review finds vulnerabilities before production
- Testing catches edge cases PMs didn't consider
- Code review ensures pattern compliance

### Common Mistakes (and How to Avoid Them)

**❌ Skipping Service Inventory**:
Why bad: Rebuild functionality that exists
How to avoid: Always run service inventory first

**❌ Vague Acceptance Criteria**:
Why bad: AI can't validate implementation
How to avoid: Make criteria specific and testable

**❌ Specifying Implementation Details**:
Why bad: Limits AI's ability to choose best approach
How to avoid: Describe user outcomes, not technical solutions

**❌ Skipping Quality Gates**:
Why bad: Issues slip to production
How to avoid: Run full workflow every time

### Patterns That Emerge

**Service Reuse is High**:
- Expect 60-75% reuse of existing code
- New features integrate with existing services
- Prevents architectural drift

**Test Coverage is Comprehensive**:
- AI generates more test cases than manual QA typically would
- Edge cases are covered automatically
- 90%+ coverage is achievable

**Security Reviews Find Issues**:
- Expect AI to catch OWASP vulnerabilities
- Permission checks are thoroughly reviewed
- Input validation is verified

**Documentation is Complete**:
- All public APIs documented automatically
- User guides generated from acceptance criteria
- Technical decisions captured

---

## Learning from These Examples

### For Your First Feature

**Choose Wisely**:
- Start with straightforward CRUD operation
- Avoid complex integrations initially
- Pick feature with clear success criteria

**Expect Learning Curve**:
- First feature takes longer (learning workflow)
- Writing AI-friendly PRDs takes practice
- Understanding quality gate outputs requires familiarity

**Build on Success**:
- Second feature will be faster
- Develop PRD templates
- Refine your review process

### Setting Realistic Expectations

**First Feature** (Week 1):
- Takes longer than experienced users
- More back-and-forth with AI
- Learning what to look for in reviews

**After 3-5 Features** (Month 1):
- Workflow becomes natural
- PRD writing improves significantly
- Reviews become faster

**Steady State** (Month 2+):
- Predictable timeframes
- Consistent quality
- Confidence in process

### Measuring Your Progress

**Quality Indicators**:
- Test coverage consistently >90%
- Security reviews pass with no critical issues
- Acceptance criteria fully met
- Documentation complete

**Velocity Indicators**:
- Time from PRD to production decreases
- Less back-and-forth on requirements
- Fewer review iterations needed

**Confidence Indicators**:
- Comfortable approving technical plans
- Can review quality gate outputs effectively
- Know when to escalate vs. iterate

---

## What to Expect from the Workflow

### Time Investment Ranges

**Learning Phase** (First 2-3 features):
- Longer than steady-state
- More experimentation
- Building templates and checklists

**Steady State** (After initial learning):
- Straightforward features: Faster delivery
- Complex features: Still faster than traditional
- Predictable timelines

**Factors That Affect Duration**:
- Feature complexity
- Codebase size and maturity
- Clarity of requirements
- Amount of new vs. reused code

### Quality You Can Expect

**Code Quality**:
- Follows existing patterns consistently
- Comprehensive error handling
- Proper abstraction and modularity

**Test Coverage**:
- 90%+ achievable for most features
- Edge cases covered
- Integration tests included

**Security**:
- OWASP Top 10 reviewed
- Permission checks verified
- Input validation comprehensive

**Documentation**:
- API documentation complete
- User guides generated
- Inline comments thorough

### When Things Don't Go Smoothly

**Common Challenges**:
- Ambiguous requirements lead to iterations
- Complex features need more planning
- Novel patterns require more guidance

**How to Recover**:
- Clarify requirements and re-run phases
- Break complex features into smaller pieces
- Provide examples of desired patterns

**Learning Opportunities**:
- Failed attempts teach what AI needs
- Iterations improve PRD writing
- Challenges build troubleshooting skills

---

## Contributing Your Experience

Have you used this workflow? Share your experience!

**What to Include**:
1. Feature type and complexity
2. What worked well
3. Challenges encountered
4. Lessons learned
5. Tips for other PMs

**How to Contribute**:
Submit a PR adding your walkthrough to this file, following the structure above.

**Help Others Learn**:
- Share PRD templates that worked
- Document review checklists you developed
- Explain how you overcame challenges

---

## Questions?

See [FAQ.md](FAQ.md) for common questions or open an issue on GitHub.

**Remember**: These walkthroughs demonstrate the process. Your actual results will vary based on your codebase, requirements, and experience level. Start small, learn the workflow, and build from there.
