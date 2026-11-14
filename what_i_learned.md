# What I Learned: My First Open-Source Contribution Exercise

## Project Overview

**Repository:** [Mantine](https://github.com/mantinedev/mantine)  
**Issue:** [#9397 — WeekView Accessibility](https://github.com/mantinedev/mantine/issues/9397)  
**Branch:** `learning/weekview-accessible-name`

This was my first experience working on a large, established open-source codebase.

My goal was not just to fix a bug, but to practice navigating unfamiliar code, understanding existing architecture, writing meaningful tests, using AI coding tools responsibly, and evaluating the broader impact of code changes.

I worked on an accessibility issue in Mantine's `WeekView` component. What initially appeared to be a small change to an `aria-label` turned into an exercise involving testing, localization, public APIs, backward compatibility, and software design decisions.

This is a local learning exercise. Another contributor submitted [PR #9401](https://github.com/mantinedev/mantine/pull/9401) addressing the same issue.

## 1. Understanding the Problem

The weekday header buttons displayed text such as:

```text
Mon 9
```

However, their accessible names were generated separately:

```text
Weekday 2026-11-09
```

This created an accessibility problem because the accessible name did not contain the visible button text.

The issue relates to **WCAG 2.5.3: Label in Name** and the axe `label-content-name-mismatch` rule.

This matters particularly for users who rely on speech-input software to activate controls using their visible labels.

### My Solution

I explored an alternative implementation that retained an explicit `aria-label` while ensuring it began with the visible weekday and day number.

**Before:**

```text
Weekday 2026-11-09
```

**After:**

```text
Mon 9, November 9, 2026
```

This preserved the full date in the accessible name while incorporating the visible text.

## 2. Navigating a Large Codebase

Before making changes, I needed to understand how Mantine organized its components, utilities, and tests.

I investigated:

- The `WeekView` component and its weekday header rendering.
- Existing date-formatting utilities.
- Component props such as `weekdayFormat` and `locale`.
- Localization through `DatesProvider`.
- Existing Jest tests.
- Mantine's Styles API and component attributes.

I also used Storybook to inspect the component and reproduce the accessibility issue.

**Lesson:** Before changing code, understand how the relevant functionality is implemented and what other parts of the system depend on it.

## 3. Practicing Test-Driven Development

I followed the Red → Green → Refactor workflow.

### Red: Write a Failing Test

I wrote a test expecting the new accessible name before implementing the fix.

The test failed because the component still returned the original accessible name.

This confirmed that the test could detect the problem.

### Green: Implement the Fix

After updating the component, the regression test passed.

However, several existing tests failed because they expected the old accessible names.

I updated those expectations while preserving the tests' original behavioral assertions.

### Refactor and Review

I reviewed the implementation and expanded the test coverage.

One important discovery involved whitespace between the weekday and day number.

Without an explicit JSX whitespace separator, the DOM text could become:

```text
Mon9
```

Even though the `aria-label` contained `Mon 9`, the axe accessibility check could still fail.

I added a test verifying the exact visible DOM text alongside the accessible name.

**Lesson:** Tests should protect specific requirements. A passing test suite does not guarantee that every important behavior is covered.

## 4. Learning to Test Edge Cases

The initial test covered one English date, but the component supported many configurations.

I expanded the tests to cover:

- Custom weekday formatting using Dayjs format strings.
- Custom formatting callbacks.
- Localization through the component's `locale` prop.
- Localization inherited from `DatesProvider`.
- Weeks spanning December and January.
- Correct spacing between visible weekday text and day numbers.

I also learned that tests can become too closely coupled to implementation details.

For example, checking the exact number and order of formatting callback invocations can detect unnecessary work, but may also make tests brittle during future refactoring.

**Lesson:** Good tests focus on meaningful behavior while avoiding unnecessary assumptions about implementation details.

## 5. Discovering Backward Compatibility Issues

One of the most important discoveries was that my change affected an existing public customization option: `labels.weekday`.

Previously, this property contributed to weekday header accessible names.

My implementation no longer used it.

This meant that an application could continue supplying `labels.weekday` without an error, but the customization would no longer affect the weekday buttons.

I learned to distinguish between:

- **Type compatibility:** Existing code still compiles.
- **Behavioral compatibility:** Existing functionality continues to behave as expected.
- **Accessibility compatibility:** Users can still interact with controls accessibly.

**Lesson:** A change can fix a bug while unintentionally changing behavior that other developers depend on.

## 6. Understanding Mantine's Styles API

Initially, I didn't realize developers could customize HTML attributes such as `aria-label` through Mantine's Styles API.

I learned that the order in which React props are applied matters.

For example:

```tsx
<UnstyledButton {...attributes} aria-label="Default label" />
```

The explicit `aria-label` overrides an `aria-label` supplied through `attributes`.

This helped me understand another advantage of the alternative PR: removing the explicit `aria-label` allows consumer-provided attributes to take effect.

**Lesson:** When modifying a component, investigate all its customization mechanisms, not just the props directly involved in the bug.

## 7. Accessibility and Customization Tradeoffs

I realized that allowing developers to customize accessible names can itself introduce accessibility risks.

For example, a button displaying `Mon 9` could receive a custom accessible name such as `November 9, 2026`, which does not contain the visible text.

This could reintroduce the original accessibility issue.

However, preventing all customization would reduce flexibility for developers with legitimate use cases.

I learned that component libraries often need to balance:

- Accessible defaults.
- Developer customization.
- Backward compatibility.
- Simplicity and maintainability.

**Lesson:** A library can provide accessible defaults while still requiring consumers to use customization APIs responsibly.

## 8. Comparing Different Solutions

I compared my implementation with PR #9401.

My solution retained the full date in the accessible name.

The other contributor's solution removed the explicit `aria-label`, allowing the accessible name to come directly from the visible button text.

The other PR also addressed the unused `labels.weekday` property through deprecation, updated documentation, and added tests for Styles API attribute overrides.

This comparison taught me that a more complex solution is not necessarily a better solution.

**Lesson:** Software engineering involves evaluating alternatives based on correctness, maintainability, compatibility, and user experience—not simply whether the code works.

## 9. Using AI as a Development Tool

I used Codex to help investigate the codebase, implement changes, create tests, and review the resulting code.

However, I also learned the importance of understanding and questioning AI-generated changes.

Some of the most valuable discoveries came from reviewing the implementation and considering its consequences.

I found it useful to ask Codex to:

- Explain existing code before modifying it.
- Create an implementation plan.
- Make focused changes.
- Run relevant tests.
- Review the diff for regressions.
- Identify compatibility risks.
- Explain its reasoning.

**Lesson:** AI can accelerate development, but I am still responsible for understanding the code, validating changes, and making engineering decisions.

## 10. Final Verification

The local implementation passed the following checks:

| Check                           | Result     |
| ------------------------------- | ---------- |
| WeekView Jest tests             | 143 passed |
| Browser axe accessibility check | Passed     |
| Oxlint                          | Passed     |
| Oxfmt                           | Passed     |
| `git diff --check`              | Passed     |

The exercise was saved to my GitHub fork on the `learning/weekview-accessible-name` branch.

These checks establish confidence in the scoped change, but they do not prove that the implementation is ready to merge upstream. The `labels.weekday` compatibility change and other design tradeoffs would still require maintainer consideration.

## 11. My Software Engineering Checklist

For future changes, I want to remember to ask:

1. **Problem:** Do I fully understand and know how to reproduce the issue?
2. **Architecture:** What components, utilities, and dependencies are involved?
3. **Public APIs:** Could this change affect existing consumers?
4. **Alternatives:** Is there a simpler solution?
5. **Testing:** What specific regressions should my tests prevent?
6. **Edge cases:** What happens with different inputs and configurations?
7. **Compatibility:** Am I changing behavior that developers rely on?
8. **Code review:** Have I examined the diff and questioned implementation decisions?
9. **Verification:** Have I run the relevant tests, lint, formatting, and other checks?
10. **Communication:** Can I clearly explain the solution and its tradeoffs?

## Final Reflection

My biggest takeaway is that **a small code change can create requirements and consequences far beyond the line being modified**.

Initially, I was focused on correcting an accessibility label. As I worked through the issue, I discovered interactions with tests, localization, component customization, public APIs, and backward compatibility.

I also learned that passing tests is not the end of the engineering process. Understanding the implications of a change and communicating its tradeoffs are equally important.

Going forward, I want to approach both open-source contributions and personal projects with a broader perspective:

**Don't just ask whether a change works. Ask what it affects, what assumptions it makes, how it can fail, and whether there's a better way to accomplish the same goal.**
