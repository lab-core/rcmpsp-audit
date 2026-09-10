# rcmpsp-audit
Instances for Multi-Project Scheduling with resources availability.

# Multi-Project Scheduling Instances with Employee Overload and Assignment Continuity

Benchmark instances accompanying the paper:

> C. Pinçon, N. Lahrichi and A. Legrain. *A Fix-and-Optimize Approach for Multi-Project
> Scheduling with Employee Overload and Assignment Continuity.*

This repository contains the **97 instances** used in the computational experiments of the
paper. They are derived from annual planning data provided by a software company for **five
audit firms**, anonymized as firms `a` to `e`. The largest instances contain thousands of
projects, tens of thousands of activities and hundreds of employees.

## The scheduling problem in one paragraph

A portfolio of client projects must be scheduled over a planning horizon. Each project consists
of activities with a release date, a due date and a processing time. An activity must be
assigned to a single qualified employee, but may be **preempted**, i.e. split over several days.
Employees differ in qualifications and daily availability. Employee capacity is a **soft**
constraint: work assigned beyond an employee's available capacity counts as *overload*. Each
activity also has a **preferred employee**, and reassigning it elsewhere costs *continuity*.
The objective combines total overload and maximum weekly overload, subject to a budget on how
many activities may be reassigned away from their preferred employee. See the paper for the
full mixed-integer formulation.

## Repository layout

```
instances/     97 instance files in JSON format (see "Instance format" below)
README.md      this file
```

## Instance naming

Each file is named after its instance code, e.g. `a1.json`, `a1'.json`, `c14.json`.
The code identifies the firm and the parameters used to generate the instance.

| Code prefix | Firm | #Instances | #Projects | #Activities | #Employees |
|---|---|--:|--:|--:|--:|
| `a` | a | 36 | 1,112 | 5,748 | 86 |
| `b` | b | 33 | 362 | 14,640 | 412 |
| `c` | c | 14 | 3,328 | 54,499 | 378 |
| `d` | d | 10 | 2,718 | 48,331 | 549 |
| `e` | e | 4 | 3,936 | 68,842 | 525 |

Each firm provides one realization of the scheduling problem. Additional instances are obtained
by varying three parameters, plus a load perturbation:

- **Time flexibility `[w⁻, w⁺]`** — an activity may be scheduled up to `w⁻` days before and
  `w⁺` days after its due date. Four combinations are used: `[0, 7]`, `[7, 0]`, `[7, 7]`
  and `[15, 15]`.
- **Resource assignment `ψ`** — defines which employees are eligible for each activity.
  `ψ = 1` uses the original team definition provided by the industrial partner; `ψ = 2–5` use
  alternative team definitions that expand or reduce the set of eligible employees. These
  definitions vary by firm and were provided by the company.
- **Precedence density `π ∈ {50, 100} %`** — the share of the original precedence constraints
  kept in the instance.
- **Load perturbation (prime `'`)** — a code ending with an apostrophe, e.g. `a1'`, is the
  *perturbed twin* of the corresponding base instance `a1`. It is built by doubling the workload
  of activities in another period of the year, creating an additional period of high demand
  later in the horizon. These instances are used when solving over the full-year horizon.
  Base and perturbed twins share the same projects, activities, employees and precedence
  structure; only the processing times differ.

Not all parameter combinations are relevant for every firm. This gives **64 base instances**
and **33 perturbed instances** (18 for firm `a` and 15 for firm `b`), for a total of **97**.
The exact parameter values of every instance are listed in the tables below.

## Instance format

Each instance is a single JSON object with four top-level keys. `Projects` and `Activites` use
a **column-oriented** layout — the field names are given once in `columns`, and `rows` holds
one array of values per record, in that same order. 

```json
{
  "Projects":  { "columns": ["id", "start", "end"],
                 "rows":    [[1, 1, 37], [2, 1, 37]] },
  "Activites": { "columns": ["id", "project_id", "duration", "release", "due", "owner",
                             "activite_start", "activite_end", "resource_pref",
                             "precedences", "resource_pool"],
                 "rows":    [[3, 1, 4, 1, 37, 186, 5, 12, 186, [1], 1]] },
  "Dispo":          { "1": { "0": 480, "1": 480, "2": 480, "5": 480 } },
  "resource_pools": { "1": [156, 184, 158, 174, 190, 186, 192, 340] }
}
```

### `Projects` — one row per project

| Field | Meaning |
|---|---|
| `id` | Project identifier, numbered `1..#Projects`. |
| `start` | Release date of the project, as a day index. |
| `end` | Due date of the project, as a day index. |

### `Activites` — one row per activity

