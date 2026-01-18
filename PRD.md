# Aire Language - Product Requirements Document (PRD)

> **Status**: Draft v0.2  
> **Date**: 2026-01-18  
> **Author**: Codex (for Pedro Piñera)  
> **Language**: Aire - Statically-typed agentic workflow language for BEAM

---

## 1. Executive Summary

**Aire** is a new statically-typed programming language designed for building agentic workflows on the Erlang BEAM virtual machine. It combines:

- **Instant incremental compilation** with Bazel-style remote caching
- **Static type system** inspired by Swift and TypeScript
- **Ruby/Elixir-inspired syntax** for approachability
- **Creative OTP abstractions** - making fault-tolerant agents intuitive
- **Full Erlang interop** - leverage the entire BEAM ecosystem
- **AI/agent-friendly** - designed for coding agents to write and understand

The language targets developers (and AI agents) building multi-agent systems, distributed services, and fault-tolerant applications who want stronger typing guarantees without sacrificing the BEAM's legendary reliability.

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

3. **Type-safe agent communication** - Currently, message passing between processes is dynamic; Aire provides typed channels and message contracts

4. **AI/agent-friendly** - Designed from the ground up for coding agents to understand, write, and refactor

5. **Creative OTP abstractions** - Making Erlang/OTP concepts accessible without the Prolog syntax

---

## 3. Target Users & Use Cases

### 3.1 Primary Users

- **AI Coding Agents** - Primary target! Agents should be able to read, write, and refactor Aire code easily
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
- AI-powered automation workflows

---

## 4. Design Decisions (Confirmed by Pedro)

### 4.1 Syntax & Style

| Decision | Choice | Rationale |
|----------|--------|-----------|
| **Braces vs Indentation** | Elixir-style (indentation, no braces) | Ruby/Elixir inspired |
| **Semicolons** | No semicolons required | Clean, minimal syntax |
| **Function calls** | Parentheses optional for simple calls | Elixir-like ergonomics |
| **Variable naming** | snake_case | Ruby/Python conventions |
| **File extension** | `.aire` | Clear language identity |

### 4.2 Type System

| Decision | Choice | Rationale |
|----------|--------|-----------|
| **Style** | Swift/TypeScript inspired | Familiar to modern developers |
| **Null handling** | Nullable `T?` (TypeScript style) | Approachability |
| **Generics** | Yes, explicit `<T>` syntax | Type safety with clarity |
| **Type inference** | Local inference, explicit annotations for public APIs | Balance of ergonomics and clarity |
| **ADTs** | Yes, with `enum` and `struct` | Powerful patterns without complexity |

### 4.3 Erlang Interop

| Decision | Choice | Rationale |
|----------|--------|-----------|
| **BEAM Access** | Full interop | Complete access to Erlang/OTP |
| **Elixir Interop** | Yes | Leverage existing ecosystem |
| **FFI** | Rust NIFs (later) | Performance when needed |

### 4.4 Target Audience

| Decision | Choice | Rationale |
|----------|--------|-----------|
| **Primary** | AI coding agents + newcomers | Fresh approach, no legacy baggage |
| **Goal** | Scalable, fault-tolerant apps | BEAM's sweet spot |

---

## 5. Creative OTP/Agent Design Proposals

*This section contains novel proposals for expressing Erlang/OTP concepts in Aire.*

### 5.1 Agent Definitions

**Proposal: Declarative agents with behavior inference**

```aire
# Simple agent with state
agent Greeter
  name: String
  greet_count: Int = 0
do
  # Public API
  pub fn greet() -> String
    "Hello, {name}! (greeted {greet_count} times)"
  end
  
  # Private helper
  fn format_greeting(prefix: String) -> String
    "{prefix}, {name}!"
  end
end

# Usage
let greeter = Greeter(name: "World")
print(greeter.greet())
```

**Agent automatically gets:**
- `start_link/1` - OTP-compliant initialization
- State management
- Process lifecycle (terminate, hibernate, etc.)

