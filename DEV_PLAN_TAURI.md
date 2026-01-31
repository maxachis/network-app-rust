# Tauri + Svelte Implementation Specification

This document provides complete specifications for building a personal network/CRM application as a standalone Windows desktop application using Tauri (Rust backend) with a Svelte frontend. An agent should be able to implement the entire application from this document alone.

## Overview

**Product**: Personal network management application for tracking people, organizations, interactions, and relationships.

**Architecture**:
- Tauri (Rust) backend with embedded SQLite database
- Svelte frontend with Tailwind CSS
- Native desktop window (uses system WebView)
- IPC commands for frontend-backend communication
- Single executable (~5-10MB)

**Key Features**:
- Dashboard with follow-up reminders
- People management with interaction tracking
- Organization management
- Relationship mapping (person-to-person, person-to-organization)
- Interactive network graph visualization
- Global search
- Native window with system tray (optional)

---

## Project Structure

```
network-app/
├── src-tauri/                      # Rust backend
│   ├── Cargo.toml
│   ├── tauri.conf.json
│   ├── src/
│   │   ├── main.rs                 # Entry point
│   │   ├── lib.rs                  # Library root
│   │   ├── database/
│   │   │   ├── mod.rs
│   │   │   ├── connection.rs       # SQLite setup
│   │   │   └── migrations.rs       # Schema
│   │   ├── models/
│   │   │   ├── mod.rs
│   │   │   ├── person.rs
│   │   │   ├── organization.rs
│   │   │   ├── interaction.rs
│   │   │   ├── interaction_type.rs
│   │   │   ├── org_type.rs
│   │   │   ├── relationship_person_person.rs
│   │   │   └── relationship_org_person.rs
│   │   └── commands/               # Tauri IPC commands
│   │       ├── mod.rs
│   │       ├── person.rs
│   │       ├── organization.rs
│   │       ├── interaction.rs
│   │       ├── relationship.rs
│   │       ├── dashboard.rs
│   │       └── graph.rs
│   └── icons/                      # App icons
├── src/                            # Svelte frontend
│   ├── app.html
│   ├── app.css                     # Tailwind imports
│   ├── lib/
│   │   ├── components/
│   │   │   ├── Table.svelte
│   │   │   ├── SortableTable.svelte
│   │   │   ├── Pagination.svelte
│   │   │   ├── Form.svelte
│   │   │   ├── FormField.svelte
│   │   │   ├── Typeahead.svelte
│   │   │   ├── Button.svelte
│   │   │   └── Card.svelte
│   │   ├── stores/
│   │   │   └── navigation.ts       # Client-side routing state
│   │   └── api/
│   │       └── commands.ts         # Tauri invoke wrappers
│   └── routes/
│       ├── +layout.svelte          # App shell
│       ├── +page.svelte            # Dashboard
│       ├── people/
│       │   ├── +page.svelte        # People list
│       │   ├── [id]/
│       │   │   └── +page.svelte    # Person detail
│       │   ├── create/
│       │   │   └── +page.svelte    # Create person
│       │   └── [id]/edit/
│       │       └── +page.svelte    # Edit person
│       ├── organizations/
│       │   ├── +page.svelte        # Organization list
│       │   └── create/
│       │       └── +page.svelte    # Create org
│       ├── interactions/
│       │   ├── +page.svelte        # Interaction list
│       │   └── create/
│       │       └── +page.svelte    # Create interaction
│       ├── relationships/
│       │   ├── +page.svelte        # Relationship list
│       │   ├── person-person/
│       │   │   └── +page.svelte    # Create person-person
│       │   └── org-person/
│       │       └── +page.svelte    # Create org-person
│       └── graph/
│           └── +page.svelte        # Graph visualization
├── static/
│   └── (static assets)
├── package.json
├── svelte.config.js
├── tailwind.config.js
├── vite.config.ts
└── tsconfig.json
```

---

## Data Models

All models use SQLite with the same schema as the Go version. The Rust structs use `serde` for serialization.

### Person

