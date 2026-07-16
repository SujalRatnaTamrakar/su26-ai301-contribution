# Contribution 2: Add deletion of inactive players after 35 days #922

**Contribution Number:** 2  
**Student:** Sujal Ratna Tamrakar  
**Issue:** [lanedirt/OGameX #922](https://github.com/lanedirt/OGameX/issues/922)  
**My Fork:** [sujalratnatamrakar/OGameX](https://github.com/SujalRatnaTamrakar/OGameX)  
**Working Branch:** `feature/922-delete-inactive-players` (branched from latest `main`)  
**Status:** Phase III Complete

---

## Why I Chose This Issue

As a graduate student with a background in deep learning, data pipeline management, and complex systems backend development, I wanted to tackle a core game-mechanic feature for my CodePath AI301 capstone project. Managing state persistence—specifically handling when and how automated cleanup pipelines execute on database entries—is a crucial aspect of scalable backend architecture. This issue directly aligns with my interest in building robust, automated workflows and writing clean, highly maintainable systems-level code.

Furthermore, this issue provides an excellent opportunity to dive deep into open-source project standards. It forces me to think critically about developer and host flexibility by implementing server-side configuration settings. I look forward to working within this codebase to learn how the project handles cron-like routine jobs, server property settings, and relational object deletions.

---

## Understanding the Issue

### Problem Description

Currently, the server tracks player inactivity markers (implemented in a previous issue, #703), but it lacks an automated lifecycle cleanup process. Inactive player accounts persist indefinitely, which artificially inflates database size and leaves un-farmed or dead planetary infrastructure cluttering the game universe. The project needs an automated backend routine that checks for prolonged inactivity and safely purges those accounts.

### Expected Behavior

The server should run a routine check on player accounts to evaluate inactivity. This behavior must be consolidated into a single server-side configuration setting to maintain flexibility across different server environments:
1. **Consolidated Configuration:** A single integer setting representing the required inactivity days threshold.
   - `0` = Feature is completely disabled (this must be the default setting).
   - `> 0` = The specific number of inactivity days required before an account is deleted (e.g., `35`).
2. **Exemptions:** The routine must apply uniformly to all players; players in "vacation mode" are **not** exempt from the deletion clock.
3. **Scope Boundary:** When triggered, the system must permanently delete the player account and abandon their planets. Because the "Destroyed Planet" logic (#146) is not yet implemented in the codebase, the planet abandonment section must include clear, structured code `TODO` markers pointing to #146 so it can be cleanly hooked in later.

### Current Behavior

Inactive player accounts remain in the database forever regardless of how many days have passed. There is currently no server setting block or automated cron/routine to handle the deletion of these expired records, nor is there an active tie-in to trigger planet modifications upon long-term inactivity.

### Affected Components

Based on early issue analysis and maintainer feedback, the following components are expected to be involved:
* **Server Configuration/Properties System:** To register the new consolidated integer threshold variable (defaulting to 0).
* **Cron/Scheduler/Task Routine Module:** The background runner where the periodic database scanning logic will live.
* **Player/Account Data Layer:** Where player records are checked against inactivity markers.
* **Planet/Universe Management Modules:** The business logic responsible for modifying planet ownership structures and abandoning assets (where the #146 `TODO` placeholders will be placed).

---

## Reproduction Process

### Environment Setup

**Setup approach.** I used the project's bundled development Docker stack rather than a manual install, following the "Install for local development" section of the `README.md`:

```bash
git checkout -b feature/922-delete-inactive-players   # branch from latest main
docker compose up -d
```

This brings up the full stack: `ogamex-app` (PHP/FPM), `ogamex-db` (MariaDB), `ogamex-scheduler`, `ogamex-queue-worker`, `ogamex-reverb`, `ogamex-webserver`, and `ogamex-phpmyadmin`.

To learn exactly which standards a PR is held to, I **inspected the CI configuration** under `.github/workflows/` before writing anything. Four gates are enforced on every push, and I mirrored each locally through the Composer scripts so I can pass them before submitting:

| CI workflow | Local command |
| --- | --- |
| `run-laravel-pint-code-style-checker.yml` | `composer run cs -- --test` |
| `run-phpstan-code-analysis.yml` | `composer run stan` |
| `run-rector-code-analysis.yml` | `composer run rector` |
| `run-tests-docker-compose.yml` | `php artisan test` |

**Real challenges encountered and how I resolved them:**

1. **First-run startup takes ~10 minutes.** The `ogamex-app` container appeared to hang on first boot. The `README.md` explains this is expected: on first run the container performs Composer installation *and* compiles the Rust battle engine. **Resolution:** waited for all containers to report healthy before loading `http://localhost`, instead of assuming a failure and restarting (which would have restarted the clock).

2. **No PHP/Composer toolchain on the host.** Running `php artisan` / `composer` directly on the WSL host fails — there is no local PHP. **Resolution:** every command must be executed inside the container, e.g. `docker compose exec ogamex-app php artisan ...` / `docker compose exec ogamex-app composer run stan`. I standardized on this prefix for all interaction.

3. **Node/npm are not present in the `ogamex-app` container.** `docker compose exec ogamex-app npm ...` returns "executable file not found". **Resolution:** confirmed Node lives on the host (WSL) and, more importantly, verified via `vite.config.js` that Vite only compiles `resources/js/*` and CSS entrypoints — Blade templates are never Vite inputs. This told me early that a change confined to server config + a Blade admin field would not require an asset rebuild, so I could scope the environment accordingly.

4. **Feature-branch naming.** Created the working branch off the latest `main` (not off another branch), matching the `CONTRIBUTING.md` "branch from main" rule, and named it after the issue number (`922`).

### Steps to Reproduce

Because this is a missing-feature gap rather than a crash, "reproduction" means demonstrating that an over-inactive account is **never** cleaned up and that no configuration or routine exists to do so. These steps are runnable on a clean checkout of `main` by anyone:

1. Start the stack: `docker compose up -d` and wait for all containers to be healthy.
2. Open `http://localhost` and register a new account. Note its planet in the galaxy view (Galaxy → its system/position).
3. Backdate the account's last-activity marker to simulate long inactivity:
   ```bash
   docker compose exec ogamex-app php artisan tinker
   >>> $u = \OGame\Models\User::orderBy('id','desc')->first();
   >>> $u->time = (string) now()->subDays(40)->timestamp;  // 40 days idle
   >>> $u->save();
   ```
4. Trigger the scheduler the way the `ogamex-scheduler` container does: `docker compose exec ogamex-app php artisan schedule:run` (repeat / wait a day).
5. Confirm no configuration exists to enable such cleanup:
   ```bash
   docker compose exec ogamex-app php artisan tinker
   >>> \OGame\Models\Setting::where('key','like','%inactiv%delet%')->get();  // empty
   ```
6. Reload the galaxy view.

### Expected vs. Actual Behavior

- **Expected:** An administrator should be able to set an inactivity-days threshold; once an account exceeds it, the account is permanently deleted and its planets are abandoned (freed in the galaxy). With the threshold at `0` (default), nothing is deleted.
- **Actual (on `main`):** There is **no** such setting, **no** scheduled routine, and **no** account-lifecycle deletion path. The backdated account persists indefinitely and its planet still occupies its galaxy slot. Inactivity is *measured* but never *acted upon*.

### Reproduction Evidence

- **Commit showing reproduction:** No code change is needed to observe the gap — it is reproduced against unmodified `main` using the steps above. Investigation happens on branch `feature/922-delete-inactive-players`.
- **Screenshots/logs:** `php artisan schedule:list` on `main` shows highscore, debris/wreck-field, message and dark-matter jobs — but **no** inactive-player job, confirming the missing routine.
- **My findings:**
  - Inactivity is already tracked: `users.time` (UNIX timestamp) is read by `PlayerService::isInactive()` (7 days) and `isLongInactive()` (28 days) — added by commit `3d3df03d` "Add player inactivity and weak/strong indicators to galaxy page" (2025-09-21), the #703 work. So the data exists; only the deletion lifecycle is missing.
  - Scheduled jobs are registered in `routes/console.php` (Laravel 13 has no `Console/Kernel.php`), each backed by a class in `app/Console/Commands/Scheduler/`.
  - A player-wipe method already exists — `PlayerService::delete()` — but it **raw-deletes** planet rows and does **not** use the abandonment flow the maintainer requires.

---

## Solution Approach

### Analysis

**Root cause (not just the symptom).** The symptom is "inactive accounts are never removed." The underlying root cause is that the inactivity signal is fully implemented but **orphaned** — nothing consumes it for lifecycle management. Concretely, three connected gaps:

1. **No configuration surface.** Settings are key/value rows via `app/Models/Setting.php`, read/written through `app/Services/SettingsService.php` (`get()`/`set()` + one typed getter per key) and edited in `app/Http/Controllers/Admin/ServerSettingsController.php` + `resources/views/ingame/admin/serversettings.blade.php`. No `inactive_player_deletion_days` key exists.
2. **No routine.** No entry in `routes/console.php` and no class in `app/Console/Commands/Scheduler/` scans `users.time` against a threshold.
3. **No abandonment-based wipe.** `PlayerService::delete()` (`app/Services/PlayerService.php`, ~line 878) exists but **raw-deletes** planet rows — it never calls `PlanetService::abandonPlanet()` (`app/Services/PlanetService.php`, ~line 222), so galaxy slots and other players' in-flight missions are not handled the way the maintainer requires. Worse, `abandonPlanet()` **guards against** abandoning the last remaining planet and any planet with active fleet missions — both conditions a to-be-deleted account will trigger — so it cannot be looped naively.

**Specific files/functions involved:**
- `app/Models/User.php` — the `time` (last-activity) and `vacation_mode` fields.
- `app/Services/PlayerService.php` — `isInactive()`, `isLongInactive()`, `delete()`.
- `app/Services/PlanetService.php` — `abandonPlanet()` (and its last-planet / active-mission guards, moon cascade, and planet-row deletion point).
- `app/Services/SettingsService.php`, `ServerSettingsController.php`, `serversettings.blade.php` — the setting surface.
- `routes/console.php` + `app/Console/Commands/Scheduler/` — the scheduler registration.

**Proactively identified edge cases** (found during investigation, before coding):
- **Last remaining planet** and **active fleet missions** — `abandonPlanet()` throws on both; a full-account delete must bypass these guards.
- **Moons** — `abandonPlanet()` cascades a planet's moon; iterating only planets (not moons) avoids double-abandonment.
- **Incoming missions from other players** must be preserved (target nulled, not deleted).
- **Admin/operator accounts** — deletion is irreversible; the codebase already protects admins from destructive actions ("Administrators cannot be banned").
- **Battle/espionage reports** — their `planet_user_id` FK is `ON DELETE SET NULL` (see Match, below), so reports must be *preserved*, not deleted.
- **Null `time`** — accounts with no recorded activity should be excluded from the numeric comparison.

### Proposed Solution

At the planning level (implementation is Phase III):
1. Add a single integer setting `inactive_player_deletion_days` (default `0` = disabled) with a typed `SettingsService` getter, a seed migration, controller wiring, and one Blade admin field.
2. Add a new abandonment-based wipe `PlayerService::deleteInactiveAccount()` that clears the player's own fleet missions, abandons each planet through `abandonPlanet()` (freeing galaxy slots and cascading to moons), and then removes the account — deliberately *not* touching battle/espionage reports.
3. Add a `bool $force = false` parameter to `abandonPlanet()` so full-account deletion can bypass the last-planet / active-mission guards without duplicating the routine (normal abandon behavior unchanged).
4. Add a daily `DeleteInactivePlayers` scheduler command that no-ops when the setting is `0`, selects non-admin users whose `time` predates the threshold (vacation mode **not** exempt), and deletes each.
5. Place `TODO (#146)` markers at the planet-removal points in `abandonPlanet()` where the unimplemented "Destroyed Planet" logic will hook in.

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** The goal is to automatically delete player accounts after a continuous number of inactive days, determined by a single configuration value (where `0` is disabled and serves as the default). Vacationing players are not exempt. Because planet destruction (#146) is missing, I will stub out that portion of the abandonment logic using explicit `TODO` blocks.

**Match:** I found a directly analogous, maintainer-authored pattern rather than guessing:

- **Closest analogue — `DeleteOldMessages`** (`app/Console/Commands/Scheduler/DeleteOldMessages.php`, introduced by commit `0c97c65d`, *"feat: delete inbox messages after 7 days…" (#1423), by Geda173, 2026-05-25*). This is a daily scheduled command that deletes DB records older than a threshold using `chunkById`, uses `#[Signature]`/`#[Description]` attributes, and returns `Command::SUCCESS`. My command mirrors this structure exactly — the same maintainer authored it, so it is the sanctioned template.
- **Setting pattern** — every setting is a one-line typed getter (`(int)$this->get('key', default)`) seeded by a small `updateOrInsert` migration and wired through `ServerSettingsController` + one Blade field (e.g. `wreckFieldMinResourcesLoss()`).
- **Guard analogue** — the admin ban flow refuses to act on admins (`ServerAdministrationController::ban()`: *"Administrators cannot be banned"*), which is the pattern I follow for excluding admins from auto-deletion.
- **git-dated design evidence (edge case)** — using `git log` I found the report FKs were changed to `ON DELETE SET NULL` specifically to *preserve* reports on player deletion: commit `4a2caffb` *"Fix FK constraint violation when deleting a player with espionage reports" (#1326, Geda173, 2026-03-28)* and the earlier `71d98a7f` for battle reports. This dated the intent and told me deletion must **not** remove reports — a non-obvious constraint I would have gotten wrong from the create-table migrations alone.
- **Dating the dependency** — `git blame` on `PlayerService::isInactive()` shows the inactivity markers landed in commit `3d3df03d` (2025-09-21), the #703 work my feature builds on.

**Plan:** Files to create/modify (root cause → concrete edits):
- *Setting:* add `SettingsService::inactivePlayerDeletionDays()`; new migration `..._add_inactive_player_deletion_setting.php` seeding `0`; add the key to `ServerSettingsController::index()`/`update()`; add one field to `serversettings.blade.php`.
- *Deletion:* add `PlayerService::deleteInactiveAccount()` (transaction-wrapped, abandon-based, orphan-safe); add `bool $force = false` to `PlanetService::abandonPlanet()` to bypass the two guards; add `TODO (#146)` markers at the planet-removal points.
- *Routine:* add `app/Console/Commands/Scheduler/DeleteInactivePlayers.php`; register it `->daily()->withoutOverlapping()` in `routes/console.php`.

**Implement:** *Completed on `feature/922-delete-inactive-players` in commit `932ddbc5` — see Implementation Notes below.*

**Review (planned self-review checklist):** match surrounding code style (snake_case where present, helper methods, no duplication per `CONTRIBUTING.md` §1); pass Pint, PHPStan, Rector, and the test suite; keep the change scoped to #922 (one issue, one PR); confirm `abandonPlanet()`'s default behavior is unchanged for existing callers.

**Evaluate (planned verification):** feature tests covering the `0`-disabled no-op, threshold selection (inactive deleted / active kept), vacation-not-exempt, admin exclusion, and a full-wipe integrity check (planets/moons/queues removed, own missions deleted, reports preserved with `planet_user_id` nulled); plus a manual smoke test — set the admin field to `1`, backdate a test account, run `php artisan ogamex:scheduler:delete-inactive-players`, and confirm the account and its galaxy slot are gone while set-to-`0` deletes nothing.

---

## Testing Strategy

### Automated tests

New file `tests/Feature/DeleteInactivePlayersCommandTest.php` — 5 feature tests that exercise the new behavior, written to follow the project's existing test conventions:

- Extends **`AccountTestCase`** (the project's base class for account-context tests) and reuses its helpers: `$this->planetService`, `planetAddResources()`, `addResourceBuildRequest()`, and `PlanetServiceFactory::createMoonForPlanet()`.
- Mirrors the **tracked-user + `tearDown()` cleanup** pattern from `tests/Feature/BanTest.php` (factory users are recorded and deleted, with `users_tech` cleaned first to avoid FK errors).
- Uses `User::factory()`, `assertDatabaseHas/Missing`, and time-travel (`Date::now()->subDays()`), consistent with `DeleteOldMessagesCommandTest`.

| Test | What it verifies |
| --- | --- |
| `testCommandIsDisabledWhenDaysIsZero` | Setting `0` → no-op; account preserved |
| `testDeletesInactivePlayerButKeepsActivePlayer` | Over-threshold account deleted; recently-active account kept |
| `testVacationModeDoesNotExemptPlayer` | Vacation-mode account is still deleted |
| `testExcludesAdminPlayers` | Admin account is preserved |
| `testDeletionAbandonsPlanetsAndPreservesReports` | Planets/moon/queues removed, own fleet mission deleted, espionage report preserved with `planet_user_id` nulled |

**Results:** all 5 new tests pass, and the **existing suite still passes** — `php artisan test` → **1065 passed (11,744 assertions), 0 failures**. Other gates: PHPStan (651 files, no errors), Rector (no changes needed). Laravel Pint passes for every file this PR touches; the only Pint failures are pre-existing style issues in ~19 files this PR does **not** modify, left untouched per the "one issue, one PR" rule.

### Manual verification

- `php artisan schedule:list` confirms `ogamex:scheduler:delete-inactive-players` is registered to run daily.
- `php artisan migrate` applies the seed migration; the setting reads back as `0` (disabled) by default.
- Running the command with the setting at `0` prints the "disabled" message and deletes nothing — confirming the safe default.
- Design smoke path: set the admin field to a threshold, backdate a test account's `users.time` via tinker, run the command, and confirm the account and its galaxy slot are removed.

All commands executed inside the container via `docker compose exec ogamex-app ...`.

---

## Implementation Notes

### Implementation Progress

All Phase III work landed in a single, scoped commit on `feature/922-delete-inactive-players`:

- **`932ddbc5`** — *feat: add automatic deletion of inactive players (#922)*

The diff is limited to the 9 files below — no unrelated formatting, no commented-out code:

| File | Change |
| --- | --- |
| `app/Services/SettingsService.php` | `inactivePlayerDeletionDays()` typed getter |
| `database/migrations/2026_07_08_000000_add_inactive_player_deletion_setting.php` | Seed default `0` |
| `app/Http/Controllers/Admin/ServerSettingsController.php` | index/update wiring for the setting |
| `resources/views/ingame/admin/serversettings.blade.php` | Admin "Player deletion settings" field |
| `app/Services/PlanetService.php` | `abandonPlanet(bool $force = false)` + `TODO (#146)` markers |
| `app/Services/PlayerService.php` | New `deleteInactiveAccount()` (abandon-based wipe) |
| `app/Console/Commands/Scheduler/DeleteInactivePlayers.php` | New daily scheduler command |
| `routes/console.php` | Register the command `->daily()->withoutOverlapping()` |
| `tests/Feature/DeleteInactivePlayersCommandTest.php` | Feature tests |

### Challenges Faced

Real obstacles hit during Phase III and how they were resolved:

1. **Report foreign keys are `ON DELETE SET NULL`, not orphaning.** My first read of the create-table migrations suggested battle/espionage reports would be orphaned on deletion. Running `git log` on those tables surfaced a later fix (#1326) that changed the FK to `ON DELETE SET NULL` *specifically to preserve reports when a player is deleted*. **Resolution:** the deletion logic deliberately does **not** delete reports, and a test asserts the report survives with `planet_user_id` nulled.
2. **`abandonPlanet()` guards block full-account deletion.** It throws on the last remaining planet and on any planet with active fleet missions — both conditions a purged account triggers. **Resolution:** added an optional `$force` flag to bypass the guards for full-account deletion, leaving the default (guarded) behavior unchanged for all existing callers.
3. **A stale factory cache caused an FK violation in tests.** The integrity test creates a moon *after* the `PlayerService` was cached; the command then deleted the user while a moon row still referenced it → FK error. **Resolution:** the command loads via `PlayerServiceFactory::make($id, true)` (reload cache) so it always operates on the account's current planets/moons — which is also the correct production behavior.
4. **No Node/npm toolchain in the app container.** The asset-build step could not run there. **Resolution:** confirmed via `vite.config.js` that only `resources/js/*`/CSS files are Vite inputs, so a Blade-only change requires no rebuild — avoiding unrelated build-artifact churn.

### Engineering Judgment

- **Edge cases the issue didn't mention** were identified and handled proactively: excluding admin accounts (mirroring the existing "Administrators cannot be banned" protection), preserving shared battle/espionage reports, cascading moon removal, and preserving other players' incoming fleet missions.
- **Reused a project-specific test helper:** built on `AccountTestCase` and the `BanTest` tracked-user/`tearDown` cleanup pattern instead of hand-rolling fixtures.
- **Descoped sensibly:** left structured `TODO (#146)` markers for the unimplemented "Destroyed Planet" logic rather than attempting to build it, exactly as the maintainer requested.

---

## Pull Request

*(To be filled in during Phase IV)*

---

## Learnings & Reflections

*(To be filled in during Phase IV)*

---

## Resources Used

* **GitHub Issue #922:** [Add deletion of inactive players after 35 days](https://github.com/lanedirt/OGameX/issues/922)
* **GitHub Related Issue #703:** [Inactivity markers implementation](https://github.com/lanedirt/OGameX/issues/703)
* **GitHub Related Issue #146:** [Destroyed planet mechanics](https://github.com/lanedirt/OGameX/issues/146)