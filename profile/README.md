# Frame

**Frame is a domain-specific language for state machines, embedded in your native source code.** Frame enables developers to add state machines as a first-class abstraction to **17 target languages** — Python, TypeScript, JavaScript, Rust, C, C++, Java, C#, Go, PHP, Kotlin, Swift, Ruby, Erlang, Lua, Dart, GDScript — plus Graphviz for state-diagram visualization.

You write your project in your chosen language. Frame `@@system` blocks inside your source file expand into idiomatic state-machine implementations in that same language. Everything outside the `@@system` blocks passes through unchanged. The result: native code, no runtime dependency, drop-in ready.

---

## Quick Install

```bash
cargo install framec
```

> Prebuilt binaries (Mac, Windows, Linux) and `brew install` coming soon. Track progress in [framec/issues](https://github.com/frame-lang/framec/issues).

## A 30-second example

A Frame source file is a native source file in your chosen target language with `@@system` blocks embedded for the state-machine pieces. Save the following as `traffic.fpy` (Python target):

```frame
@@[target("python_3")]

import logging

@@system TrafficLight {
    interface:
        next()

    machine:
        $Red {
            next() { -> $Green }
        }
        $Green {
            next() { -> $Yellow }
        }
        $Yellow {
            next() { -> $Red }
        }
}

if __name__ == "__main__":
    light = TrafficLight()
    for _ in range(6):
        light.next()
```

Run `framec traffic.fpy > traffic.py` and you get a complete Python file: the `import` and the `if __name__` block pass through verbatim, and the `@@system TrafficLight` block is replaced by a Python class that implements the state machine. Run the output with `python traffic.py` to exercise 6 transitions (Red → Green → Yellow → Red → Green → Yellow). The example handlers don't print anything, so the run is silent — add `logging.info(...)` calls inside the handler bodies to see each transition. (`framec` autodetects the target from the `@@[target]` directive in the source — no extra flag needed.)

The same approach works in any of the 17 supported languages — change `@@[target("python_3")]` to `@@[target("rust")]`, rename the file to use a Rust-flavored extension (`.frust`), and `framec traffic.frust > traffic.rs` produces Rust. Frame's job is to make state machines first-class in *your* language, not to translate between languages.

For visualization, Frame can also emit a Graphviz `.dot` diagram of any state machine — pipe through `dot` for an image: `framec -l graphviz traffic.fpy | dot -Tpng -o traffic.png`.

---

## Repositories

| Repo | Purpose |
|---|---|
| **[framec](https://github.com/frame-lang/framec)** | Frame language transpiler — Rust implementation |
| **[framec-test-env](https://github.com/frame-lang/framec-test-env)** | 17-language differential test harness |

## Documentation

- **[Getting Started](https://github.com/frame-lang/framec/blob/main/docs/frame_getting_started.md)** — tutorial introduction
- **[Language Reference](https://github.com/frame-lang/framec/blob/main/docs/frame_language.md)** — full syntax + semantics
- **[Cookbook](https://github.com/frame-lang/framec/blob/main/docs/frame_cookbook.md)** — 111 recipes from basic state machines to enterprise patterns
- **[Per-Language Guides](https://github.com/frame-lang/framec/tree/main/docs/per_language_guides)** — backend-specific notes for each of the 17 targets
- **[Roadmap](https://github.com/frame-lang/framec/blob/main/ROADMAP.md)** — what's shipping, what's next

## Testing & Quality

Frame's correctness across 17 target languages is verified by a multi-tier test infrastructure:

- **396 unit tests** in the framec compiler covering lexer, parser, validator, and codegen layers
- **111 cookbook recipes** that double as runnable examples — from basic state machines to enterprise integration patterns and protocol-as-FSM
- **17-language differential test matrix** in [framec-test-env](https://github.com/frame-lang/framec-test-env) — 321 common positive fixtures executed against every backend in Docker for hermetic, parallelized runs
- **Fuzz infrastructure** — 33,000+ generated test cases across 21 generators covering correctness axes including hierarchical state machines, persist round-trip, async, multi-system composition, push/pop modal stack, state-arg propagation, identifier shadowing, and control-flow embedding
- **Negative test suite** — every validator error code (E000-series, W000-series) has cases that fire it and counter-cases that pass cleanly
- **Per-language backend tests** for behaviors specific to a single target (Erlang gen_statem, GDScript Godot integration, Java CompletableFuture async, etc.)

## Status

Frame V4 is the current iteration. Initial public release: May 2026. Active development. See the [roadmap](https://github.com/frame-lang/framec/blob/main/ROADMAP.md) for what's coming.

---

## License

[Apache 2.0](https://github.com/frame-lang/framec/blob/main/LICENSE)