```rust
#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct Person {
    pub id: i64,
    pub first_name: String,
    pub middle_name: Option<String>,
    pub last_name: String,
    pub notes: Option<String>,
    pub follow_up_cadence_days: Option<i32>,
    pub created_at: String,  // ISO 8601
    pub updated_at: String,
}

#[derive(Debug, Deserialize)]
pub struct CreatePersonInput {
    pub first_name: String,
    pub middle_name: Option<String>,
    pub last_name: String,
    pub notes: Option<String>,
    pub follow_up_cadence_days: Option<i32>,
}

#[derive(Debug, Deserialize)]
pub struct UpdatePersonInput {
    pub id: i64,
    pub first_name: String,
    pub middle_name: Option<String>,
    pub last_name: String,
    pub notes: Option<String>,
    pub follow_up_cadence_days: Option<i32>,
}
```

### Organization

```rust
#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct Organization {
    pub id: i64,
    pub name: String,
    pub org_type_id: i64,
    pub org_type_name: String,  // Joined from org_type
    pub notes: Option<String>,
    pub created_at: String,
    pub updated_at: String,
}

#[derive(Debug, Deserialize)]
pub struct CreateOrganizationInput {
    pub name: String,
    pub org_type_id: i64,
    pub notes: Option<String>,
}
```

### Interaction

```rust
#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct Interaction {
    pub id: i64,
    pub person_id: i64,
    pub person_name: String,  // Joined: "LastName, FirstName"
    pub interaction_type_id: i64,
    pub interaction_type_name: String,
    pub date: String,  // YYYY-MM-DD
    pub notes: Option<String>,
    pub created_at: String,
    pub updated_at: String,
}

#[derive(Debug, Deserialize)]
pub struct CreateInteractionInput {
    pub person_id: i64,
    pub interaction_type_id: i64,
    pub date: String,
    pub notes: Option<String>,
}
```

### InteractionType / OrgType

```rust
#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct InteractionType {
    pub id: i64,
    pub name: String,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct OrgType {
    pub id: i64,
    pub name: String,
}
```

### RelationshipPersonPerson

```rust
#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct RelationshipPersonPerson {
    pub id: i64,
    pub person_1_id: i64,
    pub person_1_name: String,
    pub person_2_id: i64,
    pub person_2_name: String,
    pub notes: Option<String>,
    pub created_at: String,
    pub updated_at: String,
}

#[derive(Debug, Deserialize)]
pub struct CreateRelationshipPersonPersonInput {
    pub person_1_id: i64,
    pub person_2_id: i64,
    pub notes: Option<String>,
}
```

### RelationshipOrganizationPerson

```rust
#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct RelationshipOrganizationPerson {
    pub id: i64,
    pub organization_id: i64,
    pub organization_name: String,
    pub person_id: i64,
    pub person_name: String,
    pub notes: Option<String>,
    pub created_at: String,
    pub updated_at: String,
}

#[derive(Debug, Deserialize)]
pub struct CreateRelationshipOrgPersonInput {
    pub organization_id: i64,
    pub person_id: i64,
    pub notes: Option<String>,
}
```

---

## Database Schema

Same as Go version. Create in `src-tauri/src/database/migrations.rs`:

```rust
pub const MIGRATIONS: &str = r#"
CREATE TABLE IF NOT EXISTS org_type (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL
);

CREATE TABLE IF NOT EXISTS interaction_type (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL
);

CREATE TABLE IF NOT EXISTS person (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    first_name TEXT NOT NULL,
    middle_name TEXT,
    last_name TEXT NOT NULL,
    notes TEXT,
    follow_up_cadence_days INTEGER,
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(first_name, last_name)
);

CREATE TABLE IF NOT EXISTS organization (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL UNIQUE,
    org_type_id INTEGER NOT NULL,
    notes TEXT,
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (org_type_id) REFERENCES org_type(id) ON DELETE CASCADE
);

CREATE TABLE IF NOT EXISTS interaction (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    person_id INTEGER NOT NULL,
    interaction_type_id INTEGER NOT NULL,
    date DATE NOT NULL DEFAULT CURRENT_DATE,
    notes TEXT,
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (person_id) REFERENCES person(id) ON DELETE CASCADE,
    FOREIGN KEY (interaction_type_id) REFERENCES interaction_type(id) ON DELETE RESTRICT
);

CREATE INDEX IF NOT EXISTS idx_interaction_person_date
ON interaction(person_id, date DESC, id DESC);

CREATE TABLE IF NOT EXISTS relationship_person_person (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    person_1_id INTEGER NOT NULL,
    person_2_id INTEGER NOT NULL,
    notes TEXT,
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (person_1_id) REFERENCES person(id) ON DELETE CASCADE,
    FOREIGN KEY (person_2_id) REFERENCES person(id) ON DELETE CASCADE,
    CHECK (person_1_id < person_2_id),
    CHECK (person_1_id != person_2_id),
    UNIQUE (person_1_id, person_2_id)
);

CREATE TABLE IF NOT EXISTS relationship_organization_person (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    organization_id INTEGER NOT NULL,
    person_id INTEGER NOT NULL,
    notes TEXT,
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (organization_id) REFERENCES organization(id) ON DELETE CASCADE,
    FOREIGN KEY (person_id) REFERENCES person(id) ON DELETE CASCADE
);
"#;
```

