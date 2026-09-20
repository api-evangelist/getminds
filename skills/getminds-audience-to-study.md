---
name: getminds-audience-to-study
description: >-
  Build a grounded synthetic Audience from a brief, run a multi-question Study
  against it (concept/message test or segment comparison), and export the report.
api: Minds Public API
generated: '2026-09-20'
method: generated
source: https://getminds.ai/docs/api/agents
operations:
  - createAudienceFromBrief
  - getAudienceFromBriefCreationJob
  - getAudienceProgress
  - createStudy
  - previewStudyResearchPlan
  - runStudyQuestions
  - getStudyResearchStudy
  - getStudySemanticSummary
  - exportStudy
  - getStudyExportStatus
  - downloadStudyExport
---

# Audience → Study → Export

Grounded in real `/api/v1` operations. Auth: `Authorization: Bearer minds_…`.
Base: `https://getminds.ai/api/v1`. Persist every returned ID.

## Steps

1. **Create a grounded Audience from a brief** — `POST /audiences/from-brief`
   (`createAudienceFromBrief`). Persist `audienceId`. This call is idempotent for
   ~6h on identical arguments — retry with the *same* arguments after a timeout to
   recover the original Audience rather than creating a duplicate.
2. **Poll the build** — `GET /audiences/from-brief/jobs/{jobId}`
   (`getAudienceFromBriefCreationJob`) and/or `GET /audiences/{audienceId}/progress`
   (`getAudienceProgress`) until settled. Honor `Retry-After`; otherwise back off
   1–2s → 10–15s with jitter.
3. **Create a Study** — `POST /studies` (`createStudy`) attaching the Audience.
   Persist `studyId`. A matching name never attaches to an existing Study.
4. **Plan the questions** — `POST /studies/{studyId}/research-plans/preview`
   (`previewStudyResearchPlan`). Put *every* known question into one request.
   Present the returned draft and its `confirmationQuestions`; planning does not
   start research. Persist `draftPlanId` and `revision`.
5. **Confirm and run** — after explicit user confirmation of the exact revision,
   `POST /studies/{studyId}/research-runs` (`runStudyQuestions`). Persist `runId`.
6. **Poll the run** — `GET /studies/{studyId}/research-runs/{runId}`
   (`getStudyResearchStudy`) until terminal. Stop on success, failure,
   cancellation, or `plan_limited` (partial artifacts preserved — never call it
   complete).
7. **Summarize / export** — `GET /studies/{studyId}/summary`
   (`getStudySemanticSummary`), then `POST /studies/{studyId}/export`
   (`exportStudy`) → poll `GET …/export-status` (`getStudyExportStatus`) →
   `GET …/export-download` (`downloadStudyExport`).

## Rules
- Label all results as synthetic; preserve citations, IDs and shared links exactly.
- Keep link sharing disabled unless the user explicitly asks to publish.
- On `403` `data.code=PLAN_LIMIT`, explain the required plan change; do not retry.
- To abort a running study, `POST /runs/{runId}/cancel` (`cancelAgentRun`).
