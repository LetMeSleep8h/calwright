# Good first issues

New to calwright and want to contribute? These are small, well-scoped tasks.
Each one touches a single area of the codebase and has tests to prove it works.

## 1. Accept `EXDATE` / exception dates for recurring events

**Problem:** A recurring event sometimes needs one occurrence skipped (a holiday,
a cancelled week). calwright passes RRULE through but has no way to express
exceptions.

**Work:** Add an `exdate` column (comma-separated dates, same flexible parsing
as `start`). Emit one `EXDATE` property per date, with the event's TZID or UTC
form. Add validation so an EXDATE that does not parse is reported by `--check`.

**Files:** `calwright/converter.py` (`Event`, `build_events`, `render_ics`),
`tests/test_calwright.py`.

**Tests to add:** parsing of a single and a comma-separated list, ICS output
contains the EXDATE lines, `--check` flags an unparseable EXDATE with the row
number.

## 2. Warn when an RRULE UNTIL is earlier than DTSTART

**Problem:** `validate_rrule` checks grammar but never compares UNTIL against the
event's start. A rule like `FREQ=DAILY;UNTIL=20200101T000000Z` on a 2026 event
produces zero occurrences, which is almost always a typo.

**Work:** In `build_events`, after parsing start and validating the RRULE,
compare UNTIL (normalized to the event's timezone) with the start datetime and
add a row error when UNTIL is earlier. Keep the check tolerant of date-only
UNTIL values.

**Files:** `calwright/converter.py`, `tests/test_calwright.py`.

**Tests to add:** UNTIL before start is flagged with the row number; UNTIL after
start passes; a date-only UNTIL equal to the start date passes.

## 3. Support per-row `DURATION` in ICS output (VEVENT DURATION property)

**Problem:** calwright currently accepts a `duration` column but always converts
it to DTEND. Some calendar clients prefer the RFC 5545 `DURATION` property on
the event itself.

**Work:** Add a `--prefer-duration` CLI flag. When set, timed events that were
given a `duration` column emit `DURATION:PT...` instead of DTEND (keep DTEND as
the default behavior). Format the ISO 8601 duration from the parsed timedelta.

**Files:** `calwright/converter.py`, `calwright/cli.py`, `tests/test_calwright.py`.

**Tests to add:** flag on produces DURATION and no DTEND; flag off keeps current
behavior; a 90-minute duration renders as `DURATION:PT1H30M`.

## 4. Add a `--from-json` input mode

**Problem:** calwright only reads CSV. Pipelines that already produce JSON
(many APIs, jq output) have to convert to CSV first.

**Work:** Add `--from-json` so the input is parsed as a JSON array of objects
with the same canonical field names (`title`, `start`, `end`, ...). Reuse
`build_events` unchanged; only the loader differs. Document it in the README
with an example.

**Files:** `calwright/converter.py` (new `load_json_rows` next to `load_rows`),
`calwright/cli.py`, `tests/test_calwright.py`, `README.md`.

**Tests to add:** JSON array loads to the same events as the equivalent CSV;
missing field names are tolerated; invalid JSON exits with code 2.