---

## Tauri Commands (IPC API)

Commands are exposed to the frontend via `#[tauri::command]` and invoked with `invoke()`.

### Person Commands

```rust
// src-tauri/src/commands/person.rs

#[tauri::command]
pub async fn get_people(
    state: State<'_, AppState>,
    sort_by: Option<String>,
    sort_dir: Option<String>,
    page: Option<i32>,
    page_size: Option<i32>,
) -> Result<PeopleListResponse, String>;

#[tauri::command]
pub async fn get_person(
    state: State<'_, AppState>,
    id: i64,
) -> Result<PersonDetailResponse, String>;

#[tauri::command]
pub async fn create_person(
    state: State<'_, AppState>,
    input: CreatePersonInput,
) -> Result<Person, String>;

#[tauri::command]
pub async fn update_person(
    state: State<'_, AppState>,
    input: UpdatePersonInput,
) -> Result<Person, String>;

#[tauri::command]
pub async fn search_people(
    state: State<'_, AppState>,
    query: String,
    limit: Option<i32>,
) -> Result<Vec<PersonSuggestion>, String>;
```

**Response Types**:

```rust
#[derive(Serialize)]
pub struct PeopleListResponse {
    pub items: Vec<PersonWithLatestInteraction>,
    pub total: i64,
    pub page: i32,
    pub page_size: i32,
    pub total_pages: i32,
}

#[derive(Serialize)]
pub struct PersonWithLatestInteraction {
    pub id: i64,
    pub first_name: String,
    pub middle_name: Option<String>,
    pub last_name: String,
    pub follow_up_cadence_days: Option<i32>,
    pub latest_interaction_date: Option<String>,
    pub is_overdue: bool,
}

#[derive(Serialize)]
pub struct PersonDetailResponse {
    pub person: Person,
    pub interactions: Vec<Interaction>,
    pub related_organizations: Vec<RelatedOrganization>,
    pub related_people: Vec<RelatedPerson>,
}

#[derive(Serialize)]
pub struct PersonSuggestion {
    pub id: i64,
    pub label: String,  // "LastName, FirstName MiddleName"
}
```

### Organization Commands

```rust
#[tauri::command]
pub async fn get_organizations(
    state: State<'_, AppState>,
) -> Result<Vec<Organization>, String>;

#[tauri::command]
pub async fn create_organization(
    state: State<'_, AppState>,
    input: CreateOrganizationInput,
) -> Result<Organization, String>;

#[tauri::command]
pub async fn search_organizations(
    state: State<'_, AppState>,
    query: String,
    limit: Option<i32>,
) -> Result<Vec<OrganizationSuggestion>, String>;

#[tauri::command]
pub async fn get_org_types(
    state: State<'_, AppState>,
) -> Result<Vec<OrgType>, String>;
```

### Interaction Commands

```rust
#[tauri::command]
pub async fn get_interactions(
    state: State<'_, AppState>,
    page: Option<i32>,
    page_size: Option<i32>,
) -> Result<InteractionListResponse, String>;

#[tauri::command]
pub async fn create_interaction(
    state: State<'_, AppState>,
    input: CreateInteractionInput,
) -> Result<Interaction, String>;

#[tauri::command]
pub async fn get_interaction_types(
    state: State<'_, AppState>,
) -> Result<Vec<InteractionType>, String>;
```

### Relationship Commands

```rust
#[tauri::command]
pub async fn get_relationships(
    state: State<'_, AppState>,
) -> Result<RelationshipsResponse, String>;

#[tauri::command]
pub async fn create_relationship_person_person(
    state: State<'_, AppState>,
    input: CreateRelationshipPersonPersonInput,
) -> Result<RelationshipPersonPerson, String>;

#[tauri::command]
pub async fn create_relationship_org_person(
    state: State<'_, AppState>,
    input: CreateRelationshipOrgPersonInput,
) -> Result<RelationshipOrganizationPerson, String>;
```