| Field | Meaning |
|---|---|
| `id` | Activity identifier, numbered `1..#Activities`. |
| `project_id` | The project this activity belongs to. |
| `duration` | Processing time, **in minutes**. |
| `release` | Release date of the activity, as a day index. |
| `due` | Due date of the activity, as a day index. |
| `activite_start` | **Earliest** possible start day of the activity. |
| `activite_end` | **Latest** possible start day of the activity. |
| `resource_pref` | Preferred employee of the activity. |
| `precedences` | List of activity `id`s that must be completed before this one may start. |
| `resource_pool` | Key into `resource_pools`, giving the employees eligible for this activity. |

The scheduling time window of an activity is `[activite_start, activite_end]`; the *time window
duration* reported in the appendix table is `activite_end − activite_start`. This window is
always contained in `[release, due]`. Precedence references are always activities of the same
project, and the preferred employee always belongs to the activity's own eligible pool.

### `Dispo` — employee availability

A dictionary mapping an employee id to a dictionary mapping a day index to the employee's
available capacity on that day, **in minutes**. Only days with non-zero availability are
stored: **a day absent from an employee's dictionary means zero availability**. Values are
usually integers (`480` = 8 h) but may be fractional, since only activity durations were
rounded to the minute.

### `resource_pools` — sets of eligible employees

A dictionary mapping a pool id to the list of employee ids in that pool. Eligibility sets are
shared by many activities, so they are stored once here and referenced by `resource_pool`
rather than repeated on every activity.

### Conventions

- **Days** are integer indices. Day `0` is the earliest date appearing anywhere in the
  instance, and every other date is its offset in days.
- **Durations and availabilities** are expressed in minutes. 
- **Identifiers** (projects, activities, employees) are renumbered from 1 and are local to each
  instance: the same id in two different instances is not the same entity.
- Employees appearing in `Dispo` are all employees of the firm. Some of them are not eligible
  for any activity of a given instance, so they do not appear in any pool.

## Correspondence with the notation of the paper

| Paper | JSON |
|---|---|
| `P`, project `p` | `Projects` |
| `I`, activity `i` | `Activites` |
| `I_p` | activities with `project_id == p` |
| `p_i` — processing time | `duration` (minutes) |
| `J_i = [r_i, d_i]` — time window | `[activite_start, activite_end]` |
| `P_i^-` — immediate predecessors | `precedences` |
| `R_i` — eligible employees | `resource_pools[resource_pool]` |
| `γ_i` — preferred employee | `resource_pref` |
| `c_rj` — capacity of employee `r` on day `j` | `Dispo[r][j]` (minutes, `0` if absent) |

## Descriptive tables

The two tables below are the appendix tables of the paper. Their LaTeX source is in `tables/`.

<details>
<summary><b>Table 1 — Characteristics of the 97 instances</b> (click to expand)</summary>

