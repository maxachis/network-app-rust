# Tauri + Rust Tutorial

A step-by-step guide for building the Network App while learning Rust. Each step introduces new concepts gradually.

For detailed specifications, see [DEV_PLAN_TAURI.md](./DEV_PLAN_TAURI.md).

---

## Phase 1: Project Setup

### Step 1.1: Create the Tauri Project
**Goal**: Get a working Tauri + Svelte app running

- [ ] Install prerequisites (Node.js, Rust, Tauri CLI)
- [ ] Run `npm create tauri-app@latest`
- [ ] Select Svelte + TypeScript template
- [ ] Run `npm install` and `npm run tauri dev`
- [ ] Verify the app window opens

**Rust concepts**: None yet - just setup

---

### Step 1.2: Explore the Generated Project
**Goal**: Understand the project structure

- [ ] Examine `src-tauri/src/main.rs` - the Rust entry point
- [ ] Examine `src-tauri/Cargo.toml` - Rust dependencies
- [ ] Examine `src-tauri/tauri.conf.json` - app configuration
- [ ] Examine `src/` - the Svelte frontend

**Rust concepts**: Project structure, Cargo.toml basics

---

### Step 1.3: Set Up Tailwind CSS
**Goal**: Add styling framework to the frontend

- [ ] Install Tailwind: `npm install -D tailwindcss postcss autoprefixer`
- [ ] Initialize: `npx tailwindcss init -p`
- [ ] Configure `tailwind.config.js`
- [ ] Add Tailwind directives to `src/app.css`
- [ ] Verify styling works

**Rust concepts**: None - frontend only

---

## Phase 2: Your First Rust Code

### Step 2.1: Create a Simple Tauri Command
**Goal**: Learn how Rust functions become callable from the frontend

- [ ] Add a simple `#[tauri::command]` function in `main.rs`
- [ ] Register it with `.invoke_handler()`
- [ ] Call it from Svelte using `invoke()`
- [ ] See the result in the browser console

**Rust concepts**: Functions, return types, the `#[tauri::command]` attribute macro

---

### Step 2.2: Add Rust Dependencies
**Goal**: Learn how Cargo manages dependencies

- [ ] Open `src-tauri/Cargo.toml`
- [ ] Add `rusqlite`, `serde`, `serde_json`, `chrono`
- [ ] Run `cargo build` to download dependencies
- [ ] Understand version specifiers

**Rust concepts**: Cargo.toml, crates, semantic versioning

---

## Phase 3: Database Foundation

### Step 3.1: Create the Database Module Structure
**Goal**: Learn Rust module organization

- [ ] Create `src-tauri/src/database/` directory
- [ ] Create `mod.rs`, `connection.rs`, `migrations.rs`
- [ ] Declare modules in `mod.rs`
- [ ] Import database module in `lib.rs` or `main.rs`

**Rust concepts**: Modules, `mod.rs`, `pub` visibility, `use` statements

---

### Step 3.2: Set Up SQLite Connection
**Goal**: Learn about Rust types, Result, and error handling

- [ ] Create a function to get the database path
- [ ] Open a SQLite connection with rusqlite
- [ ] Handle the `Result` type
- [ ] Understand `unwrap()` vs proper error handling

**Rust concepts**: `Result<T, E>`, `Option<T>`, error handling, `?` operator

---

### Step 3.3: Run Database Migrations
**Goal**: Learn about string literals and SQL execution

- [ ] Create the schema as a raw string literal (`r#"..."#`)
- [ ] Execute the migration SQL
- [ ] Understand `execute_batch()`

**Rust concepts**: String types (`&str`, `String`), raw strings, multi-line strings

---

### Step 3.4: Create Application State
**Goal**: Learn about structs and Tauri's state management

- [ ] Create an `AppState` struct to hold the database connection
- [ ] Use `Mutex` for thread-safe access
- [ ] Register state with Tauri using `.manage()`
- [ ] Access state in commands with `State<'_, AppState>`

**Rust concepts**: Structs, `Mutex`, thread safety, lifetimes (introduction)

---

## Phase 4: Your First Model

### Step 4.1: Define the Person Struct
**Goal**: Learn about structs and derive macros

- [ ] Create `src-tauri/src/models/` directory
- [ ] Create `person.rs` with the `Person` struct
- [ ] Add `#[derive(Debug, Clone, Serialize, Deserialize)]`
- [ ] Understand each derive macro's purpose

**Rust concepts**: Structs, derive macros, Serde serialization

---

### Step 4.2: Create Input Structs
**Goal**: Learn about Option types and struct variations

- [ ] Create `CreatePersonInput` struct
- [ ] Create `UpdatePersonInput` struct
- [ ] Use `Option<T>` for nullable fields
- [ ] Understand why we have separate input/output structs

