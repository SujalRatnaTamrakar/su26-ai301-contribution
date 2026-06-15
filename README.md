# Contribution 1: [Feature Request]: Image Resizing Inside Editor #91

**Contribution Number:** 1  
**Student:** Sujal Ratna Tamrakar  
**Issue:** [GitHub issue link](https://github.com/rawilk/filament-quill/issues/91)  
**Status:** Phase II Complete

---

## Why I Chose This Issue

I chose this issue because it aligns perfectly with my existing experience working within the Laravel and Filament PHP ecosystems. The feature request addresses a highly practical capability—allowing users to intuitively resize images directly within the rich-text editor UI. Currently, integrating third-party resizing packages with this specific Filament wrapper is complex, leaving a clear gap that would highly benefit the project's user base.

Furthermore, this issue presents an excellent open-source opportunity because the project maintainer explicitly stated that while they appreciate the feature, it is not a high priority for them right now and they are openly welcoming a community pull request. Working on this will allow me to deepen my understanding of custom form component integrations in Filament, learn how to bridge frontend editor modules with backend configurations, and deliver a feature that the community is actively waiting for.

---

## Understanding the Issue

### Feature Description

The objective is to introduce a seamless, interactive image resizing mechanism directly within the Quill rich-text editor instance managed by this Filament package. Currently, users have no inline layout controls to scale, resize, or modify dimensions of an image post-upload inside the workspace. Implementing this will add critical editorial flexibility, ensuring content creators can format complex layouts intuitively without depending on raw HTML overrides.

### Expected Behavior

When an image node is selected or clicked within the rich-text input field:
1. Interactive resize handles or a dedicated overlay container should appear around the perimeter of the image element.
2. Users should be able to click and drag the corners to scale the image proportionally, or select predefined percentage/pixel sizing controls via a popup tool block.
3. The newly generated height and width dimensions must automatically synchronize with the element's styling or attributes, correctly updating the underlying saved HTML output payload seamlessly.

### Current Behavior

Once an image is inserted or dropped into the active workspace editor:
- It displays rigidly at its initial or native dimensions.
- Clicking or focusing on the image element fails to evoke any responsive resizing tools, handles, or adjustment settings within the component canvas interface.

### Affected Components

- **Local Package Architecture Source Space:** Located within the manually registered path route `/var/www/filament-quill/src/`. This directory hosts the backend class logic, field component definitions, and trait structures that build the interface wrapper.
- **Frontend JavaScript & Asset Compilation Pipelines:** The underlying scripts where the third-party Quill engine and its default module systems are configured, compiled, and registered.
- **Component Blade Views & Livewire Bridges:** The field templates rendering the asset layer that must pass custom user/system initialization configurations downstream into the active editor instance.

---

## Understanding the Issue

### Feature Description

The objective is to introduce a seamless, interactive image resizing mechanism directly within the Quill rich-text editor instance managed by this Filament package. Currently, users have no inline layout controls to scale, resize, or modify dimensions of an image post-upload inside the workspace. Implementing this will add critical editorial flexibility, ensuring content creators can format complex layouts intuitively without depending on raw HTML overrides.

### Expected Behavior

When an image node is selected or clicked within the rich-text input field:
1. Interactive resize handles or a dedicated overlay container should appear around the perimeter of the image element.
2. Users should be able to click and drag the corners to scale the image proportionally, or select predefined percentage/pixel sizing controls via a popup tool block.
3. The newly generated height and width dimensions must automatically synchronize with the element's styling or attributes, correctly updating the underlying saved HTML output payload seamlessly.

### Current Behavior

Once an image is inserted or dropped into the active workspace editor:
- It displays rigidly at its initial or native dimensions.
- Clicking or focusing on the image element fails to evoke any responsive resizing tools, handles, or adjustment settings within the component canvas interface.

### Affected Components

- **Local Package Architecture Source Space:** Located within the manually registered path route `/var/www/filament-quill/src/`. This directory hosts the backend class logic, field component definitions, and trait structures that build the interface wrapper.
- **Frontend JavaScript & Asset Compilation Pipelines:** The underlying scripts where the third-party Quill engine and its default module systems are configured, compiled, and registered.
- **Component Blade Views & Livewire Bridges:** The field templates rendering the asset layer that must pass custom user/system initialization configurations downstream into the active editor instance.

---

## Reproduction Process (Local Development Setup)

### Environment Setup

To establish a local development and testing environment for this feature, I constructed an isolated Docker runtime sandbox using Windows WSL2 and Laravel Sail, bypassing complex package version dependency deadlocks through a manual tracking layout.

- **WSL2 Subsystem Integration:** Configured Docker Desktop settings to activate the Ubuntu integration bridge, running `wsl --shutdown` via PowerShell to completely flush the subsystem's active memory layout before initializing a clean terminal.
- **Host Sandbox Infrastructure:** Utilized Laravel's automated installation wrapper (`laravel.build`) to initialize a clean development host application named `filament-test-app` inside the Linux `~/code` workspace.
- **Dependency Bypass via PSR-4 Autoloading:** Due to strict framework versioning constraints matching Laravel 12 and older dependency conflicts, typical local path configurations via Composer were blocked. To circumvent this calculation loop, I bypassed standard package registry installations and manually registered the package fork directly inside `filament-test-app/composer.json` using a native PSR-4 mapping strategy:
  ```json
  "autoload": {
      "psr-4": {
          "App\\": "app/",
          "Database\\Factories\\": "database/factories/",
          "Database\\Seeders\\": "database/seeders/",
          "Rawilk\\FilamentQuill\\": "/var/www/filament-quill/src/"
      }
  }
  ```
  Following registration, I ran `composer dump-autoload` inside the active container to map the file routes.
- **Database Engine Optimization:** Configured the empty MySQL container using `php artisan migrate`, removed mass assignment protections on the base User model, and populated admin authentication credentials using `php artisan make:filament-user`.

### Steps to Verify Environment Setup

1. Spin up the local containerized environment via Laravel Sail using `./vendor/bin/sail up -d`.
2. Generate a fresh dashboard test area using `php artisan make:filament-resource Post` paired with its database structural model `php artisan make:model Post -m`.
3. Execute `php artisan migrate` to synchronize the structural post schemas into the database instance.
4. Launch the local application workspace by navigating to `http://localhost/admin` inside the browser.

### Development Evidence

- **Development Branch Link:** [Link to your local development branch on GitHub]
- **Screenshots/Logs:** 
![alt text](image.png)
- **My Findings:** The host development platform successfully mounts, resolves, and loads the backend class files nested inside the local package mapping space (`/var/www/filament-quill/src/`). The development environment is fully operational and listening to structural code modifications. The next step is evaluating the frontend build system setup to determine how the Quill editor initializes its JavaScript module ecosystem.

---

## Solution Approach

### Analysis

[Your analysis of the root cause - what's causing the issue?]

### Proposed Solution

[High-level description of your fix approach]

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** [Restate the problem]

**Match:** [What similar patterns/solutions exist in the codebase?]

**Plan:** [Step-by-step implementation plan]
1. [Modify file X to do Y]
2. [Add function Z]
3. [Update tests]

**Implement:** [Link to your branch/commits as you work]

**Review:** [Self-review checklist - does it follow the project's contribution guidelines?]

**Evaluate:** [How will you verify it works?]

---

## Testing Strategy

### Unit Tests

- [ ] Test case 1: [Description]
- [ ] Test case 2: [Description]
- [ ] Test case 3: [Description]

### Integration Tests

- [ ] Integration scenario 1
- [ ] Integration scenario 2

### Manual Testing

[What you tested manually and results]

---

## Implementation Notes

### Week [X] Progress

[What you built this week, challenges faced, decisions made]

### Week [Y] Progress

[Continue documenting as you work]

### Code Changes

- **Files modified:** [List]
- **Key commits:** [Links to important commits]
- **Approach decisions:** [Why you chose certain approaches]

---

## Pull Request

**PR Link:** [GitHub PR URL when submitted]

**PR Description:** [Draft or final PR description - much of the content above can be adapted]

**Maintainer Feedback:**
- [Date]: [Summary of feedback received]
- [Date]: [How you addressed it]

**Status:** [Awaiting review / Iterating / Approved / Merged]

---

## Learnings & Reflections

### Technical Skills Gained

[What you learned technically]

### Challenges Overcome

[What was hard and how you solved it]

### What I'd Do Differently Next Time

[Reflection on your process]

---

## Resources Used

- [Link to helpful documentation]
- [Tutorial or Stack Overflow post that helped]
- [GitHub issues or discussions that helped]