Characteristics of the 97 instances. `[w⁻, w⁺]`: allowed shift (days) before/after the due date; `π`: share of precedence constraints kept; `ψ`: resource-assignment rule; `p̄ᵢ`: average processing time; `D`: total demand; `A`: available capacity.
| Code | ψ | [w⁻, w⁺] | π (%) | #Proj. | #Act. | #Res. | p̄ᵢ (h) | #Pred. avg | #Pred. max | D (h) | A (h) | D/A |
|---|--:|--:|--:|--:|--:|--:|--:|--:|--:|--:|--:|--:|
| a1 | 1 | [15, 15] | 100 | 1,112 | 5,748 | 86 | 5.23 | 2.66 | 32 | 30,068 | 69,511 | 0.43 |
| a1&#39; | 1 | [15, 15] | 100 | 1,112 | 5,748 | 86 | 5.94 | 2.66 | 32 | 34,138 | 69,511 | 0.49 |
| a2 | 1 | [15, 15] | 50 | 1,112 | 5,748 | 86 | 5.23 | 1.61 | 16 | 30,068 | 69,511 | 0.43 |
| a2&#39; | 1 | [15, 15] | 50 | 1,112 | 5,748 | 86 | 5.94 | 1.61 | 16 | 34,138 | 69,511 | 0.49 |
| a3 | 2 | [15, 15] | 100 | 1,112 | 5,748 | 86 | 5.23 | 2.66 | 32 | 30,068 | 67,922 | 0.44 |
| a3&#39; | 2 | [15, 15] | 100 | 1,112 | 5,748 | 86 | 5.94 | 2.66 | 32 | 34,138 | 67,607 | 0.50 |
| a4 | 2 | [15, 15] | 50 | 1,112 | 5,748 | 86 | 5.23 | 1.61 | 16 | 30,068 | 67,348 | 0.45 |
| a4&#39; | 2 | [15, 15] | 50 | 1,112 | 5,748 | 86 | 5.94 | 1.61 | 16 | 34,138 | 67,481 | 0.51 |
| a5 | 5 | [15, 15] | 100 | 1,112 | 5,748 | 86 | 5.23 | 2.66 | 32 | 30,068 | 149,416 | 0.20 |
| a5&#39; | 5 | [15, 15] | 100 | 1,112 | 5,748 | 86 | 5.94 | 2.66 | 32 | 34,138 | 149,129 | 0.23 |
| a6 | 5 | [15, 15] | 50 | 1,112 | 5,748 | 86 | 5.23 | 1.61 | 16 | 30,068 | 149,136 | 0.20 |
| a6&#39; | 5 | [15, 15] | 50 | 1,112 | 5,748 | 86 | 5.94 | 1.61 | 16 | 34,138 | 148,891 | 0.23 |
| a7 | 1 | [7, 0] | 100 | 1,112 | 5,748 | 86 | 5.23 | 2.66 | 32 | 30,068 | 59,886 | 0.50 |
| a7&#39; | 1 | [7, 0] | 100 | 1,112 | 5,748 | 86 | 5.94 | 2.66 | 32 | 34,138 | 59,886 | 0.57 |
| a8 | 1 | [7, 0] | 50 | 1,112 | 5,748 | 86 | 5.23 | 1.61 | 16 | 30,068 | 59,886 | 0.50 |
| a8&#39; | 1 | [7, 0] | 50 | 1,112 | 5,748 | 86 | 5.94 | 1.61 | 16 | 34,138 | 59,886 | 0.57 |
| a9 | 2 | [7, 0] | 100 | 1,112 | 5,748 | 86 | 5.23 | 2.66 | 32 | 30,068 | 58,920 | 0.51 |
| a9&#39; | 2 | [7, 0] | 100 | 1,112 | 5,748 | 86 | 5.94 | 2.66 | 32 | 34,138 | 59,060 | 0.58 |
| a10 | 2 | [7, 0] | 50 | 1,112 | 5,748 | 86 | 5.23 | 1.61 | 16 | 30,068 | 58,941 | 0.51 |
| a10&#39; | 2 | [7, 0] | 50 | 1,112 | 5,748 | 86 | 5.94 | 1.61 | 16 | 34,138 | 58,948 | 0.58 |
| a11 | 5 | [7, 0] | 100 | 1,112 | 5,748 | 86 | 5.23 | 2.66 | 32 | 30,068 | 130,964 | 0.23 |
| a11&#39; | 5 | [7, 0] | 100 | 1,112 | 5,748 | 86 | 5.94 | 2.66 | 32 | 34,138 | 131,139 | 0.26 |
| a12 | 5 | [7, 0] | 50 | 1,112 | 5,748 | 86 | 5.23 | 1.61 | 16 | 30,068 | 131,013 | 0.23 |
| a12&#39; | 5 | [7, 0] | 50 | 1,112 | 5,748 | 86 | 5.94 | 1.61 | 16 | 34,138 | 131,069 | 0.26 |
| a13 | 1 | [7, 7] | 100 | 1,112 | 5,748 | 86 | 5.23 | 2.66 | 32 | 30,068 | 63,526 | 0.47 |
| a13&#39; | 1 | [7, 7] | 100 | 1,112 | 5,748 | 86 | 5.94 | 2.66 | 32 | 34,138 | 63,526 | 0.54 |
| a14 | 1 | [7, 7] | 50 | 1,112 | 5,748 | 86 | 5.23 | 1.61 | 16 | 30,068 | 63,526 | 0.47 |
| a14&#39; | 1 | [7, 7] | 50 | 1,112 | 5,748 | 86 | 5.94 | 1.61 | 16 | 34,138 | 63,526 | 0.54 |
| a15 | 2 | [7, 7] | 100 | 1,112 | 5,748 | 86 | 5.23 | 2.66 | 32 | 30,068 | 62,392 | 0.48 |
| a15&#39; | 2 | [7, 7] | 100 | 1,112 | 5,748 | 86 | 5.94 | 2.66 | 32 | 34,138 | 62,147 | 0.55 |
| a16 | 2 | [7, 7] | 50 | 1,112 | 5,748 | 86 | 5.23 | 1.61 | 16 | 30,068 | 62,413 | 0.48 |
| a16&#39; | 2 | [7, 7] | 50 | 1,112 | 5,748 | 86 | 5.94 | 1.61 | 16 | 34,138 | 62,420 | 0.55 |
| a17 | 5 | [7, 7] | 100 | 1,112 | 5,748 | 86 | 5.23 | 2.66 | 32 | 30,068 | 137,936 | 0.22 |
| a17&#39; | 5 | [7, 7] | 100 | 1,112 | 5,748 | 86 | 5.94 | 2.66 | 32 | 34,138 | 137,880 | 0.25 |
| a18 | 5 | [7, 7] | 50 | 1,112 | 5,748 | 86 | 5.23 | 1.61 | 16 | 30,068 | 137,558 | 0.22 |
| a18&#39; | 5 | [7, 7] | 50 | 1,112 | 5,748 | 86 | 5.94 | 1.61 | 16 | 34,138 | 137,831 | 0.25 |
| b1 | 1 | [0, 7] | 100 | 362 | 14,640 | 412 | 7.63 | 35.18 | 406 | 111,733 | 411,536 | 0.27 |
| b1&#39; | 1 | [0, 7] | 100 | 362 | 14,640 | 412 | 9.88 | 35.18 | 406 | 144,634 | 411,536 | 0.35 |
| b2 | 1 | [0, 7] | 50 | 362 | 14,640 | 412 | 7.63 | 17.84 | 203 | 111,733 | 411,536 | 0.27 |
| b2&#39; | 1 | [0, 7] | 50 | 362 | 14,640 | 412 | 9.88 | 17.84 | 203 | 144,634 | 411,536 | 0.35 |
| b3 | 2 | [0, 7] | 100 | 362 | 14,640 | 412 | 7.63 | 35.18 | 406 | 111,733 | 392,688 | 0.28 |
| b3&#39; | 2 | [0, 7] | 100 | 362 | 14,640 | 412 | 9.88 | 35.18 | 406 | 144,634 | 392,888 | 0.37 |
| b4 | 2 | [0, 7] | 50 | 362 | 14,640 | 412 | 7.63 | 17.84 | 203 | 111,733 | 392,248 | 0.28 |
| b4&#39; | 2 | [0, 7] | 50 | 362 | 14,640 | 412 | 9.88 | 17.84 | 203 | 144,634 | 392,624 | 0.37 |
| b5 | 5 | [0, 7] | 100 | 362 | 14,640 | 412 | 7.63 | 35.18 | 406 | 111,733 | 941,200 | 0.12 |
| b5&#39; | 5 | [0, 7] | 100 | 362 | 14,640 | 412 | 9.88 | 35.18 | 406 | 144,634 | 940,728 | 0.15 |
| b6 | 5 | [0, 7] | 50 | 362 | 14,640 | 412 | 7.63 | 17.84 | 203 | 111,733 | 941,840 | 0.12 |
| b6&#39; | 5 | [0, 7] | 50 | 362 | 14,640 | 412 | 9.88 | 17.84 | 203 | 144,634 | 940,512 | 0.15 |
| b7 | 1 | [15, 15] | 100 | 362 | 14,640 | 412 | 7.63 | 35.18 | 406 | 111,733 | 628,696 | 0.18 |
| b7&#39; | 1 | [15, 15] | 100 | 362 | 14,640 | 412 | 9.88 | 35.18 | 406 | 144,634 | 628,696 | 0.23 |
| b8 | 1 | [15, 15] | 50 | 362 | 14,640 | 412 | 7.63 | 17.84 | 203 | 111,733 | 628,696 | 0.18 |
| b9 | 2 | [15, 15] | 100 | 362 | 14,640 | 412 | 7.63 | 35.18 | 406 | 111,733 | 617,376 | 0.18 |
| b9&#39; | 2 | [15, 15] | 100 | 362 | 14,640 | 412 | 9.88 | 35.18 | 406 | 144,634 | 618,200 | 0.23 |
| b10 | 2 | [15, 15] | 50 | 362 | 14,640 | 412 | 7.63 | 17.84 | 203 | 111,733 | 617,600 | 0.18 |
| b11 | 5 | [15, 15] | 100 | 362 | 14,640 | 412 | 7.63 | 35.18 | 406 | 111,733 | 1,071,608 | 0.10 |
| b11&#39; | 5 | [15, 15] | 100 | 362 | 14,640 | 412 | 9.88 | 35.18 | 406 | 144,634 | 1,071,584 | 0.13 |
| b12 | 5 | [15, 15] | 50 | 362 | 14,640 | 412 | 7.63 | 17.84 | 203 | 111,733 | 1,071,448 | 0.10 |
| b13 | 1 | [7, 7] | 100 | 362 | 14,640 | 412 | 7.63 | 35.18 | 406 | 111,733 | 488,336 | 0.23 |
| b13&#39; | 1 | [7, 7] | 100 | 362 | 14,640 | 412 | 9.88 | 35.18 | 406 | 144,634 | 488,336 | 0.30 |
| b14 | 1 | [7, 7] | 50 | 362 | 14,640 | 412 | 7.63 | 17.84 | 203 | 111,733 | 488,336 | 0.23 |
| b14&#39; | 1 | [7, 7] | 50 | 362 | 14,640 | 412 | 9.88 | 17.84 | 203 | 144,634 | 488,336 | 0.30 |
| b15 | 2 | [7, 7] | 100 | 362 | 14,640 | 412 | 7.63 | 35.18 | 406 | 111,733 | 476,248 | 0.23 |
| b15&#39; | 2 | [7, 7] | 100 | 362 | 14,640 | 412 | 9.88 | 35.18 | 406 | 144,634 | 476,280 | 0.30 |
| b16 | 2 | [7, 7] | 50 | 362 | 14,640 | 412 | 7.63 | 17.84 | 203 | 111,733 | 475,896 | 0.23 |
| b16&#39; | 2 | [7, 7] | 50 | 362 | 14,640 | 412 | 9.88 | 17.84 | 203 | 144,634 | 476,040 | 0.30 |
| b17 | 5 | [7, 7] | 100 | 362 | 14,640 | 412 | 7.63 | 35.18 | 406 | 111,733 | 1,000,208 | 0.11 |
| b17&#39; | 5 | [7, 7] | 100 | 362 | 14,640 | 412 | 9.88 | 35.18 | 406 | 144,634 | 997,776 | 0.14 |
| b18 | 5 | [7, 7] | 50 | 362 | 14,640 | 412 | 7.63 | 17.84 | 203 | 111,733 | 997,512 | 0.11 |
| b18&#39; | 5 | [7, 7] | 50 | 362 | 14,640 | 412 | 9.88 | 17.84 | 203 | 144,634 | 998,744 | 0.14 |
| c1 | 1 | [0, 7] | 100 | 3,328 | 54,499 | 378 | 5.01 | 7.71 | 157 | 272,929 | 763,637 | 0.36 |
| c2 | 1 | [0, 7] | 50 | 3,328 | 54,499 | 378 | 5.01 | 4.15 | 79 | 272,929 | 763,637 | 0.36 |
| c3 | 2 | [0, 7] | 100 | 3,328 | 54,499 | 378 | 5.01 | 7.71 | 157 | 272,929 | 750,994 | 0.36 |
| c4 | 2 | [0, 7] | 50 | 3,328 | 54,499 | 378 | 5.01 | 4.15 | 79 | 272,929 | 750,944 | 0.36 |
| c5 | 5 | [0, 7] | 100 | 3,328 | 54,499 | 378 | 5.01 | 7.71 | 157 | 272,929 | 1,091,754 | 0.25 |
| c6 | 5 | [0, 7] | 50 | 3,328 | 54,499 | 378 | 5.01 | 4.15 | 79 | 272,929 | 1,091,396 | 0.25 |
| c7 | 2 | [15, 15] | 100 | 3,328 | 54,499 | 378 | 5.01 | 7.71 | 157 | 272,929 | 827,559 | 0.33 |
| c8 | 2 | [15, 15] | 50 | 3,328 | 54,499 | 378 | 5.01 | 4.15 | 79 | 272,929 | 828,751 | 0.33 |
| c9 | 1 | [7, 7] | 100 | 3,328 | 54,499 | 378 | 5.01 | 7.71 | 157 | 272,929 | 789,663 | 0.35 |
| c10 | 1 | [7, 7] | 50 | 3,328 | 54,499 | 378 | 5.01 | 4.15 | 79 | 272,929 | 789,663 | 0.35 |
| c11 | 2 | [7, 7] | 100 | 3,328 | 54,499 | 378 | 5.01 | 7.71 | 157 | 272,929 | 780,482 | 0.35 |
| c12 | 2 | [7, 7] | 50 | 3,328 | 54,499 | 378 | 5.01 | 4.15 | 79 | 272,929 | 781,261 | 0.35 |
| c13 | 5 | [7, 7] | 100 | 3,328 | 54,499 | 378 | 5.01 | 7.71 | 157 | 272,929 | 1,128,109 | 0.24 |
| c14 | 5 | [7, 7] | 50 | 3,328 | 54,499 | 378 | 5.01 | 4.15 | 79 | 272,929 | 1,128,212 | 0.24 |
| d1 | 3 | [0, 7] | 100 | 2,718 | 48,169 | 517 | 5.08 | 2.33 | 71 | 244,605 | 630,232 | 0.39 |
| d2 | 3 | [0, 7] | 50 | 2,718 | 48,331 | 549 | 5.09 | 1.49 | 36 | 245,812 | 630,307 | 0.39 |
| d3 | 4 | [0, 7] | 50 | 2,718 | 48,331 | 549 | 5.09 | 1.49 | 36 | 245,812 | 694,681 | 0.35 |
| d4 | 3 | [15, 15] | 100 | 2,718 | 48,169 | 517 | 5.08 | 2.33 | 71 | 244,605 | 654,160 | 0.37 |
| d5 | 3 | [15, 15] | 50 | 2,718 | 48,331 | 549 | 5.09 | 1.49 | 36 | 245,812 | 654,160 | 0.38 |
| d6 | 4 | [7, 0] | 100 | 2,718 | 48,169 | 517 | 5.08 | 2.33 | 71 | 244,605 | 693,046 | 0.35 |
| d7 | 3 | [7, 7] | 100 | 2,718 | 48,169 | 517 | 5.08 | 2.33 | 71 | 244,605 | 641,332 | 0.38 |
| d8 | 3 | [7, 7] | 50 | 2,718 | 48,331 | 549 | 5.09 | 1.49 | 36 | 245,812 | 641,347 | 0.38 |
| d9 | 4 | [7, 7] | 100 | 2,718 | 48,169 | 517 | 5.08 | 2.33 | 71 | 244,605 | 700,723 | 0.35 |
| d10 | 4 | [7, 7] | 50 | 2,718 | 48,331 | 549 | 5.09 | 1.49 | 36 | 245,812 | 700,723 | 0.35 |
| e1 | 1 | [0, 7] | 50 | 3,936 | 68,842 | 525 | 5.01 | 3.37 | 68 | 345,221 | 487,190 | 0.71 |
| e2 | 1 | [7, 0] | 100 | 3,936 | 68,842 | 525 | 5.01 | 6.07 | 135 | 345,221 | 478,238 | 0.72 |
| e3 | 1 | [7, 7] | 100 | 3,936 | 68,842 | 525 | 5.01 | 6.07 | 135 | 345,221 | 532,033 | 0.65 |
| e4 | 1 | [7, 7] | 50 | 3,936 | 68,842 | 525 | 5.01 | 3.37 | 68 | 345,221 | 532,033 | 0.65 |

