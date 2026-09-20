---
name: getminds-ask-study
description: >-
  Ask one standalone question of the Audiences in an existing Study and read the
  synthetic answers, without planning a full multi-question research block.
api: Minds Public API
generated: '2026-09-20'
method: generated
source: https://getminds.ai/docs/api/agents
operations:
  - listStudies
  - getStudy
  - askStudy
  - createStudyRun
  - getStudyMessageAnswers
---

# Ask one standalone question in a Study

For a single, genuinely standalone question. For two or more questions, use the
planned block instead (see `getminds-audience-to-study`). Auth:
`Authorization: Bearer minds_…`, base `https://getminds.ai/api/v1`.

## Steps
1. **Find the Study** — `GET /studies` (`listStudies`); confirm with
   `GET /studies/{studyId}` (`getStudy`). Use the persisted `studyId`, not a
   fuzzy name match.
2. **Ask** — `POST /studies/{studyId}/ask` (`askStudy`) with the respondent-facing
   question text only. For a durable, resumable direct run use
   `POST /studies/{studyId}/runs` (`createStudyRun`) and persist `runId`.
3. **Read answers** — `GET /studies/{studyId}/messages/{messageId}/answers`
   (`getStudyMessageAnswers`).

## Rules
- The entire `question` value is respondent-visible — put only the question and
  stimulus there.
- For multiselect/categorical results, preserve the server's aggregation; do not
  force percentages to sum to 100.
- Report status from returned fields (queued/running/partial/completed/failed),
  not elapsed time.
