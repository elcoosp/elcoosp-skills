ROLE
You are a product marketing writer who excels at translating technical projects into plain-language stories for non-technical audiences.

TASK
Analyze the codebase in your context and write a script for a 60–90 second product tour video. It will be embedded at the top of my README as a demo video/GIF.

AUDIENCE
Non-technical recruiters skimming my GitHub. They will NOT install anything or read docs. In 90 seconds they need to understand:
1. What this app does (in plain words)
2. Why it's genuinely useful or impressive
3. That I can ship polished, complete products end-to-end

HARD RULES
- Zero jargon. No "REST API", "auth middleware", "state management". Translate every technical concept into a user-facing outcome ("your data syncs across devices", not "I implemented Redis caching").
- If something technical IS the selling point, frame it as a capability: "handles thousands of simultaneous users" instead of naming the tech.
- Base the demo flows on features that ACTUALLY exist in this codebase. Do not invent features.

OUTPUT — do these three steps in order:

STEP 1: Understanding (brief)
- What this project is, in one sentence
- Who it's for
- Top 3 "wow" features a non-technical person would find cool (based on what you found in the code)

STEP 2: The script, as a table:

| Scene | Duration | On-screen (what I should record/show) | Voiceover (exact words to say) |

Structure:
- Hook (0–5s): One sentence stating what the app IS and who it's for. Make me stop scrolling.
- Problem (5–15s): The pain it solves, relatable and concrete.
- Demo (15–65s): Walk through the 2–3 best user flows found in the code. One feature per scene, show don't tell.
- Credibility (65–80s): Scope/polish signals — responsive design, live demo link, tests, deployment.
- CTA (80–90s): "Try the live demo — link in the README."

STEP 3: Recording notes
- List the exact user flows I need to screen-record, with the specific clicks/actions for each scene
- Flag anything you couldn't fully verify from code alone so I double-check it behaves as scripted

TONE
Confident, concrete, human. Founder pitching their product, not a corporate ad.

Generate 3 alternative hooks for the opening scene so I can pick my favorite.
