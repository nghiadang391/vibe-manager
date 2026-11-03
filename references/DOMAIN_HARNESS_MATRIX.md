# Domain-Adaptive Verification & Architecture Matrix

Use this reference when executing **Phase 1 (Brief & Stack Recommendation)** and **Phase 4 (Automated Verification Harness)** across different engineering domains.

---

## 1. SystemC / Hardware / RTL Modeling

- **Mental Model**: Discrete-event simulation modeling hardware concurrency, clocks, signals (`sc_signal`), ports (`sc_in`/`sc_out`), and Transaction-Level Modeling (TLM-2.0) before or alongside RTL synthesis.
- **Recommended Stack**:
  - Language: C++17 / C++20 with Accellera SystemC (`systemc` / `tlm`)
  - Build System: CMake + Ninja (`ctest`)
- **Automated Quality Gates (`make verify`)**:
  - **Compiler Flags**: `-Wall -Wextra -Werror -Wpedantic -Wshadow -Wconversion`
  - **Static Analysis**: `clang-tidy` (checks for memory leaks, uninitialized signals, thread-unsafe globals) + `cppcheck`
  - **Runtime Sanitizers**: AddressSanitizer (`-fsanitize=address,undefined`) during simulation testbenches
  - **Test Harness**: Self-checking SystemC Testbench (`sc_main` stimulus generator + golden reference scoreboard + `SC_REPORT_ERROR` assertions) + GoogleTest (`gtest`) for algorithmic sub-blocks
  - **Staging / Artifacts**: Automated VCD waveform trace generation (`sc_create_vcd_trace_file`) + assertion log verification
- **Top High-Level Risks for Manager**:
  1. **Abstraction Level Trade-off**: Cycle-Accurate (CA) vs. Loosely-Timed (LT) / Approximately-Timed (AT) TLM-2.0 (speed vs. timing precision).
  2. **Delta-Cycle Non-Determinism**: Processes writing/reading shared variables instead of `sc_signal`/`sc_fifo`, causing order-dependent simulation bugs.
  3. **Infinite Simulation Loops**: Missing `wait()` in `SC_THREAD` or combinational feedback loops freezing simulation.

---

## 2. Blockchain / Smart Contracts (EVM, Solana, Move)

- **Mental Model**: Deterministic, adversarial state machines where every computation costs gas/compute units and deployed code is immutable or high-risk to upgrade.
- **Recommended Stack**:
  - EVM: Solidity + **Foundry** (`forge`, `cast`, `anvil`)
  - Solana: Rust + Anchor + Bankrun/LiteSVM
- **Automated Quality Gates (`make verify`)**:
  - **Static & Security Analysis**: `slither`, `aderyn`, `solhint`, compiler warnings as errors
  - **Automated Testing**: Unit tests + **Stateful Fuzz Testing** + **Invariant Testing** (`forge test`)
  - **Coverage & Gas Gate**: `forge coverage` (target >= 95% branch coverage on state-mutating functions) + `forge snapshot` (gas regression check)
  - **Staging Environment**: Local mainnet/testnet fork via `anvil` with realistic token/oracle state
- **Top High-Level Risks for Manager**:
  1. **Reentrancy & External Call Ordering**: Strictly enforce Checks-Effects-Interactions (CEI) and reentrancy guards.
  2. **Access Control & Upgradeability**: Who holds admin keys, timelocks, or proxy upgrade rights?
  3. **Precision Loss & Economic Exploits**: Integer division rounding direction (always round in favor of the protocol) and oracle manipulation (flash-loan attacks).

---

## 3. Web Applications (Full-Stack / Frontend)

- **Mental Model**: Client-server reactive UI with state synchronization, authentication boundaries, and asynchronous API contracts.
- **Recommended Stack**:
  - TypeScript (`strict: true`), Next.js / React / Vite, TailwindCSS, Zod (runtime I/O validation)
