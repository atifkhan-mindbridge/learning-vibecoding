# Step 07: Test-Driven Development with AI

**Level:** Intermediate
**Time:** ~35 minutes

## Learning Objectives

By the end of this step, you will be able to:

1. Apply TDD with AI (Anthropic's "favorite workflow")
2. Prevent mock implementations that hide real issues
3. Use the "fail first, then implement" pattern
4. Verify with independent subagents to prevent overfitting

---

You've mastered the Explore-Plan-Code-Commit workflow from [Step 06](06-explore-plan-code-commit-workflow.md). Now we'll add a powerful technique that Anthropic calls their "favorite workflow": Test-Driven Development with AI.

## Why TDD + AI Is Powerful

Test-Driven Development becomes even more effective with AI assistance:

**Traditional TDD:**
1. Human writes test
2. Human writes code to pass test
3. Human refactors

**TDD + AI:**
1. Human (with AI help) writes test
2. AI writes code to pass test
3. AI can iterate until tests pass
4. Human verifies the solution

**The key advantage:** AI performs best when it has a clear target to iterate against. Tests provide that target. Instead of hoping AI generates correct code, you define correctness upfront through tests.

> "Test-driven development becomes even more powerful with agentic coding."
> — Anthropic, Claude Code Best Practices [^anthropic]

### TDD as Verification Quality

In the SASE (Structured Agentic Software Engineering) framework [^sase], tests serve as part of your **verification quality**—the evidence that work was done correctly. In any **Minimum Reviewable Product (MRP)**, verification evidence must include:

- Tests that prove the implementation works
- Evidence that tests actually tested the right things
- Independent verification that the code isn't overfitted to tests

TDD naturally produces this evidence as a byproduct of the workflow.

---

## The Complete TDD + AI Session

Before diving into individual steps, let's understand what a complete TDD session looks like from start to finish.

### The 6-Step TDD + AI Process

The following diagram illustrates the complete TDD + AI workflow:

```
┌─────────────────────────────────────────────────────────────────┐
│              TDD + AI WORKFLOW                                  │
│                                                                 │
│   ┌───────────┐                                                 │
│   │   Human   │ Defines requirements                            │
│   └─────┬─────┘                                                 │
│         │                                                       │
│         ▼                                                       │
│   ┌───────────┐      ┌───────────┐                             │
│   │    AI     │ ───► │   Tests   │  (AI writes tests)          │
│   └─────┬─────┘      └─────┬─────┘                             │
│         │                  │                                    │
│         │                  ▼                                    │
│         │            ┌───────────┐                             │
│         │            │   Fail?   │  (Human verifies)           │
│         │            └─────┬─────┘                             │
│         │                  │ Yes                                │
│         │                  ▼                                    │
│         │            ┌───────────┐                             │
│         │            │  Commit   │  (Lock specification)       │
│         │            │  Tests    │                             │
│         │            └─────┬─────┘                             │
│         │                  │                                    │
│         ▼                  ▼                                    │
│   ┌───────────┐      ┌───────────┐                             │
│   │    AI     │ ───► │   Code    │  (AI implements)            │
│   └─────┬─────┘      └─────┬─────┘                             │
│         │                  │                                    │
│         │                  ▼                                    │
│         │            ┌───────────┐                             │
│         │            │   Pass?   │  (Automated)                │
│         │            └─────┬─────┘                             │
│         │                  │ Yes                                │
│         ▼                  ▼                                    │
│   ┌───────────┐      ┌───────────┐                             │
│   │ Subagent  │ ───► │  Verify   │  (Independent check)        │
│   └─────┬─────┘      └─────┬─────┘                             │
│         │                  │                                    │
│         ▼                  ▼                                    │
│   ┌──────────────────────────────┐                             │
│   │      Human Approves          │                             │
│   │      Commit Implementation   │                             │
│   └──────────────────────────────┘                             │
└─────────────────────────────────────────────────────────────────┘
```

1. **Write tests** based on expected behavior (AI assists, human reviews)
2. **Run tests** to confirm they fail (proves tests are valid)
3. **Commit tests** to lock in the specification
4. **Implement** to pass tests (AI iterates until tests pass)
5. **Verify independently** using subagents (detect overfitting)
6. **Commit implementation** once verified

### Key Mindset Shifts

**From "write code that works" to "define what working means, then make it work"**

This shift is crucial because:
- Requirements become executable (tests ARE the specification)
- AI has a clear iteration target
- "Done" is objectively measurable
- Refactoring is safe (tests catch regressions)

**From "tests verify code" to "tests are evidence"**

Tests aren't just for the developer—they're evidence for reviewers, future maintainers, and AI agents that the code behaves as intended.

---

## Complete TDD Session Transcript

Before diving into individual steps, let's see a complete TDD session from start to finish. This transcript shows every step with explanations.

### Task: Password Strength Validator

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ STEP 1: WRITE TESTS (based on expected behavior)                            │
└─────────────────────────────────────────────────────────────────────────────┘

USER:
Write comprehensive tests for a password strength function called
`check_password_strength` that:
- Takes a password string
- Returns "weak", "medium", or "strong"
- Rules:
  - Weak: less than 8 characters
  - Medium: 8+ characters with letters OR numbers (not both)
  - Strong: 8+ characters with letters, numbers, AND special characters

Write the test file only. Do NOT implement the function yet.
Use pytest.

───────────────────────────────────────────────────────────────────────────────

CLAUDE:
I'll create comprehensive tests covering all the rules and edge cases.

[Creates tests/test_password_strength.py]

```python
import pytest
from password_validator import check_password_strength


class TestPasswordStrengthWeak:
    """Tests for weak password classification."""

    def test_empty_password_is_weak(self):
        """Empty string should be classified as weak."""
        assert check_password_strength("") == "weak"

    def test_single_character_is_weak(self):
        """Single character passwords are weak."""
        assert check_password_strength("a") == "weak"

    def test_seven_characters_is_weak(self):
        """Passwords with exactly 7 characters are weak."""
        assert check_password_strength("abcdefg") == "weak"
        assert check_password_strength("1234567") == "weak"
        assert check_password_strength("!@#$%^&") == "weak"

    def test_seven_mixed_characters_is_weak(self):
        """Even mixed characters under 8 chars are weak."""
        assert check_password_strength("Abc123!") == "weak"  # 7 chars


class TestPasswordStrengthMedium:
    """Tests for medium password classification."""

    def test_eight_letters_only_is_medium(self):
        """8+ characters with only letters is medium."""
        assert check_password_strength("abcdefgh") == "medium"
        assert check_password_strength("ABCDEFGH") == "medium"
        assert check_password_strength("AbCdEfGh") == "medium"

    def test_eight_numbers_only_is_medium(self):
        """8+ characters with only numbers is medium."""
        assert check_password_strength("12345678") == "medium"
        assert check_password_strength("00000000") == "medium"

    def test_long_letters_only_is_medium(self):
        """Even very long letter-only passwords are medium."""
        assert check_password_strength("abcdefghijklmnop") == "medium"

    def test_letters_and_numbers_no_special_is_medium(self):
        """Letters + numbers but no special chars is medium."""
        assert check_password_strength("abcd1234") == "medium"
        assert check_password_strength("Password1") == "medium"


class TestPasswordStrengthStrong:
    """Tests for strong password classification."""

    def test_eight_chars_all_types_is_strong(self):
        """8 characters with letters, numbers, and special is strong."""
        assert check_password_strength("Abcd123!") == "strong"

    def test_letters_numbers_special_is_strong(self):
        """Mix of all character types is strong."""
        assert check_password_strength("MyP@ssw0rd") == "strong"
        assert check_password_strength("Test#123") == "strong"

    def test_various_special_characters(self):
        """Various special characters should all count."""
        assert check_password_strength("Abcd123!") == "strong"
        assert check_password_strength("Abcd123@") == "strong"
        assert check_password_strength("Abcd123#") == "strong"
        assert check_password_strength("Abcd123$") == "strong"

    def test_long_strong_password(self):
        """Long passwords with all types stay strong."""
        assert check_password_strength("MyVeryStr0ng!Password") == "strong"


class TestPasswordStrengthEdgeCases:
    """Edge cases and boundary conditions."""

    def test_exactly_eight_characters_boundary(self):
        """Test the boundary at exactly 8 characters."""
        assert check_password_strength("1234567") == "weak"    # 7 chars
        assert check_password_strength("12345678") == "medium" # 8 chars

    def test_special_chars_only_eight_plus(self):
        """Only special characters (no letters/numbers) - edge case."""
        # This should be medium - has 8 chars but only special
        assert check_password_strength("!@#$%^&*") == "medium"

    def test_spaces_in_password(self):
        """Spaces should be handled (count as special chars)."""
        assert check_password_strength("abc 1234!") == "strong"

    def test_unicode_characters(self):
        """Unicode passwords should be handled."""
        assert check_password_strength("Пароль123!") == "strong"  # Cyrillic


class TestPasswordStrengthInputValidation:
    """Input validation tests."""

    def test_none_raises_type_error(self):
        """None input should raise TypeError."""
        with pytest.raises(TypeError):
            check_password_strength(None)

    def test_non_string_raises_type_error(self):
        """Non-string input should raise TypeError."""
        with pytest.raises(TypeError):
            check_password_strength(12345678)
        with pytest.raises(TypeError):
            check_password_strength(["password"])
```

I've created 23 test cases covering:
- Weak passwords (under 8 chars)
- Medium passwords (8+ chars, limited character types)
- Strong passwords (8+ chars, all character types)
- Edge cases (boundaries, unicode, special chars only)
- Input validation (None, wrong types)

───────────────────────────────────────────────────────────────────────────────

┌─────────────────────────────────────────────────────────────────────────────┐
│ STEP 2: RUN TESTS TO CONFIRM FAILURE                                        │
└─────────────────────────────────────────────────────────────────────────────┘

USER:
Run the tests. They should all fail since the function doesn't exist.

───────────────────────────────────────────────────────────────────────────────

CLAUDE:
> pytest tests/test_password_strength.py -v

FAILED tests/test_password_strength.py - ModuleNotFoundError: No module named 'password_validator'

All 23 tests fail with ModuleNotFoundError because password_validator.py
doesn't exist yet. This is exactly what we want - confirms our tests are
actually trying to import and call the function.

Tests correctly fail. Ready to commit the tests.

───────────────────────────────────────────────────────────────────────────────

┌─────────────────────────────────────────────────────────────────────────────┐
│ STEP 3: COMMIT THE TESTS                                                    │
└─────────────────────────────────────────────────────────────────────────────┘

USER:
Good. Commit the tests.

───────────────────────────────────────────────────────────────────────────────

CLAUDE:
> git add tests/test_password_strength.py
> git commit -m "test: add password strength validator tests

Comprehensive test suite covering:
- Weak passwords (< 8 chars)
- Medium passwords (8+ chars, limited types)
- Strong passwords (8+ chars, all types)
- Edge cases (unicode, boundaries)
- Input validation (None, wrong types)

23 test cases total. Implementation to follow."

Tests committed. The specification is now locked in version control.

───────────────────────────────────────────────────────────────────────────────

┌─────────────────────────────────────────────────────────────────────────────┐
│ STEP 4: IMPLEMENT TO PASS TESTS                                             │
└─────────────────────────────────────────────────────────────────────────────┘

USER:
Now implement check_password_strength to pass all tests.

IMPORTANT:
- Do NOT modify the tests
- Keep iterating until all tests pass
- Run tests after each change

───────────────────────────────────────────────────────────────────────────────

CLAUDE:
[Creates password_validator.py]

```python
def check_password_strength(password: str) -> str:
    """
    Evaluate password strength.

    Args:
        password: The password to evaluate

    Returns:
        "weak", "medium", or "strong"

    Raises:
        TypeError: If password is not a string
    """
    if not isinstance(password, str):
        raise TypeError("Password must be a string")

    if len(password) < 8:
        return "weak"

    has_letters = any(c.isalpha() for c in password)
    has_numbers = any(c.isdigit() for c in password)
    has_special = any(not c.isalnum() for c in password)

    if has_letters and has_numbers and has_special:
        return "strong"

    return "medium"
```

Running tests:

> pytest tests/test_password_strength.py -v

[Output shows test results]

22 passed, 1 failed

FAILED test_special_chars_only_eight_plus
  AssertionError: assert 'weak' == 'medium'

The implementation returns "weak" for special-chars-only passwords because
`has_letters` and `has_numbers` are both False. Let me check the requirement...

Looking at the test: `check_password_strength("!@#$%^&*")` expects "medium".

The rules say medium is "8+ characters with letters OR numbers (not both)".
Special chars only doesn't fit that description. Let me check if the test
correctly interprets the requirements...

Actually, the test comment says "Only special characters (no letters/numbers)
- edge case". The requirement says "letters OR numbers", so special-only
doesn't qualify for medium.

I believe the TEST has a bug. The requirement says:
- Medium: 8+ chars with letters OR numbers

Special chars only (no letters, no numbers) shouldn't be medium per the rules.

Should the test be "weak" for special-chars-only, or should we update the
requirements to include special-chars-only as medium?

───────────────────────────────────────────────────────────────────────────────

USER:
Good catch. The test is wrong - special chars only should be "weak" since it
doesn't have letters OR numbers. Fix the test.

───────────────────────────────────────────────────────────────────────────────

CLAUDE:
Updating the test to match the actual requirements:

[Edits tests/test_password_strength.py]

```python
def test_special_chars_only_eight_plus(self):
    """Only special characters (no letters/numbers) is weak per requirements."""
    # Doesn't have letters OR numbers, so it's weak despite length
    assert check_password_strength("!@#$%^&*") == "weak"
```

> pytest tests/test_password_strength.py -v

23 passed

All tests pass!

───────────────────────────────────────────────────────────────────────────────

[ANNOTATION: This is the one valid case where tests can be modified: when
the TEST has a bug, not when trying to make failing tests pass. The human
confirmed the test was wrong before the change was made.]

───────────────────────────────────────────────────────────────────────────────

┌─────────────────────────────────────────────────────────────────────────────┐
│ STEP 5: VERIFY WITH INDEPENDENT SUBAGENT                                    │
└─────────────────────────────────────────────────────────────────────────────┘

USER:
Use a subagent to verify the implementation handles cases not in the tests.
The subagent should NOT see the test file - only the requirements and
implementation.

───────────────────────────────────────────────────────────────────────────────

CLAUDE:
[Spawns subagent with only requirements and implementation]

**Subagent Review Results:**

I reviewed check_password_strength against these requirements:
- Weak: less than 8 characters
- Medium: 8+ characters with letters OR numbers
- Strong: 8+ characters with letters, numbers, AND special characters

**Verification Findings:**

✅ Length check works correctly for boundary (7 vs 8 chars)

✅ Character type detection handles:
  - Uppercase and lowercase letters
  - All digit characters
  - Special characters including punctuation and spaces

⚠️ **Potential Issue Found:** Emoji handling

The implementation uses `isalpha()` and `isdigit()` which may not handle
emoji characters as expected:
- `'😀'.isalpha()` returns False
- `'😀'.isdigit()` returns False
- `not '😀'.isalnum()` returns True (counts as special)

So a password like "Test123😀" would count as strong (has letters, numbers,
and emoji=special), which seems correct.

But a password of just "😀😀😀😀😀😀😀😀" (8 emoji) would be "weak" since
no letters and no numbers. This matches the "special chars only = weak" logic.

✅ **Unicode letter handling:**
"Пароль123!" (Cyrillic letters) - `isalpha()` returns True for Cyrillic
letters, so this correctly classifies as strong.

**Edge Cases Tested Manually:**
- "password" → medium ✓
- "12345678" → medium ✓
- "Pass123!" → strong ✓
- "        " (8 spaces) → weak ✓ (no letters/numbers)
- "a1!" → weak ✓ (under 8)
- "aaaa1111" → medium ✓ (letters + numbers, no special)

**Conclusion:** Implementation appears correct for all tested edge cases.
No overfitting detected - the logic is general, not case-specific.

───────────────────────────────────────────────────────────────────────────────

┌─────────────────────────────────────────────────────────────────────────────┐
│ STEP 6: COMMIT THE IMPLEMENTATION                                           │
└─────────────────────────────────────────────────────────────────────────────┘

USER:
Subagent verification passed. Commit the implementation.

───────────────────────────────────────────────────────────────────────────────

CLAUDE:
> git add password_validator.py
> git add tests/test_password_strength.py  # Include the fixed test
> git commit -m "feat: implement password strength validator

Implementation:
- Checks length (< 8 = weak)
- Checks character diversity (letters, numbers, special)
- Strong requires all three types, medium requires letters OR numbers

Handles edge cases:
- Unicode characters (Cyrillic, etc.)
- Spaces as special characters
- Emoji as special characters

Verified by independent subagent review.

Co-Authored-By: Claude <noreply@anthropic.com>"

Implementation committed with verification evidence.
```

**What this transcript demonstrates:**
1. Tests written first define expected behavior
2. Tests verified to fail before implementation
3. Implementation iterates until tests pass
4. When a test was wrong (not the implementation), it was identified and fixed with human approval
5. Subagent verification caught no overfitting
6. Both tests and implementation committed with clear messages

---

## The TDD + AI Workflow

### Step 1: Write Tests Based on Expected Behavior

Define what you expect through test cases:

```
Write tests for a function called `calculate_shipping_cost` that:
- Takes order_total (float) and destination (string: 'domestic' or 'international')
- Returns shipping cost as a float
- Rules:
  - Domestic: $5 flat rate, free over $50
  - International: $20 flat rate, free over $100
  - Invalid destination raises ValueError

Write the test file only. Do NOT implement the function yet.
Use pytest.
```

### Step 2: Run Tests and Confirm Failure

Critical step: Verify tests fail before implementation:

```
Run the tests and confirm they fail. We expect failures because
the function doesn't exist yet.

Do NOT write any implementation code at this stage.
```

**Why this matters:**
- Confirms tests are actually testing something
- Ensures tests will catch missing functionality
- Prevents "tests that pass with no implementation"

### Step 3: Commit the Tests

Once tests correctly represent expected behavior:

```
The tests look good and fail as expected. Commit them with message:
"test: add shipping cost calculation tests"
```

### Step 4: Implement to Pass Tests

Now ask for implementation:

```
Write the `calculate_shipping_cost` function to pass all the tests.

Important:
- Do NOT modify the tests
- Keep iterating until all tests pass
- Run tests after each change to verify progress
```

### Step 5: Verify with Independent Subagent

Prevent overfitting to specific test cases:

```
Use a subagent to verify the implementation:
- Check if the code handles cases not explicitly tested
- Look for edge cases that might fail
- Verify the logic matches the requirements, not just the tests

The subagent should evaluate independently, not just run existing tests.
```

### Step 6: Commit the Implementation

Once verified:

```
Implementation passes all tests and verification. Commit with message:
"feat: implement shipping cost calculation"
```

---

## Preventing Mock Implementations

### The Problem with Mocks

AI models sometimes generate "mock" implementations—code that looks right but doesn't actually work:

```python
# Mock implementation (BAD)
def calculate_shipping_cost(order_total, destination):
    # TODO: implement actual logic
    return 5.0  # Hardcoded to pass basic test
```

### How to Prevent Mocks

**Be explicit about TDD:**

```
We are doing test-driven development. This means:
1. You should NOT create mock implementations
2. You should NOT create placeholder functions
3. Each function should be fully implemented
4. Tests should test real behavior, not mocks

Do not use unittest.mock or pytest.mock for the core logic.
Only mock external dependencies like APIs or databases.
```

**Include realistic test data:**

Instead of:
```python
def test_shipping():
    assert calculate_shipping(10, "domestic") == 5
```

Use:
```python
def test_domestic_shipping_under_threshold():
    """Orders under $50 should have $5 shipping."""
    assert calculate_shipping_cost(10.00, "domestic") == 5.00
    assert calculate_shipping_cost(49.99, "domestic") == 5.00

def test_domestic_shipping_over_threshold():
    """Orders $50+ should have free domestic shipping."""
    assert calculate_shipping_cost(50.00, "domestic") == 0.00
    assert calculate_shipping_cost(100.00, "domestic") == 0.00
```

---

## The "Fail First" Pattern

### Why Failing Tests Matter

A test that never fails is worthless. Before implementation:

```
Run the test suite. I expect ALL tests to fail at this point because
the function doesn't exist. If any test passes, it means the test
is not actually testing our new functionality.

Report which tests fail and why.
```

### Handling Tests That Pass Prematurely

If a test passes before implementation:

```
The test `test_shipping_free_over_threshold` passed before we
implemented the function. This means:
1. The test might have a bug
2. The functionality already exists somewhere
3. The test isn't actually testing what we think

Investigate why this test passes and fix it if needed.
```

---

## Visual TDD with Screenshots

For UI features, extend TDD with visual verification:

### Setting Up Visual TDD

1. **Give Claude screenshot capability** (Puppeteer MCP server)
2. **Provide a visual mock** of expected UI
3. **Write tests that verify visual behavior**
4. **Use screenshot comparison in iteration**

### Example: Visual TDD Workflow

```
We're implementing a new notification badge. Here's the expected design:
[paste screenshot or image]

1. Write a Playwright test that:
   - Navigates to the dashboard
   - Verifies the badge appears when there are unread notifications
   - Verifies the badge shows the correct count
   - Verifies the badge hides when count is zero

2. Run the test to confirm it fails (badge doesn't exist yet)

3. Don't implement yet - just confirm the test correctly describes
   the expected behavior.
```

After test is confirmed:

```
Now implement the notification badge to pass the tests.

After implementing:
1. Run the Playwright test
2. Take a screenshot of the result
3. Compare to the mock
4. Iterate until it matches

Do NOT modify the tests to make them pass.
```

---

## Preventing Overfitting

### What Is Overfitting?

AI might generate code that passes specific tests but fails on:
- Edge cases not explicitly tested
- Different but valid inputs
- Real-world usage patterns

The following diagram illustrates overfitting and how to detect it:

```
┌─────────────────────────────────────────────────────────────────┐
│              DETECTING OVERFITTING                              │
│                                                                 │
│   Tests:                   Implementation:                      │
│   ┌──────────────────┐     ┌────────────────────────────────┐  │
│   │ double(2) == 4   │     │ if n == 2: return 4            │  │
│   │ double(3) == 6   │     │ if n == 3: return 6            │  │
│   └──────────────────┘     │ return 0  # OVERFITTED!        │  │
│                            └────────────────────────────────┘  │
│                                                                 │
│   ─────────────────────────────────────────────────────────     │
│                                                                 │
│   Detection Strategy:                                           │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │  1. Independent subagent reviews code                   │   │
│   │  2. Subagent does NOT see the test file                 │   │
│   │  3. Subagent evaluates: "Would this work for any n?"    │   │
│   │  4. Finds: "No, only works for n=2 and n=3"             │   │
│   │  5. Reports overfitting detected                        │   │
│   └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│   Proper Implementation:                                        │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │  def double(n):                                         │   │
│   │      return n * 2  # Works for ANY input                │   │
│   └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│   Key Insight: Overfitting happens when AI "memorizes" test     │
│   cases instead of implementing general logic.                  │
└─────────────────────────────────────────────────────────────────┘
```

**Example of overfitting:**

```python
# Tests
def test_double_2():
    assert double(2) == 4

def test_double_3():
    assert double(3) == 6

# Overfitted implementation (BAD)
def double(n):
    if n == 2:
        return 4
    if n == 3:
        return 6
    return 0  # Fails for any other input!
```

### Using Subagents to Detect Overfitting

```
Use an independent subagent to review the implementation:

1. Do NOT share the test file with the subagent
2. Give it only the requirements and implementation
3. Ask it to evaluate:
   - Does the code implement the requirements logically?
   - Would it work for inputs not shown in examples?
   - Are there obvious edge cases it would fail?

The subagent should judge the code's correctness independently,
not by running the existing tests.
```

### Adding Edge Case Tests

After initial implementation:

```
The basic tests pass. Now let's verify edge cases:

1. Add tests for boundary conditions:
   - Exactly at thresholds (49.99, 50.00, 50.01)
   - Zero values
   - Negative values (if applicable)
   - Very large values

2. Run the new tests
3. Fix any failures WITHOUT changing the original tests
```

---

## Property-Based Testing with AI

### What Is Property-Based Testing?

**Property-based testing** (also called generative testing) defines properties that should hold for ALL inputs, then generates hundreds or thousands of test cases automatically. Instead of specific examples, you define invariants.

**Example-based testing:**
```
double(2) should equal 4
double(3) should equal 6
```

**Property-based testing:**
```
For any integer n, double(n) should equal n * 2
For any integer n, double(n) should be even if n is an integer
```

### Why Property-Based Testing Matters for AI

Property-based tests are harder to "overfit" because:
- AI can't just handle specific cases
- Generated inputs cover edge cases you might not think of
- Properties describe behavior, not just examples

### Test Pyramid with AI

Understanding where AI excels and where humans add value helps you build effective test suites:

```
┌─────────────────────────────────────────────────────────────────┐
│              TEST PYRAMID + AI ROLES                            │
│                                                                 │
│                         /\                                      │
│                        /  \                                     │
│                       / E2E \      AI: Screenshot verification  │
│                      /      \     Human: Scenario design        │
│                     /────────\                                  │
│                    /          \                                 │
│                   / Integration \   AI: Generate tests          │
│                  /              \  Human: Define boundaries     │
│                 /────────────────\                              │
│                /                  \                             │
│               /       Unit         \  AI: High volume           │
│              /                      \ Human: Edge cases         │
│             /────────────────────────\                          │
│                                                                 │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │  AI excels at:                                          │   │
│   │  • Generating high volume of unit tests                 │   │
│   │  • Covering boilerplate test scenarios                  │   │
│   │  • Maintaining test consistency                         │   │
│   │  • Visual verification with screenshots                 │   │
│   │                                                         │   │
│   │  Human excels at:                                       │   │
│   │  • Defining test strategy and priorities                │   │
│   │  • Identifying edge cases from domain knowledge         │   │
│   │  • Setting boundaries for integration tests             │   │
│   │  • Designing meaningful E2E scenarios                   │   │
│   │  • Catching overfitting to test cases                   │   │
│   └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

### When to Use Property-Based Testing

**Good candidates:**
- Mathematical functions (parsers, encoders, calculations)
- Data transformations (serialization, mapping)
- Invariant-preserving operations (sort preserves length)
- Reversible operations (encode/decode, compress/decompress)

**Less suitable:**
- UI behavior (hard to define properties)
- External API interactions (non-deterministic)
- Complex business rules (properties hard to specify)

### Combining TDD with Property-Based Testing

The TDD + AI workflow adapts naturally:

1. **Write example-based tests** for key cases (human-readable specification)
2. **Write property tests** for invariants (comprehensive coverage)
3. **Run all tests** to confirm failures
4. **Implement** to pass both types
5. **Property tests reveal edge cases** the implementation might miss

### Asking AI for Property Tests

```
For the password strength function, in addition to example tests,
write property-based tests using Hypothesis that verify:
1. No password under 8 characters is ever rated "strong"
2. Password rating is deterministic (same input = same output)
3. Adding characters never decreases strength rating
```

### Complete Property-Based Testing Example

Here's a comprehensive property-based test suite for the password strength validator:

```python
"""
Property-based tests for password strength validator.
Uses Hypothesis to generate hundreds of test cases automatically.
"""
import pytest
from hypothesis import given, strategies as st, assume, settings
from password_validator import check_password_strength


# ============================================================================
# PROPERTY 1: Length determines minimum strength
# "No password under 8 characters can ever be strong or medium"
# ============================================================================

@given(password=st.text(min_size=0, max_size=7))
def test_short_passwords_never_strong_or_medium(password):
    """Any password under 8 characters cannot be strong or medium."""
    result = check_password_strength(password)
    assert result == "weak", f"Password '{password}' ({len(password)} chars) should be weak"


@given(password=st.text(min_size=0, max_size=7))
def test_short_passwords_weak_regardless_of_content(password):
    """Even short passwords with letters+numbers+special are weak."""
    result = check_password_strength(password)
    assert result == "weak"


# ============================================================================
# PROPERTY 2: Determinism
# "Same input always produces same output"
# ============================================================================

@given(password=st.text(min_size=0, max_size=100))
@settings(max_examples=500)
def test_deterministic_results(password):
    """Calling the function twice with same input gives same result."""
    result1 = check_password_strength(password)
    result2 = check_password_strength(password)
    assert result1 == result2, f"Non-deterministic result for '{password}'"


# ============================================================================
# PROPERTY 3: Valid output range
# "Output is always one of the three valid values"
# ============================================================================

@given(password=st.text(min_size=0, max_size=100))
def test_output_always_valid(password):
    """Result must be one of: weak, medium, strong."""
    result = check_password_strength(password)
    assert result in {"weak", "medium", "strong"}, f"Invalid result: {result}"


# ============================================================================
# PROPERTY 4: Length requirement for strength
# "Strong passwords must have at least 8 characters"
# ============================================================================

@given(password=st.text(min_size=8, max_size=50))
def test_long_passwords_not_weak_if_has_content(password):
    """8+ char passwords with letters or numbers are at least medium."""
    # Only test if password has at least letters OR numbers
    has_letters = any(c.isalpha() for c in password)
    has_numbers = any(c.isdigit() for c in password)

    assume(has_letters or has_numbers)  # Skip if neither

    result = check_password_strength(password)
    assert result in {"medium", "strong"}


# ============================================================================
# PROPERTY 5: Strong password composition
# "Strong passwords have all three character types"
# ============================================================================

@given(
    letters=st.text(alphabet=st.characters(whitelist_categories=['L']), min_size=1, max_size=10),
    numbers=st.text(alphabet='0123456789', min_size=1, max_size=10),
    special=st.text(alphabet='!@#$%^&*()-_=+[]{}|;:,.<>?', min_size=1, max_size=10)
)
def test_mixed_content_at_least_8_chars_is_strong(letters, numbers, special):
    """Password with letters, numbers, and special (8+ chars) is strong."""
    password = letters + numbers + special
    assume(len(password) >= 8)

    result = check_password_strength(password)
    assert result == "strong", f"'{password}' has all types but is {result}"


# ============================================================================
# PROPERTY 6: Monotonicity - adding good characters doesn't decrease strength
# "Adding a character type should never weaken a password"
# ============================================================================

@given(password=st.text(min_size=8, max_size=30))
def test_adding_number_doesnt_weaken(password):
    """Adding a digit to a password shouldn't decrease strength."""
    original_strength = check_password_strength(password)
    new_password = password + "7"
    new_strength = check_password_strength(new_password)

    strength_order = {"weak": 0, "medium": 1, "strong": 2}
    assert strength_order[new_strength] >= strength_order[original_strength], \
        f"Adding '7' weakened '{password}' from {original_strength} to {new_strength}"


@given(password=st.text(min_size=8, max_size=30))
def test_adding_special_doesnt_weaken(password):
    """Adding a special character shouldn't decrease strength."""
    original_strength = check_password_strength(password)
    new_password = password + "!"
    new_strength = check_password_strength(new_password)

    strength_order = {"weak": 0, "medium": 1, "strong": 2}
    assert strength_order[new_strength] >= strength_order[original_strength]


# ============================================================================
# PROPERTY 7: Unicode handling
# "Unicode letters should count as letters"
# ============================================================================

@given(
    base=st.text(alphabet=st.characters(whitelist_categories=['L']), min_size=5, max_size=10),
    numbers=st.text(alphabet='0123456789', min_size=1, max_size=3),
    special=st.text(alphabet='!@#$%', min_size=1, max_size=3)
)
def test_unicode_letters_count_as_letters(base, numbers, special):
    """Unicode letters (Cyrillic, Greek, etc.) should count as letters."""
    password = base + numbers + special
    assume(len(password) >= 8)

    result = check_password_strength(password)
    # Should be strong since we have letters + numbers + special
    assert result == "strong"


# ============================================================================
# PROPERTY 8: Whitespace handling
# "Whitespace should count as special characters (or be handled consistently)"
# ============================================================================

@given(
    letters=st.text(alphabet='abcdefgh', min_size=3, max_size=5),
    numbers=st.text(alphabet='12345', min_size=2, max_size=3)
)
def test_space_counts_as_special(letters, numbers):
    """Space character should count as special, making password strong."""
    password = letters + " " + numbers
    assume(len(password) >= 8)

    result = check_password_strength(password)
    # letters + numbers + space (special) = strong
    assert result == "strong", f"'{password}' with space should be strong"


# ============================================================================
# Regression tests found by Hypothesis
# (These get added when Hypothesis finds edge cases)
# ============================================================================

class TestHypothesisRegression:
    """Test cases discovered by Hypothesis that we want to preserve."""

    def test_null_byte_handling(self):
        """Null bytes shouldn't crash the function."""
        result = check_password_strength("pass\x00word123!")
        assert result in {"weak", "medium", "strong"}

    def test_very_long_password(self):
        """Very long passwords should be handled."""
        long_pass = "A" * 1000 + "1" * 100 + "!" * 100
        result = check_password_strength(long_pass)
        assert result == "strong"
```

**Running Property Tests:**

```bash
> pytest tests/test_password_strength_properties.py -v

test_short_passwords_never_strong_or_medium PASSED  (100 examples)
test_short_passwords_weak_regardless_of_content PASSED  (100 examples)
test_deterministic_results PASSED  (500 examples)
test_output_always_valid PASSED  (100 examples)
test_long_passwords_not_weak_if_has_content PASSED  (100 examples)
test_mixed_content_at_least_8_chars_is_strong PASSED  (100 examples)
test_adding_number_doesnt_weaken PASSED  (100 examples)
test_adding_special_doesnt_weaken PASSED  (100 examples)
test_unicode_letters_count_as_letters PASSED  (100 examples)
test_space_counts_as_special PASSED  (100 examples)

1300+ test cases generated and passed
```

**Why Property Tests Complement Example Tests:**

| Example Tests | Property Tests |
|---------------|----------------|
| Specific, readable scenarios | General rules that always hold |
| Easy to understand intent | Find edge cases you didn't think of |
| Fast to run (fixed inputs) | Slower (generates many inputs) |
| Document requirements | Verify invariants |

Use both: Example tests for documentation and specific behavior, property tests for confidence that edge cases are handled.

---

## Security-Focused Testing

### Why Security Testing Matters in AI-Assisted Development

AI-generated code can introduce security vulnerabilities:
- AI may not consider malicious inputs
- Common patterns in training data may include insecure approaches
- AI optimization for "works" may not include "secure"

TDD with security tests catches these issues early.

### OWASP-Inspired Test Categories

Test for common vulnerabilities from the OWASP Top 10:

**Injection Testing**
- SQL injection attempts in user input
- Command injection in shell-related functions
- XSS payloads in text processing

**Input Validation Testing**
- Boundary values (very long, very short, empty)
- Type confusion (numbers as strings, objects as arrays)
- Encoding issues (Unicode, null bytes, special characters)

**Authentication/Authorization Testing**
- Missing authentication checks
- Privilege escalation attempts
- Session handling edge cases

### Writing Security Tests

When writing tests for any function that handles user input:

```
Also write security-focused tests that include:
1. Input with SQL injection attempt: "'; DROP TABLE users; --"
2. Input with XSS: "<script>alert('xss')</script>"
3. Very long input (10,000+ characters)
4. Unicode edge cases
5. Null and undefined values

These should either be handled gracefully or raise appropriate exceptions.
```

### Security Test Mindset

Think like an attacker when writing tests:
- What would a malicious user try?
- What assumptions does the code make that could be violated?
- What happens with unexpected input types?

AI can help generate malicious test cases:

```
Act as a security tester. For this input validation function,
generate 20 test cases that attempt to bypass or break validation.
Include various injection attacks, encoding tricks, and edge cases.
```

### Complete Security Testing Example

Here's a comprehensive security test suite for a user registration function:

```python
"""
Security-focused tests for user registration.
Tests for OWASP Top 10 vulnerabilities and common attack vectors.
"""
import pytest
from app.auth import register_user, validate_email, sanitize_username


class TestSQLInjection:
    """SQL Injection attack prevention tests."""

    def test_email_with_sql_injection(self):
        """SQL injection in email should be handled safely."""
        malicious_emails = [
            "user@test.com'; DROP TABLE users; --",
            "admin'--@evil.com",
            "user@test.com' OR '1'='1",
            "'; INSERT INTO admin VALUES('hacker'); --",
            "user@test.com\"; DROP TABLE users; --",
        ]
        for email in malicious_emails:
            # Should either reject or safely escape - never execute SQL
            result = validate_email(email)
            assert result is False or isinstance(result, str), \
                f"SQL injection not handled: {email}"

    def test_username_with_sql_injection(self):
        """SQL injection in username should be sanitized."""
        malicious_usernames = [
            "admin'; DELETE FROM users WHERE '1'='1",
            "user\"; UPDATE users SET role='admin' WHERE name=\"",
            "Robert'); DROP TABLE Students;--",  # Little Bobby Tables
        ]
        for username in malicious_usernames:
            sanitized = sanitize_username(username)
            # Dangerous characters should be removed or escaped
            assert "'" not in sanitized
            assert '"' not in sanitized
            assert ";" not in sanitized


class TestXSSPrevention:
    """Cross-Site Scripting (XSS) prevention tests."""

    def test_username_with_script_tags(self):
        """Script tags in username should be sanitized."""
        xss_payloads = [
            "<script>alert('XSS')</script>",
            "<img src=x onerror=alert('XSS')>",
            "<svg onload=alert('XSS')>",
            "javascript:alert('XSS')",
            "<iframe src='javascript:alert(1)'>",
            "<<script>script>alert('XSS')<</script>/script>",  # Nested
        ]
        for payload in xss_payloads:
            sanitized = sanitize_username(payload)
            assert "<script" not in sanitized.lower()
            assert "javascript:" not in sanitized.lower()
            assert "onerror" not in sanitized.lower()
            assert "onload" not in sanitized.lower()

    def test_email_with_xss_in_name_part(self):
        """XSS in email local part should not execute."""
        email = "<script>alert(1)</script>@test.com"
        result = validate_email(email)
        # Should reject as invalid email format
        assert result is False


class TestInputValidation:
    """Input validation and boundary testing."""

    def test_extremely_long_inputs(self):
        """Very long inputs should not cause denial of service."""
        # 1 MB of data
        long_email = "a" * (1024 * 1024) + "@test.com"
        long_username = "x" * (1024 * 1024)

        # Should reject without crashing or taking too long
        import time
        start = time.time()
        result_email = validate_email(long_email)
        result_username = sanitize_username(long_username)
        elapsed = time.time() - start

        assert elapsed < 1.0, f"Long input took {elapsed}s - DoS vulnerability"
        assert result_email is False  # Should reject

    def test_null_byte_injection(self):
        """Null bytes should not truncate validation."""
        # Attacker tries: "admin\x00@evil.com" hoping validation sees "admin"
        email_with_null = "admin\x00@evil.com"
        username_with_null = "admin\x00;DROP TABLE users"

        result = validate_email(email_with_null)
        assert result is False, "Null byte injection not prevented"

        sanitized = sanitize_username(username_with_null)
        assert "\x00" not in sanitized

    def test_unicode_homoglyph_attacks(self):
        """Unicode characters that look like ASCII should be handled."""
        # Cyrillic 'а' (U+0430) looks like Latin 'a'
        homoglyph_username = "аdmin"  # First char is Cyrillic
        sanitized = sanitize_username(homoglyph_username)

        # Should either normalize to ASCII or reject
        # Depends on requirements - but shouldn't silently allow "аdmin" = "admin"

    def test_empty_and_whitespace_inputs(self):
        """Empty and whitespace-only inputs should be rejected."""
        invalid_inputs = ["", "   ", "\t\n", "\u200b"]  # Last is zero-width space

        for inp in invalid_inputs:
            email_valid = validate_email(inp + "@test.com")
            # Empty local part should be invalid
            assert validate_email(inp) is False

    def test_special_email_formats(self):
        """Valid but unusual email formats should be handled correctly."""
        # These ARE valid per RFC 5321 but often cause issues
        unusual_emails = [
            '"john doe"@example.com',      # Quoted local part
            'user+tag@example.com',         # Plus addressing
            'user.name@example.co.uk',      # Multiple dots in domain
            'user@[192.168.1.1]',           # IP address domain
        ]
        for email in unusual_emails:
            result = validate_email(email)
            # Should handle without crashing (accept or reject based on policy)
            assert result is True or result is False


class TestAuthenticationSecurity:
    """Authentication-related security tests."""

    def test_password_not_in_response(self):
        """Password should never appear in registration response."""
        result = register_user(
            email="test@example.com",
            username="testuser",
            password="SecureP@ss123"
        )
        result_str = str(result)
        assert "SecureP@ss123" not in result_str, "Password leaked in response"

    def test_password_not_logged(self, caplog):
        """Password should never be logged."""
        import logging
        with caplog.at_level(logging.DEBUG):
            register_user(
                email="test@example.com",
                username="testuser",
                password="SecureP@ss123"
            )

        for record in caplog.records:
            assert "SecureP@ss123" not in record.message, \
                f"Password logged: {record.message}"

    def test_timing_attack_resistance(self):
        """Email existence check should have constant time."""
        import time

        # Test with existing email
        existing_times = []
        for _ in range(10):
            start = time.perf_counter()
            validate_email("existing@example.com")
            existing_times.append(time.perf_counter() - start)

        # Test with non-existing email
        nonexisting_times = []
        for _ in range(10):
            start = time.perf_counter()
            validate_email("nonexisting@example.com")
            nonexisting_times.append(time.perf_counter() - start)

        # Times should be similar (within 10% variance)
        avg_existing = sum(existing_times) / len(existing_times)
        avg_nonexisting = sum(nonexisting_times) / len(nonexisting_times)

        # This test is probabilistic - in real code, use constant-time comparison
        variance = abs(avg_existing - avg_nonexisting) / max(avg_existing, avg_nonexisting)
        # Note: This test may be flaky; real constant-time checks are better


class TestRateLimitingAndAbuse:
    """Tests for abuse prevention."""

    def test_rapid_registration_attempts(self):
        """Rapid registrations should be rate limited."""
        results = []
        for i in range(100):
            result = register_user(
                email=f"spam{i}@attacker.com",
                username=f"spammer{i}",
                password="Password123!"
            )
            results.append(result)

        # After some threshold, should start rejecting
        rejected = sum(1 for r in results if r.get('error') == 'rate_limited')
        assert rejected > 0, "No rate limiting on registration"

    def test_email_enumeration_prevention(self):
        """Should not reveal whether email exists."""
        # Register an email
        register_user("exists@test.com", "existsuser", "Password123!")

        # Try to register same email
        result1 = register_user("exists@test.com", "newuser", "Password456!")

        # Try to register different email
        result2 = register_user("new@test.com", "anotheruser", "Password789!")

        # Error messages should be identical (don't reveal which email exists)
        # Bad: "Email already registered" vs "Invalid email"
        # Good: "Unable to complete registration" for both


class TestFilePathTraversal:
    """If registration involves file operations."""

    def test_username_path_traversal(self):
        """Username should not enable path traversal."""
        dangerous_usernames = [
            "../../../etc/passwd",
            "..\\..\\..\\windows\\system32\\config\\sam",
            "user/../../admin",
            "user%2F..%2F..%2Fadmin",  # URL encoded
        ]
        for username in dangerous_usernames:
            sanitized = sanitize_username(username)
            assert ".." not in sanitized
            assert "/" not in sanitized
            assert "\\" not in sanitized


class TestHeaderInjection:
    """HTTP header injection prevention."""

    def test_email_with_crlf_injection(self):
        """CRLF in email should not enable header injection."""
        # Attacker tries to inject headers via email
        malicious = "user@test.com\r\nBcc: attacker@evil.com"
        result = validate_email(malicious)
        assert result is False, "CRLF injection not prevented"

        malicious2 = "user@test.com%0d%0aBcc:%20attacker@evil.com"
        result2 = validate_email(malicious2)
        assert result2 is False, "URL-encoded CRLF not prevented"
```

**How to use security tests in TDD:**

1. **Write security tests alongside functional tests** - They're part of the specification
2. **Include in CI pipeline** - Security tests must pass on every commit
3. **Run with fuzzing** - Combine with property-based tests for broader coverage
4. **Update when new vulnerabilities discovered** - Security tests grow over time

**Asking AI to generate security tests:**

```
For this user input processing function, write security tests that check:
1. SQL injection in all string parameters
2. XSS prevention in user-visible outputs
3. Path traversal if any file operations
4. Denial of service with large inputs
5. Null byte handling
6. Unicode edge cases

Use OWASP testing guidelines as reference.
```

---

## The Golden Rule: Never Modify Tests to Pass

The following visual reinforces this critical rule:

```
┌─────────────────────────────────────────────────────────────────┐
│       THE GOLDEN RULE: NEVER MODIFY TESTS TO PASS               │
│                                                                 │
│   [X] BAD: Test fails                                           │
│       └──► "Let me adjust the test assertion..."                │
│            └──► Test passes (but code is still wrong)           │
│                 └──► Bug ships to production                    │
│                                                                 │
│   ─────────────────────────────────────────────────────────     │
│                                                                 │
│   [✓] GOOD: Test fails                                          │
│       └──► "Let me fix the implementation..."                   │
│            └──► Test passes (code is now correct)               │
│                 └──► Quality maintained                         │
│                                                                 │
│   ─────────────────────────────────────────────────────────     │
│                                                                 │
│   Exception: Test itself has a bug                              │
│       └──► Human reviews the test                               │
│            └──► Confirms test is wrong (not implementation)     │
│                 └──► Fix test, document why                     │
│                      └──► Get approval before changing          │
│                                                                 │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │  In AI prompts, be explicit:                            │   │
│   │                                                         │   │
│   │  "Do NOT modify tests. Fix the implementation only.     │   │
│   │   If you believe a test has a bug, explain why and      │   │
│   │   wait for my approval before changing it."             │   │
│   └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

```
IMPORTANT: It is unacceptable to remove or edit tests to make them pass.

If a test fails:
1. Verify the test is correct
2. If test is correct: fix the implementation
3. If test has a bug: fix the test (rare - usually implementation is wrong)

Never "fix" a test by making it less strict or removing it.
```

### Why This Rule Exists

- Tests represent the specification
- Modifying tests changes the requirements
- It's easy to "pass" by weakening tests
- Encourages implementation solutions, not test avoidance

---

## Complete TDD + AI Example

### Task: Email Validation Function

**Step 1: Write Tests**
```
Write comprehensive tests for an email validation function. Include:
- Valid email formats (user@domain.com, user.name@domain.co.uk)
- Invalid formats (missing @, missing domain, etc.)
- Edge cases (empty string, None, very long emails)

Use pytest. Do NOT implement the function.
```

**Step 2: Verify Failure**
```
Run the tests. All should fail with "function not defined" or similar.
Confirm this before we proceed.
```

**Step 3: Commit Tests**
```
Commit the tests: "test: add email validation tests"
```

**Step 4: Implement**
```
Implement is_valid_email to pass all tests.
- Do NOT modify tests
- Run tests after each iteration
- Continue until all pass
```

**Step 5: Independent Verification**
```
Use a subagent to verify the implementation handles cases not in tests:
- Emails with numbers
- Emails with + symbols
- Subdomains
- International characters

Report any issues found.
```

**Step 6: Commit Implementation**
```
Implementation verified. Commit: "feat: implement email validation"
```

---

## TDD Anti-Patterns to Avoid

### Anti-Pattern 1: Tests After Implementation

**Problem:** Writing tests to match implementation (tests pass by definition)

**Solution:** Always write tests first; verify they fail

### Anti-Pattern 2: Over-Mocking

**Problem:** Tests that mock so much they don't test real behavior

**Solution:** Mock only external dependencies; test real logic

### Anti-Pattern 3: Implementation-Aware Tests

**Problem:** Tests that verify internal implementation, not behavior

**Solution:** Test inputs and outputs, not how the function works internally

### Anti-Pattern 4: Single Happy Path

**Problem:** Only testing the "it works" case

**Solution:** Include error cases, edge cases, and boundary conditions

### Anti-Pattern 5: Letting AI Modify Tests

**Problem:** AI "fixes" failing tests instead of fixing implementation

**Solution:** Explicitly forbid test modification; review all changes

---

## Key Takeaways

1. **TDD + AI is powerful** because tests give AI a clear target to iterate against
2. **Tests are verification evidence** — Part of your MRP quality artifacts
3. **Be explicit about TDD** to prevent mock implementations
4. **Verify tests fail first** before implementing
5. **Never modify tests** to make them pass
6. **Use independent subagents** to detect overfitting
7. **Include property-based tests** for comprehensive coverage against overfitting
8. **Add security tests** for any code handling user input
9. **Include edge cases** to ensure robust implementations
10. **Visual TDD** extends the pattern to UI development

---

## Practical Exercise: TDD a Password Strength Checker

Practice the full TDD + AI workflow:

### Requirements
Create a function that evaluates password strength:
- Returns "weak", "medium", or "strong"
- Weak: less than 8 characters
- Medium: 8+ characters with letters only OR numbers only
- Strong: 8+ characters with letters, numbers, AND special characters

### Your Task

1. **Write tests first** (all should fail)
2. **Commit tests**
3. **Implement to pass tests**
4. **Verify with subagent for edge cases**
5. **Add edge case tests and fix any issues**
6. **Commit implementation**

Document any places where you were tempted to modify tests instead of fixing implementation.

---

## Next Step

[Step 08: Agent Architecture and Patterns](08-agent-architecture-and-patterns.md) — Understand how AI agents work and learn multi-agent patterns.

---

## Sources

[^anthropic]: Anthropic. (2025). "Claude Code Best Practices." Official documentation on TDD workflows. https://docs.anthropic.com/en/docs/claude-code/best-practices

[^sase]: Guo, P. (2025). "Structured Agentic Software Engineering." arXiv:2509.06216v2. Sections on Verification Quality and MRP Evidence. https://arxiv.org/abs/2509.06216

[^tdd]: Chen, X., et al. (2025). "TDD Framework for LLM Code Generation." arXiv:2510.15585. Research on test-driven approaches with AI code generation.

- Stanford CS146S, Week 6: AI Testing and Security. Stanford University, 2025.
- Anthropic: "Building Agents with the Claude Agent SDK." https://docs.anthropic.com/en/docs/agents-and-tools
- Hypothesis Documentation: Property-Based Testing. https://hypothesis.readthedocs.io/
- OWASP: Testing Guide and Top 10. https://owasp.org/www-project-web-security-testing-guide/

---

## Further Reading

1. **Anthropic: Claude Code Best Practices (TDD Section)** — Official guidance on test-driven development with AI assistance. https://docs.anthropic.com/en/docs/claude-code/best-practices

2. **Hypothesis Documentation** — Python library for property-based testing, essential for detecting AI code overfitting. https://hypothesis.readthedocs.io/

3. **OWASP Testing Guide** — Comprehensive guide to security testing, applicable to AI-generated code. https://owasp.org/www-project-web-security-testing-guide/

4. **arXiv:2510.15585: TDD Framework for LLM Code Generation** — Academic research on formalizing TDD for AI-assisted development. https://arxiv.org/abs/2510.15585

5. **Martin Fowler: TDD** — Classic resource on test-driven development fundamentals. https://martinfowler.com/bliki/TestDrivenDevelopment.html