**Response Types**:

```rust
#[derive(Serialize)]
pub struct RelationshipsResponse {
    pub person_person: Vec<RelationshipPersonPerson>,
    pub org_person: Vec<RelationshipOrganizationPerson>,
}
```

### Dashboard Commands

```rust
#[tauri::command]
pub async fn get_dashboard(
    state: State<'_, AppState>,
) -> Result<DashboardResponse, String>;

#[tauri::command]
pub async fn global_search(
    state: State<'_, AppState>,
    query: String,
) -> Result<SearchResults, String>;
```

**Response Types**:

```rust
#[derive(Serialize)]
pub struct DashboardResponse {
    pub stats: DashboardStats,
    pub overdue_followups: Vec<FollowUpItem>,
    pub upcoming_followups: Vec<FollowUpItem>,
    pub recent_interactions: Vec<Interaction>,
}

#[derive(Serialize)]
pub struct DashboardStats {
    pub total_people: i64,
    pub interactions_this_month: i64,
    pub overdue_count: i64,
    pub upcoming_count: i64,
}

#[derive(Serialize)]
pub struct FollowUpItem {
    pub person_id: i64,
    pub person_name: String,
    pub latest_interaction_date: Option<String>,
    pub days_overdue: Option<i32>,      // For overdue
    pub days_until_due: Option<i32>,    // For upcoming
}

#[derive(Serialize)]
pub struct SearchResults {
    pub people: Vec<PersonSuggestion>,
    pub organizations: Vec<OrganizationSuggestion>,
    pub interactions: Vec<InteractionSearchResult>,
}
```

### Graph Commands

```rust
#[tauri::command]
pub async fn get_graph_data(
    state: State<'_, AppState>,
) -> Result<GraphData, String>;
```

**Response Type**:

```rust
#[derive(Serialize)]
pub struct GraphData {
    pub nodes: Vec<GraphNode>,
    pub edges: Vec<GraphEdge>,
}

#[derive(Serialize)]
pub struct GraphNode {
    pub id: String,         // "person-1" or "organization-1"
    pub label: String,
    pub node_type: String,  // "person" or "organization"
    pub details: serde_json::Value,
}

#[derive(Serialize)]
pub struct GraphEdge {
    pub id: String,
    pub source: String,
    pub target: String,
    pub edge_type: String,  // "person-person" or "organization-person"
    pub notes: Option<String>,
}
```

---

## Frontend API Wrapper

Create type-safe wrappers for Tauri commands:

```typescript
// src/lib/api/commands.ts
import { invoke } from '@tauri-apps/api/core';

// Types
export interface Person {
  id: number;
  first_name: string;
  middle_name: string | null;
  last_name: string;
  notes: string | null;
  follow_up_cadence_days: number | null;
  created_at: string;
  updated_at: string;
}

export interface PeopleListResponse {
  items: PersonWithLatestInteraction[];
  total: number;
  page: number;
  page_size: number;
  total_pages: number;
}

// ... other types matching Rust structs

// API functions
export async function getPeople(
  sortBy?: string,
  sortDir?: string,
  page?: number,
  pageSize?: number
): Promise<PeopleListResponse> {
  return invoke('get_people', {
    sortBy,
    sortDir,
    page: page ?? 1,
    pageSize: pageSize ?? 10,
  });
}

export async function getPerson(id: number): Promise<PersonDetailResponse> {
  return invoke('get_person', { id });
}

export async function createPerson(input: CreatePersonInput): Promise<Person> {
  return invoke('create_person', { input });
}

export async function updatePerson(input: UpdatePersonInput): Promise<Person> {
  return invoke('update_person', { input });
}

export async function searchPeople(
  query: string,
  limit?: number
): Promise<PersonSuggestion[]> {
  return invoke('search_people', { query, limit: limit ?? 8 });
}

export async function getDashboard(): Promise<DashboardResponse> {
  return invoke('get_dashboard');
}

export async function globalSearch(query: string): Promise<SearchResults> {
  return invoke('global_search', { query });
}

export async function getGraphData(): Promise<GraphData> {
  return invoke('get_graph_data');
}

// ... etc for all commands
```

---

## Page Specifications

### 1. Dashboard (`/`)

