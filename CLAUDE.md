# CLAUDE.md

## Project Context

This is a learning project for someone new to Rust. The goal is to learn Rust concepts thoroughly while building a working application.

## Project Documentation

- `TUTORIAL.md` - Step-by-step learning guide with checkboxes (follow this)
- `DEV_PLAN_TAURI.md` - Detailed technical specification (reference for implementation details)
- `TUTORIAL_HISTORY.md` - Log of Rust concepts learned (updated as we go)

## Guidelines for Assistance

### Prioritize Learning Over Speed
- Explain Rust concepts as they come up, especially ownership, borrowing, and lifetimes
- Show why Rust does things a certain way, not just how
- When suggesting code, explain what each part does and why it's idiomatic Rust

### Keep Changes Small and Digestible
- Make incremental changes rather than large rewrites
- Introduce one new concept at a time when possible
- Pause after significant changes to allow for questions

### Explain Compiler Errors
- When errors occur, explain what the compiler is telling us
- Show how to read and interpret Rust error messages
- Use errors as teaching moments for Rust's safety guarantees

### Code Style
- Prefer explicit code over clever shortcuts while learning
- Add comments explaining non-obvious Rust patterns
- Show alternative approaches when relevant and explain tradeoffs

### Before Writing Code
- Ask clarifying questions if the task is ambiguous
- Discuss approach before implementing when appropriate
- Explain relevant Rust concepts that will be used

### Follow the Tutorial Progression
- Work through `TUTORIAL.md` step by step
- Check off items as they're completed
- Don't skip ahead - each step builds on previous knowledge
- Reference `DEV_PLAN_TAURI.md` for detailed specs when implementing

### Track Learning Progress
- Maintain `TUTORIAL_HISTORY.md` as a running log of Rust concepts covered
- When introducing a new concept, add an entry with:
  - The concept name
  - A brief explanation
  - Where it was used in the project (file/context)
  - Date learned
- This serves as a personal Rust reference built from hands-on experience

### Build Commands
- `cargo build` - compile the project
- `cargo run` - build and run
- `cargo check` - fast type checking without full compilation
- `cargo test` - run tests
- `cargo clippy` - run the linter for idiomatic Rust suggestions
