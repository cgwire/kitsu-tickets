# Tickets

Kitsu plugin for ticket management (issues on tasks).

Reference: [Kitsu Plugin Installation](https://dev.kitsu.cloud/kitsu-plugins/installation.html)

## Installation

### Option A: From a local folder

1. Download the plugin folder.
2. Install the plugin:

   ```bash
   git clone https://github.com/cgwire/kitsu-tickets.git
   zou install-plugin --path ./kitsu-tickets
   ```

3. Restart the Zou server.

### Option B: From the Git repository

1. Clone the repository and install from the cloned folder:

   ```bash
   zou install-plugin --path https://github.com/cgwire/kitsu-tickets.git
   ```

2. Restart the Zou server.

## Tables added

| Table | Description |
|-------|-------------|
| `plugin_tickets_tickets` | Tickets (title, text, status, task_id, project_id, episode_id, person_id, assignee_id, etc.) |

## Routes added

All routes are prefixed by **`plugins/tickets`**. JWT authentication required.

| Method | Path | Description |
|--------|------|-------------|
| GET | `/tickets` | List all tickets |
| POST | `/tickets` | Create a ticket |
| GET | `/tickets/<id>` | Get a ticket |
| DELETE | `/tickets/<id>` | Delete a ticket |

Example full URL: `https://your-zou/api/plugins/tickets/tickets`.