# Lift Log: Requirements

**Working title.** A web app for three people running Jim Stoppani's Shortcut to Size program together. It replaces the paper log, keeps everyone's data in one database, and turns it into progress charts.

**Owner:** Lennon Crow
**Status:** Requirements, v1 (2026-10-15)
**Priority key:** **Must** = needed for first release, **Should** = next release, **Could** = later or optional

---

## 1. Goals

1. Stop carrying paper. Log every set on a phone during the workout.
2. Keep all three people's data in one place so it can be analyzed.
3. Show progress in a way that stays fair while reps drop and weight goes up week to week.

## 2. Users and access

| User | Role |
|---|---|
| Lennon | Admin and lifter |
| Lifter 1 | Lifter |
| Lifter 2 | Lifter |

- **FR-1 (Must):** Each person has their own login. There is no public sign-up; the admin creates accounts.
- **FR-2 (Must):** Passwords are stored hashed (Argon2 or bcrypt), never in plain text.
- **FR-3 (Must):** The site only runs over HTTPS.
- **FR-4 (Must):** Repeated failed logins are rate limited.
- **FR-5 (Must):** The admin can reset a user's password.
- **FR-6 (Must):** Every user can see every user's logs and charts. Users can only create or edit their own entries.

## 3. Program model

The app follows the program as printed on the group's paper log.

- 12 weeks, split into 3 phases of 4 weeks
- Rep range by week of phase: week 1 = 12-15, week 2 = 9-11, week 3 = 6-8, week 4 = 3-5
- Training days: Monday chest and triceps, Tuesday back and biceps, Thursday shoulders, Friday legs
- Abs and calves use higher rep ranges
- Finishing technique: rest-pause in weeks 1 and 2 of each phase, drop sets in weeks 3 and 4
- One program, same exercises for all three users

- **FR-7 (Must):** Store the program (phases, weeks, days, exercises, prescribed sets and rep ranges) as data, not hard-coded.
- **FR-8 (Must):** The group shares one current week. The app opens to today's workout for that week.
- **FR-9 (Must):** A missed workout affects only that person. It shows as missed for them without changing the group's week.
- **FR-10 (Should):** Support running the program again as a new 12-week cycle, keeping the old cycle's data for comparison.

## 4. Logging a workout

- **FR-11 (Must):** For each set, log weight (lbs) and reps.
- **FR-12 (Must):** Log the finishing set by type: rest-pause (reps after the 15-second pause) or drop set (weight and reps after the drop).
- **FR-13 (Must):** Each set saves as soon as it is entered. No "save" button, no lost data if the phone locks.
- **FR-14 (Must):** Next to each exercise, show what that person lifted last time at the same rep range.
- **FR-15 (Must):** Add a notes line per workout.
- **FR-16 (Must):** Users can edit or delete their own entries.
- **FR-17 (Must):** Screens are built for a phone held in one hand: large inputs and buttons, number keypad for weight and reps.
- **FR-18 (Must):** The app can be added to the home screen on iPhone and Android and opens like an app (progressive web app).
- **FR-19 (Should):** While the group works out, each person sees the others' sets appear live as they are entered.
- **FR-20 (Could):** If the phone loses signal, entries queue on the phone and sync when the connection returns.

## 5. Timer

- **FR-21 (Should):** When a person marks a set as done, their rest timer starts.
- **FR-22 (Should):** Each person sets their own rest length and can change it anytime.
- **FR-23 (Should):** A separate 15-second timer for rest-pause sets.
- **FR-24 (Should):** Timers are per person. One person's timer never starts, stops or changes anyone else's.
- **FR-25 (Should):** The timer is based on start time, not a running counter, so it shows the right time after the phone is unlocked or the app is reopened.
- **FR-26 (Should):** Android phones vibrate when the timer ends.

**Known limit:** iPhone web apps cannot vibrate, and sound is unreliable once the screen locks. On iPhone the timer is visual only.

## 6. Back-filling the paper logs

Workouts already done on paper get imported once.

- **FR-27 (Must):** Photos of the filled-in paper pages are converted to a CSV in a defined format, with a column flagging any value that was hard to read.
- **FR-28 (Must):** An import script loads the CSV into the database.
- **FR-29 (Must):** The import checks every row first (known user, valid week and exercise, numbers in a sane range) and produces a report of rejected rows without loading anything bad.
- **FR-30 (Must):** Running the same import twice does not create duplicates.
- **FR-31 (Must):** Flagged values are listed for a person to confirm before import.

## 7. Progress and analytics

