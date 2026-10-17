# Capability Map: AI-Powered Learning Platform

This map decomposes the approved initiative into independently testable modules.
Each module will receive its own specification and will be developed in the
listed dependency order.

| Module id | Responsibility | Depends on |
|---|---|---|
| `identity-and-roles` | User login, instructor/learner roles, authorization, and user-specific content access | — |
| `course-authoring` | Create and edit mini-courses, manage uploaded source files and references, instructor customization, and learner-created learning paths | `identity-and-roles` |
| `ai-content-generation` | Generate and refine text, images, simulations, role play, mock interviews, games, narrated animations, slides, and coding activities from topics, requirements, and sources | `identity-and-roles`, `course-authoring` |
| `enrollment-and-rosters` | Instructor course membership, Excel student import, invitations, and learner enrollment | `identity-and-roles`, `course-authoring` |
| `learning-delivery` | Present course content and interactive activities, personalized teaching/learning flows, progress, and marks capture | `identity-and-roles`, `course-authoring`, `ai-content-generation`, `enrollment-and-rosters` |
| `analytics-and-reporting` | Participation and engagement tracking, course performance views, marks, and generated learner-performance reports | `identity-and-roles`, `enrollment-and-rosters`, `learning-delivery` |

## Build Order

`identity-and-roles` → `course-authoring` → (`ai-content-generation` and
`enrollment-and-rosters` in parallel) → `learning-delivery` →
`analytics-and-reporting`

## Scope Notes

- The repository currently contains no application implementation; technology
  choices will be captured in the first module specification and kept
  consistent across subsequent module specifications.
- Identity owns secure invitation-token acceptance and invited learner
  registration; enrollment owns student import and course-specific invitation
  triggering.
- The existing untracked `temp.txt` is outside this initiative and must remain
  untouched.
