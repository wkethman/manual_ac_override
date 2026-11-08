# manual_ac_override

Home Assistant automation blueprint: when a user manually changes a
thermostat's target temperature, an override toggle (input_boolean) is
turned on for a set duration so scheduled automations (e.g. an AC
setpoint controller) stand down. When the countdown ends, the toggle is
turned off and automation resumes.

## Changelog

### v2.1 (2026-11-08)
- Versioned, self-documenting `description` field (what the blueprint does +
  changelog), so the automation UI shows the version and how it works.

### v2.0 (2026-11-08) — countdown moved to an `input_timer`
The previous version held the countdown as an in-automation `delay` under
`mode: restart`. Any device-initiated change of the thermostat's
`temperature` attribute (which the thermostat itself does periodically)
would *cancel* the pending delay, and the replacement run re-armed nothing,
so the override toggle could get stuck `on` indefinitely (observed: stuck
for 5+ days, permanently blocking the setpoint controller).

- **New required input:** `override_timer` (an `input_timer` helper) now
  holds the countdown.
- Repeated manual temperature changes **reset** the timer instead of
  cancelling the countdown (the old design silently ignored them).
- `mode: single` — runs are short (no blocking delays), so device-side
  attribute changes can no longer kill the countdown.
- **Restart self-heal:** on `homeassistant_started`, if the override is
  still on and the timer is idle, the countdown is re-armed so the
  override can no longer survive a Home Assistant restart.
- Manual `on` of the override toggle still starts a full countdown window.

**Migration:** create one `input_timer` helper per thermostat (e.g.
`input_timer.downstairs_ac_override_timer`, `input_timer.upstairs_ac_override_timer`)
and provide it to the new `override_timer` input when (re)configuring the
automation. Existing automations built on v1.x must be updated to supply
the new input.
