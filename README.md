# Aire

A statically-typed programming language for agentic workflows on the BEAM.

## Quick Start

```aire
// Define an agent
agent Greeter {
  name: String,
  
  pub fn greet() -> String {
    "Hello, {self.name}!"
  }
}

fn main() {
  let greeter = Greeter(name: "World");
  print(greeter.greet());
}
```

