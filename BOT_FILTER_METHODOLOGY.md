# Bot-filtering methodology for windowed open-rate metrics (2025 data)

## Why this exists

The dashboard's existing `openRate`/`clickRate` figures come from HubSpot's exported
summary PDFs, which apply HubSpot's own "exclude bots" filter before totaling opens.
That filter is proprietary - we don't know its exact rules.

To compute a time-windowed metric (e.g. "% opened within 14 days of send") we instead
work from HubSpot's raw "Advanced Recipient Events List" CSV export, which has no bot
flag at all: every automated/prefetch open counts the same as a genuine human open.
Using it unfiltered roughly **doubles** the open rate vs. the dashboard's existing
numbers (e.g. 39.3% raw vs. ~20% dashboard for Homebuilder Survey, Jan 2025).

## Filter used

An open's `User Agent` is treated as a bot/non-human signal, and excluded, if it matches
any of:

- Bare `Mozilla/5.0` or `Mozilla` with no OS/engine info (real browsers always report more)
- Contains `lua-resty-http` (a Lua HTTP library - not a mail client)
- Contains `GoogleImageProxy` (Gmail's image-proxy prefetch)

Deliberately **not** filtered: `Mozilla/4.0 (compatible; ms-office; MSOffice ...)`
(Outlook desktop's automatic remote-image fetch). Testing showed excluding this
undercounts real opens - it dropped every report tested to 12-15%, well below the
dashboard baseline - so despite being a common "phantom open" suspect, it's kept as
genuine here.

This is a **best-effort approximation**, not a reproduction of HubSpot's actual bot
list. Treat the resulting rates as directionally comparable to the dashboard, not
exact matches.

## Validation (January 2025 sends, dashboard baseline = typical range for that report across other 2025/2026 months)

| Report | Sent | Bot-filtered open rate | Dashboard baseline range |
|---|---|---|---|
| Homebuilder Survey | 11,017 | 20.53% | 19.2-21.5% |
| Homebuilder Survey First Glance | 7,125 | 21.26% | 19.6-23.9% |
| HBAF | 2,102 | 21.65% | 24.2-25.7% |
| Building Products Dealer Survey | 2,819 | 19.23% | 17.4-21.6% |
| BP Quarterly | 2,760 | 21.34% | 19.9-21.4% |
| Regional Analysis and Forecast | 5,825 | 20.5% | (not yet cross-checked) |
| RCAF / BTR Apartment Update | 6,259 | 21.28% | (not yet cross-checked) |
| Real Estate Agent Survey | 12,060 | 17.88% | (not yet cross-checked) |
| Residential Land Survey | 11,964 | 18.97% | (not yet cross-checked) |
| NHTI Insights | 1,360 | 24.93% | (not yet cross-checked) |

Most land within a few points of their report's typical dashboard range. HBAF runs
consistently ~3-4 points low under this filter - a known gap, not a bug to chase further
without ground truth from HubSpot.

## Applied consistently

This same filter (see `parse_events.py` in the scratchpad) is used for every 2025
monthly file processed for the windowed open-rate project, so results stay comparable
across months/reports even though the filter isn't a perfect match to HubSpot's own.
