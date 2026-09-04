# TaskGraph Model

## Overview

TaskGraph is a hierarchical task planning model for managing complex work through recursively nested tasks.

It supports:

- manually created tasks
- unlimited parent-child task depth
- reusable task templates
- grouped alternative templates for the same outcome
- template execution with input parameters
- calendar and list views of concrete tasks
- progress tracking derived through the task tree
- structured JSON results stored on tasks

The model is designed so that the actual executable unit is always the `TaskTemplate`, while `TaskTemplateGroup` is only used to organize related templates into alternative strategies.

---

## Core Concepts

### Tasks
A task is the concrete unit of work.  
A task may stand alone or belong to another task as a child.  
Tasks support unbounded nesting through a self-referential parent relationship.

### Templates
A template is a reusable task generator.  
It takes input parameters and produces concrete tasks, potentially spread across time.

### Template Groups
A template group is a container for multiple templates that represent alternative ways of achieving the same goal.  
A group is organizational, not executable.

### Progress
Every task has a progress value between `0` and `100`.

- If a task has no children, its progress is manually managed.
- If a task has children, its progress is derived from the average progress of its direct children.

### Result
A task may store a structured JSON result representing the output it produced.

---

## Entities

## 1. Task

### Description
A `Task` is the central unit of work in the system.

A task can be:

- a root task
- a child task
- manually created
- generated from a template instance

A task may store scheduling data for calendar rendering, progress information, and a structured JSON result.

### Fields

| Field | Type | Nullable | Description |
|---|---|---:|---|
| `id` | UUID / string | No | Unique identifier for the task. |
| `parent_task_id` | UUID / string | Yes | References another `Task.id`. Null means the task is a root task. |
| `title` | string | No | Short title of the task. |
| `description` | text | Yes | Longer explanation or instructions for the task. |
| `scheduled_start_at` | datetime | Yes | Intended start time of the task. Used for calendar placement. |
| `scheduled_end_at` | datetime | Yes | Intended end time of the task. Used for calendar placement and time blocking. |
| `due_at` | datetime | Yes | Deadline for the task, if applicable. |
| `status` | enum / string | No | Current state of the task. |
| `progress` | decimal / integer | No | Progress percentage from `0` to `100`. Manually managed for leaf tasks and derived for parent tasks. |
| `result` | JSON | Yes | Structured output produced by the task. Null means no output has been recorded. |
| `sort_order` | integer | Yes | Relative ordering among sibling tasks under the same parent. |
| `template_instance_id` | UUID / string | Yes | References the `TemplateInstance` that generated this task. Null if the task was created manually. |
| `created_at` | datetime | No | Timestamp when the task was created. |
| `updated_at` | datetime | No | Timestamp when the task was last updated. |

### Behavior
- A task with `parent_task_id = null` is a root task.
- A task with a non-null `parent_task_id` is a child of another task.
- A task may exist without any scheduling fields.
- A task may exist without a result.

### Progress Rules
- If a task has no children, `progress` is manually editable.
- If a task has one or more children, `progress` is derived.
- Parent progress is calculated as the arithmetic average of the progress values of its direct children.
- Any update to a leaf task’s progress triggers recursive recalculation of all ancestor tasks.

### Result Rules
- `result` stores the structured output of the task itself.
- `result` is not automatically derived from children.
- Parent tasks may also have their own `result`, but that is independent from child task results.

---

## 2. TaskTemplate

### Description
A `TaskTemplate` is a reusable task generator.

It defines a code-backed template that accepts input parameters and generates concrete tasks.  
A template may be standalone or may belong to a `TaskTemplateGroup`.

### Fields

| Field | Type | Nullable | Description |
|---|---|---:|---|
| `id` | UUID / string | No | Unique identifier for the task template. |
| `task_template_group_id` | UUID / string | Yes | References `TaskTemplateGroup.id`. Null means the template is standalone. |
| `code` | string | No | Stable code identifier used by the application to locate the template implementation. |
| `name` | string | No | Human-readable name of the template. |
| `description` | text | Yes | Optional explanation of what the template generates. |
| `created_at` | datetime | No | Timestamp when the template was created. |
| `updated_at` | datetime | No | Timestamp when the template was last updated. |

### Behavior
- A template is always the executable generation unit.
- A standalone template has no group.
- A grouped template represents one alternative strategy inside a template group.

---

## 3. TaskTemplateGroup

### Description
A `TaskTemplateGroup` is an organizational container for related task templates.

It is used when multiple templates represent alternative ways to achieve the same conceptual outcome.

A group does not generate tasks by itself. Its purpose is to group executable templates.