```svelte
<!-- src/routes/+page.svelte -->
<script lang="ts">
  import { onMount } from 'svelte';
  import { getDashboard, globalSearch } from '$lib/api/commands';
  import type { DashboardResponse, SearchResults } from '$lib/api/commands';

  let dashboard: DashboardResponse | null = null;
  let searchQuery = '';
  let searchResults: SearchResults | null = null;

  onMount(async () => {
    dashboard = await getDashboard();
  });

  async function handleSearch() {
    if (searchQuery.length >= 2) {
      searchResults = await globalSearch(searchQuery);
    } else {
      searchResults = null;
    }
  }
</script>
```

**Layout**:
1. **Stats Bar**: 4 cards showing total people, interactions this month, overdue, upcoming
2. **Search Bar**: Input with debounced search (250ms), results dropdown
3. **Overdue Follow-ups Table**: Clickable rows navigate to person detail
4. **Upcoming Follow-ups Table**: Next 14 days
5. **Recent Interactions Table**: Last 10 interactions

---

### 2. People List (`/people`)

**Features**:
- Paginated (10 per page)
- Sortable columns via header click
- Click row to navigate to detail

```svelte
<script lang="ts">
  import { getPeople } from '$lib/api/commands';
  import SortableTable from '$lib/components/SortableTable.svelte';
  import Pagination from '$lib/components/Pagination.svelte';

  let sortBy = 'last_name';
  let sortDir = 'asc';
  let page = 1;
  let data = null;

  $: loadData(sortBy, sortDir, page);

  async function loadData(sort: string, dir: string, p: number) {
    data = await getPeople(sort, dir, p, 10);
  }

  function handleSort(field: string) {
    if (sortBy === field) {
      sortDir = sortDir === 'asc' ? 'desc' : 'asc';
    } else {
      sortBy = field;
      sortDir = 'asc';
    }
  }
</script>
```

**Columns**:
| Column | Field | Sortable |
|--------|-------|----------|
| First | first_name | Yes |
| Middle | middle_name | Yes |
| Last | last_name | Yes |
| Follow-up Cadence | follow_up_cadence_days | Yes |
| Latest Interaction | latest_interaction_date | Yes |

---

### 3. Person Detail (`/people/[id]`)

**Sections**:
1. **Header**: Name + Edit button
2. **Info Card**: Cadence, notes, timestamps
3. **Interactions Table**: With "Add Interaction" button
4. **Related Organizations**: List
5. **Related People**: List

```svelte
<script lang="ts">
  import { page } from '$app/stores';
  import { getPerson } from '$lib/api/commands';

  $: personId = Number($page.params.id);
  $: loadPerson(personId);

  let detail = null;

  async function loadPerson(id: number) {
    detail = await getPerson(id);
  }
</script>
```

---

### 4. Person Create/Edit Form

**Component**: Reusable form for both create and edit

```svelte
<!-- src/lib/components/PersonForm.svelte -->
<script lang="ts">
  import { createPerson, updatePerson } from '$lib/api/commands';
  import { goto } from '$app/navigation';

  export let person = null; // null for create, object for edit
  export let isEdit = false;

  let firstName = person?.first_name ?? '';
  let middleName = person?.middle_name ?? '';
  let lastName = person?.last_name ?? '';
  let followUpCadenceDays = person?.follow_up_cadence_days ?? null;
  let notes = person?.notes ?? '';
  let error = '';

  async function handleSubmit() {
    try {
      const input = {
        first_name: firstName,
        middle_name: middleName || null,
        last_name: lastName,
        follow_up_cadence_days: followUpCadenceDays,
        notes: notes || null,
      };

      let result;
      if (isEdit) {
        result = await updatePerson({ id: person.id, ...input });
      } else {
        result = await createPerson(input);
      }
      goto(`/people/${result.id}`);
    } catch (e) {
      error = e.toString();
    }
  }
</script>
```

**Fields**:
| Field | Type | Required | Validation |
|-------|------|----------|------------|
| first_name | text | Yes | max 255 chars |
| middle_name | text | No | max 255 chars |
| last_name | text | Yes | max 255 chars |
| follow_up_cadence_days | number | No | positive integer |
| notes | textarea | No | — |

---

### 5. Typeahead Component