- **Automated Quality Gates (`make verify`)**:
  - **Type Checker**: `tsc --noEmit` (`strict: true`, `noUncheckedIndexedAccess: true`)
  - **Linter**: ESLint / Biome (ban `any`, enforce exhaustive hooks and boundary imports)
  - **Unit / Integration Tests**: Vitest + Testing Library
  - **End-to-End (Staging Gate)**: Playwright headless browser tests verifying critical user flows
- **Top High-Level Risks for Manager**:
  1. **Client vs. Server Trust Boundary**: Never trust client-side validation; enforce schema (Zod) and auth checks on every server endpoint.
  2. **State Desynchronization & Race Conditions**: Stale cache, duplicate form submissions, and optimistic UI rollbacks.
  3. **Bundle & Render Performance**: Waterfall network requests and unindexed database queries.

---

## 4. Backend / Distributed Systems / APIs

- **Mental Model**: Concurrent request processing, persistent storage with ACID/eventual consistency guarantees, and resilient external integrations.
- **Recommended Stack**:
  - Go / Rust / TypeScript (Node/Bun) / Python (FastAPI + Pydantic v2), PostgreSQL, Docker Compose
- **Automated Quality Gates (`make verify`)**:
  - **Static Analysis**: `golangci-lint` (Go), `cargo clippy -- -D warnings` (Rust), `mypy --strict` + `ruff` (Python)
  - **Contract & Integration Tests**: API schema validation (OpenAPI/Protobuf) + real database integration tests (Testcontainers / Docker)
  - **Concurrency / Race Detector**: `go test -race` or concurrent load smoke test
- **Top High-Level Risks for Manager**:
  1. **Idempotency & Partial Failures**: What happens when a client retries a payment/write request after a network timeout?
  2. **Database Transactions & Locking**: Deadlocks, N+1 queries, and missing indexes under load.
  3. **Secret & Auth Leakage**: Hardcoded credentials, missing rate limits, or broken object-level authorization (BOLA).

---

## 5. Embedded / Systems C & C++

- **Mental Model**: Resource-constrained execution (strict RAM/ROM budgets, real-time deadlines, hardware interrupts, no garbage collector).
- **Recommended Stack**:
  - C11/C17 or Modern C++ (no exceptions/RTTI/dynamic heap after init), CMake, QEMU or host-based HAL mock
- **Automated Quality Gates (`make verify`)**:
  - **Compiler & Static Analysis**: `-Wall -Wextra -Werror`, `clang-tidy`, `cppcheck`, MISRA/CERT-C rule checks
  - **Host Unit Testing**: Unity / Ceedling / GoogleTest with mocked Hardware Abstraction Layer (HAL)
  - **Memory & Bounds Check**: Valgrind / ASan / UBSan on host test runner + binary size check (`size`)
- **Top High-Level Risks for Manager**:
  1. **ISR (Interrupt) Safety & Volatile State**: Shared data between main loop and interrupts without atomic/critical sections.
  2. **Stack Overflow & Heap Fragmentation**: Unbounded recursion or dynamic allocation (`malloc`) in long-running firmware.
  3. **Watchdog & Hardware Hang Recovery**: Missing timeouts on busy-wait hardware register loops (`while (!(REG & READY))`).

---

## 6. Data Pipelines & AI / ML Systems

- **Mental Model**: Probabilistic or batch/streaming data transformations where silent data corruption ("garbage in, garbage out") is worse than a crash.
- **Recommended Stack**:
  - Python 3.11+, `uv`, Pydantic / Pandera (data schema validation), Pytest, Ruff, Mypy
- **Automated Quality Gates (`make verify`)**:
  - **Type & Lint**: `ruff check` + `mypy --strict`
  - **Data Contract Tests**: Schema validation on input/output DataFrames/JSONs (null rates, value ranges, distribution sanity checks)
  - **Deterministic Evaluation**: Golden dataset regression suite with fixed random seeds
- **Top High-Level Risks for Manager**:
  1. **Silent Schema / Data Drift**: Upstream data changes format or units without throwing an error.
  2. **Train-Serve Skew / Data Leakage**: Future information leaking into training/evaluation sets.
  3. **Cost & Latency Spikes**: Unbounded token consumption, missing caching, or O(N²) memory blowups on large datasets.