**Rust concepts**: `Option<T>`, `Some`, `None`, struct design patterns

---

### Step 4.3: Write Your First Database Query
**Goal**: Learn rusqlite query patterns

- [ ] Write a function to insert a Person
- [ ] Use parameter binding (`:named` or `?`)
- [ ] Get the last inserted ID
- [ ] Return the created Person

**Rust concepts**: Closures (brief intro), `map()`, database row mapping

---

### Step 4.4: Create the Person Command
**Goal**: Connect Rust to the frontend

- [ ] Create `src-tauri/src/commands/person.rs`
- [ ] Implement `create_person` command
- [ ] Register in `main.rs`
- [ ] Test from Svelte frontend

**Rust concepts**: Async functions (intro), `async fn`, command registration

---

## Phase 5: CRUD Operations

### Step 5.1: Implement Get All People
**Goal**: Learn iterators and collecting results

- [ ] Write SQL SELECT query
- [ ] Map rows to Person structs
- [ ] Collect into a `Vec<Person>`
- [ ] Return from command

**Rust concepts**: Iterators, `map()`, `collect()`, `Vec<T>`

---

### Step 5.2: Implement Get Single Person
**Goal**: Learn query_row and handling not found

- [ ] Write SQL SELECT with WHERE clause
- [ ] Use `query_row()` for single result
- [ ] Handle "not found" case
- [ ] Return appropriate error

**Rust concepts**: Pattern matching, `match`, error variants

---

### Step 5.3: Implement Update Person
**Goal**: Learn UPDATE queries and ownership

- [ ] Write SQL UPDATE statement
- [ ] Handle optional fields in the update
- [ ] Return the updated Person
- [ ] Understand borrowing vs ownership

**Rust concepts**: Ownership (intro), borrowing, `&` references

---

### Step 5.4: Implement Delete Person
**Goal**: Learn DELETE and cascade behavior

- [ ] Write SQL DELETE statement
- [ ] Handle foreign key constraints
- [ ] Return success/failure

**Rust concepts**: Unit type `()`, success without return value

---

## Phase 6: Frontend Integration

### Step 6.1: Create the API Wrapper
**Goal**: Set up type-safe frontend calls

- [ ] Create `src/lib/api/commands.ts`
- [ ] Define TypeScript interfaces matching Rust structs
- [ ] Create wrapper functions using `invoke()`

**Rust concepts**: None - TypeScript focus, but reinforces struct design

---

### Step 6.2: Build the People List Page
**Goal**: Display data from Rust backend

- [ ] Create `/people` route
- [ ] Fetch people on mount
- [ ] Display in a table
- [ ] Handle loading and error states

**Rust concepts**: None - Svelte focus

---

### Step 6.3: Build the Create Person Form
**Goal**: Send data to Rust backend

- [ ] Create form with fields
- [ ] Call `createPerson` on submit
- [ ] Handle success (navigate to list)
- [ ] Handle errors (display message)

**Rust concepts**: None - Svelte focus

---

## Phase 7: Pagination and Sorting

### Step 7.1: Add Pagination to Queries
**Goal**: Learn SQL LIMIT/OFFSET and response structs

- [ ] Create `PeopleListResponse` struct
- [ ] Add `page`, `page_size`, `total` fields
- [ ] Implement count query
- [ ] Calculate `total_pages`

**Rust concepts**: Compound structs, integer math

---

### Step 7.2: Add Sorting to Queries
**Goal**: Learn dynamic SQL building safely

- [ ] Accept `sort_by` and `sort_dir` parameters
- [ ] Validate sort field against allowed list
- [ ] Build ORDER BY clause safely (no SQL injection)
- [ ] Understand why we validate instead of interpolate

**Rust concepts**: Match expressions, string formatting, security

---

### Step 7.3: Update Frontend for Pagination
**Goal**: Interactive table with sorting

- [ ] Add pagination controls
- [ ] Add sortable column headers
- [ ] Manage state reactively

**Rust concepts**: None - Svelte focus

---

## Phase 8: Additional Models

### Step 8.1: Organization Model
**Goal**: Practice struct patterns with a new model

- [ ] Create Organization struct with joined fields
- [ ] Create OrgType lookup table
- [ ] Implement CRUD commands
- [ ] Add to frontend

**Rust concepts**: Reinforcement of previous concepts

---

### Step 8.2: Interaction Model
**Goal**: Learn foreign key relationships

- [ ] Create Interaction struct
- [ ] Create InteractionType lookup
- [ ] Join Person data in queries
- [ ] Implement CRUD commands

**Rust concepts**: SQL JOINs in Rust, composite data

---

### Step 8.3: Relationship Models
**Goal**: Learn many-to-many relationships