### 5.2 Typed Message Passing

**Proposal: Channels with type safety**

```aire
# Define a message channel
channel GreeterChannel[Message: struct {
  greet: String -> String
}] end

# Agent receives from typed channel
agent Greeter
  name: String
do
  # Inbox is automatically typed
  receive on GreeterChannel[Message]:
    Message.greet("Hi")
  end
end
```

**Alternative: Simpler approach (Elixir-like)**

```aire
agent Worker
  state: Int
do
  receive do
    :increment -> 
      state += 1
      :ok
    :get -> 
      state
    other ->
      {:error, :unknown_message}
  end
  
  # Pattern matching on messages
  receive :stop, timeout: 5000 do
    :stopping
  end
end
```

### 5.3 Supervisor Trees

**Proposal: Declarative supervision**

```aire
# Simple one-for-one (like DynamicSupervisor)
supervisor TaskSupervisor
  strategy: :one_for_one
  max_restarts: 10
  max_seconds: 3600
do
  # Workers can be started dynamically
  worker: TaskWorker
end

# Supervisor for linked workers
supervisor AppSupervisor
  strategy: :rest_for_one
  max_restarts: 3
do
  agent: DatabasePool
  agent: CachePool
  agent: APIHandlers
end
```

**Starting supervisors:**

```aire
# Start with configuration
let supervisor = AppSuperviser.start!(
  DatabasePool: pool_size: 10,
  CachePool: pool_size: 5,
  APIHandlers: port: 8080
)
```

### 5.4 GenServer-like Abstractions

**Proposal: Built-in server behavior**

```aire
# Counter with GenServer semantics
server Counter
  initial_state: Int = 0
do
  # Call (synchronous)
  def handle_call(:get, _from, state)
    {:reply, state, state}
  end
  
  # Cast (asynchronous)
  def handle_cast(:increment, state)
    {:noreply, state + 1}
  end
  
  # Info (timers, etc.)
  def handle_info(:reset, _state)
    {:noreply, 0}
  end
end

# Usage
let counter = Counter.start!()
Counter.increment(counter)  # async cast
let value = Counter.get(counter)  # sync call
```

### 5.5 Workflow Abstractions

**Proposal: First-class workflow DSL**

```aire
# Define a workflow
workflow OrderProcessing
  steps: [
    step :validate_order,
    step :check_inventory,
    step :process_payment,
    step :ship_order
  ]
  error_handler: :handle_order_error
do
  fn handle_order_error(step: String, error: Error) -> Void
    print("Failed at {step}: {error}")
    notify_team(step, error)
  end
end

# Execute workflow
let result = OrderProcessing.run(order: my_order)
```

### 5.6 Agent Spawning

**Proposal: Simple, expressive spawning**

```aire
# Spawn with options
let agent = spawn Greeter(name: "World")
  as: :registered_name  # optional
  linked: true           # link to parent
  monitor: true          # receive exit signals
end

# Spawn under supervisor
let worker = supervisor.worker(
  TaskWorker,
  args: [queue: my_queue],
  restart: :transient
)
```

---

## 6. Technical Architecture

### 6.1 High-Level Stack

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
│  │  (Rowan)    │  │ (Swift-ish) │  │  (AST → .beam)      │  │
│  └─────────────┘  └─────────────┘  └─────────────────────┘  │
└─────────────────────────┬───────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                   Erlang BEAM VM                             │
│         (.beam files, OTP, supervision trees)                │
└─────────────────────────────────────────────────────────────┘
```

### 6.2 Build System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     Aire Build CLI                           │
├─────────────────────────────────────────────────────────────┤
│  • Incremental compilation graph                             │
│  • Content-addressed caching (SHA-256)                       │
│  • Remote cache sync (configurable backend)                  │
│  • Deterministic builds                                       │
│  • AI-friendly output (structured JSON logging)              │
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

### 6.3 Key Technical Decisions

- **Compiler**: Rust for performance and safety
- **Parser**: Rowan (LALR, similar to Elixir's, handles indentation-based syntax)
- **Type System**: Based on row polymorphism with Swift/TypeScript influence
- **Code Generation**: Direct AST → Erlang AST → BEAM bytecode
- **Remote Cache**: Content-addressed, pluggable backends (S3, GCS, custom HTTP)
- **Agent Model**: Declarative agents that compile to gen_server/gen_statem

---

## 7. Type System Details

### 7.1 Basic Types

```aire
# Primitives
let name: String = "Aire"
let count: Int = 42
let price: Float = 3.14
let is_active: Bool = true

