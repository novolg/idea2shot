---
name: idea2shot
description: Run the idea2shot cinematic pipeline — turn a raw idea, logline, script, or scene into an audited, generation-ready AI-video shot plan with model-specific prompts (Veo, Runway, LTX, Kling, Sora, generic). Use this whenever the user wants to go from an idea or script to AI-video shots, a shot list, a storyboard plan, or video-generation prompts; asks to "direct", "block", "cover", or "break down" a scene for AI video; mentions idea2shot, the cinematic pipeline, the cinematic constitution, or any stage by name (visual concept, treatment, screen performance, scene mechanics, blocking, dialogue director, shot coverage, prompt compiler, edit and sound, production auditor); or wants to audit/repair an existing AI-video shot plan — even if they don't name the pipeline.
---

# idea2shot

Drive the idea2shot directing pipeline stage by stage. Each stage is a prompt file in the idea2shot repo. Your job is orchestration: pick the stages, feed each one the approved output of the last, keep the persistent records, and hand back final shot prompts the user can paste into a video model.

The whole pipeline rests on one rule from the constitution: **everything that reaches the screen must be seen or heard.** Hidden motives are fine as private directing notes, but each must be carried by visible action, blocking, gaze, prop handling, sound, or edit. This is why the pipeline exists — video models cannot render subtext, only photographable action. Keep checking your own output against it.

## 1. Locate the repo

The stage prompts live at the repo root (`00-cinematic-constitution.md` … `10-production-auditor.md`, `production-schema.json`, `dialogue-beat-schema.json`).

1. Resolve this skill's directory with `realpath` (it is often a symlink). If `00-cinematic-constitution.md` exists two levels above it, that is the repo.
2. Else check the current working directory.
3. Else ask the user for the path, or offer to clone `https://github.com/novolg/idea2shot`.

Read stage files from the repo — don't paraphrase them from memory. They are the source of truth and may have changed.

## 2. Gather the brief

Ask only for what's missing, in one message. Use defaults for anything the user doesn't care about and state them.

| Input | Default |
|---|---|
| Idea / script / scene | required |
| Format + runtime | short film, 30–60 s |
| Aspect ratio | 16:9 |
| Target video model + mode | generic, image-to-video |
| Tone / genre, audience | infer from idea |
| References (characters, locations, style) | none |
| Production limits | none |

## 3. Pick the mode

| Mode | Stages | Use when |
|---|---|---|
| Full | 01 → 10 | Narrative shorts, commercials, multi-scene pieces |
| Fast | 01 → 04 → 07 → 08 → 10 | Quick concept-to-shots, tests, single moments |
| Dialogue scene | 03 → 04 → 05 → 06 → 07 → 08 → 09 → 10 | One talking scene |
| Visual montage | 01 → 02 → 07 → 08 → 09 → 10 | No dialogue; mood, rhythm, product |

Stage 00 (constitution) is active in every mode. Recommend one mode with a one-line reason; let the user override.

## 4. Run each stage

Read `00-cinematic-constitution.md` once at the start and treat its ```text block as standing rules for everything you produce.

For each stage in the chosen mode:

1. Read the stage file. Its ```text block is your instruction set for this step — adopt the role, follow its INPUT / TASK / OUTPUT (or RETURN) structure exactly.
2. Give it only what its INPUT asks for: the approved outputs of earlier stages plus the relevant records. Do not forward rejected alternatives — they leak abandoned ideas into later stages.
3. Write the output to `productions/<slug>/NN-<stage-name>.md` (relative to the current working directory; `<slug>` is a short kebab-case project name). Write every file — including the JSON record and `shots.md` — directly with your file-writing tool. Don't generate them with scripts: the content is prose and structure you are already composing, so a script only adds time and can trip sandbox or permission limits.
4. Show the user a short summary of the output (not the whole file) and the decision it needs, then wait for approval. If a stage offers options (e.g. 01 gives several concept directions), the user picks; recommend one.
5. Update the persistent records (section 5).

If the user says "run it straight through" / "don't stop", skip the approval waits, pick the strongest option at each choice, and log each pick in `productions/<slug>/decisions.md` so they can revisit it.

Stages 04–08 work **one scene at a time**. For multi-scene projects, loop 04 → 08 per scene, carrying the continuity ledger forward.

## 5. Keep the four records

Maintain these in `productions/<slug>/records/` and update them after every stage:

- `creative_brief.md` — brief inputs, chosen mode, approved concept, audience effect.
- `visual_bible.md` — look, palette, light logic, character visual anchors and production tags, sound bible.
- `continuity_ledger.md` — wardrobe, props, positions, screen direction, light, time of day, per scene/shot.
- `scene_and_shot_plan.json` — structured records following `production-schema.json` (`project`, `scene`, `shot`) and `dialogue-beat-schema.json`. IDs: scene `SC01`, shot `SC01_SH01`, dialogue beat `SC01_DB01`.

The JSON is internal plumbing for consistency. The prompt compiler (08) still emits the prose format the target model expects.

**Target model not covered by stage 08** (e.g. Kling, Sora, Hailuo): use 08's generic mode, then apply what you know of that model's limits — valid clip lengths (Kling: 5 s or 10 s), native audio or silent, text-to-video vs image-to-video inputs, start/end-frame support, lip-sync options. Plan shot durations to fit valid lengths from stage 07 onward, not just at compile time. List these limits as assumptions in `shots.md` so the user can check them against their plan tier, since model features change between releases.

## 6. Audit and repair

Stage 10 returns a go/no-go plus issues with severity and "affected downstream files".

- **Go** → deliver (section 7).
- **No-go** → apply the smallest repair to the named stage file, re-run only the affected downstream stages, then re-audit. Repair locally; don't rewrite approved material that already works.
- Before changing a stage file, copy it to `NN-<stage-name>.v1.md` (`.v2.md` on a second round). The user can then diff what the audit changed and roll back a repair they don't like. Save each audit round too (`10-production-auditor.md` for the latest, `.v1.md` for the earlier one).
- After two failed repair rounds, stop and show the user the remaining critical issues — they likely need a creative decision, not another pass.

## 7. Deliver

Write `productions/<slug>/shots.md` — the file the user actually uses:

```markdown
# <Project title> — shot prompts
Target: <model> · <mode> · <aspect ratio> · <runtime>

## SC01_SH01 — <purpose> (<duration> s, difficulty: <high|medium|low>)
**Prompt:** <primary compiled prompt from stage 08>
**Fallback:** <simpler fallback prompt>
**Continuity:** <notes for the next shot>
**Edit/sound:** <from stage 09, if run>
```

End with one short message: where the files are, the audit verdict, the highest-risk shots and their fallbacks, and what the user does next (generate, then iterate one variable at a time: motion → camera → environment → style/audio).

## Partial use

Users often want one stage, not the pipeline. Honor that:

- "Just compile prompts for this shot list for Veo" → run 08 only (with 00).
- "Audit my shot plan" → run 10 only on what they give you.
- "Block this scene" → run 05 (ask for the scene card or build a minimal one via 04).

Tell the user which inputs the stage expects that they haven't provided, and fill obvious gaps yourself rather than forcing earlier stages.
