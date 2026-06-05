# Data Verification — Slides 6 & 7

**Subject:** Accuracy of the charts on slides 6 and 7 of `presentation.html`
**Data source:** PostgreSQL database `uni_admin_db`, schema `banner_mirror` (Banner 9 table layout)
**Programme under discussion:** `DL_SCYPY_O`
**Core module (Stage 3):** `PSYL_H3010` — "Intro to Cyberpsychology"

---

## Summary

Both charts were checked row-by-row against the `banner_mirror` data. **Every plotted value
is accurate, and both charts reconcile exactly with the methodology used by the AdminPortal
application** that is to be demonstrated live. No numbers require correction for accuracy or
for consistency with the app.

One analytical limitation is worth noting before the presentation: both the slides and
AdminPortal measure the inactive rate **among students still registered for the module**, which
does not capture students who left the programme entirely. This is described in the
"Methodological caveat" section below.

---

## Slide 7 — `PSYL_H3010` Active vs Inactive by year

The chart plots, per academic year, the count of students taking `PSYL_H3010` with an Active
(`SGBSTDN_STST_CODE = 'AS'`) versus Inactive (`'IS'`) student status at year-end.

**Result: fully accurate.** All eighteen values match the database exactly.

| Year  | 17/18 | 18/19 | 19/20 | 20/21 | 21/22 | 22/23 | 23/24 | 24/25 | 25/26 |
|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|
| Active   | 17 | 15 | 18 | 30 | 24 | 19 | 25 | 20 | 22 |
| Inactive |  5 |  4 |  5 |  8 |  3 |  3 |  4 |  6 |  3 |

(Slide value = database value for every cell.)

---

## Slide 6 — `DL_SCYPY_O` intake and inactive rate by year

The chart plots annual student count (bars) and the proportion carrying Inactive status (line).

**Result: the plotted values are accurate and identical to the AdminPortal calculation.**
The bars equal the module population (Active + Inactive from slide 7) and the line equals
Inactive ÷ total for that population.

| Year    | Students (slide = DB) | Inactive rate (slide = DB) |
|---------|-----------------------|----------------------------|
| 2018/19 | 19 | 21.1% |
| 2019/20 | 23 | 21.7% |
| 2020/21 | 38 | 21.1% |
| 2021/22 | 27 | 11.1% |
| 2022/23 | 22 | 13.6% |
| 2023/24 | 29 | 13.8% |
| 2024/25 | 26 | 23.1% |
| 2025/26 | 25 | 12.0% |

---

## Reconciliation with AdminPortal

AdminPortal's dropout metric is defined (`programme_enrollments` entity):

- The query spine is the registration table `SFRSTCR`.
- Student status is resolved point-in-time: for each registration term, the student's most
  recent `SGBSTDN` record with `SGBSTDN_TERM_CODE_EFF <= term` is used.
- `dropout_count` = distinct students registered that term whose status is `'IS'`.
- `dropout_rate` = `dropout_count` ÷ distinct registered students.
- "Study year" is derived from the module code: `SSBSECT_CRSE_NUMB ~ '^[A-Za-z][0-9]'` takes
  the second character, so `H3010` resolves to **study year 3**.

Three views were computed against the database and compared:

| View | Definition | Result |
|------|------------|--------|
| A | AdminPortal `programme_enrollments` logic, `DL_SCYPY_O`, all study years | Matches slide |
| B | Same, restricted to study year 3 | Matches slide |
| C | `PSYL_H3010` module population only (the slide's basis) | Matches slide |

**All three views are identical.** For `DL_SCYPY_O`, the programme population, the Stage 3
population, and the `PSYL_H3010` population are effectively the same cohort, so the slide and
the application produce the same figures.

### Module is the correct Stage 3 lens

`PSYL_H3010` is the programme's core Stage 3 module: **311 students / 331
registrations** pass through it, far exceeding any other module taken by these students (all
others are 1–4 students — electives, transfers, and stray repeats). The CRN structure is as
expected: **CRN 12939 is the core CRN** (38 / 29 / 25 students in 2020 / 2023 / 2025), with
the additional CRNs (12940, 12941) holding 0–1 repeat students. Including all CRNs versus the
core CRN alone makes no material difference.

---

## Methodological caveat

The measure *"of students **registered** for the module in a given
year, what proportion carry Inactive status."* This captures in-cohort disengagement, **not**
true programme attrition. Because a student who fully drops out stops registering, they leave
the denominator entirely and become invisible to the metric.

This gap is material in **2023/24**. The registration-based view (slide and AdminPortal) shows
**4 inactive of 29 (13.8%)**. Querying `SGBSTDN` programme membership directly, however, shows
**22 inactive of 47 (46.8%)** for `DL_SCYPY_O` that year — i.e. 18 additional students were
flagged Inactive but were no longer registered for the module and are therefore not counted.

This is not an error in the slide: the slide is internally consistent with the application and
with the database. It is a structural property of a registration-based inactive rate.

---