# Collection types
let names: List[String] = ["Alice", "Bob"]
let scores: Map[String, Int] = %{ alice: 10, bob: 20 }
let unique_ids: Set[UUID] = #{"uuid1", "uuid2"}

# Function types
let callback: fn(Int) -> String = fn x: Int { x.to_string() }
```

### 7.2 Nullable Types (TypeScript style)

```aire
# T? means "T or nil"
let name: String? = nil
let name: String? = "Alice"

# Null-coalescing operator
let display_name = name ?? "Anonymous"

# Optional chaining
let length = name?.length || 0
```

### 7.3 Generics

```aire
# Generic function
fn first<T>(list: List[T]) -> T?
  list.first()
end

# Generic struct
struct Container[T]
  value: T
end

# Generic enum
enum Result[T, E]
  ok: T
  err: E
end
```

### 7.4 Algebraic Data Types

```aire
# Simple enum
enum Color
  red
  green
  blue
end

# Enum with data (like Rust/OCaml)
enum Option[T]
  none
  some: T
end

enum Result[T, E]
  ok: T
  err: E
end

# Struct types
struct Point
  x: Float
  y: Float
end

# Usage
let point = Point(x: 1.0, y: 2.0)
match point:
  Point(x: 0, y: 0) -> "Origin"
  Point(x: x, y: 0) -> "On X axis: {x}"
  Point(x, y) -> "Point({x}, {y})"
end
```

### 7.5 Type Inference

```aire
# Local inference works
let x = 42  # inferred as Int
let name = "Alice"  # inferred as String

# Explicit annotations for public APIs
pub fn process(user: User) -> Result[Order, Error]
  # ...
end
```

---

## 8. Erlang Interoperability

### 8.1 Calling Erlang Functions

```aire
# Standard library
let current_time = :erlang.now()

# OTP modules
let pid = :gen_server.start_link(MyServer, [], [])

# Any Erlang module
let result = :lists.map(fn x: x * 2, [1, 2, 3])

# Custom Erlang modules
let result = :my_erlang_module.my_function(arg1, arg2)
```

### 8.2 Using Elixir Libraries

```aire
# Access Mix config
let config = Mix.env()

# Use Elixir structs (with type coercion)
let Poison.json_decode(json_string)  # Returns dynamic or typed?

# Phoenix channels (future)
# let channel = Phoenix.Channel.join("room:lobby", socket)
```

### 8.3 Defining NIFs (Future)

```aire
# aire.toml
[dependencies]
rust = "0.1.0"

[dependencies.rust.nifs]
fast_math = "path/to/rust/code"
```

---

## 9. Error Handling

### 9.1 Result Type

```aire
fn parse_int(s: String) -> Result[Int, String]
  if s.matches(~r/^\d+$/) do
    Ok(s.to_int())
  else
    Err("Not a valid integer: {s}")
  end
end
```

### 9.2 Try Operator

```aire
let result = try parse_int(user_input) {
  Ok(n) -> n * 2
  Err(e) -> print("Error: {e}")
  nil  # fallback value
}

# Propagate error
let n = try parse_int(input)?
```

### 9.3 Pattern Matching

```aire
let result = parse_int("42")

match result:
  Ok(n) -> print("Parsed: {n}")
  Err(e) -> print("Error: {e}")
end

# Destructuring in match
match user:
  User(name: "Admin", role: role) -> handle_admin(role)
  User(name: name, role: "guest") -> greet_guest(name)
  User(name) -> greet_user(name)