All metrics are calculated in the database (SQL views or queries), not in the browser.

- **FR-32 (Must):** **Estimated max per lift**, using the Epley formula: weight × (1 + reps ÷ 30). This puts a 12-rep week and a 3-rep week on the same scale. Less accurate above about 10 reps; good for trends.
- **FR-33 (Must):** Chart of estimated max over time, per exercise, per person.
- **FR-34 (Should):** **Phase-vs-phase comparison:** the same rep-range week across phases (weeks 1, 5 and 9; weeks 2, 6 and 10; and so on).
- **FR-35 (Should):** Total volume (weight × reps) per workout and per week.
- **FR-36 (Should):** Personal record markers on charts and on the workout screen when a PR is set.
- **FR-37 (Should):** Consistency: workouts done vs planned, per person.
- **FR-38 (Should):** Side-by-side charts for all three users.
- **FR-39 (Should):** **Leaderboard by percent improvement:** each person's estimated max growth since their own week 1, averaged across lifts. No raw-weight rankings.
- **FR-40 (Should):** Export any user's data or the whole database as CSV.
- **FR-41 (Could):** Power BI report connected to the database through a read-only account.

## 8. Non-functional requirements

- **NFR-1 Security:** Secrets (database password, session key) live in AWS Systems Manager Parameter Store or Secrets Manager, never in the repo.
- **NFR-2 Security:** Reporting tools connect with a read-only database user.
- **NFR-3 Backups:** Automatic daily database backup to S3. A restore is tested at least once and the steps are written down.
- **NFR-4 Speed:** Saving a set takes under 1 second on a normal phone connection.
- **NFR-5 Privacy:** The public GitHub repo contains no real user data. Sample data in the repo is made up.
- **NFR-6 Reliability:** Single server, best effort. If the site is down, nothing is lost that was already saved.
- **NFR-7 Monitoring:** Health check endpoint, structured logs, and an alert when the site is down.
- **NFR-8 Cost:** Keep the monthly AWS bill low and check it monthly.

## 9. Tech stack

| Layer | Choice |
|---|---|
| Backend | Python, FastAPI |
| Database | PostgreSQL |
| Frontend | Server-rendered pages (Jinja templates), small amount of JavaScript, Chart.js |
| Live set updates | WebSockets |
| Containers | Docker, Docker Compose |
| Hosting | AWS: EC2 server, S3 for backups, DNS for a subdomain of lennoncrow.com |
| HTTPS | Caddy or nginx with a free Let's Encrypt certificate |
| Infrastructure as code | Terraform |
| CI/CD | GitHub Actions: run tests on every push, deploy on merge to main |
| Testing | pytest (backend), Playwright (end-to-end) |
| Monitoring | Prometheus and Grafana |
| Phase 3 | k3s (lightweight Kubernetes), Power BI, optional Kotlin Android app on the same backend |

## 10. How the project is run

These are part of the requirements because the project has to hold up in an interview.

- **Lennon writes the core code** (database schema, API, auth, import script, tests). AI tools can review and explain, but they don't write it.
- All work happens on branches and merges through pull requests.
- CI must pass before a merge.
- Every outage or bug that reaches the live site gets a short incident write-up: what broke, how it was found, root cause, fix.
- Big design choices are recorded as short decision notes in `docs/decisions/`.
- The README has a plain explanation, an architecture diagram, and how to run it locally.

## 11. Release plan

| Release | Contents |
|---|---|
| **1. Get off paper** | Accounts and login, program data, logging screens, last-time reference, deployed on AWS with HTTPS, CI running tests, daily backups |
| **2. Catch up** | CSV format, import script with validation, back-filled paper logs |
| **3. Work out together** | Rest and rest-pause timers, live set updates |
| **4. See progress** | Estimated max charts, phase comparison, volume, PRs, consistency, leaderboard, CSV export |
| **5. Harden** | Terraform for all AWS resources, Prometheus and Grafana, incident write-ups, k3s, Power BI |

## 12. Out of scope

- Bodyweight tracking
- Any AI features inside the app
- Per-person custom programs or exercise swaps
- Users beyond the three listed
- Native iPhone app
- Snowflake, Databricks, Kafka, microservices

## 13. Open questions

1. Where is the DNS for lennoncrow.com managed today?
2. Starting rest time for a new user, before they change their own?
3. Do you work out somewhere with weak signal? (Decides whether FR-20 moves up.)
4. After week 12, do you plan to run the program again? (Decides whether FR-10 moves up.)
5. Which AWS account does this go in?
