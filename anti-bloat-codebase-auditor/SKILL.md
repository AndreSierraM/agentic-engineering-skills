---
name: anti-bloat-codebase-auditor
description: Use when the user explicitly asks to scan, audit, review, or analyze a codebase or scoped path for unnecessary code, duplicated logic, over-engineering, inflated implementations, near-duplicate functions, redundant abstractions, dead wrappers, excessive branching, or code that could be solved with fewer clearer lines. Works without Git and does not require branches, commits, PRs, or diffs. Do not use for bare skill activation without an audit request.
license: MIT
---

# Codebase Bloat Auditor

## Mission

Find code that is larger, more duplicated, more abstract, more fragmented, or more complex than the product behavior requires.

The goal is not to rewrite the whole project. The goal is to identify high-confidence simplification opportunities that a skeptical senior engineer would accept.

Prefer deletion, inlining, reuse, consolidation, and clearer local logic over new abstractions.

## Operating Law

After a concrete audit request, scan the agreed scope for unnecessary expansion, then report the smallest safe simplifications with evidence, file references, risk, and verification requirements.

## Activation

Use this skill only when the user requests a codebase bloat audit.

A concrete audit task includes at least one of:

* an explicit ask to scan, audit, review, or analyze a codebase, project, repo, or path for bloat, duplication, over-engineering, or simplification;
* phrases such as “analyze all the code”, “find bloated code”, “find duplicated code”, “audit the project for AI-generated bloat”, “run anti-bloat on this repo”, “review the whole codebase for simplification”, or “scan without Git”;
* a named directory, package, module, or area to audit (for example `src/api`);
* an invocation like `Run anti-bloat-codebase-auditor on this project`.

Bare activation is not a task.

If the user only says to enable, activate, use, or apply this skill/mode without requesting an audit, do not build a filesystem inventory, run `find`/`fd`/`rg` across the project, read source files, or produce an audit report.

For bare activation, respond only with a compact acknowledgement such as:

```text
Codebase bloat auditor ready. Request an audit with a scope (whole project or a path such as src/services). I will not scan files or run repo-wide commands until you do.
```

Then stop.

When the user requests an audit but does not specify a path, use the current workspace/project root as the audit scope.

When the user specifies a path, limit inventory and signal collection to that scope unless broader context is required to interpret a finding.

## When Not to Use

Do not use this skill for:

* normal bug fixes unless the user asks for bloat analysis;
* feature implementation;
* style-only formatting;
* dependency update review unless bloat or duplication is the focus;
* generated code cleanup unless the user explicitly includes generated files;
* legal, product, or UX review unrelated to code structure.

## No Git Requirement

This skill must work without Git.

Do not require:

* `git status`
* `git diff`
* branches
* commits
* PR metadata
* merge bases
* remote repositories

Git may be used only if the user explicitly asks for a branch, PR, or diff audit.

Default audit mode is filesystem-based.

## Default Safety Rules

By default, this skill is read-only.

Do not modify files unless the user explicitly asks for patches or refactoring.

Do not delete files.

Do not rewrite broad areas.

Do not reformat unrelated files.

Do not create new abstractions as part of the audit.

The default output is a ranked report.

## Default Scope

Include likely source code and tests.

Exclude by default:

* `node_modules`
* `.git`
* `vendor`
* `dist`
* `build`
* `coverage`
* `.next`
* `.nuxt`
* `target`
* `out`
* `tmp`
* `cache`
* generated files
* lockfiles
* minified files
* binary files
* snapshots unless specifically relevant
* migrations unless the user explicitly asks
* vendored SDKs
* generated API clients
* compiled artifacts

If the project has ignore files such as `.gitignore`, `.ignore`, `.eslintignore`, or tool-specific excludes, respect them as hints, but do not depend on Git.

## Audit Objectives

Find and classify:

### 1. Exact duplication

Code blocks, functions, types, constants, tests, fixtures, or utilities that are identical or almost identical.

Look for:

* copy-pasted functions;
* repeated validation logic;
* repeated mapping logic;
* repeated API error handling;
* repeated test setup;
* repeated constants;
* repeated conditional trees;
* repeated type definitions.

### 2. Semantic duplication

Different code that appears to do the same thing.

Look for:

* functions with different names but equivalent behavior;
* utilities that wrap the same primitive;
* similar parsing or formatting logic;
* multiple implementations of the same normalization;
* duplicated permission checks;
* duplicated request/response shaping;
* duplicated error mappers.

### 3. Near-duplicate code

Code that differs only by:

* variable names;
* literal strings;
* field names;
* one or two branches;
* minor type names;
* different wrapper layers;
* repeated structure with small local variations.

Recommend consolidation only when the shared behavior is real and stable.

Do not force abstraction when duplication is clearer than indirection.

### 4. Unnecessary expansion

Find code that is much longer than necessary.

Examples:

