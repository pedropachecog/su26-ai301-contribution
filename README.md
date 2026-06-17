# Contribution 1: test(agent): cover Ollama provider HTTP response branches

**Contribution Number:** 1  
**Student:** Pedro Pacheco  
**Issue:** [InnerWarden #850: test(agent): cover Ollama provider HTTP response branches](https://github.com/InnerWarden/innerwarden/issues/850)  
**Status:** Phase II Complete Draft

---

## Why I Chose This Issue

I chose this issue because it lines up with what I want to practice: local LLM tooling, Rust, and test coverage. It also looked realistic for a first open-source contribution. The issue names the file, describes the missing branches, and says no real Ollama server or API key is required.

I have worked with C/C++, Python, LLM tooling, GitHub Issues, and debugging. Rust is the part I am newer to, so I wanted an issue where the Rust learning curve was real but the feature scope was not huge.

---

## Understanding the Issue

### Problem Description

The `crates/agent/src/ai/ollama.rs` provider adapter has existing tests for construction and JSON extraction helpers, but it is missing coverage for important HTTP response and error-handling branches in `chat` and `decide`.

The functions involved are `OllamaProvider::chat`, `OllamaProvider::decide`, and the existing test module in the same file.

### Expected Behavior

The provider needs mocked HTTP tests for successful responses, malformed responses, HTTP errors, and request failures. Those tests should not need a real Ollama server.

### Current Behavior

The issue reports 72.86% line coverage for this file, with missing coverage in the `chat` and `decide` HTTP handling paths.

My baseline matches that gap. The targeted Ollama test command passes, but it only runs six helper/constructor tests. None of them exercise the provider's HTTP behavior.

### Affected Components

- `crates/agent/src/ai/ollama.rs`
- Existing `#[cfg(test)] mod tests` in the same file
- Existing HTTP mocking pattern used elsewhere in the InnerWarden repository

---

## Reproduction Process

### Environment Setup

I started by setting up the project on my Windows machine. Rust itself installed correctly on Windows:

- `rustc 1.93.1`
- `cargo 1.93.1`
- `rustup 1.28.2`
- `rustfmt` and `clippy` installed through `rustup component add rustfmt clippy`

However, running the targeted test command natively on Windows did not reach the Ollama tests. The `innerwarden-agent` crate pulls in dependencies that are not currently Windows-clean in this checkout:

- `innerwarden-smm` failed around Unix/Linux-oriented `nix` usage.
- `tikv-jemalloc-sys` attempted to build jemalloc for `x86_64-pc-windows-msvc` and failed.

Because InnerWarden is a Linux/macOS security agent, I used Docker for the Linux Rust environment instead of using my personal WSL distributions.

The Docker setup exposes only the project work folder:

- Host folder exposed to Docker: `sandbox/docker-work`
- Repo inside that folder: `sandbox/docker-work/innerwarden`
- Container name: `innerwarden-rust`
- Docker volume for Rust/Cargo/build output: `innerwarden-rust-data`
- Container repo path: `/workspace/innerwarden`
- Rust/Cargo data path: `/rust-data`

The container does not mount my user directory, Windows drives, WSL home, Docker socket, or the parent sandbox folder. Only `docker-work` is exposed.

Inside Docker, the Linux Rust environment is:

- `rustc 1.96.0`
- `cargo 1.96.0`
- `rustup 1.29.0`

The command pattern I used for project commands from Windows was:

```powershell
docker exec innerwarden-rust bash -lc "cd /workspace/innerwarden && <command>"
```

### Steps to Reproduce

For this issue, reproduction means showing that the existing Ollama provider tests skip the HTTP branches named in the issue.

1. Fork `InnerWarden/innerwarden`.
2. Clone the fork and check out the issue branch:

   ```bash
   git clone --branch test/issue-850-ollama-provider-http-branches --single-branch https://github.com/pedropachecog/innerwarden.git
   cd innerwarden
   ```

3. Install the Rust formatting and linting components:

   ```bash
   rustup component add rustfmt clippy
   ```

4. Run the existing Ollama provider tests:

   ```bash
   cargo test -p innerwarden-agent ai::ollama -- --nocapture
   ```

5. Observe that the test suite passes, but only six Ollama tests run.
6. Inspect the existing Ollama tests and confirm they cover helper/constructor behavior only:

   - `extract_json_bare_object`
   - `extract_json_strips_prose`
   - `extract_json_returns_none_for_no_braces`
   - `new_uses_supplied_values`
   - `new_stores_api_key`
   - `url_construction_strips_trailing_slash`

7. Confirm that the baseline tests do not mock `/api/chat`, do not call `OllamaProvider::chat`, and do not call `OllamaProvider::decide`.

**Expected:** The baseline shows that the Ollama provider tests are limited to helpers, constructor storage, API key storage, and URL construction. The issue remains valid because the HTTP response branches are not covered.

**Actual:** The targeted test command passed with six Ollama tests, and all six were helper/constructor tests. No baseline test exercised a mocked `/api/chat` response, `chat`, or `decide`.

### Branch Link

[test/issue-850-ollama-provider-http-branches](https://github.com/pedropachecog/innerwarden/tree/test/issue-850-ollama-provider-http-branches)

### Reproduction Evidence

- **Working branch:** [test/issue-850-ollama-provider-http-branches](https://github.com/pedropachecog/innerwarden/tree/test/issue-850-ollama-provider-http-branches)
- **Baseline commit:** `4e947149` (`feat(ctl,agent): notify discord command + dashboard card (spec 078 P3b) (#1029)`)
- **Targeted command:**

  ```bash
  cargo test -p innerwarden-agent ai::ollama -- --nocapture
  ```

- **Docker command used from Windows:**

  ```powershell
  docker exec innerwarden-rust bash -lc "cd /workspace/innerwarden && cargo test -p innerwarden-agent ai::ollama -- --nocapture"
  ```

- **Observed result:**

  ```text
  running 6 tests
  test ai::ollama::tests::extract_json_returns_none_for_no_braces ... ok
  test ai::ollama::tests::extract_json_bare_object ... ok
  test ai::ollama::tests::extract_json_strips_prose ... ok
  test ai::ollama::tests::url_construction_strips_trailing_slash ... ok
  test ai::ollama::tests::new_uses_supplied_values ... ok
  test ai::ollama::tests::new_stores_api_key ... ok

  test result: ok. 6 passed; 0 failed; 0 ignored; 0 measured; 3848 filtered out
  ```

- **My findings:** The existing test file verifies JSON extraction helpers, constructor storage, API key storage, and URL string construction. It does not verify the real HTTP response branches in `chat` or `decide`, which matches the issue.
- **Coverage tooling:** I did not run `cargo tarpaulin` during Phase II. The targeted test output and source inspection were enough for this reproduction step.

---

## Solution Approach

### Analysis

The root cause is missing test coverage, not a runtime bug I can trigger manually. The Ollama provider has several branches in its HTTP paths:

- It posts to `/api/chat`.
- It optionally sends bearer auth.
- It handles non-2xx HTTP responses.
- It parses Ollama-style JSON responses.
- It rejects empty content.
- `decide` also asks for JSON output, extracts JSON from prose, handles auth failures, and gives a local model-not-found hint.

The existing tests do not drive those branches. They stop at helper and constructor coverage. The code already has the branches named in the issue, so the first PR should add tests before changing provider behavior.

### Proposed Solution

Add async tests under the existing Ollama test module. Use mocked HTTP responses, not a live Ollama server and not a real API key. Keep production changes minimal.

### Implementation Plan

Using UMPIRE framework:

**Understand:** `chat` and `decide` both build POST requests to the Ollama `/api/chat` endpoint, but the current test suite does not exercise those requests. The missing coverage is in HTTP success/error handling and response parsing.

**Match:** Follow the existing async `mockito` style already used in InnerWarden tests. The agent crate already has `mockito = "1"` as a dev dependency, and other tests use `mockito::Server::new_async().await`, `server.mock(...)`, `create_async().await`, and `assert_async().await`.

**Plan:**
1. Add a mocked successful `chat` test that posts to `/api/chat` and returns `message.content`.
2. Include trailing slash handling for `base_url` through a mocked request path instead of checking only string formatting.
3. Add a `chat` test that asserts `Authorization: Bearer <key>` is sent when an API key is configured.
4. Add `chat` error tests for non-2xx responses, malformed JSON responses, and empty `message.content`.
5. Add a `decide` success test where the model wraps the JSON object in prose and `extract_json` still allows parsing.
6. Add `decide` error tests for auth failures, empty content, malformed JSON/no-JSON content, and the local model-not-found hint.
7. Avoid unrelated refactors. Extract a small test helper only if the repeated mock setup gets awkward.

**Implement:** Implementation will happen in Phase III on the existing working branch.

**Review:** Self-review against InnerWarden's `CONTRIBUTING.md`: small PR, conventional commit format, no real Ollama credentials, no live network dependency in tests, and no unrelated code churn.

**Evaluate:** Run the targeted Ollama test command after adding tests. Then run `make check`. If time/resources allow, run `make test` before opening a PR.

---

## Phase II Checklist

- [x] Local setup documented, including Windows issue and Docker Linux setup.
- [x] Numbered reproduction steps written under Reproduction Process.
- [x] Branch link added.
- [x] Reproduction evidence recorded with command output.
- [x] Solution plan written using UMPIRE.
- [x] No Phase III implementation started.

---

## Testing Strategy

### Unit Tests

- [ ] `chat` posts to `/api/chat`, handles trailing slashes in `base_url`, and returns `message.content` from a successful mock response.
- [ ] `chat` sends bearer auth when `api_key` is configured.
- [ ] `chat` returns a parse error for malformed Ollama JSON.
- [ ] `chat` returns an error for empty message content.
- [ ] `chat` returns an error for non-2xx HTTP responses.
- [ ] `decide` handles successful mock responses, including JSON wrapped in prose.
- [ ] `decide` covers authentication failure messaging.
- [ ] `decide` covers empty content and no-JSON/malformed decision content.
- [ ] `decide` covers the local model-not-found hint.

### Integration Tests

No live integration test is planned for the initial PR. The issue says no real Ollama server or API key should be required.

### Manual Testing

No manual Ollama server testing is planned for this issue. The acceptance criteria call for mocked HTTP tests.

---

## Implementation Notes

### Week 1 Progress

Selected the issue, reviewed the issue requirements, created this contribution README, and prepared to comment on the GitHub issue and fork the repository.

### Phase II Progress

Set up a repeatable Linux Rust environment through Docker on Windows, published the working branch, and reproduced the coverage gap by running the targeted Ollama provider tests. The baseline passes, but it only runs six helper/constructor tests and does not exercise the HTTP branches named in the issue.

### Code Changes

- **Files expected to be modified in Phase III:** `crates/agent/src/ai/ollama.rs`
- **Key commits:** To be added during implementation.
- **Approach decisions:** Keep the first PR limited to test coverage and avoid unrelated refactors.

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

I learned how this Rust workspace is organized around crates, how `cargo test -p <crate>` selects a package in a workspace, and why even a targeted Rust test still needs the whole crate and its dependencies to compile first.

### Challenges Overcome

The main setup challenge was that native Windows Rust worked, but this project did not compile cleanly on Windows because some dependencies assume Linux/macOS behavior. I used Docker as a contained Linux Rust environment instead of relying on my personal WSL setup.

### What I'd Do Differently Next Time

For Rust projects that describe themselves as Linux/macOS system tools, I would check the supported OS and CI configuration earlier, then start from a Linux dev environment instead of first trying native Windows.

---

## Resources Used

- [InnerWarden issue #850](https://github.com/InnerWarden/innerwarden/issues/850)
- [InnerWarden repository](https://github.com/InnerWarden/innerwarden)
- [InnerWarden contribution guide](https://github.com/InnerWarden/innerwarden/blob/main/CONTRIBUTING.md)
- CodePath AI301 Phase II instructions