- [ ] Create RelationshipPersonPerson struct
- [ ] Create RelationshipOrganizationPerson struct
- [ ] Implement bidirectional queries
- [ ] Understand the CHECK constraint pattern

**Rust concepts**: Complex queries, data modeling

---

## Phase 9: Search and Typeahead

### Step 9.1: Implement Person Search
**Goal**: Learn LIKE queries and suggestions

- [ ] Create `PersonSuggestion` struct
- [ ] Write search query with LIKE
- [ ] Limit results
- [ ] Format display label

**Rust concepts**: String manipulation, `format!()`

---

### Step 9.2: Build Typeahead Component
**Goal**: Create reusable search input

- [ ] Build Svelte component with debounce
- [ ] Handle keyboard navigation
- [ ] Emit selection events

**Rust concepts**: None - Svelte focus

---

## Phase 10: Dashboard

### Step 10.1: Calculate Follow-up Status
**Goal**: Learn date arithmetic in Rust

- [ ] Parse dates with chrono
- [ ] Calculate days since last interaction
- [ ] Determine overdue status
- [ ] Sort by urgency

**Rust concepts**: chrono crate, date parsing, comparisons

---

### Step 10.2: Build Dashboard Query
**Goal**: Complex aggregation query

- [ ] Create `DashboardResponse` struct
- [ ] Query overdue follow-ups
- [ ] Query upcoming follow-ups
- [ ] Get recent interactions
- [ ] Calculate stats

**Rust concepts**: Multiple queries, data aggregation

---

### Step 10.3: Global Search
**Goal**: Search across multiple tables

- [ ] Create `SearchResults` struct
- [ ] Search people, organizations, interactions
- [ ] Combine results
- [ ] Rank by relevance (optional)

**Rust concepts**: Combining multiple Result types

---

## Phase 11: Graph Visualization

### Step 11.1: Build Graph Data Query
**Goal**: Transform relational data to graph format

- [ ] Create `GraphNode` and `GraphEdge` structs
- [ ] Query all people as nodes
- [ ] Query all organizations as nodes
- [ ] Query relationships as edges
- [ ] Use `serde_json::Value` for flexible details

**Rust concepts**: Enums (node types), JSON serialization

---

### Step 11.2: Build Graph Frontend
**Goal**: D3.js force-directed graph

- [ ] Set up D3.js
- [ ] Create force simulation
- [ ] Render nodes and edges
- [ ] Add drag interaction

**Rust concepts**: None - JavaScript/D3 focus

---

## Phase 12: Polish and Production

### Step 12.1: Error Handling Improvements
**Goal**: Learn proper Rust error handling

- [ ] Create custom error types
- [ ] Implement `From` trait for conversions
- [ ] Use `thiserror` or `anyhow` crate
- [ ] Return user-friendly messages

**Rust concepts**: Custom errors, traits, `From` implementation

---

### Step 12.2: Input Validation
**Goal**: Validate data before database

- [ ] Add validation to input structs
- [ ] Return validation errors
- [ ] Display errors in frontend

**Rust concepts**: Validation patterns, early returns

---

### Step 12.3: Loading States
**Goal**: Better UX during async operations

- [ ] Add loading indicators
- [ ] Disable buttons during submission
- [ ] Handle slow connections gracefully

**Rust concepts**: None - frontend focus

---

### Step 12.4: Build for Production
**Goal**: Create distributable executable

- [ ] Run `npm run tauri build`
- [ ] Locate the executable
- [ ] Test on a clean machine
- [ ] Understand the bundle contents

**Rust concepts**: Release builds, optimization

---

## Checkpoint Markers

Use these to track your overall progress:

- [ ] **Milestone 1**: App runs and displays "Hello World" from Rust
- [ ] **Milestone 2**: Can create and list People from the database
- [ ] **Milestone 3**: Full CRUD for People with pagination
- [ ] **Milestone 4**: Organizations and Interactions working
- [ ] **Milestone 5**: Relationships and search working
- [ ] **Milestone 6**: Dashboard with follow-up tracking
- [ ] **Milestone 7**: Graph visualization complete
- [ ] **Milestone 8**: Production build ready

---

## Quick Reference

### Common Cargo Commands
```bash
cargo build          # Compile the project
cargo run            # Build and run
cargo check          # Fast type checking
cargo test           # Run tests
cargo clippy         # Linter suggestions
cargo fmt            # Format code
```

### Common Tauri Commands
```bash
npm run tauri dev    # Development mode
npm run tauri build  # Production build
```

### Getting Help
- Rust Book: https://doc.rust-lang.org/book/
- Rust by Example: https://doc.rust-lang.org/rust-by-example/
- Tauri Docs: https://tauri.app/v1/guides/
- rusqlite Docs: https://docs.rs/rusqlite/
