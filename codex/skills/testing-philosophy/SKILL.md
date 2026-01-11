---
name: testing-philosophy
description: |
  Enforces test accuracy over coverage quantity. Use when:
  - Writing tests: "write tests", "add tests", "test this", "improve coverage", "testing phase"
  - Fixing tests: "tests failing", "fix tests", "debug test", "test suite broken", "run tests"
  - CI issues: "CI failing", "pipeline broken", "build red", "tests red", "skip tests"
  - Asking about tests: "why is this test failing", "how do I test this", "what should I test"
  - Creating/editing: *.spec.ts, *.test.ts, *.spec.js, *.test.js, __tests__/*, *.test.py, *_test.go

  Enforces fixing ALL existing broken tests BEFORE writing new tests. Read actual API before mocking.
  Gate sequence: Fix existing -> Discover API -> Compile -> Execute -> Coverage (secondary).
---

# Testing Philosophy

Fix broken tests BEFORE writing new tests. Accurate running tests > high coverage with broken tests.

## Mandatory Gate Sequence

### Gate 0: Fix Existing Tests (FIRST)

**BLOCKER: Do not write new tests until existing tests pass.**

```bash
# 1. Find existing tests for affected modules
ls src/modules/[module]/**/*.spec.ts 2>/dev/null

# 2. Run existing tests
npm test -- --testPathPattern="[module]" --run

# 3. Fix ALL failures (see below), then verify 100% pass
```

Fix strategies:
- API changed -> Update test to match new API
- Bug found -> Fix production code, not the test
- Wrong assumption -> Update test to correct behavior

### Gate 1: API Discovery (Before New Tests)

**NEVER assume. ALWAYS verify:**

```bash
# Read actual implementation
cat src/modules/[module]/[module].service.ts

# Find actual enums and methods
grep -r "enum\|async.*(" src/modules/[module]/
```

Document findings before writing tests:
```markdown
## API Discovery: EmailService
- Method: sendEmail(params: SendEmailParams): Promise<EmailResult>
- Enum: EmailType.USER_WELCOME (NOT "WELCOME")
```

### Gate 2: Compilation (100%)
```bash
npx tsc --noEmit src/**/*.spec.ts  # Zero TS errors
```

### Gate 3: Execution (100%)
```bash
npm test -- --testPathPattern="[module]" --run  # Zero runtime errors
```

### Gate 4: Coverage (Secondary)
- Target: 70-80% (not 90%+)
- Focus on complex logic
- Skip trivial code

## Test Structure

Study passing tests first, then follow pattern:

```javascript
describe('PaymentService', () => {
  let service: PaymentService;
  let mockRepository: jest.Mocked<PaymentRepository>;

  beforeEach(() => {
    mockRepository = createMockRepository();
    service = new PaymentService(mockRepository);
  });

  describe('processPayment', () => {
    it('should return transaction ID for valid payment', async () => {
      // Arrange
      mockRepository.save.mockResolvedValue({ id: 'txn_123' });
      // Act
      const result = await service.processPayment(validPayment);
      // Assert
      expect(result.transactionId).toBe('txn_123');
    });
  });
});
```

## When TO Test

- New functionality with no coverage
- Complex business logic
- Security-sensitive operations
- Critical user paths

## When NOT TO Test

- Already well-covered
- Trivial code (getters, pass-throughs)
- Just for coverage percentage
- Framework/library code

**Questions before each test:** Is this new? Is there a gap? Can this fail? Will it catch real bugs?

## Common Mistakes

```javascript
// WRONG - Assumed API
await emailService.sendWelcomeEmail(user); // Doesn't exist!

// RIGHT - Verified via reading implementation
await emailService.sendEmail({ type: EmailType.USER_WELCOME });

// WRONG - Testing workaround behavior
expect(await service.failingOp()).toBeNull(); // Tests workaround!

// RIGHT - Testing correct behavior
await expect(service.failingOp()).rejects.toThrow(ValidationError);
```

## Production vs Test Code

| Location | Mocks | Fixtures | Workarounds |
|----------|-------|----------|-------------|
| src/, lib/, app/ | NO | NO | NO |
| *.test.*, __tests__/ | YES | YES | N/A |

**Remember: 50% coverage with correct tests > 90% with broken tests.**

## Related Skills

- **production-code-standards**: Production code quality standards (test code has different rules)
- **verify-implementation**: Run and show test output before claiming "tests pass"

See `examples/test-templates.md` for well-structured test file templates and examples.

---

## How to Use This Skill in Codex

Include this skill's content in your Codex prompt when:
- Writing new tests for any code
- Fixing failing tests
- Reviewing test coverage strategy
- Discussing testing approach

Copy the gate sequence into your prompt to enforce the fix-first, accuracy-first approach.