```svelte
<!-- src/lib/components/Typeahead.svelte -->
<script lang="ts">
  import { createEventDispatcher } from 'svelte';

  export let placeholder = 'Search...';
  export let searchFn: (query: string) => Promise<Array<{id: number, label: string}>>;
  export let value: number | null = null;
  export let displayValue = '';

  const dispatch = createEventDispatcher();

  let suggestions = [];
  let showSuggestions = false;
  let activeIndex = -1;
  let inputValue = displayValue;

  async function handleInput() {
    if (inputValue.length >= 2) {
      suggestions = await searchFn(inputValue);
      showSuggestions = true;
      activeIndex = -1;
    } else {
      suggestions = [];
      showSuggestions = false;
      value = null;
      dispatch('select', null);
    }
  }

  function selectSuggestion(suggestion) {
    value = suggestion.id;
    inputValue = suggestion.label;
    showSuggestions = false;
    dispatch('select', suggestion);
  }

  function handleKeydown(e: KeyboardEvent) {
    if (e.key === 'ArrowDown') {
      activeIndex = Math.min(activeIndex + 1, suggestions.length - 1);
    } else if (e.key === 'ArrowUp') {
      activeIndex = Math.max(activeIndex - 1, 0);
    } else if (e.key === 'Enter' && activeIndex >= 0) {
      selectSuggestion(suggestions[activeIndex]);
    } else if (e.key === 'Escape') {
      showSuggestions = false;
    }
  }
</script>

<div class="relative">
  <input
    type="text"
    bind:value={inputValue}
    on:input={handleInput}
    on:keydown={handleKeydown}
    on:focus={() => inputValue.length >= 2 && (showSuggestions = true)}
    on:blur={() => setTimeout(() => showSuggestions = false, 150)}
    {placeholder}
    class="w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500"
  />

  {#if showSuggestions && suggestions.length > 0}
    <ul class="absolute z-10 mt-1 w-full bg-white border border-gray-300 rounded-md shadow-lg max-h-60 overflow-auto">
      {#each suggestions as suggestion, i}
        <li
          class="px-3 py-2 cursor-pointer hover:bg-indigo-50"
          class:bg-indigo-100={i === activeIndex}
          on:mousedown={() => selectSuggestion(suggestion)}
        >
          {suggestion.label}
        </li>
      {/each}
    </ul>
  {/if}
</div>
```

---

### 6. Interaction Create Form

```svelte
<script lang="ts">
  import { page } from '$app/stores';
  import { createInteraction, getInteractionTypes, searchPeople } from '$lib/api/commands';
  import Typeahead from '$lib/components/Typeahead.svelte';

  // Pre-select person if passed via query param
  let preselectedPersonId = $page.url.searchParams.get('person_id');

  let personId: number | null = preselectedPersonId ? Number(preselectedPersonId) : null;
  let interactionTypeId: number | null = null;
  let date = new Date().toISOString().split('T')[0]; // Today
  let notes = '';
  let interactionTypes = [];

  onMount(async () => {
    interactionTypes = await getInteractionTypes();
  });
</script>
```

---

### 7. Graph Page

Uses D3.js for force-directed graph visualization.