end
```

---

## 10. Build System & Tooling

### 10.1 aire.toml Configuration

```toml
name = "my-app"
version = "0.1.0"

[dependencies]
erlang = ["gen_server", "supervisor"]
elixir = ["poison", "httpoison"]

[build]
target = "27"  # BEAM version
optimization = "speed"

[cache]
remote = "s3://aire-cache-bucket"
region = "us-east-1"
```

### 10.2 CLI Commands

```bash
aire init          # Initialize project
aire build         # Compile with caching
aire test          # Run tests
aire run           # Run entry point
aire check         # Type check only (fast)
aire fmt           # Format code
aire REPL          # Interactive REPL
aire deps get      # Fetch dependencies
aire cache sync    # Sync remote cache
```

### 10.3 Remote Caching

```bash
# Configure remote cache
aire config cache.remote s3://my-bucket/path

# Set credentials (env var or config)
export AIRE_AWS_KEY=xxx
export AIRE_AWS_SECRET=xxx

# Force clean build from remote
aire build --force-remote
```

### 10.4 IDE Support

- **LSP Server**: Full language server for autocomplete, goto definition, refactoring
- **Formatter**: Opinionated formatter (Elixir-like)
- **Debugger**: BEAM debugger integration
- **Type Hints**: Inline type display in editors

---

## 11. Competitive Analysis

### 11.1 Direct Competitors

| Feature | **Aire** | **Gleam** | **Elixir** | **Erlang** |
|---------|----------|-----------|------------|------------|
| Static typing | ✅ | ✅ | ❌ | ❌ |
| TypeScript-style `T?` | ✅ | ❌ | ❌ | ❌ |
| Agent workflows | ✅ | Partial | ✅ | ✅ |
| Creative OTP DSL | ✅ | ❌ | ❌ | ❌ |
| Remote caching | ✅ | ❌ | ❌ | ❌ |
| AI/agent-friendly | ✅ | ❌ | ❌ | ❌ |
| BEAM native | ✅ | ✅ | ✅ | ✅ |
| Ruby/Elixir syntax | ✅ | ❌ | ✅ | ❌ |
| Full Erlang interop | ✅ | Limited | ✅ | ✅ |

### 11.2 Indirect Competitors

- **Rust**: Type safety, systems programming → but not BEAM
- **TypeScript**: Familiar types → but not fault-tolerant distributed
- **Go**: Simple, scalable → but not actor-based, no hot reload

### 11.3 Aire's Unique Positioning

> **Aire = Static Types + Ruby/Elixir Syntax + Creative OTP + Bazel Caching + AI-Ready**

No existing language offers this specific combination.

---

## 12. MVP Scope

### 12.1 Must Have (v0.1)

- [ ] Basic syntax (functions, let bindings, types)
- [ ] Static type checker with Swift/TypeScript influence
- [ ] Nullable types (`T?`)
- [ ] Generics and ADTs (enum, struct)
- [ ] Pattern matching
- [ ] Code generation → .beam files
- [ ] Full Erlang interop
- [ ] Basic agent spawning (declarative)
- [ ] Message passing (`pid <- message`)
- [ ] Incremental compilation (local)
- [ ] Basic CLI tool
- [ ] Simple formatter

### 12.2 Should Have (v0.2)

- [ ] Remote caching layer (S3 backend)
- [ ] Supervisor declarations
- [ ] GenServer-like server behavior
- [ ] LSP server
- [ ] Package manager (basic)
- [ ] Typed channels
- [ ] Error handling with `Result`

### 12.3 Could Have (v0.3+)

- [ ] Effect system
- [ ] Distributed computing primitives
- [ ] Workflow DSL
- [ ] FFI to Rust
- [ ] Elixir interop (full)
- [ ] WebAssembly target?
- [ ] Hot code reloading API

---

## 13. Success Metrics

### 13.1 Technical Metrics

| Metric | Target (6 months) |
|--------|-------------------|
| Incremental build speed | <1s for unchanged modules |
| Cold compilation | <10s for 100 modules |
| Cache hit rate (local) | >90% |
| Cache hit rate (remote) | >70% |
| Type inference coverage | >80% of declarations |
| LSP response time | <100ms |

### 13.2 AI/Agent Metrics

| Metric | Target |
|--------|--------|
| Agent code generation success | >70% first-pass |
| Agent refactoring success | >80% |
| Code readability (human + AI) | High |

### 13.3 Adoption Metrics

| Metric | Target (1 year) |
|--------|-----------------|
| GitHub stars | 1,000+ |
| Production users | 10+ companies |
| Ecosystem packages | 20+ published |
| Discord/Slack members | 500+ |
| AI agent experiments | 50+ |

---

## 14. Next Steps

1. **Review OTP/Agent proposals** (Section 5) - Do these resonate?
2. **Prototype the parser** - Rowan-based indentation handling
3. **Prototype the type checker** - Swift/TypeScript inspired
4. **Implement code generation** - AST → Erlang → BEAM
5. **Build MVP compiler** - End-to-end compilation
6. **Create documentation** - AI-friendly guides

---

## Appendix A: Example Code (Draft)

### A.1 Basic Agent

```aire
# Greeter agent with typed state
agent Greeter
  name: String
  greet_count: Int = 0
