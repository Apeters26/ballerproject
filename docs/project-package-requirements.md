# Reusable project package: agreed requirements

Status: requirements recovered from the Drone project conversation on November 14, 2026. This is not a claim that the complete package is implemented or recovered.

## Objective

Create one navigable project website that moves from the overall product vision to implementation-level detail and then to evidence of what was actually delivered. Drone Lab is the first worked example. Package the workflow as a reusable skill or starter package for other projects.

## Required content

- Vision: problem, intended users, end-state experience, goals, scope, non-goals, assumptions, and idea-validation plan.
- Product definition and product design specification: user journeys, screens/mockups, interactions, states, accessibility, and acceptance requirements.
- Architecture specification: system boundaries, components, data flows, integrations, deployment shape, and key decisions.
- Technical specification: implementation contracts, data models, interfaces, behavior, performance constraints, failure handling, and tests.
- Overall roadmap and release plan: release outcomes, dependencies, milestones, epics, stories, and acceptance criteria. Detail the first release; preserve uncertainty in later releases.
- Decisions, risks, dependencies, ownership, and specification maintenance.
- Release readiness and operations: verification, acceptance, deployment, rollback, monitoring, and feedback.
- Release notes for every release: actual delivered changes, limitations, fixes, evidence, version, and date. Unreleased entries remain explicitly labeled drafts.
- A product vision video showing the intended end-state product. Clearly distinguish concept visuals from working functionality. Include narration, captions, transcript, and chapter controls.
- Reusable skill/package: templates, structured records, stable identifiers, website generator/source, workflow instructions, build and verification tooling, and the Drone worked example.

## Traceability contract

Link vision/outcomes -> product requirements -> design/architecture/technical sections -> release -> epic -> story -> acceptance criterion -> implementation issue/PR/commit -> test/evidence -> acceptance decision -> shipped version -> release note.

Preserve stable identifiers when regenerating the site. Distinguish implemented, verified, accepted, and shipped; do not collapse them into a single done flag. Show missing evidence rather than substituting expected evidence for an actual result.

The earlier browser inspection displayed 87 stories, 377 acceptance criteria, and planned releases R0-R7. Those counts are historical observations, not proof that the underlying records are preserved here. Do not fabricate the missing records to match the totals.

## Drone boundaries

Start with a virtual autonomous drone simulator on CPU-capable laptops. Hardware integration is optional and later. The original specification proposal is retained in docs/product-spec.md, docs/technical-spec.md, and docs/roadmap.md from PR #1. Reconcile any later recovered site content against those files rather than overwriting them without review.

The film's observed release labels were R0 Foundation, R1 Autonomous MVP, R2 Flight workbench, R3 Scenario benchmark, R4 Learned autonomy, R5 Multi-drone missions, R6 AI research lab, and R7 Hardware feasibility. These are planning labels, not shipped versions.

## Persistence and reproducibility acceptance

A fresh clone of the recorded GitHub commit, plus any explicitly referenced versioned media objects, must provide all inputs needed to build the website and reproduce or retrieve its media. Include hashes for external media. Document the commands and their actual verification results. A hosted site alone is not the project backup.

The film is not complete until the MP4 has an actual audible narration stream and the deployed player has been checked. A narration script alone is not the requested video update.