```svelte
<!-- src/routes/graph/+page.svelte -->
<script lang="ts">
  import { onMount } from 'svelte';
  import { getGraphData } from '$lib/api/commands';
  import * as d3 from 'd3';

  let container: HTMLElement;
  let graphData = null;

  onMount(async () => {
    graphData = await getGraphData();
    renderGraph();
  });

  function renderGraph() {
    // D3 force simulation setup
    const width = container.clientWidth;
    const height = container.clientHeight;

    const svg = d3.select(container)
      .append('svg')
      .attr('width', width)
      .attr('height', height);

    // Transform data for D3
    const nodes = graphData.nodes.map(n => ({
      id: n.id,
      label: n.label,
      type: n.node_type,
      ...n.details,
    }));

    const links = graphData.edges.map(e => ({
      source: e.source,
      target: e.target,
      type: e.edge_type,
      notes: e.notes,
    }));

    // Force simulation
    const simulation = d3.forceSimulation(nodes)
      .force('link', d3.forceLink(links).id(d => d.id).distance(140))
      .force('charge', d3.forceManyBody().strength(-500))
      .force('center', d3.forceCenter(width / 2, height / 2));

    // Draw links
    const link = svg.append('g')
      .selectAll('line')
      .data(links)
      .join('line')
      .attr('stroke', '#9ca3af')
      .attr('stroke-width', 1.5);

    // Draw nodes
    const node = svg.append('g')
      .selectAll('circle')
      .data(nodes)
      .join('circle')
      .attr('r', d => d.type === 'person' ? 7 : 9)
      .attr('fill', d => d.type === 'person' ? '#3b82f6' : '#f97316')
      .call(drag(simulation));

    // Labels
    const labels = svg.append('g')
      .selectAll('text')
      .data(nodes)
      .join('text')
      .text(d => d.label)
      .attr('font-size', 11)
      .attr('text-anchor', 'middle')
      .attr('dy', -12);

    // Tick function
    simulation.on('tick', () => {
      link
        .attr('x1', d => d.source.x)
        .attr('y1', d => d.source.y)
        .attr('x2', d => d.target.x)
        .attr('y2', d => d.target.y);

      node
        .attr('cx', d => d.x)
        .attr('cy', d => d.y);

      labels
        .attr('x', d => d.x)
        .attr('y', d => d.y);
    });
  }

  function drag(simulation) {
    return d3.drag()
      .on('start', (event, d) => {
        if (!event.active) simulation.alphaTarget(0.3).restart();
        d.fx = d.x;
        d.fy = d.y;
      })
      .on('drag', (event, d) => {
        d.fx = event.x;
        d.fy = event.y;
      })
      .on('end', (event, d) => {
        if (!event.active) simulation.alphaTarget(0);
        d.fx = null;
        d.fy = null;
      });
  }
</script>

<div bind:this={container} class="w-full h-screen"></div>
```

**Controls** (sidebar):
- Spacing slider (link distance)
- Repulsion slider
- Centering slider
- Label size slider
- Hide isolated nodes checkbox
- Show edge notes checkbox
- Node type filters
- Search box

---

## Reusable Components

### SortableTable.svelte

```svelte
<script lang="ts">
  export let columns: Array<{label: string, field: string, sortable?: boolean}>;
  export let rows: any[];
  export let sortBy: string;
  export let sortDir: 'asc' | 'desc';
  export let onSort: (field: string) => void;
  export let onRowClick: (row: any) => void = () => {};
</script>

<table class="min-w-full divide-y divide-gray-200">
  <thead class="bg-gray-50">
    <tr>
      {#each columns as col}
        <th
          class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider"
          class:cursor-pointer={col.sortable}
          on:click={() => col.sortable && onSort(col.field)}
        >
          {col.label}
          {#if col.sortable && sortBy === col.field}
            <span class="ml-1">{sortDir === 'asc' ? '▲' : '▼'}</span>
          {/if}
        </th>
      {/each}
    </tr>
  </thead>
  <tbody class="bg-white divide-y divide-gray-200">
    {#each rows as row}
      <tr
        class="hover:bg-gray-50 cursor-pointer"
        on:click={() => onRowClick(row)}
      >
        <slot {row} />
      </tr>
    {:else}
      <tr>
        <td colspan={columns.length} class="px-6 py-4 text-center text-gray-500">
          No rows.
        </td>
      </tr>
    {/each}
  </tbody>
</table>
```

### Pagination.svelte

```svelte
<script lang="ts">
  export let page: number;
  export let totalPages: number;
  export let total: number;
  export let onPageChange: (page: number) => void;
</script>

<div class="flex items-center justify-between px-4 py-3 bg-white border-t">
  <div class="text-sm text-gray-700">
    Page {page} of {totalPages} ({total} total)
  </div>
  <div class="flex gap-2">
    <button
      disabled={page === 1}
      on:click={() => onPageChange(1)}
      class="px-3 py-1 border rounded disabled:opacity-50"
    >
      First
    </button>
    <button
      disabled={page === 1}
      on:click={() => onPageChange(page - 1)}
      class="px-3 py-1 border rounded disabled:opacity-50"
    >
      Prev
    </button>
    <button
      disabled={page === totalPages}
      on:click={() => onPageChange(page + 1)}
      class="px-3 py-1 border rounded disabled:opacity-50"
    >
      Next
    </button>
    <button
      disabled={page === totalPages}
      on:click={() => onPageChange(totalPages)}
      class="px-3 py-1 border rounded disabled:opacity-50"
    >
      Last
    </button>
  </div>
</div>
```

---

## Styling (Tailwind CSS)

Same color palette as Go version:

