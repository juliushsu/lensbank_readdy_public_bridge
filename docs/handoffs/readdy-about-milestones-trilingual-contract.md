# Readdy Handoff: About Milestones Tri-Lingual CMS Contract

Status: HANDOFF / DO NOT PUBLISH

Canonical phase: About milestones tri-lingual schema/service contract

## Summary

The LensBank About page timeline/milestones are CMS content. They are not fixed frontend i18n strings.

Readdy must not use automatic frontend i18n translation for milestone content. English and Japanese milestone copy must be stored as manually reviewed fields.

## Required UI Sync

`/admin/milestones` needs fields for:

- Chinese Title
- Chinese Description
- English Title
- English Description
- Japanese Title
- Japanese Description

`/about` needs to use a localized helper such as:

```ts
getLocalizedMilestone(milestone, locale, field)
```

Required fallback order:

- `zh-TW`: `zh` -> `en` -> `ja`
- `en`: `en` -> `zh` -> `ja`
- `ja`: `ja` -> `zh` -> `en`

## Forbidden

Do not:

- Readdy Publish this change directly.
- Assume production DB already has `title_en` or `description_en`.
- Use i18n keys to auto-translate milestone CMS content.
- Add live frontend machine translation.
- Overwrite admin security patches, scheduling patches, Edge Functions, RLS, or migrations.
- Change production DB, secrets, DNS, or deployment settings.

## Required Sequence

1. Codex prepares schema migration draft.
2. Owner reviews Production Approval Gate.
3. Codex applies migration only after approval.
4. Codex or Readdy may update UI after the schema contract is accepted.
5. Codex builds and audits.
6. Only then may a release candidate be considered.

## Notes for Readdy

AI translation may be offered only as a draft suggestion in the admin editor. A human operator must confirm and save final text before it appears on the public site.
