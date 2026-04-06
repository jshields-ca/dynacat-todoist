# dynacat-todoist

Connect your Todoist tasks, projects, and stats to your [Dynacat](https://github.com/Panonim/dynacat) self-hosted dashboard.

This repository provides ready-to-paste `custom-api` widget configurations for Dynacat. No extra server or service required — Dynacat fetches the Todoist API directly.

---

## Widgets

| Widget | File | Description |
|--------|------|-------------|
| Task List | `widgets/todoist-tasks.yml` | Open tasks with priority indicators and due dates. Filter by project, sort by priority, due date, or added date. |
| Projects Overview | `widgets/todoist-projects.yml` | All your Todoist projects with open task counts. |
| Stats | `widgets/todoist-stats.yml` | User profile, open task count, and karma/streak data if available from the API. |

A complete example page combining all three is in `examples/full-page.yml`.

---

## Setup

### 1. Get your Todoist API token

1. Log in to [Todoist](https://todoist.com)
2. Go to **Settings → Integrations → Developer**
3. Copy your **API token**

### 2. Add the token to your Dynacat `.env` file

```env
TODOIST_API_TOKEN=your_personal_api_token_here
```

See `.env.example` for the full template.

### 3. Add a widget to your Dynacat `config.yml`

Copy the widget block you want from the files in `widgets/` and paste it into a column's `widgets:` list in your Dynacat config. For example:

```yaml
pages:
  - name: Home
    columns:
      - size: small
        widgets:
          # Paste widget block here
          - type: custom-api
            title: Todoist Inbox
            cache: 5m
            url: https://api.todoist.com/api/v1/tasks
            headers:
              Authorization: Bearer ${TODOIST_API_TOKEN}
            parameters:
              sorted_by: priority
              sort_order: asc
            template: |
              ...
```

### 4. Restart Dynacat

```bash
# Docker
docker compose restart

# Binary
systemctl restart dynacat
```

---

## Task List Widget

**File:** `widgets/todoist-tasks.yml`

Shows open tasks with colour-coded priority dots and due dates.

**Priority colours** (matching Dynacat's native palette):

| Priority | Dot | Label |
|----------|-----|-------|
| 4 — Urgent | Red (`color-negative`) | ● |
| 3 — High | Yellow/orange (`color-highlight`) | ● |
| 2 — Medium | Blue (`color-primary`) | ● |
| 1 — Normal | Grey (`color-subdue`) | ○ |

### Sorting options

Set via the `parameters` block in your config:

```yaml
parameters:
  sorted_by: priority     # priority | due_date | added_date
  sort_order: asc         # asc | desc
```

### Filtering by project

Add `project_id` to the parameters:

```yaml
parameters:
  project_id: "1234567890"
  sorted_by: due_date
  sort_order: asc
```

**How to find your project ID:**
1. Open Todoist in your browser
2. Navigate to the project
3. The numeric ID is in the URL: `todoist.com/app/project/2349823456`

Or use the Projects widget (Variant 2, simplified list) which shows IDs next to project names.

### Multiple project widgets

You can have as many task list widgets as you like — one per project, each with its own title and sort settings:

```yaml
- type: custom-api
  title: Work
  ...
  parameters:
    project_id: "111111111"
    sorted_by: due_date

- type: custom-api
  title: Personal
  ...
  parameters:
    project_id: "222222222"
    sorted_by: priority
```

---

## Projects Widget

**File:** `widgets/todoist-projects.yml`

**Variant 1** (default): Lists all your projects with open task count badges. Makes two API calls concurrently (projects + tasks).

**Variant 2** (simple): Lists projects with their IDs — useful for first-time setup to find your project IDs. Single API call.

---

## Stats Widget

**File:** `widgets/todoist-stats.yml`

Shows your Todoist user profile and productivity stats.

**Always available:**
- Full name
- Premium status
- Open task count

**Available if returned by Todoist API v1:**
- Karma score
- Karma trend (up/down/same)
- Daily goal
- Weekly goal
- Completed today
- Current streak (days)

> **Note on karma & streaks:** The old Todoist Sync API v9 (which explicitly exposed karma, streaks, and daily goals) was shut down in February 2026. The new API v1 `/user` endpoint may include these fields — the widget displays them if present and gracefully skips them if not. See troubleshooting below if you want to check what your account returns.

---

## Caching

Dynacat caches widget responses. Recommended values:

| Widget | Suggested cache |
|--------|-----------------|
| Task list | `5m` |
| Projects | `10m` |
| Stats / karma | `30m` |

Change the `cache:` field in any widget block to suit your needs. During development/testing, set `cache: 1s` to see changes immediately without restarting Dynacat.

---

## Troubleshooting

### Widget shows no content / blank

1. Check that `TODOIST_API_TOKEN` is set correctly in your Dynacat `.env`
2. Check Dynacat's logs for HTTP errors: `docker compose logs -f` or `journalctl -u dynacat`
3. Test the API directly:
   ```bash
   curl -H "Authorization: Bearer YOUR_TOKEN" https://api.todoist.com/api/v1/tasks
   ```
   You should get a JSON array back.

### Template errors in logs

- The Go template syntax is strict. Check that indentation in the `template: |` block is consistent.
- Ensure you're using Dynacat (or a recent Glance fork) that supports `custom-api` with `subrequests`.

### Checking what the API v1 user endpoint returns (for karma fields)

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" \
  https://api.todoist.com/api/v1/user | python3 -m json.tool
```

Look for fields like `karma`, `karma_trend`, `streak_days`, `daily_goal`, `weekly_goal` in the output. If they're present, the stats widget will display them automatically.

### Project ID not working

Verify the project ID by listing your projects:
```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" \
  https://api.todoist.com/api/v1/projects | python3 -m json.tool
```

Make sure you're passing the ID as a string in YAML:
```yaml
parameters:
  project_id: "1234567890"   # quotes required
```

---

## API Reference

All widgets use [Todoist API v1](https://developer.todoist.com/api/v1/).

- **Auth:** `Authorization: Bearer {token}` header
- **Tasks:** `GET https://api.todoist.com/api/v1/tasks`
- **Projects:** `GET https://api.todoist.com/api/v1/projects`
- **User:** `GET https://api.todoist.com/api/v1/user`
- **Rate limit:** 1000 requests per 15 minutes per user

---

## Dynacat `custom-api` reference

For template syntax and all available options, see the [Dynacat / Glance configuration docs](https://github.com/glanceapp/glance/blob/main/docs/configuration.md#custom-api).

Templates use [gjson](https://github.com/tidwall/gjson) path syntax for JSON access.

---

## Contributing

Issues and pull requests welcome. If you find that additional Todoist API v1 fields are available (especially karma/streak data), please open an issue or PR to update the stats widget.

---

## License

MIT