### Fields

| Field | Type | Nullable | Description |
|---|---|---:|---|
| `id` | UUID / string | No | Unique identifier for the template group. |
| `name` | string | No | Human-readable name of the group. |
| `description` | text | Yes | Optional explanation of the purpose shared by the grouped templates. |
| `created_at` | datetime | No | Timestamp when the group was created. |
| `updated_at` | datetime | No | Timestamp when the group was last updated. |

### Behavior
A template group may contain multiple templates such as:

- automation-friendly
- all manual
- fastest
- lowest risk

Each member of the group is still a normal `TaskTemplate`.

---

## 4. TemplateInstance

### Description
A `TemplateInstance` represents one execution of a task template with a specific set of input parameters.

It records which template was executed, what input was supplied, and when task generation occurred.

This allows the same template to be executed multiple times with different parameters and schedules.

### Fields

| Field | Type | Nullable | Description |
|---|---|---:|---|
| `id` | UUID / string | No | Unique identifier for the template execution instance. |
| `task_template_id` | UUID / string | No | References `TaskTemplate.id`. |
| `input` | JSON | No | Input parameters supplied to the template at generation time. |
| `generated_at` | datetime | No | Timestamp when the template was executed and tasks were generated. |
| `created_at` | datetime | No | Timestamp when the instance record was created. |
| `updated_at` | datetime | No | Timestamp when the instance record was last updated. |

### Behavior
- A template instance always references exactly one `TaskTemplate`.
- A template instance may generate one or many tasks.
- Generated tasks reference the template instance through `Task.template_instance_id`.

---

## Relationships

### Task Hierarchy
- `Task.parent_task_id -> Task.id`
- One task may have many child tasks.
- One task may belong to zero or one parent task.

### Template Grouping
- `TaskTemplate.task_template_group_id -> TaskTemplateGroup.id`
- One template group may contain many templates.
- One template may belong to zero or one template group.

### Template Execution
- `TemplateInstance.task_template_id -> TaskTemplate.id`
- One template may have many execution instances.
- One execution instance belongs to exactly one template.

### Generated Tasks
- `Task.template_instance_id -> TemplateInstance.id`
- One template instance may generate many tasks.
- A task may or may not originate from a template instance.

---

## Derived Behavior

### Manual Task Creation
A task may be created directly:

- as a root task by leaving `parent_task_id` null
- as a child task by setting `parent_task_id`

### Template-Based Task Generation
A template is selected through `TaskTemplate.code`, executed with `TemplateInstance.input`, and produces concrete `Task` records.

### Alternative Strategies
When multiple templates belong to the same `TaskTemplateGroup`, they represent alternative generation strategies for the same conceptual outcome.

### Calendar View
Calendar rendering is derived from:

- `scheduled_start_at`
- `scheduled_end_at`
- `due_at`

### List View
List rendering is derived from:

- hierarchy via `parent_task_id`
- ordering via `sort_order`
- filtering and grouping via `status`

### Progress Propagation
- Leaf task progress is manually updated.
- Parent task progress is recalculated from direct children.
- Any leaf progress change recursively updates all ancestors.

### Result Storage
- Task output is stored on `Task.result`.
- Template generation input is stored on `TemplateInstance.input`.

This creates a clear separation between:

- generation input
- concrete task execution output

---

## Nullability Rationale

### `parent_task_id`
Nullable because a task may exist at the root level.

### `description`
Nullable because some tasks, templates, or groups may be self-explanatory from their name or title alone.

### Scheduling Fields
Nullable because not every task needs time-blocking or a deadline.

### `result`
Nullable because not every task produces a structured output, and some tasks may not yet be completed.

### `template_instance_id`
Nullable because tasks may be created manually instead of being template-generated.

### `task_template_group_id`
Nullable because templates may be standalone and not part of an alternative strategy set.

---

## Minimal First-Version Scope

This model includes the minimum clean set of entities required to support the system:

- `Task`
- `TaskTemplate`
- `TaskTemplateGroup`
- `TemplateInstance`

This is sufficient for:

- complex hierarchical task breakdown
- unlimited subtask depth
- reusable code-defined templates
- grouped alternative strategies
- template execution with inputs
- weekly calendar and list rendering
- recursive progress rollup
- structured JSON task outputs

---

## Summary

TaskGraph separates the model into four simple concerns:

- `Task` for concrete work
- `TaskTemplate` for reusable generation logic
- `TaskTemplateGroup` for organizing alternatives
- `TemplateInstance` for capturing one execution of a template

This keeps the model minimal while supporting both manual and template-driven planning at arbitrary depth.
