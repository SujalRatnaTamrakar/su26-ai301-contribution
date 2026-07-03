# Contribution 2: Add deletion of inactive players after 35 days #922

**Contribution Number:** 2  
**Student:** Sujal Ratna Tamrakar  
**Issue:** [lanedirt/OGameX #922](https://github.com/lanedirt/OGameX/issues/922)  
**Status:** Phase I Complete

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

*(To be filled in during Phase II)*

### Steps to Reproduce

*(To be filled in during Phase II)*

### Reproduction Evidence

- **Commit showing reproduction:** *[Link to commit in your fork]*
- **Screenshots/logs:** *[If applicable]*
- **My findings:** *[What you discovered during reproduction]*

---

## Solution Approach

### Analysis

*(To be filled in during Phase II)*

### Proposed Solution

*(To be filled in during Phase II)*

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** The goal is to automatically delete player accounts after a continuous number of inactive days, determined by a single configuration value (where `0` is disabled and serves as the default). Vacationing players are not exempt. Because planet destruction (#146) is missing, I will stub out that portion of the abandonment logic using explicit `TODO` blocks.

**Match:** I need to examine how the codebase currently reads server configuration files, how issue #703 reads/updates the inactivity timestamps, and how the core game loop or cron-tasks delete or modify user profiles.

**Plan:** *(To be filled in during Phase II)*

**Implement:** *[Link to your branch/commits as you work]*

**Review:** *[Self-review checklist]*

**Evaluate:** *[How will you verify it works?]*

---

## Testing Strategy

*(To be filled in during Phase III)*

---

## Implementation Notes

*(To be filled in during Phase III)*

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