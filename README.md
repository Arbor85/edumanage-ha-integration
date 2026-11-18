# EduManage — Home Assistant Integration

Expose your [EduManage](https://github.com/Arbor85/edumanage-portal) routines, workout queue, and controls to Home Assistant. Entities-first, configured entirely through the HA UI.

## What you get

| Entity | Type | Purpose |
|---|---|---|
| `sensor.edumanage_queued_routine` | sensor | Name of the queued routine, or `none`. Attributes include queue id, routine id, source, created-at. |
| `sensor.edumanage_routines` | sensor | Count of available routines. |
| `button.edumanage_start_<routine>` | button (one per routine, dynamic) | Pressing queues that routine. |

Services:

- `edumanage.queue_routine` — field `routine_id` (string). Queues a routine.
- `edumanage.cancel_queue` — clears any pending queued routines.

## Install via HACS

1. In Home Assistant, open **HACS → Integrations**.
2. **⋮** (top right) → **Custom repositories**.
3. Add `https://github.com/Arbor85/edumanage-ha-integration`, category **Integration** → **Add**.
4. Find **EduManage** → **Download** → pick the latest release.
5. Restart Home Assistant.
6. **Settings → Devices & services → Add integration → EduManage**.
7. Enter the backend base URL (e.g. `http://192.168.1.10:5090`) and your API key from EduManage → Settings → MCP API Keys.

## Install manually

1. Copy `custom_components/edumanage/` into your HA config dir:
   ```
   <ha-config>/custom_components/edumanage/
   ```
2. Restart Home Assistant and continue with step 6 above.

## Automations

Queue a routine every weekday morning:

```yaml
alias: Queue morning routine
trigger:
  - platform: time
    at: "06:30:00"
condition:
  - condition: time
    weekday: [mon, tue, wed, thu, fri]
action:
  - service: edumanage.queue_routine
    data:
      routine_id: 01HXYZ...  # from the Routines sensor attributes or the EduManage web app
```

Announce when a routine gets queued:

```yaml
trigger:
  - platform: state
    entity_id: sensor.edumanage_queued_routine
    from: "none"
action:
  - service: tts.google_translate_say
    data:
      entity_id: media_player.kitchen
      message: "Your {{ states('sensor.edumanage_queued_routine') }} routine is queued."
```

Every routine also appears as a standard button entity — drop them into any `entities` or `grid` card.

## Options

**Settings → Devices & services → EduManage → Configure** lets you change the poll interval (10–600 s, default 30 s).

## Troubleshooting

- **Config flow rejects the key** — backend returned 401; rotate the key in EduManage.
- **"Could not reach the EduManage backend"** — verify the URL is reachable from the HA host. Container deployments often need `http://host.docker.internal:5090` or the LAN IP, not `localhost`.
- **Buttons go unavailable** — the matching routine was deleted from EduManage. The entity stays registered so automations don't silently break.

## Related

- [Lovelace card](https://github.com/Arbor85/edumanage-portal/tree/main/ha-card) — the dashboard card version, installed as a HACS Dashboard plugin from the main `edumanage-portal` repo.

## License

MIT