</details>

<details>
<summary><b>Table 2 — Activity-level statistics</b> (click to expand)</summary>

Activity-level statistics of the 97 instances: number of predecessors, number of eligible employees, and time-window duration in days (`activite_end − activite_start`), aggregated over the activities of each instance.
| Code | #Pred. min | #Pred. avg | #Pred. max | #Elig. min | #Elig. avg | #Elig. max | Window min | Window avg | Window max |
|---|--:|--:|--:|--:|--:|--:|--:|--:|--:|
| a1 | 0 | 2.66 | 32 | 1 | 13.97 | 24 | 15 | 29.8 | 128 |
| a1&#39; | 0 | 2.66 | 32 | 1 | 13.97 | 24 | 15 | 29.8 | 128 |
| a2 | 0 | 1.61 | 16 | 1 | 13.97 | 24 | 15 | 29.8 | 128 |
| a2&#39; | 0 | 1.61 | 16 | 1 | 13.97 | 24 | 15 | 29.8 | 128 |
| a3 | 0 | 2.66 | 32 | 1 | 7.62 | 13 | 15 | 29.8 | 128 |
| a3&#39; | 0 | 2.66 | 32 | 1 | 7.64 | 13 | 15 | 29.8 | 128 |
| a4 | 0 | 1.61 | 16 | 1 | 7.63 | 13 | 15 | 29.8 | 128 |
| a4&#39; | 0 | 1.61 | 16 | 1 | 7.62 | 13 | 15 | 29.8 | 128 |
| a5 | 0 | 2.66 | 32 | 2 | 27.94 | 48 | 15 | 29.8 | 128 |
| a5&#39; | 0 | 2.66 | 32 | 2 | 27.94 | 48 | 15 | 29.8 | 128 |
| a6 | 0 | 1.61 | 16 | 2 | 27.94 | 48 | 15 | 29.8 | 128 |
| a6&#39; | 0 | 1.61 | 16 | 2 | 27.94 | 48 | 15 | 29.8 | 128 |
| a7 | 0 | 2.66 | 32 | 1 | 13.97 | 24 | 0 | 7.1 | 105 |
| a7&#39; | 0 | 2.66 | 32 | 1 | 13.97 | 24 | 0 | 7.1 | 105 |
| a8 | 0 | 1.61 | 16 | 1 | 13.97 | 24 | 0 | 7.1 | 105 |
| a8&#39; | 0 | 1.61 | 16 | 1 | 13.97 | 24 | 0 | 7.1 | 105 |
| a9 | 0 | 2.66 | 32 | 1 | 7.63 | 13 | 0 | 7.1 | 105 |
| a9&#39; | 0 | 2.66 | 32 | 1 | 7.63 | 13 | 0 | 7.1 | 105 |
| a10 | 0 | 1.61 | 16 | 1 | 7.63 | 13 | 0 | 7.1 | 105 |
| a10&#39; | 0 | 1.61 | 16 | 1 | 7.63 | 13 | 0 | 7.1 | 105 |
| a11 | 0 | 2.66 | 32 | 2 | 27.94 | 48 | 0 | 7.1 | 105 |
| a11&#39; | 0 | 2.66 | 32 | 2 | 27.94 | 48 | 0 | 7.1 | 105 |
| a12 | 0 | 1.61 | 16 | 2 | 27.94 | 48 | 0 | 7.1 | 105 |
| a12&#39; | 0 | 1.61 | 16 | 2 | 27.94 | 48 | 0 | 7.1 | 105 |
| a13 | 0 | 2.66 | 32 | 1 | 13.97 | 24 | 7 | 14.1 | 112 |
| a13&#39; | 0 | 2.66 | 32 | 1 | 13.97 | 24 | 7 | 14.1 | 112 |
| a14 | 0 | 1.61 | 16 | 1 | 13.97 | 24 | 7 | 14.1 | 112 |
| a14&#39; | 0 | 1.61 | 16 | 1 | 13.97 | 24 | 7 | 14.1 | 112 |
| a15 | 0 | 2.66 | 32 | 1 | 7.63 | 13 | 7 | 14.1 | 112 |
| a15&#39; | 0 | 2.66 | 32 | 1 | 7.63 | 13 | 7 | 14.1 | 112 |
| a16 | 0 | 1.61 | 16 | 1 | 7.64 | 13 | 7 | 14.1 | 112 |
| a16&#39; | 0 | 1.61 | 16 | 1 | 7.63 | 13 | 7 | 14.1 | 112 |
| a17 | 0 | 2.66 | 32 | 2 | 27.94 | 48 | 7 | 14.1 | 112 |
| a17&#39; | 0 | 2.66 | 32 | 2 | 27.94 | 48 | 7 | 14.1 | 112 |
| a18 | 0 | 1.61 | 16 | 2 | 27.94 | 48 | 7 | 14.1 | 112 |
| a18&#39; | 0 | 1.61 | 16 | 2 | 27.94 | 48 | 7 | 14.1 | 112 |
| b1 | 0 | 35.18 | 406 | 1 | 16.52 | 43 | 6 | 7.0 | 8 |
| b1&#39; | 0 | 35.18 | 406 | 1 | 16.52 | 43 | 6 | 7.0 | 8 |
| b2 | 0 | 17.84 | 203 | 1 | 16.52 | 43 | 6 | 7.0 | 8 |
| b2&#39; | 0 | 17.84 | 203 | 1 | 16.52 | 43 | 6 | 7.0 | 8 |
| b3 | 0 | 35.18 | 406 | 1 | 9.04 | 23 | 6 | 7.0 | 8 |
| b3&#39; | 0 | 35.18 | 406 | 1 | 9.04 | 23 | 6 | 7.0 | 8 |
| b4 | 0 | 17.84 | 203 | 1 | 9.04 | 23 | 6 | 7.0 | 8 |
| b4&#39; | 0 | 17.84 | 203 | 1 | 9.04 | 23 | 6 | 7.0 | 8 |
| b5 | 0 | 35.18 | 406 | 2 | 33.03 | 86 | 6 | 7.0 | 8 |
| b5&#39; | 0 | 35.18 | 406 | 2 | 33.03 | 86 | 6 | 7.0 | 8 |
| b6 | 0 | 17.84 | 203 | 2 | 33.03 | 86 | 6 | 7.0 | 8 |
| b6&#39; | 0 | 17.84 | 203 | 2 | 33.03 | 86 | 6 | 7.0 | 8 |
| b7 | 0 | 35.18 | 406 | 1 | 16.52 | 43 | 21 | 30.0 | 31 |
| b7&#39; | 0 | 35.18 | 406 | 1 | 16.52 | 43 | 21 | 30.0 | 31 |
| b8 | 0 | 17.84 | 203 | 1 | 16.52 | 43 | 21 | 30.0 | 31 |
| b9 | 0 | 35.18 | 406 | 1 | 9.04 | 23 | 21 | 30.0 | 31 |
| b9&#39; | 0 | 35.18 | 406 | 1 | 9.04 | 23 | 21 | 30.0 | 31 |
| b10 | 0 | 17.84 | 203 | 1 | 9.05 | 23 | 21 | 30.0 | 31 |
| b11 | 0 | 35.18 | 406 | 2 | 33.03 | 86 | 21 | 30.0 | 31 |
| b11&#39; | 0 | 35.18 | 406 | 2 | 33.03 | 86 | 21 | 30.0 | 31 |
| b12 | 0 | 17.84 | 203 | 2 | 33.03 | 86 | 21 | 30.0 | 31 |
| b13 | 0 | 35.18 | 406 | 1 | 16.52 | 43 | 13 | 14.0 | 15 |
| b13&#39; | 0 | 35.18 | 406 | 1 | 16.52 | 43 | 13 | 14.0 | 15 |
| b14 | 0 | 17.84 | 203 | 1 | 16.52 | 43 | 13 | 14.0 | 15 |
| b14&#39; | 0 | 17.84 | 203 | 1 | 16.52 | 43 | 13 | 14.0 | 15 |
| b15 | 0 | 35.18 | 406 | 1 | 9.04 | 23 | 13 | 14.0 | 15 |
| b15&#39; | 0 | 35.18 | 406 | 1 | 9.04 | 23 | 13 | 14.0 | 15 |
| b16 | 0 | 17.84 | 203 | 1 | 9.04 | 23 | 13 | 14.0 | 15 |
| b16&#39; | 0 | 17.84 | 203 | 1 | 9.05 | 23 | 13 | 14.0 | 15 |
| b17 | 0 | 35.18 | 406 | 2 | 33.03 | 86 | 13 | 14.0 | 15 |
| b17&#39; | 0 | 35.18 | 406 | 2 | 33.03 | 86 | 13 | 14.0 | 15 |
| b18 | 0 | 17.84 | 203 | 2 | 33.03 | 86 | 13 | 14.0 | 15 |
| b18&#39; | 0 | 17.84 | 203 | 2 | 33.03 | 86 | 13 | 14.0 | 15 |
| c1 | 0 | 7.71 | 157 | 1 | 24.27 | 44 | 7 | 7.0 | 18 |
| c2 | 0 | 4.15 | 79 | 1 | 24.27 | 44 | 7 | 7.0 | 18 |
| c3 | 0 | 7.71 | 157 | 1 | 12.77 | 23 | 7 | 7.0 | 18 |
| c4 | 0 | 4.15 | 79 | 1 | 12.77 | 23 | 7 | 7.0 | 18 |
| c5 | 0 | 7.71 | 157 | 2 | 48.55 | 88 | 7 | 7.0 | 18 |
| c6 | 0 | 4.15 | 79 | 2 | 48.55 | 88 | 7 | 7.0 | 18 |
| c7 | 0 | 7.71 | 157 | 1 | 12.78 | 23 | 15 | 29.6 | 41 |
| c8 | 0 | 4.15 | 79 | 1 | 12.78 | 23 | 15 | 29.6 | 41 |
| c9 | 0 | 7.71 | 157 | 1 | 24.27 | 44 | 7 | 13.9 | 25 |
| c10 | 0 | 4.15 | 79 | 1 | 24.27 | 44 | 7 | 13.9 | 25 |
| c11 | 0 | 7.71 | 157 | 1 | 12.78 | 23 | 7 | 13.9 | 25 |
| c12 | 0 | 4.15 | 79 | 1 | 12.78 | 23 | 7 | 13.9 | 25 |
| c13 | 0 | 7.71 | 157 | 2 | 48.55 | 88 | 7 | 13.9 | 25 |
| c14 | 0 | 4.15 | 79 | 2 | 48.55 | 88 | 7 | 13.9 | 25 |
| d1 | 0 | 2.33 | 71 | 1 | 11.50 | 21 | 7 | 7.0 | 37 |
| d2 | 0 | 1.49 | 36 | 1 | 11.48 | 21 | 7 | 7.0 | 37 |
| d3 | 0 | 1.49 | 36 | 1 | 36.60 | 75 | 7 | 7.0 | 37 |
| d4 | 0 | 2.33 | 71 | 1 | 11.50 | 21 | 15 | 29.5 | 60 |
| d5 | 0 | 1.49 | 36 | 1 | 11.48 | 21 | 15 | 29.5 | 60 |
| d6 | 0 | 2.33 | 71 | 1 | 36.63 | 75 | 0 | 6.9 | 37 |
| d7 | 0 | 2.33 | 71 | 1 | 11.50 | 21 | 7 | 13.9 | 44 |
| d8 | 0 | 1.49 | 36 | 1 | 11.48 | 21 | 7 | 13.9 | 44 |
| d9 | 0 | 2.33 | 71 | 1 | 36.63 | 75 | 7 | 13.9 | 44 |
| d10 | 0 | 1.49 | 36 | 1 | 36.60 | 75 | 7 | 13.9 | 44 |
| e1 | 0 | 3.37 | 68 | 1 | 1.00 | 1 | 1 | 7.0 | 30 |
| e2 | 0 | 6.07 | 135 | 1 | 1.00 | 1 | 0 | 7.0 | 30 |
| e3 | 0 | 6.07 | 135 | 1 | 1.00 | 1 | 7 | 14.0 | 37 |
| e4 | 0 | 3.37 | 68 | 1 | 1.00 | 1 | 7 | 14.0 | 37 |

</details>

