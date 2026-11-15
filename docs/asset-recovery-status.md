# Drone website and video: recovery status

Audit date: November 14, 2026 (America/Chicago).

## Confirmed repository inventory before this recovery commit

The live GitHub branch listing returned only:

- main: 7e2750955f65d458141b1225a30602a8665a87a0. Recursive tree: README.md only.
- docs/autonomous-drone-mvp: 07fee9310b2f20167501e57b4d75a10205c639ff. Recursive tree: README.md, docs/product-spec.md, docs/technical-spec.md, docs/roadmap.md.

The all-state pull-request listing returned PR #1 only; it is open and not merged. Neither branch contained the website source, MP4, captions, narration audio, full backlog/evidence data, or reusable project-package implementation. This does not establish that the separately hosted site or its internal source has been deleted.

## Material preserved in this commit

- Existing PR #1 specification files, unchanged, inherited from its head commit.
- AGENTS.md: repository-first preservation and truthful delivery rules.
- docs/project-package-requirements.md: agreed package requirements recovered from the conversation.
- media/vision/transcript-recovered.md: the seven chapter passages visible in the earlier browser inspection.
- media/vision/storyboard-recovered.json: observed chapter timings and plan references, transcribed from that inspection.

The transcript/storyboard are conversation recoveries, not downloaded copies of the original files. They preserve enough intent to resume work, but do not recover the animation, graphics, video renderer, or audio.

## Still missing / not recovered

- Complete website source, assets, build configuration, and deployment configuration.
- Full structured backlog, all 87 stories and 377 acceptance criteria, and evidence register.
- The later product-design and architecture specifications shown on the website.
- Original 72-second silent MP4 and poster/visual sources.
- Original WebVTT caption file.
- Narration audio and final narrated MP4: not created or published by this recovery step.
- Reusable package/skill implementation and any original installer/test tooling.

## Known original locations

Recorded hosted site: https://drone-lab-review-baller.apeters26.chatgpt.site/

Observed original paths:

- vision-video.html
- project-package.html
- specifications.html
- traceability.html
- releases.html
- release-notes.html
- operations.html
- assets/vision/drone-vision.mp4
- assets/vision/drone-vision-transcript.md
- assets/vision/drone-vision.vtt

Earlier local browser verification used localhost:4173 and reported a 72-second video with one caption track. These are historical observations, not current published-file verification.

Current external fetch attempts for vision-video.html and project-package.html returned HTTP 401 (sign-in required). The connected browser could not be accessed from this session. The original working directory and video are absent from the current working environment. Therefore the original hosted assets remain inaccessible here; they are not confirmed lost.

## Remaining recovery work

1. Access the original authenticated site/editor session or its source export. Recover the complete original source and media without asking the user to supply a previously saved copy.
2. Commit the recovered site sources and structured records on the recovery branch. Record provenance and hashes of recovered assets.
3. Preserve existing IDs. Reconcile plan/spec conflicts; do not invent missing stories or evidence.
4. Add timed narration to the original film, retaining captions and chapter navigation. Validate actual audio streams, non-silence, timing, and playback.
5. Save the final MP4 and its narration/caption/source inputs in Git or versioned Git-linked release/LFS storage. Record SHA-256 hashes and reproducible build instructions.
6. Restore the reusable package from original source, or explicitly identify a replacement as newly reconstructed and test it on a second project.
7. Open/review changes, publish from the recorded commit, verify the deployed video, and record the deployment result. Do not mark these steps done based on this document.
