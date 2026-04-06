# dynacat-todoist

Connect your Todoist productivity stats to your [Dynacat](https://github.com/Panonim/dynacat) self-hosted dashboard.

A single `custom-api` widget that shows karma, streaks, task stats, priority breakdown, and projects — all in one place. No extra server required; Dynacat fetches the Todoist API directly.

---

## What It Shows

| Section | Details |
|---------|---------|
| **Karma** | Current score with up/down trend indicator |
| **Completed today** | Tasks done today vs. your daily goal |
| **All-time completed** | Total tasks completed across all time |
| **Open tasks** | Total count with priority breakdown (urgent / high / medium / normal) |
| **Labels** | Tag cloud of all labels in use across open tasks |
| **Projects** | All project names as compact badges, with total project count |
| **Open Todoist** | Direct link to the Todoist app |

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

### 3. Add the widget to your Dynacat `config.yml`

Copy the full widget block from `widgets/todoist-dashboard.yml` and paste it into a column's `widgets:` list:

```yaml
pages:
  - name: Home
    columns:
      - size: small
        widgets:
          # paste contents of widgets/todoist-dashboard.yml here
```

### 4. Restart Dynacat

```bash
# Docker
docker compose restart

# Binary
systemctl restart dynacat
```

---

## Caching

The widget defaults to `cache: 15m`. Change it to suit your needs. Use `cache: 1s` while developing to see changes without restarting Dynacat.

---

## Troubleshooting

### Widget shows no content / blank

1. Confirm `TODOIST_API_TOKEN` is set in your Dynacat `.env`
2. Check logs: `docker compose logs -f`
3. Test the API directly:
   ```bash
   curl -s -H "Authorization: Bearer YOUR_TOKEN" \
     https://api.todoist.com/api/v1/user | python3 -m json.tool
   ```

### Streaks not showing

Streak data (current streak, best streak) is **not exposed by Todoist API v1**. It is only visible inside the Todoist app under the Productivity section. If Todoist adds streak fields to the API in a future update, the widget will display them automatically since it uses `.Exists` guards.

---

## API Reference

Uses [Todoist API v1](https://developer.todoist.com/api/v1/).

- **Auth:** `Authorization: Bearer {token}` header
- **User/Stats:** `GET https://api.todoist.com/api/v1/user`
- **Tasks:** `GET https://api.todoist.com/api/v1/tasks`
- **Projects:** `GET https://api.todoist.com/api/v1/projects`
- **Rate limit:** 1000 requests / 15 minutes per user

---

## Dynacat `custom-api` reference

For template syntax and all available options, see the [Glance configuration docs](https://github.com/glanceapp/glance/blob/main/docs/configuration.md#custom-api).

Templates use [gjson](https://github.com/tidwall/gjson) path syntax for JSON access.

---

## Contributing

Issues and pull requests welcome. If you discover additional Todoist API v1 fields (streaks, goals, etc.), please open an issue or PR.

---

## License

MIT
