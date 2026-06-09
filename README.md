# Contribution 1: test(agent): cover Ollama provider HTTP response branches

**Contribution Number:** 1  
**Student:** Pedro Pacheco  
**Issue:** [InnerWarden #850: test(agent): cover Ollama provider HTTP response branches](https://github.com/InnerWarden/innerwarden/issues/850)  
**Status:** Phase I Complete

---

## Why I Chose This Issue

I chose this issue because it is a focused test-coverage task around an Ollama provider adapter, which connects to my interest in local LLM tooling while staying bounded enough for a first open-source contribution. The issue gives clear acceptance criteria, names the exact file involved, and says no real Ollama server or API key should be required.

This also fits what I want to practice during AI301: using AI tools to ramp into an unfamiliar codebase, understand existing test patterns, and contribute a small but useful PR. I have experience with C/C++, Python, LLM tooling, GitHub Issues, and debugging, so Rust is the main learning stretch here, but the scope is narrow enough to make that realistic.

---

## Understanding the Issue

### Problem Description

The `crates/agent/src/ai/ollama.rs` provider adapter has existing tests for construction and JSON extraction helpers, but it is missing coverage for important HTTP response and error-handling branches in `chat` and `decide`.

### Expected Behavior

The provider should be tested with mocked HTTP responses so successful responses, malformed responses, HTTP errors, and request failures are covered without needing a real Ollama server.

### Current Behavior

The issue reports that local coverage for this file is only 72.86% line coverage, with useful missing coverage in the `chat` and `decide` HTTP handling paths.

### Affected Components

- `crates/agent/src/ai/ollama.rs`
- Existing `#[cfg(test)] mod tests` in the same file
- Existing HTTP mocking pattern used elsewhere in the InnerWarden repository

---

## Reproduction Process

### Environment Setup

To be completed in Phase II after forking and cloning the InnerWarden repository locally.

### Steps to Reproduce

1. Fork and clone `InnerWarden/innerwarden`.
2. Install the project dependencies described in the repository README and CONTRIBUTING guide.
3. Run the existing Ollama provider tests.
4. Confirm the current test coverage gap described in the issue.

### Reproduction Evidence

- **Commit showing reproduction:** To be added in Phase II.
- **Screenshots/logs:** To be added in Phase II if useful.
- **My findings:** To be added after running the tests locally.

---

## Solution Approach

### Analysis

The likely solution is to add focused async unit tests under the existing test module in `crates/agent/src/ai/ollama.rs`. These tests should use the repository's existing HTTP mocking approach and avoid requiring a live Ollama server.

### Proposed Solution

Add tests for successful `chat` behavior, response parsing, malformed payloads, non-2xx responses, and request errors. Keep the change test-focused unless a small production-code adjustment is necessary to make the behavior testable.

### Implementation Plan

Using UMPIRE framework:

**Understand:** Confirm how `chat` and `decide` build requests, parse responses, and return errors.

**Match:** Find similar async HTTP mocking tests in the InnerWarden codebase and follow the same style.

**Plan:**
1. Run the existing `cargo test -p innerwarden-agent ai::ollama` command.
2. Inspect `crates/agent/src/ai/ollama.rs` and nearby provider tests.
3. Add mocked HTTP tests for successful and failing response branches.
4. Run the targeted test command again.
5. Run any formatting or lint commands required by the project.

**Implement:** Branch and commit links will be added during Phase II and Phase III.

**Review:** Check that the PR is small, test-focused, follows existing style, and does not require real Ollama credentials.

**Evaluate:** Verify with `cargo test -p innerwarden-agent ai::ollama` and any additional project checks recommended by maintainers.

---

## Testing Strategy

### Unit Tests

- [ ] `chat` posts to `/api/chat`, handles trailing slashes in `base_url`, and returns `message.content` from a successful mock response.
- [ ] `chat` returns a useful error for malformed or missing message content.
- [ ] `chat` returns an error for non-2xx HTTP responses.
- [ ] `decide` handles successful mock responses.
- [ ] `decide` covers malformed response and request-error branches.

### Integration Tests

No live integration test is planned for the initial PR because the issue explicitly says no real Ollama server or API key should be required.

### Manual Testing

Manual testing notes will be added after the local environment is set up in Phase II.

---

## Implementation Notes

### Week 1 Progress

Selected the issue, reviewed the issue requirements, created this contribution README, and prepared to comment on the GitHub issue and fork the repository.

### Code Changes

- **Files modified:** To be added during implementation.
- **Key commits:** To be added during implementation.
- **Approach decisions:** Keep the first PR focused on test coverage and avoid unrelated refactors.

---

## Pull Request

**PR Link:** To be added after implementation.

**PR Description:** To be drafted after the solution is implemented and verified.

**Maintainer Feedback:**

- To be updated as feedback is received.

**Status:** Not submitted yet.

---

## Learnings & Reflections

### Technical Skills Gained

To be updated as the contribution progresses.

### Challenges Overcome

To be updated as the contribution progresses.

### What I'd Do Differently Next Time

To be updated as the contribution progresses.

---

## Resources Used

- [InnerWarden issue #850](https://github.com/InnerWarden/innerwarden/issues/850)
- [InnerWarden repository](https://github.com/InnerWarden/innerwarden)
- CodePath AI301 issue selection instructions