- **Primary**: Indigo-600 (`#4f46e5`)
- **Primary Hover**: Indigo-700
- **Accent**: Orange-400 (organization nodes)
- **Text**: Gray-700 (primary), Gray-500 (secondary)
- **Error**: Red-600
- **Borders**: Gray-300

```css
/* src/app.css */
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer components {
  .btn-primary {
    @apply rounded-md bg-indigo-600 px-4 py-2 text-white
           hover:bg-indigo-700 focus:outline-none focus:ring-2
           focus:ring-indigo-500 focus:ring-offset-2;
  }

  .input {
    @apply w-full rounded-md border-gray-300 shadow-sm
           focus:border-indigo-500 focus:ring-indigo-500;
  }

  .label {
    @apply block text-sm font-medium text-gray-700;
  }
}
```

---

## Implementation Steps

### Phase 1: Project Setup
1. Create Tauri project: `npm create tauri-app`
2. Select Svelte + TypeScript template
3. Set up Tailwind CSS
4. Configure tauri.conf.json (window size, title, etc.)

### Phase 2: Database Layer (Rust)
1. Add rusqlite dependency
2. Create database module with connection pool
3. Implement migrations
4. Create model structs with serde

### Phase 3: Core Commands (Rust)
1. Implement Person CRUD commands
2. Implement Organization CRUD commands
3. Implement Interaction CRUD commands
4. Add to main.rs command registration

### Phase 4: Frontend Foundation (Svelte)
1. Create reusable components (Table, Form, etc.)
2. Set up routing structure
3. Create API wrapper module
4. Implement navigation layout

### Phase 5: Core Pages
1. People list with sorting/pagination
2. Person detail page
3. Person create/edit forms
4. Organization pages
5. Interaction pages

### Phase 6: Relationships
1. Implement relationship commands
2. Create relationship list page
3. Create relationship forms with typeahead
4. Add relationships to person detail

### Phase 7: Dashboard
1. Implement dashboard command with follow-up logic
2. Create dashboard page
3. Add global search

### Phase 8: Graph
1. Implement graph data command
2. Create graph page with D3.js
3. Add interactive controls

### Phase 9: Polish
1. Error handling and validation messages
2. Loading states
3. Window configuration (size, icon)
4. Build and test executable

---

## Tauri Configuration

```json
// src-tauri/tauri.conf.json
{
  "productName": "Network App",
  "version": "1.0.0",
  "identifier": "com.networkapp.app",
  "build": {
    "beforeDevCommand": "npm run dev",
    "beforeBuildCommand": "npm run build",
    "frontendDist": "../build"
  },
  "app": {
    "windows": [
      {
        "title": "Network App",
        "width": 1200,
        "height": 800,
        "resizable": true,
        "fullscreen": false
      }
    ],
    "security": {
      "csp": null
    }
  },
  "bundle": {
    "active": true,
    "icon": [
      "icons/32x32.png",
      "icons/128x128.png",
      "icons/128x128@2x.png",
      "icons/icon.icns",
      "icons/icon.ico"
    ],
    "windows": {
      "wix": null
    }
  }
}
```

---

## Dependencies

### Rust (Cargo.toml)

```toml
[dependencies]
tauri = { version = "2", features = [] }
serde = { version = "1", features = ["derive"] }
serde_json = "1"
rusqlite = { version = "0.31", features = ["bundled"] }
chrono = { version = "0.4", features = ["serde"] }
tokio = { version = "1", features = ["full"] }
```

### Frontend (package.json)

```json
{
  "dependencies": {
    "@tauri-apps/api": "^2"
  },
  "devDependencies": {
    "@sveltejs/adapter-static": "^3",
    "@sveltejs/kit": "^2",
    "@sveltejs/vite-plugin-svelte": "^3",
    "autoprefixer": "^10",
    "d3": "^7",
    "postcss": "^8",
    "svelte": "^4",
    "tailwindcss": "^3",
    "typescript": "^5",
    "vite": "^5"
  }
}
```

---

## Build Commands

```bash
# Development
npm run tauri dev

# Production build
npm run tauri build

# Output: src-tauri/target/release/network-app.exe
```

Expected binary size: 5-10 MB (uses system WebView)

---

## Default Seed Data

Same as Go version:

### Interaction Types
- Phone Call
- Email
- In Person
- Video Call
- Text Message

### Organization Types
- Company
- Non-Profit
- Government
- Educational
- Other
