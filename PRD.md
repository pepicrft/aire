# Aire Language - Product Requirements Document (PRD)

> **Status**: Draft v0.1  
> **Date**: 2026-01-18  
> **Author**: Codex (for Pedro Piñera)  
> **Language**: Aire - Statically-typed agentic workflow language for BEAM

---

## 1. Executive Summary

**Aire** is a new statically-typed programming language designed for building agentic workflows on the Erlang BEAM virtual machine. It combines:

- **Instant incremental compilation** with Bazel-style remote caching
- **Static type system** for excellent developer experience and agent feedback
- **First-class concurrency** leveraging Erlang's actor model
- **Clean syntax** bridging the gap between Elixir's ergonomics and Gleam's type safety

The language targets developers building multi-agent systems, distributed services, and fault-tolerant applications who want stronger typing guarantees without sacrificing the BEAM's legendary reliability.

---

## 2. Problem Statement

### 2.1 Current Landscape

| Language | Pros | Cons |
|----------|------|------|
| **Erlang** | Battle-tested, mature BEAM citizen | Dynamic typing, verbose syntax, steep learning curve |
| **Elixir** | Productive, excellent tooling, large ecosystem | Dynamic typing, may need more compile-time safety |
| **Gleam** | Static typing, clean syntax, great ergonomics | Younger ecosystem, less focus on agentic workflows |

### 2.2 Gaps Aire Addresses

1. **No BEAM language with static types AND agentic workflow focus** - Gleam has types but isn't designed around agent workflows; Elixir/OTP are agent-focused but dynamically typed

2. **Compilation speed at scale** - As projects grow, compilation times suffer; no BEAM language has built-in remote caching like Bazel

3. **Type-safe agent communication** - Currently, message passing between processes is dynamic; Aire could provide typed channels and message contracts

4. **IDE/agent feedback** - Static types enable better autocomplete, refactoring, and AI assistant integration

---

## 3. Target Users & Use Cases

### 3.1 Primary Users

- **Agent Framework Authors** - Building libraries like LangChain, AutoGPT, but for BEAM
- **Distributed Systems Engineers** - Need type guarantees across node boundaries
- **Erlang/Elixir Developers** - Want static typing without leaving the BEAM
- **AI/ML Engineers** - Building agentic AI systems with reliability requirements

### 3.2 Use Cases

- Multi-agent orchestration platforms
- Distributed task queues with type-safe workers
- Fault-tolerant microservices
- Real-time streaming pipelines with typed transformations
- Chatbot/conversational AI backends

---

## 4. Technical Architecture

### 4.1 High-Level Stack

```
┌─────────────────────────────────────────────────────────────┐
│                      Aire Source (.aire)                     │
└─────────────────────────┬───────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                    Aire Compiler (Rust)                      │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐  │
│  │  Parser     │→│ Type Checker│→│  Erlang Code Gen     │  │
│  └─────────────┘  └─────────────┘  └─────────────────────┘  │
└─────────────────────────┬───────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                   Erlang BEAM VM                             │
│         (.beam files, OTP, supervision trees)                │
└─────────────────────────────────────────────────────────────┘
```

### 4.2 Build System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     Aire Build CLI                           │
├─────────────────────────────────────────────────────────────┤
│  • Incremental compilation graph                             │
│  • Content-addressed caching (SHA-256)                       │
│  • Remote cache sync (configurable backend)                  │
│  • Deterministic builds                                       │
└─────────────────────────┬───────────────────────────────────┘
                          │
          ┌───────────────┼───────────────┐
          ▼               ▼               ▼
    ┌──────────┐   ┌──────────┐    ┌──────────┐
    │ Local    │   │ Remote   │    │ Build    │
    │ Cache    │   │ Cache    │    │ Graph    │
    │ (~/.aire) │   │ (S3/GCS) │    │ (.aire)  │
    └──────────┘   └──────────┘    └──────────┘
