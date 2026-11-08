<!-- Generated from a private repo by export-public.py; edit the source there, not here. -->

# Pattern — Canon TDD: test list first, one test at a time

**Use when** building new behaviour on a sizable ticket (a new endpoint, business rule, cross-cutting concern). **Status: adopted** (2026-11-03), after a trial on the day sheet of Repair Table, a repair café booking app (two PRs).

Source: Kent Beck, [Canon TDD](https://newsletter.kentbeck.com/p/canon-tdd) (Dec 2023).

## The steps

1. **Test list.** Write the behaviour variants before any code: the basic case, failure cases, existing behaviour that must not break. Behaviour only, no implementation design. For a sizable ticket, post it on the PR description (or a ticket comment) and invite the person who owns the domain to add cases — they know the domain cases (e.g. "a second café on a fresh page load").
2. **One test.** Turn exactly one item into a real test: setup, call, assertions. This is where interface design happens.
3. **Make it pass**, along with every earlier test. Run only the targeted test in the loop (`mvn -Dtest=Class#method`, `vitest <file>`). The red must fail for the expected reason (missing behaviour, not a typo or compile error). Tick the item; add anything discovered to the list. If a discovery invalidates the work so far, consider restarting with a different test order.
4. **Refactor** — a named step, after green. This is where implementation design happens. Only what this session needs; duplication is a hint, not an order to abstract.
5. **Repeat** until the list is empty. Full suite before each commit: green commits only. **Gate the commit on the runner's exit code** (`npx vitest run --silent; [ $? -eq 0 ] && git commit …`), not on grepping its output: in the trial a grep-and-`&&` chain committed a red test once.

## Avoid

- Turning the whole list into tests up front ("batch red") — early passes make speculative tests wrong, and a long all-red stretch hides design flaws until everything meets.
- Pasting actual output in as the expected value.
- Deleting or weakening assertions to get green.
- Refactoring while still getting to green.
- Tests without assertions.

## Showing the work

Commits bundle tests and code, so git can't show the order. Put the test list, with ticks and the items added on the way, in the PR description.

## Exemptions

Mechanical refactors (signature propagation), one-liners and config: change-then-fix is fine.

## Why

Observed on the same app in October 2026: the agent wrote all tests per slice, saw one red, implemented everything, saw one green. Red-first held, but no test list existed, and two order/interaction defects reached dev and were found by hand (café header installed after the first calls; a 404 thrown before Spring Security had no CORS headers). Both are the kind of case a behaviour list surfaces.
