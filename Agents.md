# Agents Guide

This repo powers a static Machine Learning System Design site.

## Repository layout

- `index.html`: homepage
- `course.html`: course content shell that loads chapter content
- `chapters/`: markdown chapters and `sidebar.json` navigation
- `assets/`: diagrams and static images
- `Machine-Learning-Interviews/`: reference content source material

## Core workflow for content changes

1. Add or edit chapter markdown in `chapters/`.
2. Keep file naming consistent with existing IDs (example: `agent-09-xyz.md`).
3. Update `chapters/sidebar.json` so the new chapter appears in navigation.
4. Verify links, headings, and image paths resolve correctly from `course.html`.

## Conventions

- Preserve the existing chapter naming scheme and section ordering style.
- Keep markdown focused, scannable, and interview-oriented.
- Prefer relative asset paths from chapter files (example: `../assets/...` when needed).
- Do not rename or delete existing chapter files unless explicitly requested.

## Change checklist

- Navigation entry added/updated in `chapters/sidebar.json`
- Chapter ID matches the markdown filename
- No broken internal links or image references
- Existing ML and Agent tracks remain intact

## Notes for automation agents

- This project has no strict build pipeline in-repo; changes are primarily content and static HTML updates.
- For new content, prioritize minimal, targeted edits over broad restructuring.
- If a change impacts structure, update both content and navigation in the same commit.
