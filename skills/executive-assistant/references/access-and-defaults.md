# Access and defaults

Customize this file for your own setup.

## Authority rules

Treat the following sender identities as principal-owned addresses that may authorize scheduling work on their behalf when the request is operationally clear:

- `{{PERSONAL_EMAIL}}`
- `{{PRIMARY_WORK_EMAIL}}`
- `{{SECONDARY_CALENDAR_EMAIL_1}}`
- `{{SECONDARY_CALENDAR_EMAIL_2}}`
- `{{SECONDARY_CALENDAR_EMAIL_3}}`

When mail comes from one of those addresses and the request is operationally clear, you may schedule on the principal's behalf without separately asking whether they are free.

### Single-operator collapse

If the install is a single-operator setup — the principal signed in to `m365 login` as themselves rather than running the pack out of a separate shared mailbox — `{{ASSISTANT_EMAIL}}` is the same mailbox as `{{PRIMARY_WORK_EMAIL}}`, every Graph call already resolves to `me`, and these authority rules collapse to "always authorized." There is no delegation boundary to police in that configuration; the authority rules only matter when a separate assistant mailbox is acting on the principal's behalf.

## Calendars to check

Verify live state with:

```bash
m365 request --url "@graph/me/calendars" --output json
```

Minimum calendar set to care about when checking availability or conflicts:

- `{{PRIMARY_WORK_EMAIL}}`
- `{{PERSONAL_EMAIL}}`
- `{{SECONDARY_CALENDAR_EMAIL_1}}`
- `{{SECONDARY_CALENDAR_EMAIL_2}}`
- `{{SECONDARY_CALENDAR_EMAIL_3}}`
- any family / personal shared calendars that materially affect availability

If one of the calendars the principal cares about is not visible in the all-calendar view, treat availability as uncertain and resolve that before booking.

## Defaults

- principal -> `{{OWNER_NAME}}`
- assistant -> `{{ASSISTANT_NAME}}`
- primary outbound operational mailbox -> `{{ASSISTANT_EMAIL}}`
- default write calendar for general business meetings -> `{{PRIMARY_WORK_EMAIL}}`
- primary proactive update route -> `{{PRIMARY_UPDATE_CHANNEL}} -> {{PRIMARY_UPDATE_TARGET}}`
- default conferencing -> Microsoft Teams (`isOnlineMeeting: true`, `onlineMeetingProvider: teamsForBusiness` when creating events via Graph)
- default event change behavior -> send updates to all attendees (Graph sends invite updates automatically when the event body changes; for cancellations use `m365 outlook event cancel` with a `--comment` so attendees get a notice)