do
  pub fn greet() -> String
    greet_count += 1
    "Hello, {name}! (greeted {greet_count} times)"
  end
end

# Spawn and use
fn main()
  let greeter = Greeter(name: "World")
  print(greeter.greet())  # "Hello, World! (greeted 1 times)"
end
```

### A.2 GenServer-like Server

```aire
# Counter with GenServer semantics
server Counter
  initial_state: Int = 0
do
  def handle_call(:get, _from, state)
    {:reply, state, state}
  end
  
  def handle_cast(:increment, state)
    {:noreply, state + 1}
  end
  
  def handle_info(:reset, _state)
    {:noreply, 0}
  end
end

# Usage
fn main()
  let counter = Counter.start!()
  Counter.cast(counter, :increment)
  let value = Counter.call(counter, :get)  # 1
end
```

### A.3 Supervisor Tree

```aire
# Application supervisor
supervisor AppSupervisor
  strategy: :rest_for_one
  max_restarts: 3
do
  agent: DatabasePool
  agent: CachePool  
  agent: APIHandlers
end

fn main()
  let supervisor = AppSupervisor.start!(
    DatabasePool: pool_size: 10,
    CachePool: pool_size: 5,
    APIHandlers: port: 8080
  )
end
```

### A.4 Typed Result Handling

```aire
fn parse_int(s: String) -> Result[Int, String]
  if s.matches(~r/^\d+$/) do
    Ok(s.to_int())
  else
    Err("Not a valid integer: {s}")
  end
end

fn main()
  let result = parse_int("42")
  
  match result:
    Ok(n) -> print("Parsed: {n}")
    Err(e) -> print("Error: {e}")
  end
end
```

### A.5 Full Erlang Interop

```aire
fn main()
  # Call any Erlang function
  let node = :net_adm.ping(:"node@host")
  
  # Use OTP
  let {:ok, pid} = :gen_server.start_link(
    MyServer,
    [],
    []
  )
  
  # Pattern match on Erlang results
  match :file.read_file("/tmp/test.txt"):
    {:ok, content} -> print(content)
    {:error, reason} -> print("Error: {reason}")
  end
end
```

---

## Appendix B: Design Principles for AI Agents

Aire is designed to be **AI-friendly**. Design decisions should prioritize:

1. **Predictability** - AI agents can reason about the code
2. **Consistency** - Similar patterns across the language
3. **Type Safety** - Types guide AI code generation
4. **Familiar Syntax** - Borrow from known languages
5. **Clear Errors** - AI needs to understand what went wrong
6. **Structured Output** - Compiler outputs should be parseable

---

*This PRD is a living document. Feedback and iterations are welcome.*