```

### 4.3 Key Technical Decisions

- **Compiler**: Rust for performance and safety
- **Type System**: Based on row polymorphism (like OCaml/Gleam)
- **Code Generation**: Direct AST → Erlang AST → BEAM bytecode
- **Remote Cache**: Content-addressed, pluggable backends (S3, GCS, custom HTTP)

---

## 5. Design Questions for Pedro

To finalize the language design, please answer the following questions. These will shape the syntax, type system, and tooling.

### 5.1 Syntax & Style

1. **Braces vs. Indentation**: Do you prefer:
   - `fn foo() { return 1; }` (braces, C-style)
   - `fn foo() -> Int: 1` (indentation, Python/Elixir-style)
   - A hybrid approach?

2. **Semicolons**: Required at end of statements? (`let x = 1;` vs `let x = 1`)

3. **Function call syntax**:
   - `foo(bar, baz)` (parentheses required)
   - `foo bar, baz` (Elixir-style, no parens for simple calls)

4. **Variable naming**: snake_case (`my_variable`) or camelCase (`myVariable`)?

5. **Module/file organization**: One module per file? What file extension (`.aire`, `.ai`)?

### 5.2 Type System

6. **Generics/Type Parameters**: Yes or no? If yes, syntax preference:
   - `fn identity<T>(x: T) -> T`
   - `fn identity(x: a) -> a` (ML-style, implicit)

7. **Algebraic Data Types (ADTs)**:
   - Should Aire have enums with data? Like:
     ```aire
     enum Option<T> {
       None,
       Some(T),
     }
     ```
   - What about struct/vrecord types?

8. **Null handling**: How to handle absence?
   - `Option<T>` (like Rust/ML)
   - Nullable types: `T?` (like Kotlin/TypeScript)
   - Throw exceptions

9. **Effect System**: Do you want effects (IO, async, etc.) tracked in types?
   - Yes, full effect system
   - Yes, basic effect tracking
   - No, keep it simple

10. **Type inference**: Full inference or explicit annotations required?

### 5.3 Concurrency & Agents

11. **Spawn syntax**: How to create agents?
    - `spawn fn worker() { ... }` (keyword-based)
    - `Agent.start(fn worker() { ... })` (function-based)

12. **Message passing**:
    - `agent.send(pid, message)` (dynamic, like Erlang)
    - `pid <- message` (operator-based, like Elixir)
    - Typed channels: `chan: Channel<Message> = ...; chan <- message`

13. **Agent definition**:
    ```aire
    // Option A: Declarative agent
    agent Greeter {
      name: String,
      
      fn greet() -> String { ... }
    }
    
    // Option B: Process-based
    fn greeter(name: String) -> Agent {
      receive {
        "greet" -> ...
      }
    }
    ```

14. **Supervision trees**: First-class support?
    ```aire
    supervisor MyApp {
      worker: Worker,
      worker2: Worker,
    }
    ```

### 5.4 Erlang Interoperability

15. **BEAM access**: How much Erlang should be accessible?
    - Full interop (call any Erlang function, use any OTP module)
    - Limited safe interop (blessed APIs only)
    - Sandboxed (compile-to-.beam only)

16. **FFI (Foreign Function Interface)**: How to call non-Erlang code?
    - Rust NIFs via a cargo-like system
    - Port drivers
    - No FFI initially (focus on BEAM)

17. **Elixir interop**: Can Aire code use Elixir libraries?

### 5.5 Error Handling

18. **Result types**: How to handle fallible operations?
    - `Result<T, E>` enum (must handle or explicitly ignore)
    - Try-catch blocks
    - `?` operator: `let x = might_fail()?`

19. **Pattern matching**: Should it be the primary control flow?
    ```aire
    match result {
      Ok(value) => process(value),
      Err(e) => handle(e),
    }
    ```

### 5.6 Build System & Tooling

20. **Package manager**: Built-in or leverage Erlang's mix/rebar?
    - Built-in Aire package manager
    - Use Mix as the build tool
    - Hybrid: Aire CLI for caching, Mix for deps

21. **Remote cache storage**: Where should compiled artifacts be stored?
    - AWS S3
    - Google Cloud Storage
    - Self-hosted HTTP server
    - All of the above (pluggable)

22. **Cache key strategy**: What determines cache hits?
    - Source file content + dependencies
    - Source + target BEAM version + compiler version
    - Just source content (fastest, less accurate)

23. **IDE support priorities**:
    - LSP (Language Server Protocol)
    - Debugger integration
    - Type hints in comments
    - Formatter/pretty-printer

### 5.7 Language Philosophy

24. **Target audience**: Who is Aire for?
    - Erlang veterans (familiar concepts, better types)
    - newcomers from TypeScript/Rust (approachable syntax)
    - AI/ML engineers (agentic workflows first)

25. **Minimal viable product vs. full-featured**:
    - Start with minimal: just types, basic agents, Erlang gen_server
    - Full features from day one: effects, supervisors,分布

26. **Stability promise**: Semantic versioning from day one? Backward compatibility?

---

## 6. Competitive Analysis

### 6.1 Direct Competitors

| Feature | **Aire** | **Gleam** | **Elixir** | **Erlang** |
|---------|----------|-----------|------------|------------|
| Static typing | ✅ | ✅ | ❌ | ❌ |
| Agent workflows | ✅ | Partial | ✅ | ✅ |
| Remote caching | ✅ (planned) | ❌ | ❌ | ❌ |
| BEAM native | ✅ | ✅ | ✅ | ✅ |
| Syntax style | New | Clean ML | Ruby-like | Prolog-like |
| Effect system | TBD | Basic | Via libraries | Via processes |

### 6.2 Indirect Competitors

- **Rust**: Type safety, systems programming → but not BEAM
- **Odin**: Fast compilation, simple syntax → not BEAM
- **Zig**: Native, comptime → not BEAM

### 6.3 Aire's Unique Positioning

> **Aire = Static Types + Agent Workflows + Bazel-style Caching + BEAM**

No existing language offers this specific combination.

---

## 7. MVP Scope

### 7.1 Must Have (v0.1)

- [ ] Basic syntax (functions, let bindings, types)
- [ ] Static type checker
- [ ] Code generation → .beam files
- [ ] Basic agent spawning
- [ ] Message passing (dynamic)
- [ ] Erlang stdlib interop
- [ ] Incremental compilation (local)
- [ ] Basic CLI tool

### 7.2 Should Have (v0.2)

- [ ] ADTs (enum, record types)
- [ ] Pattern matching
- [ ] Result/Option types
- [ ] Remote caching layer (S3 backend)
- [ ] LSP server
- [ ] Package manager

### 7.3 Could Have (v0.3+)

- [ ] Effect system
- [ ] Typed message channels
- [ ] First-class supervision
- [ ] Distributed computing primitives
- [ ] FFI to Rust
- [ ] WebAssembly target?

---

## 8. Success Metrics

### 8.1 Technical Metrics

| Metric | Target (6 months) |
|--------|-------------------|
| Incremental build speed | <1s for unchanged modules |
| Cold compilation | <10s for 100 modules |
| Cache hit rate (local) | >90% |
| Cache hit rate (remote) | >70% |
| Type inference coverage | >80% of declarations |

### 8.2 Adoption Metrics

| Metric | Target (1 year) |
|--------|-----------------|
| GitHub stars | 1,000+ |
| Production users | 10+ companies |
| Ecosystem packages | 20+ published |
| Discord/Slack members | 500+ |

---

## 9. Next Steps

1. **Pedro reviews and answers** the Design Questions (Section 5)
2. **Finalize language syntax** based on answers
3. **Create technical design document** for compiler architecture
4. **Prototype** the type checker and code generator
5. **Build MVP** with core language features

---

## Appendix A: Example Code (Draft)

### A.1 Basic Agent

```aire
// Greeter agent with typed state
agent Greeter {
  name: String,
  greet_count: Int = 0,
  
  pub fn new(name: String) -> Greeter {
    Greeter(name: name, greet_count: 0)
  },
  
  pub fn greet(self) -> String {
    self.greet_count += 1;
    "Hello, {self.name}! (greeted {self.greet_count} times)"
  }
}

// Spawn and use
fn main() {
  let greeter = Greeter.new(name: "World");
  print(greeter.greet());  // "Hello, World! (greeted 1 times)"
}
```

### A.2 Agent Communication

```aire
agent Counter {
  count: Int = 0,
  
  pub fn increment(self) {
    self.count += 1;
  },
  
  pub fn get(self) -> Int {
    self.count
  }
}

agent Printer {
  pub fn print_messages(self) {
    receive {
      message -> print("Received: {message}")
    }
  }
}

fn main() {
  let counter = Counter();
  let printer = Printer();
  
  // Send messages
  counter <- increment();
  printer <- "Hello from Aire!";
}
```

### A.3 Typed Result Handling

```aire
fn parse_int(s: String) -> Result<Int, String> {
  if s.matches(~r/^\d+$/) {
    Ok(s.parse_int())
  } else {
    Err("Not a valid integer")
  }
}

fn main() {
  let result = parse_int("42");
  
  match result {
    Ok(n) => print("Parsed: {n}"),
    Err(e) => print("Error: {e}"),
  }
}
```

---

*This PRD is a living document. Please answer the questions in Section 5 to help finalize the language design.*