* verbose conditionals that can be direct expressions;
* manual loops where existing primitives are clearer;
* multi-step transformations that can be collapsed;
* unnecessary temporary variables;
* needless classes for data-only behavior;
* excessive object spreading;
* repeated guard clauses with same outcome;
* overbuilt factories;
* needless async wrappers;
* pointless try/catch blocks;
* functions that only call another function.

### 5. Abstraction bloat

Find abstractions with no current justification.

Examples:

* generic helpers with one call site;
* interfaces with one implementation;
* classes with no state or polymorphism;
* strategy/factory/provider patterns for one behavior;
* config options with one value;
* wrapper modules that hide simple imports;
* hooks/services/managers that only forward calls;
* “future-proof” extension points with no current use.

### 6. Dead or low-value code

Find likely dead code, but label it as “likely” unless verified.

Look for:

* exported functions with no internal use;
* files not imported anywhere;
* unused types;
* unused constants;
* unreachable branches;
* old compatibility paths;
* duplicate legacy paths;
* stale feature flags;
* TODO-driven scaffolding that is not wired in.

Do not claim code is dead if it may be externally imported, framework-loaded, or reflection-based.

### 7. Bad complexity

Find code whose complexity is disproportionate to behavior.

Look for:

* deeply nested conditionals;
* long functions with multiple responsibilities;
* broad catch blocks;
* hidden global state;
* mutable shared state;
* fragile ordering dependencies;
* confusing boolean flags;
* functions with too many parameters;
* tests that require excessive setup for simple behavior.

### 8. Test bloat

Find tests that add maintenance cost without proving behavior.

Look for:

* tests that only verify mocks;
* snapshot spam;
* duplicated test cases;
* tests coupled to private implementation;
* huge fixture factories;
* random sleeps;
* over-mocked integration paths;
* repeated setup that can be local and simpler.

## Audit Procedure

### Phase 0: Confirm audit mode

Proceed only after a concrete audit task (see Activation). If the user only enabled the skill, stop per Activation.

Classify the task as:

```text
Task: Codebase bloat audit
Mode: Read-only by default
Scope: user-provided path, else current workspace/project root
Git: not required
```

If multiple roots are visible, choose the most likely project root based on source/config files. If this is ambiguous and proceeding may inspect the wrong project, ask one concise blocking question instead of scanning everything.

### Phase 1: Build a scoped filesystem inventory

Within the audit scope only, inspect project structure without Git.

Identify:

* languages and frameworks;
* main source directories;
* test directories;
* generated/vendor/build directories to exclude;
* package/module boundaries;
* obvious entry points;
* major utility/shared folders.

Use fast local search tools when available, such as:

* `find`
* `fd`
* `rg`
* language-aware tooling
* AST parsers
* typecheck/lint metadata

Do not depend on any single tool existing.

Do not expand beyond the audit scope by default. Do not inventory unrelated packages, vendored trees, or the whole monorepo when the user scoped the audit to one path.

### Phase 2: Collect bloat signals

Within the audit scope, collect candidate signals before making judgments.

Useful signals include:

* largest files;
* longest functions;
* files with many exports;
* helpers with one call site;
* similar function names;
* repeated constants;
* repeated error strings;
* repeated validation branches;
* repeated condition structures;
* similar import sets;
* duplicated test setup;
* near-identical code blocks;
* modules with wrapper-only functions;
* files with high churn-like symptoms, even without Git;
* code paths that implement the same behavior in parallel.

Prefer AST or structural comparison when possible. Fall back to text search when needed.

### Phase 3: Deduplicate findings

Before reporting, collapse overlapping findings.

Do not report the same problem five times.

Group related findings under one root cause when appropriate.

Example:

```text
Repeated request validation exists in 7 handlers.
```

is better than seven separate low-value findings.

### Phase 4: Prove each finding

For every finding, include evidence.

Evidence may include:

* exact file paths;
* function/component/type names;
* duplicated snippets summarized, not copied in full;
* call-site count;
* why the code appears redundant;
* what behavior seems shared;
* why simplification is safe or risky;
* verification needed before changing it.

Never claim equivalence solely because code “looks similar.” Distinguish:

* exact duplicate;
* near duplicate;
* likely semantic duplicate;
* suspicious but unverified.

### Phase 5: Rank by value

Rank findings by expected payoff and confidence.

Use this scale:

```text
Impact:
- High: reduces repeated logic, bug surface, security risk, or major maintenance burden.
- Medium: simplifies local code or removes a confusing abstraction.
- Low: minor cleanup, mostly readability.

Confidence:
- High: directly evidenced by identical or clearly equivalent code.
- Medium: strong similarity, but behavior needs tests before merging.
- Low: suspicious pattern; requires human/domain confirmation.

Risk:
- Low: local, testable, no public contract impact.
- Medium: shared code, multiple call sites, behavior must be verified.
- High: auth, payments, data, migrations, security, public APIs, concurrency, or framework magic.
```

Prioritize high-impact, high-confidence, low/medium-risk findings.

### Phase 6: Recommend minimal fixes

For each finding, recommend the smallest safe action.

Allowed recommendations:

* delete unused wrapper;
* inline one-use helper;
* merge duplicate functions;
* replace custom logic with existing local primitive;
* move repeated literals to existing constant;
* consolidate repeated validation;
* remove unused option/config;
* simplify conditional;
* split long function only if responsibilities are truly separate;
* add focused regression tests before refactor;
* leave duplication alone if abstraction would be worse.

Do not recommend broad rewrites.

Do not recommend new architecture unless the current code proves it is necessary.

Do not introduce dependencies just to reduce line count.

### Phase 7: Human rewrite comparison

For important findings, include:

```text
Human-handwritten target:
- Concepts needed:
- Files likely touched:
- Abstractions avoided:
- Expected shape:
```

This comparison should describe what a strong engineer would likely have written by hand.

Use it to avoid preserving AI-generated bloat out of politeness.

### Phase 8: Verification plan

Because the default mode is read-only, provide a verification plan instead of claiming verified behavior.

For each proposed change, state the narrowest verification needed:

* focused unit test;
* existing test file to run;
* typecheck;
* lint;
* build;
* manual smoke test;
* authorization negative test;
* snapshot update only if unavoidable;
* before/after behavior check.

If no tests appear to exist, say so and recommend the smallest test needed.

## Output Format

Use this format:

```text
Codebase bloat audit

Scope:
- Root:
- Included:
- Excluded:
- Git used: No

Executive summary:
- High-confidence simplification opportunities:
- Highest-risk area:
- Biggest duplication cluster:
- Best first refactor:

Findings:

1. [Title]
   Category:
   Impact:
   Confidence:
   Risk:
   Evidence:
   - file/path.ext:functionName
   - file/path.ext:otherFunctionName

   Why this is bloat:
   - ...

   Minimal fix:
   - ...

   Human-handwritten target:
   - Concepts needed:
   - Files likely touched:
   - Abstractions avoided:
   - Expected shape:

   Verification:
   - ...

2. ...

Not recommended:
- Things that look duplicate but should probably stay separate:
  - ...

Unverified / limitations:
- ...

Recommended next step:
- ...
```

## Patch Mode

Only enter patch mode if the user explicitly asks to modify code.

Patch mode rules:

* Start with the highest-confidence, lowest-risk finding.
* Change the fewest files possible.
* Prefer one refactor at a time.
* Preserve behavior.
* Add or run focused tests.
* Do not combine unrelated cleanup.
* Do not reformat whole files.
* Do not introduce new abstractions unless consolidation has at least two real call sites and improves clarity.
* After patching, perform an anti-bloat compression pass.

Patch mode final response:

```text
Summary:
- ...

Changed:
- ...

Why this is minimal:
- ...

Verification:
- Passed:
- Not run:

Risk:
- ...

Remaining bloat candidates:
- ...
```

## False Positive Controls

Do not mark code as bloat when duplication is justified by:

* different bounded contexts;
* intentionally separate public APIs;
* security-sensitive separation;
* framework-required shape;
* generated code;
* migrations/history;
* test clarity;
* performance-critical specialization;
* different lifecycle or ownership;
* external consumers;
* regulatory/audit requirements.

When unsure, label the finding as “possible” and state what must be checked.

## Stop-the-Line Conditions

Warn before recommending consolidation if the candidate touches:

* authentication;
* authorization;
* payments;
* personal data;
* encryption;
* secrets;
* data deletion;
* migrations;
* public API contracts;
* distributed locks;
* concurrency;
* file system writes;
* production configuration;
* generated code that could overwrite manual edits.

For these areas, prefer review findings and verification plans over direct refactor proposals.

## Forbidden Behaviors

Do not:

* treat skill activation as permission to scan the repository, run search commands, or read files without an explicit audit request;
* run `git status`, compare branches, or inspect PR metadata unless the user explicitly asked for a Git-, branch-, or PR-scoped audit;
* produce a full audit report after bare activation;
* claim bare activation or naming this skill alone is a concrete audit task;
* modify files during a read-only audit unless the user explicitly asked for patches;
* recommend broad rewrites or new architecture when a minimal consolidation suffices.

## Bloat Scoring

For each proposed simplification, assign a bloat score to the current code:

```text
Bloat score:
- 0: no issue; leave it alone.
- 1: minor verbosity; optional cleanup.
- 2: real local bloat; simplify when nearby.
- 3: harmful duplication or abstraction; should fix.
- 4: architecture bloat causing correctness, security, or maintenance risk; prioritize.
```

Only report score 2+ by default.

Mention score 1 only if the user asks for exhaustive cleanup.

## Final Checklist

Before answering, confirm internally:

```text
- Confirmed a concrete audit request (not bare activation only).
- Used filesystem-based audit, not Git.
- Stayed within the agreed audit scope.
- Excluded generated/vendor/build artifacts.
- Distinguished exact, near, and semantic duplication.
- Avoided style-only complaints.
- Avoided broad rewrite recommendations.
- Ranked findings by impact, confidence, and risk.
- Included file/function evidence.
- Included minimal fixes.
- Included verification plan.
- Disclosed uncertainty.
```
