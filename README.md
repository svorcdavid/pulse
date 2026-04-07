# Pulse

A simple task management application built on Alpine.js and PocketBase for managing tasks with support for priorities, statuses, assignments, and task hierarchies.

## Requirements

- PocketBase instance
- Modern web browser with ES6 support

## Installation

1. Clone or download this repository.
2. Open `index.html` in your browser.
3. Enter your PocketBase instance URL and log in.

## PocketBase Setup

### Tables

Create the following collections in the PocketBase admin panel:

#### 1. users (standard collection)
- Add a custom field `apps` (JSON array, e.g., `["pulse"]` or list) for filtering/enabling app users.

#### 2. pulse_tasks
Fields:
- `title` (text, required)
- `description` (text)
- `due_date` (date)
- `assignee` (relation to users, multiple)
- `creator` (relation to users)
- `status` (relation to pulse_statuses)
- `parent_task` (relation to pulse_tasks, self-reference)
- `priority` (relation to pulse_priorities)

#### 3. pulse_statuses
Fields:
- `title` (text, required)
- `sort_order` (number)
- `is_closed` (bool)

Create at least basic statuses, e.g.:
- "Open" (is_closed: false)
- "Closed" (is_closed: true)

#### 4. pulse_priorities
Fields:
- `title` (text, required)
- `level` (number, lower number = higher priority)
- `color` (text, hex code, e.g., "#ff0000")

Create priorities, e.g.:
- "High" (level: 1, color: "#ef4444")
- "Medium" (level: 2, color: "#f59e0b")
- "Low" (level: 3, color: "#10b981")

### PocketBase API Rules (Security)

Set the following API rules to restrict access:

#### users
- **List**: `@request.auth.id != "" && apps ~ '{"pulse"}'` (only authenticated users with app "pulse")
- **View**: `@request.auth.id != ""`
- **Create**: `true`
- **Update**: `@request.auth.id = id` (user can only edit themselves)
- **Delete**: `false` (disable user deletion)

#### pulse_tasks
- **List**: `@request.auth.id != "" && (assignee ~ @request.auth.id || creator = @request.auth.id)`
- **View**: `@request.auth.id != "" && (assignee ~ @request.auth.id || creator = @request.auth.id)`
- **Create**: `@request.auth.id != ""` (automatically set creator)
- **Update**: `@request.auth.id != "" && (assignee ~ @request.auth.id || creator = @request.auth.id)`
- **Delete**: `@request.auth.id != "" && creator = @request.auth.id` (only creator can delete)

#### pulse_statuses
- **List**: `@request.auth.id != ""`
- **View**: `@request.auth.id != ""`
- **Create/Update/Delete**: `false` (admin only in admin panel)

#### pulse_priorities
- **List**: `@request.auth.id != ""`
- **View**: `@request.auth.id != ""`
- **Create/Update/Delete**: `false` (admin only in admin panel)

These rules ensure that:
- Unauthenticated users cannot access data
- Users see only their tasks or tasks they created
- Sensitive collections (statuses, priorities) are read-only for users
- Loading all users is restricted to authenticated users with app "pulse"

## How to Run

1. Start your PocketBase server.
2. Open `index.html` in your browser.
3. Enter the PocketBase URL and log in.
4. Start adding tasks!

## Features

- **Core Mode**: Task list with priorities and deadlines
- **Master Mode**: Hierarchical project tree
- **Task Assignment**: Assigning tasks to users
- **Priorities and Statuses**: Managing task priorities and statuses
- **Real-time Updates**: Automatic updates via PocketBase subscriptions
- **Responsive Design**: Works on desktop and mobile

## Security

The application uses:
- Authentication via PocketBase
- Input sanitization to prevent XSS
- API rules to restrict data access
- HTTPS recommended for production deployment
