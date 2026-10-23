# First Words

A voice-dictation onboarding prototype that A/B tests two first-time experiences:

- **Variant A (standard setup):** welcome → microphone explanation → instructions → empty box, "say anything".
- **Variant B (guided first win):** say a given sentence straight away → "you just saved X seconds" screen → a second, real dictation → choose a day-2 reminder.

Both end with: "How likely are you to use voice typing again tomorrow?" (1–5).

Each visitor is randomly assigned one variant and keeps it. Every step is logged as an event tagged with the variant.

## 1. Put it online (GitHub Pages, free, ~5 minutes)

Speech recognition only works on a secure (https) page, which GitHub Pages gives you.

1. On GitHub, create a new **public** repository, e.g. `first-words`.
2. Click **Add file → Upload files**, upload `index.html` (and this README), and commit.
3. Go to **Settings → Pages**. Under "Branch", choose `main` and `/ (root)`, then **Save**.
4. After a minute your link appears: `https://<your-username>.github.io/first-words/`.

Testers should open it in **Chrome or Edge** (desktop or Android). Safari and Firefox have limited speech support; the page falls back to typing there and records that.

## 2. Turn on analytics (PostHog, free)

1. Sign up at posthog.com (the free tier is plenty) and create a project.
2. Copy your **Project API key** (starts with `phc_`) from Project settings.
3. In `index.html`, near the top of the script, paste it:
   `const POSTHOG_KEY = "phc_...";`
   If your project is in the EU region, also set `POSTHOG_HOST = "https://eu.i.posthog.com"`.
4. Commit the change on GitHub. Events now go to PostHog.

Without a key, events are stored only in each tester's own browser.

## 3. Read the results in PostHog

- **Funnel:** Product analytics → New insight → Funnel. Steps: `onboarding_started` → `first_dictation_success` → `onboarding_completed`. Set "Breakdown" to the event property `variant`.
- **Time to first success:** Trends on `first_dictation_success`, aggregating the property `time_to_success_ms` by median, broken down by `variant`.
- **Survey:** Trends on `survey_submitted`, average of `likelihood`, broken down by `variant`. Comments are in the `comment` property.
- **Guardrails:** counts of `onboarding_skipped`, `mic_permission_denied` and `dictation_error` by variant.
- **Check the split:** `onboarding_started` counts should be roughly equal between A and B.

## Testing tips

- Add `?debug=1` to the link to see the variant and a live event log.
- Force a variant with `?v=a` or `?v=b` (for your own testing only; send testers the plain link).
- A tester keeps their first variant in that browser, so repeat visits don't mix groups.

## Events

| Event | When | Key properties |
| --- | --- | --- |
| `page_viewed` | page loads | `speech_supported` |
| `onboarding_started` | first button pressed | |
| `permissions_step_completed`, `howto_step_completed` | Variant A setup steps | |
| `dictation_started` / `dictation_completed` | each dictation | `words`, `seconds`, `step`, `typed_fallback` |
| `first_dictation_success` | first dictation of 4+ words | `time_to_success_ms`, `words` |
| `aha_screen_viewed` | Variant B time-saved screen | `wpm`, `seconds_saved` |
| `real_use_context_selected` / `second_dictation_success` | Variant B real-use step | `context` |
| `habit_nudge_choice` | Variant B reminder step | `choice`, `opted_in` |
| `onboarding_completed` / `onboarding_skipped` | end of flow | `at` |
| `survey_submitted` | final question | `likelihood`, `comment` |
| `mic_permission_denied`, `dictation_error`, `speech_unsupported` | problems | `error` |

---
An independent prototype by Bhanavi D, not affiliated with any company.
