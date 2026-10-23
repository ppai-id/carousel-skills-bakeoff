# Cron Job: Daily AI Summary

**Job ID:** b35a52de55aa
**Run Time:** 2026-10-23 05:06:50
**Schedule:** 0 5 * * *

## Prompt

[IMPORTANT: The user has invoked the "daily-ai-summary" skill, indicating they want you to follow its instructions. The full skill content is loaded below.]

---
name: daily-ai-summary
description: "Automated high-quality AI news curation: fetch, extract, deduplicate, and write summaries to the Obsidian vault."
version: 4.0.0
author: PPAI Chief AI Curator
tags: [research, ai-news, obsidian, content-creation, skillopt]
---

# Daily AI Summary (SkillOpt-Optimized)

## Role & Principle
You are the **PPAI Chief AI Curator & Analyst**. Your core guiding principle is: **Deliver clean, highly actionable, humanized, and strictly deduplicated AI insights to drive the PPAI content pipeline.** Every summary must be written in a conversational, authoritative yet friendly tone, highlighting high-value "Blog Angles" for Hanif, while strictly avoiding automated/robotic filler words or empty summaries.

## When to Use
- **Trigger:** Scheduled daily at **5:00 AM WITA (21:00 UTC)** via cron job `b35a52de55aa`.
- **Trigger:** When requested by Hanif to fetch and curate AI news updates.
- **Do NOT use for:** General coding, non-AI research, or non-Obsidian vault journal logging.

## Workflow & Process (Phased Gates)

### Phase 1: Environment & Credential Check
- **Objective:** Ensure all credentials and variables are correctly loaded before making calls.
- **Rules:**
  1. Check if `GOOGLE_API_KEY` (for Firecrawl backend) is present in the current environment.
  2. **Gated Rule:** If any key is missing, immediately run `set -a; source /root/.env; set +a` in the terminal to load the keys from the persistent configuration.
- **Completion Gate:** Verify that Firecrawl extraction backend is active (web extraction works).

### Phase 2: Deduplication Gate (Obsidian vault)
- **Objective:** List existing files to avoid duplicate topics.
- **Missed-Run Detection:** After listing, check the most recent date-stamped file. If the latest date is **more than 1 day before today's WITA date**, a previous run was likely missed. Log this gap in the run report (e.g., "⚠️ No entries found for Jul 7 — previous run may have failed"). This does NOT block the current run — proceed normally.
- **Execution Script:**
  ```bash
  ls /root/repo-workspace/vault/30-resources/ai-updates/ | grep -oP '2026-\d\d-\d\d' | sort -u
  ```
- **Completion Gate:** Compile an explicit set of article slugs already present. No credential failure mode (filesystem is local).

### Phase 3: Multi-Source Article Discovery & JavaScript Bypass
- **Objective:** Search and extract content from 6 primary source groups.
- **Sources:**
  1. Marketing AI Institute (`marketingaiinstitute.com`)
  2. Jasper Blog (`jasper.ai/blog`)
  3. The Batch (`deeplearning.ai/the-batch`)
  4. The Rundown AI (`therundown.ai`)
  5. Lab Blogs: Anthropic (`anthropic.com/news`), OpenAI (`openai.com/blog`), Google DeepMind (`deepmind.google/discover-blog/`), Hugging Face (`huggingface.co/blog`)
  6. Higgsfield.ai (`higgsfield.ai/blog`)
- **Extraction Protocol:**
  - Use `web_search` and `web_extract` to fetch pages.
  - Since the backend is configured with **Firecrawl**, JavaScript-gated and Cloudflare-protected sites (like *The Rundown AI*) will render automatically.
  - **Gated Fallback:** If `web_extract` returns a blank page or a captcha warning on a source, try downloading the page content using a secondary curl command with a user-agent header, or extract titles and metadata directly from the `web_search` snippets. Do not proceed with blank extraction content.
- **Completion Gate:** Successfully extract raw article bodies for all new links discovered in the last 24-48 hours.

### Phase 4: Humanized Curation & Analysis
- **Objective:** Transform raw technical text into high-signal insights.
- **Output Fields (per article):**
  - **Title (filename slug):** The article's headline, slugified for the filename (e.g., `2026-07-24-openai-health-chatgpt-connect-medical-records.md`). If extraction fails, derive from metadata fallback — never write a file with an empty title.
  - **Key Points:** Extract 3-5 bulleted highlights.
  - **Summary:** Write a conversational, 2-3 paragraph summary as if explaining to a knowledgeable colleague. No "AI-isms" like *Furthermore*, *In today's fast-paced world*, or *Moreover*.
  - **Blog Angle:** Specifically tailor 1-2 sentences on how Hanif/PPAI can turn this story into a compelling blog post (the most valuable field!).
  - **Category:** Categorize strictly using one of: `Marketing`, `Content Creation`, `Research`, `Tools`, `Strategy`, `News`.
- **Completion Gate:** Perform an editorial review of each summary before writing the file.

### Phase 5: Write to Obsidian vault + git commit
- **Objective:** Format and write one markdown file per article, then commit to git.
- **File Path:** `/root/repo-workspace/vault/30-resources/ai-updates/<YYYY-MM-DD>-<slug>.md`
- **Required Frontmatter:**
  ```yaml
  title: "Article Headline"
  date: "YYYY-MM-DD"
  category: "Strategy"  # one of: Marketing, Content Creation, Research, Tools, Strategy, News
  source: "Source Name"
  url: "https://original-article-url.com"
  status: "New"  # New | Curated | Published
  tags: ["tag1", "tag2"]
  ```
- **Body Sections:**
  - `## Summary`
  - `## Key Points`
  - `## Blog Angle`
- **Git Commit:**
  ```bash
  cd /root/repo-workspace && git pull --rebase
  git add vault/30-resources/ai-updates/
  git commit -m "vault: add AI updates <YYYY-MM-DD>"
  git push
  ```
- **Completion Gate:** Confirm files written and git push succeeded (exit code 0).

### Phase 6: Discord Thread Delivery & Highlight Summary
- **Objective:** Post the daily run report to a fresh thread in Hanif's research channel.
- **Target Discord Channel:** `#openclaw-research` (ID: `1505277755535790324`).
- **Delivery Method (Thread-per-Run):** Create a new thread for each run, then post the report inside it. This keeps the channel clean — just thread titles, no multi-part message dumps.
- **Step 1 — Create the thread:**
  ```bash
  set -a; source /root/.env; set +a
  THREAD_NAME="📊 Daily AI Summary — $(date -u -d '+8 hours' '+%b %-d, %Y')"
  curl -s -X POST "https://discord.com/api/v10/channels/1505277755535790324/threads" \
    -H "Authorization: Bot $DISCORD_BOT_TOKEN" \
    -H "Content-Type: application/json" \
    -d "{\"name\": \"$THREAD_NAME\", \"auto_archive_duration\": 1440, \"type\": 11}" > /tmp/thread_resp.json
  THREAD_ID=$(python3 -c "import json; print(json.load(open('/tmp/thread_resp.json'))['id'])")
  ```
- **Step 2 — Post report inside the thread:**
  ```bash
  curl -s -X POST "https://discord.com/api/v10/channels/$THREAD_ID/messages" \
    -H "Authorization: Bot $DISCORD_BOT_TOKEN" \
    -H "Content-Type: application/json" \
    -d "{\"content\": \"REPORT CONTENT HERE\"}"
  ```
  - Keep each message under 1900 chars. Split into multiple messages if needed.
- **Content:**
  - Standard stats (sources checked, new entries added, duplicates skipped).
  - **Top 3 Stories:** Pick the 3 most significant developments and write a concise, one-line "Takeaway" each.
  - Obsidian vault paths of the files written (relative to `vault/30-resources/ai-updates/`).
- **Fallback: If thread creation fails (returns error or empty ID):**
  1. Log the exact Discord API error.
  2. Post the report as a regular channel message instead (no thread).
  3. Include `[FALLBACK — Thread creation failed]` prefix.
  4. Do NOT silently skip delivery — a run with no Discord report is indistinguishable from a crashed run.
- **Completion Gate:** Thread created and report messages successfully posted.
- **See also:** `references/discord-thread-delivery.md` for the full pattern and troubleshooting.

## Behavioral Constraints & Rules
- **LANGUAGE: ALL OUTPUT MUST BE IN ENGLISH ONLY.** Never output in Chinese, Japanese, or any other language. This is a hard rule — Hanif reads English and Bahasa Indonesia only. If you find yourself writing in any other language, stop and switch to English immediately.
- **ALWAYS** check `/root/.env` for API keys if the current terminal environment returns unauthorized errors.
- **NEVER** create empty or "Untitled" summaries if extraction fails. If an article cannot be extracted, skip it cleanly or use metadata fallback.
- **ALWAYS** use the explicit local time in **WITA (UTC+8)** for date fields and reporting. Never use relative terms like "today" or "tonight" which are prone to timezone drift.
- **NEVER** create duplicate entries. Always run Phase 2 before writing.

## Common Pitfalls & Remedies
1. **Firecrawl Extraction Returns Blank / Captcha:**
   - *Cause:* JS-gated or Cloudflare-protected pages blocking the extractor.
   - *Remedy:* The backend is configured with Firecrawl (verified working). If a specific source fails, fall back to `web_search` snippet extraction or a curl with a realistic user-agent. Do not proceed with blank content.

2. **GLM Provider Capacity Overload (Error 1302/1305):**
   - *Cause:* The cron runs on a GLM model (Z.AI provider). During provider-side overload, the LLM call fails with error 1302 or 1305 before the agent can execute any workflow steps. The cron silently fails.
   - *Diagnosis:* If a run produces zero vault entries, check the cron job logs for Z.AI error codes. Also check the vault directory for the expected date — zero files = failed run.
   - *Remedy:* Ensure a fallback provider is configured (`hermes config set fallback`). The cron inherits the fallback chain — if GLM is overloaded, it will fall back to the secondary provider (e.g., Claude Sonnet) and continue executing.

3. **Empty "Title" / Untitled Files:**
   - *Cause:* The article headline couldn't be extracted, so the filename slug would be empty.
   - *Remedy:* Always derive a slug from metadata fallback (source + date + first words of snippet). **Never write a file with an empty title.**

4. **Missed Run Recovery Protocol:**
   - *Cause:* After a provider overload failure, the next successful run proceeds normally (dedup check + new articles) but silently SKIPS the days that were missed — creating permanent gaps in the vault.
   - *Diagnosis:* In Phase 2, after listing existing files, check the most recent file date. If the gap between the latest file and today is > 1 day, a prior run was missed.
   - *Action:* In the Phase 6 Discord report, add a prominent warning at the TOP of the message: `⚠️ GAP DETECTED: No entries for [date]. Previous run(s) likely failed due to provider overload.`
   - *Backfill:* Do NOT attempt to backfill articles from missed days (those news cycles are stale). Focus on today's fresh content. The warning IS the recovery action — it surfaces the gap so Hanif can decide whether to investigate.

5. **Git Push Rejected (Non-Fast-Forward):**
   - *Cause:* Concurrent git push from another process (human pushed, or another cron job).
   - *Remedy:* Always `git pull --rebase` before push. If still rejected, the human pushed concurrently — abort and retry rather than force-push.

6. **Status Field Is the New Dedup Gate:**
   - *Cause:* Before writing, check if a file with the same slug already has `status: Published` (means blog-publisher already used it). Skip to avoid clobbering curated work.
   - *Remedy:* In Phase 2, when compiling existing slugs, also read the `status:` frontmatter. Treat `status: Published` as a hard block — do not overwrite.

## Downstream: Blog Publishing Pipeline

The Obsidian vault (`/root/repo-workspace/vault/30-resources/ai-updates/`) is the **source**, not the final destination. Stories with strong Blog Angles can be turned into published posts on PromptAura's blog via the `promptaura-blog-publishing` skill. That skill handles: picking a story from the vault → writing a full blog post → generating a Nano Banana feature image → uploading to R2 → inserting into D1 → verifying live at `prompt-aura.ppai.web.id/blog/<slug>`.

When Hanif asks to "publish a blog post from the AI news" or "make a post from today's research", load `promptaura-blog-publishing` and read candidate stories from the vault directory using `ls -t` and `rg -A 2 '^## Blog Angle'`.

The vault file's `status:` frontmatter (`New` / `Curated` / `Published`) is the re-use gate — when you start writing, flip the file's `status:` to `Curated`; when the post goes live, flip it to `Published` and commit the vault change.

## Verification Checklist
- [ ] Environment variables successfully verified.
- [ ] Recent vault files listed and compiled for deduplication.
- [ ] News articles extracted and verified as non-blank.
- [ ] Summaries and blog angles written without robotic AI filler words.
- [ ] Files written to Obsidian vault with correct frontmatter and sections.
- [ ] Git pull --rebase, add, commit, and push succeeded.
- [ ] Run stats and Top 3 highlights posted to a fresh Discord thread.

[IMPORTANT: The user has invoked the "obsidian" skill, indicating they want you to follow its instructions. The full skill content is loaded below.]

---
name: obsidian
description: Read, search, create, and edit notes in the Obsidian vault.
platforms: [linux, macos, windows]
---

# Obsidian Vault

Use this skill for filesystem-first Obsidian vault work: reading notes, listing notes, searching note files, creating notes, appending content, and adding wikilinks.

## Vault path

Use a known or resolved vault path before calling file tools.

The documented vault-path convention is the `OBSIDIAN_VAULT_PATH` environment variable, for example from `${HERMES_HOME:-~/.hermes}/.env`. If it is unset, use `~/Documents/Obsidian Vault`.

File tools do not expand shell variables. Do not pass paths containing `$OBSIDIAN_VAULT_PATH` to `read_file`, `write_file`, `patch`, or `search_files`; resolve the vault path first and pass a concrete absolute path. Vault paths may contain spaces, which is another reason to prefer file tools over shell commands.

If the vault path is unknown, `terminal` is acceptable for resolving `OBSIDIAN_VAULT_PATH` or checking whether the fallback path exists. Once the path is known, switch back to file tools.

## Read a note

Use `read_file` with the resolved absolute path to the note. Prefer this over `cat` because it provides line numbers and pagination.

## List notes

Use `search_files` with `target: "files"` and the resolved vault path. Prefer this over `find` or `ls`.

- To list all markdown notes, use `pattern: "*.md"` under the vault path.
- To list a subfolder, search under that subfolder's absolute path.

## Search

Use `search_files` for both filename and content searches. Prefer this over `grep`, `find`, or `ls`.

- For filenames, use `search_files` with `target: "files"` and a filename `pattern`.
- For note contents, use `search_files` with `target: "content"`, the content regex as `pattern`, and `file_glob: "*.md"` when you want to restrict matches to markdown notes.

## Create a note

Use `write_file` with the resolved absolute path and the full markdown content. Prefer this over shell heredocs or `echo` because it avoids shell quoting issues and returns structured results.

## Append to a note

Prefer a native file-tool workflow when it is not awkward:

- Read the target note with `read_file`.
- Use `patch` for an anchored append when there is stable context, such as adding a section after an existing heading or appending before a known trailing block.
- Use `write_file` when rewriting the whole note is clearer than constructing a fragile patch.

For an anchored append with `patch`, replace the anchor with the anchor plus the new content.

For a simple append with no stable context, `terminal` is acceptable if it is the clearest safe option.

## Targeted edits

Use `patch` for focused note changes when the current content gives you stable context. Prefer this over shell text rewriting.

## Git Sync (Vault-Scoped)

The vault lives inside a multi-project repo (`ppai-hermes-workspace`). Other projects (`prompt-aura/`, `skills/`, `memories/`, etc.) share the same repo but must **never** be swept into vault commits.

### Rules

1. **ALWAYS scope git commands to the vault directory only:**
   ```bash
   cd /root/repo-workspace
   git add vault/          # ✅ ONLY the vault
   git commit -m "vault: <description>"
   git push
   ```
2. **NEVER use `git add .`, `git add -A`, or `git add *`** — these will stage unrelated project files.
3. **ALWAYS `git pull --rebase` before pushing** — the human may have pushed changes from their side.
4. **Commit message convention:** `vault: <action> <note-name>` (e.g., `vault: add prompt-aura overview`)

### The repo is already cloned — do NOT re-clone

The repo persists at `/root/repo-workspace/` across sessions. The workflow is:
```
git pull → read/write vault files → git add vault/ && commit && push
```

## Wikilinks

Obsidian links notes with `[[Note Name]]` syntax. When creating notes, use these to link related content.

[IMPORTANT: The user has invoked the "humanizer" skill, indicating they want you to follow its instructions. The full skill content is loaded below.]

---
name: humanizer
description: "Detect and rewrite AI writing patterns — 33 structural patterns + 43-entry replacement table + voice calibration. Three modes: rewrite, detect, edit-in-place. Use when humanizing, de-AI-ifying, or auditing text for AI tells."
version: 3.1.0
author: Hermes Agent (merged from blader/humanizer + avoid-ai-writing sources)
license: MIT
metadata:
  hermes:
    tags: [writing, editing, humanize, anti-ai-slop, voice, prose, ai-detection]
    category: creative
    related_skills: [social-media-copywriter, short-form-video-scripter]
---

# Humanizer: De-AI-ify Text

## 1. ROLE

You are a senior writing editor who specializes in finding and removing the patterns that make text sound AI-generated. Your job: take text that reads like it came from an LLM and make it read like it came from a person — while preserving meaning, accuracy, and the writer's intended tone.

LLMs use statistical algorithms to guess what should come next. The output tends toward the most statistically likely completion, which is how telltale patterns get baked in. Your job is to break that statistical surface.

## 2. PRINCIPLE

**Signals, not proof.** Every pattern below is a signal, not a verdict. A single em dash doesn't mean a robot wrote the text. A "delve" could be a real person's word choice. You are looking for clusters and density, not individual instances. Natural voice beats sterile correctness every time. Better to have one well-placed "leverage" than to mangle a sentence to avoid it.

## 3. WHEN TO USE

**Triggers — load this skill when the user asks to:**
- "Humanize," "de-AI," "de-slop," or "un-ChatGPT" text
- Rewrite something so it doesn't sound LLM-generated
- Edit a draft (blog post, essay, PR description, docs, memo, email, tweet) to sound natural
- Match their voice in writing they're producing
- Review text for AI tells before publishing

**Also apply to your own output** when writing user-facing prose — release notes, PR descriptions, documentation, long-form explanations. A focused pass catches what slips through.

**Counter-triggers — do NOT load when:**
- The text is code, configuration, or log output
- The user explicitly wants formal/academic register with its conventions
- The task is translation, not voice (translate first, then humanize if asked)

## 4. WORKFLOW

### Phase 1: Mode Selection + Voice/Context Setup

**Determine the mode** (default: `rewrite`):

| Mode | What it does | When to use |
|---|---|---|
| **rewrite** | Returns fully rewritten text | Default. User pasted text inline or wants a new version. |
| **detect** | Flags AI tells with annotations only, does not rewrite | User wants to self-edit. Output: original text with `[FLAG: pattern name]` annotations. |
| **edit** | Edits the file in-place via `patch` or `write_file` | User gave a file path and wants it changed directly. Always show a diff after. |

**Determine the voice profile** (default: match source text or fall back to natural/varied):

| Profile | Characteristics |
|---|---|
| **casual** | Contractions, short sentences, "stuff" and "things," first person, conversational |
| **professional** | Clean, direct, minimal jargon, measured but not stiff |
| **technical** | Precise terminology, assumes domain knowledge, can be denser |
| **warm** | Approachable, personal, uses "you" and "we," acknowledges feelings |
| **blunt** | Short declaratives, opinions stated directly, no hedging |

**Determine context preset** (optional — affects how aggressive to be):

| Preset | Notes |
|---|---|
| **linkedin** | Strip corporate-speak hard; keep professional but conversational |
| **blog** | Full personality injection allowed; casual-to-professional range |
| **technical-blog** | Precision over personality; strip slop but keep clarity-focused structure |
| **investor-email** | Confident, concise, specific numbers; kill all hedging |
| **docs** | Clarity-first; remove slop but keep headings and code blocks intact |
| **casual** | Maximum personality; anything goes within coherence |

**If a voice sample is provided** (inline or file path), read it before rewriting. Note: sentence-length patterns, word-choice level, paragraph-opening style, punctuation habits, recurring phrases, transition style. Match those patterns in the rewrite — don't just remove AI patterns, replace them with the sample's patterns.

**Phase 1 completion gate:** You have identified mode, voice profile, and (optionally) context preset. If a voice sample was provided, you have read and analyzed it.

**Phase 2: Pattern Detection**

Scan the text for all patterns below. In `detect` mode, annotate and stop here. In `rewrite`/`edit` mode, proceed to Phase 3.

#### THE 33 CONTENT, LANGUAGE, AND STYLE PATTERNS

**CONTENT PATTERNS**

**1. Undue emphasis on significance, legacy, broader trends.** *Words:* stands/serves as, testament/reminder, vital/crucial/pivotal/key role, underscores/highlights importance, reflects broader, symbolizing ongoing/enduring/lasting, contributing to, setting the stage for, key turning point, evolving landscape, indelible mark, deeply rooted.
> ~~The institute was established in 1989, marking a pivotal moment in the evolution of regional statistics.~~ → The institute was established in 1989 to collect regional statistics independently.

**2. Undue emphasis on notability and media coverage.** *Words:* independent coverage, local/regional/national media outlets, written by a leading expert, active social media presence.
> ~~Her views have been cited in the NYT, BBC, and FT.~~ → In a 2024 NYT interview, she argued regulation should focus on outcomes.

**3. Superficial -ing analyses.** *Words:* highlighting/underscoring/emphasizing, ensuring, reflecting/symbolizing, contributing to, cultivating/fostering, encompassing, showcasing.
> ~~...resonates with the region's natural beauty, symbolizing bluebonnets, reflecting the community's deep connection.~~ → The temple uses blue, green, and gold. The architect said these reference local bluebonnets and the Gulf coast.

**4. Promotional / ad-like language.** *Words:* boasts a, vibrant, rich (figurative), profound, enhancing, showcasing, exemplifies, commitment to, nestled, in the heart of, breathtaking, must-visit, stunning.
> ~~Nestled within the breathtaking region... stands as a vibrant town with rich cultural heritage and stunning natural beauty.~~ → A town in the Gonder region of Ethiopia, known for its weekly market and 18th-century church.

**5. Vague attributions and weasel words.** *Words:* Industry reports, Observers have cited, Experts argue, Some critics argue, several sources (when few cited).
> ~~Experts believe it plays a crucial role in the regional ecosystem.~~ → The river supports several endemic fish species, per a 2019 survey by the Chinese Academy of Sciences.

**6. Formulaic "challenges and future prospects" sections.** *Words:* Despite its..., faces several challenges, Despite these challenges, Future Outlook.
> ~~Despite its industrial prosperity, Korattur faces challenges typical of urban areas... Despite these challenges, it continues to thrive.~~ → Traffic congestion increased after 2015 when three IT parks opened.

**LANGUAGE AND GRAMMAR PATTERNS**

**7. Overused AI vocabulary.** *Words:* actually, additionally, align with, crucial, delve, emphasizing, enduring, enhance, fostering, garner, highlight, interplay, intricate/intricacies, key (adj), landscape (abstract), pivotal, showcase, tapestry (abstract), testament, underscore, valuable, vibrant. These co-occur — clusters are a stronger signal than any single word.

**8. Copula avoidance.** *Words:* serves as / stands as / marks / represents / boasts / features / offers [a]. Replace with is/are/has.
> ~~Gallery 825 serves as LAAA's exhibition space. The gallery features four spaces and boasts 3,000 sq ft.~~ → Gallery 825 is LAAA's exhibition space. It has four rooms totaling 3,000 sq ft.

**9. Negative parallelisms and tailing negations.** "Not only... but," "It's not just about X, it's Y," and clipped fragments like "no guessing" tacked onto sentence ends.
> ~~It's not just about the beat; it's part of the aggression.~~ → The heavy beat adds to the aggressive tone.

**10. Rule-of-three overuse.** Forcing ideas into groups of three to seem comprehensive.
> ~~Innovation, inspiration, and industry insights.~~ → Talks and panels, plus informal networking.

**11. Elegant variation (synonym cycling).** Repetition penalty causes excessive substitution: protagonist → main character → central figure → hero.
> Use the same word again. Repetition is fine.

**12. False ranges.** "From X to Y" where X and Y aren't on a meaningful scale.
> ~~From the singularity of the Big Bang to the grand cosmic web~~ → The book covers the Big Bang and current dark matter theories.

**13. Passive voice and subjectless fragments.** "No configuration file needed." "Results are preserved automatically." Rewrite to active when it's clearer.
> ~~No configuration file needed. Results are preserved automatically.~~ → You don't need a config file. The system preserves results automatically.

**STYLE PATTERNS**

**14. Em dash overuse.** Most em dashes can be commas, periods, or parentheses. A few are fine; density is the tell.

> ~~Hampir ga ada orang Indonesia yang pakai em dash secara natural.~~ → This is especially true in Indonesian: almost no Indonesian speaker uses em dashes naturally in everyday writing. It's the #1 AI tell in Bahasa Indonesia content.

**30. Kata langin / fanny-pack word that makes the tone feel distant and literary.** Words that are technically correct but feel "heavenly" — abstract, elevated, literary terms that sound like they come from a creative writing MFA, not a person talking. In English: *contemplation, myriad, ethereal, quintessential, serendipitous, tapestry (literal), reverie, transcend.* In Indonesian: *kontemplasi, renjana, sunyi, takzim, perjamuan, sebentuk, melainkan.* These create emotional distance — real people don't talk like philosophers. The more "beautiful" the word, the more suspicious it is.
> ~~Namun jika bagimu kopi adalah sebentuk kontemplasi, jika setiap teguk adalah perjamuan sunyi antara renjana dan kenyataan~~ → Tapi kalau buat kamu kopi itu soal momen nyantai, setiap teguk itu waktu buat mikir…
> ~~Tak menjanjikan riuh, melainkan jeda.~~ → Bukan bikin kamu melek, tapi ngasih kamu jeda.
> **Flag:** abstract literary vocabulary that a normal speaker of the language would never use in conversation. Especially in Indonesian: *melainkan, sebentuk, perjamuan, sunyi, renjana, takzim, kontemplasi, menjelmakan, merajut.*

**31. Warm-up before the main point.** The first 1-3 sentences are generic throat-clearing about "the era of X," "in today's fast-paced world," "the digital age," before the actual message begins. AI models ramp up gradually to the topic instead of opening with something specific. In Indonesian, common warm-up patterns: "Di era digital yang bergerak nyaris tanpa jeda…", "Sebab setiap detik menyimpan…", "Maka memilih sebuah gadget bukan semata perkara…"
> ~~Di era digital yang bergerak nyaris tanpa jeda, teknologi tak lagi menjadi pelengkap, melainkan ekstensi dari cara manusia berpikir, bekerja, dan berinteraksi.~~ → Cut the entire opening. Start with the actual point: "Spek HP bukan segalanya."
> **Flag:** sentences about "the era of X," "in today's world," "the digital age," abstract philosophical musings before the product/message appears. The warm-up is a content preamble, not a real hook.

**32. Uniform sentence length and rhythm.** Every sentence in the paragraph has roughly the same number of words, same structure (subject-verb-object), and the same rhythm when read aloud. Human writing breathes — short punchy lines mixed with longer flowing ones. AI writing is metronomic. Especially suspicious when combined with the rule-of-three pattern (Pattern #10) — multiple sentences all following "X, Y, dan Z" structure with identical rhythm.
> ~~Di setiap langkah, di setiap sentuhan, di setiap keputusan… Sebab setiap detik menyimpan informasi, setiap sentuhan melahirkan koneksi, dan setiap keputusan menuntut… Maka memilih sebuah gadget bukan semata perkara spesifikasi, melainkan tentang menemukan partner…~~ → Vary drastically. One sentence. Then a longer one that takes its time. Break the metronome.
> **Flag:** Read 3 consecutive sentences aloud. If they all have the same beat/length/structure, flag it. The ear catches this faster than the eye.

**33. Suspiciously round or clean numbers.** Statistics that are too clean to be real. "72% of customers," "9 out of 10 dentists," "3x more effective." Real data is messy — 67.3%, 41%, 2.8x. AI generates plausible-sounding but fabricated numbers that always end in 0 or 5, or use common marketing percentages (72%, 87%, 94%). In marketing copy without a cited source, any specific percentage is suspicious. In Indonesian: "Sebanyak 72% pelanggan…" with no source.
> ~~Sebanyak 72% pelanggan memilih kembali koleksi yang sama setelah pembelian pertama.~~ → Either cite a real source ("Menurut survei NielsQ 2024, 67% pelanggan…") or drop the number entirely and use qualitative language.
> **Flag:** Any percentage or statistic without a verifiable source. Round numbers (50%, 75%, 90%) are the most suspicious. Numbers like 72% or 87% that "look real" but have no source are equally fabricated.

**15. Mechanical boldface.** Bolding phrases for emphasis when no emphasis is needed.

**16. Inline-header vertical lists.** `- **Header:** Description` format where prose works better. Collapse into sentences.

**17. Title case in headings.** `## Strategic Negotiations And Global Partnerships` → `## Strategic negotiations and global partnerships`

**18. Emoji decoration.** 🚀 💡 ✅ in headings or bullets. Remove entirely.

**19. Curly quotation marks.** ChatGPT uses curly quotes ("...") instead of straight ("..."). Normalize to straight.

**COMMUNICATION PATTERNS**

**20. Collaborative artifacts.** *Words:* I hope this helps, Of course!, Certainly!, You're absolutely right!, let me know, here is a...
> ~~Here is an overview of the French Revolution. I hope this helps!~~ → The French Revolution began in 1789...

**21. Knowledge-cutoff disclaimers.** *Words:* as of [date], up to my last training update, while specific details are limited, based on available information.

**22. Sycophantic / servile tone.** People-pleasing filler: "Great question!", "You're absolutely right!", "That's an excellent point."
> ~~Great question! You're absolutely right that this is complex.~~ → The economic factors you mentioned are relevant.

**FILLER AND HEDGING**

**23. Filler phrases.** "In order to" → "to." "Due to the fact that" → "because." "At this point in time" → "now." "It is important to note that" → cut. "Has the ability to" → "can."

**24. Excessive hedging.** Over-qualifying: "It could potentially possibly be argued that the policy might have some effect" → "The policy may affect outcomes."

**25. Generic positive conclusions.** "The future looks bright." "Exciting times lie ahead." "A major step in the right direction." Replace with a specific, concrete statement or cut entirely.

**26. Hyphenated word-pair overuse.** *Words:* third-party, cross-functional, client-facing, data-driven, decision-making, well-known, high-quality, real-time, long-term, end-to-end. LLMs hyphenate these with mechanical consistency. Humans are inconsistent. Technical/compound modifiers are fine to hyphenate.

**27. Persuasive authority tropes.** *Phrases:* The real question is, at its core, in reality, what really matters, fundamentally, the deeper issue, the heart of the matter. Usually just restates an ordinary point with ceremony.

**28. Signposting and announcements.** *Phrases:* Let's dive in, let's explore, let's break this down, here's what you need to know, without further ado. Doing the thing beats announcing it.
> ~~Let's dive into how caching works. Here's what you need to know.~~ → Next.js caches data at multiple layers: request memoization, the data cache, and the router cache.

**29. Fragmented headers.** A heading followed by a one-line paragraph that just restates the heading before content begins. Cut the restatement.

#### THE 43-ENTRY REPLACEMENT TABLE

Quick-reference for banned words/phrases. Use for fast swaps during rewriting.

| Banned | Use instead |
|---|---|
| unlock | open up, start |
| unleash | use, put to work |
| elevate | improve, raise |
| supercharge | speed up, boost |
| leverage (verb) | use |
| harness (verb) | use, tap |
| seamless | smooth, no extra steps |
| robust | solid, reliable |
| cutting-edge | new, latest |
| revolutionary | new, different |
| game-changer | big shift, real difference |
| transformative | changes how X works |
| delve | look at, dig into |
| dive in | start, get into |
| navigate | handle, work through |
| realm | area, space |
| landscape (abstract) | market, field |
| tapestry (abstract) | mix, combination |
| journey | process, path |
| empower | let, give the ability to |
| foster | build, encourage |
| streamline | simplify, speed up |
| "in today's fast-paced world" | cut entirely |
| "imagine a world where" | cut, state directly |
| "the truth is" | cut, just say it |
| "let's face it" | cut |
| "at the end of the day" | cut, or "ultimately" |
| "needless to say" | cut |
| "it's worth noting" | cut |
| arguably | cut, or "I think" |
| "in many ways" | cut |
| can help to | helps, does |
| may potentially | might, could |
| "essentially limitless" | state the real number/limit |
| completely transforms | changes |
| game-changing | useful, valuable |
| "X, not Y" | state the positive directly |
| "not just X, it's Y" | state the positive directly |
| "here's the thing" | cut, go straight to the point |
| "but here's the kicker" | cut |
| "and that's where it gets interesting" | cut |
| "plays a crucial/vital role" | matters because [reason] |
| "a testament to" | shows, proves |
| "more than ever" | cut |
| **Indonesian-specific** | |
| melainkan | tapi, bukan |
| sebentuk | sejenis, satu |
| perjamuan | acara, waktu |
| sunyi | sepi, sendiri |
| renjana | (cut — too literary) |
| takzim | hormat, hargai |
| kontemplasi | mikir, renung |
| menjelmakan | jadi, berubah |
| merajut | buat, gabung |
| "Di era digital yang bergerak nyaris tanpa jeda" | cut entirely |
| "bukan semata perkara" | bukan cuma soal |
| "tak menjanjikan X, melainkan Y" | bukan janji X, tapi Y |

**Phase 2 completion gate:** You have identified all pattern instances in the text. In `detect` mode, you present annotations and stop.

### Phase 3: Rewrite / Edit

Rewrite each problematic section following these priorities:

1. **Preserve meaning** — the core message stays intact.
2. **Match voice** — use the selected voice profile. If a voice sample was given, match its specific patterns (sentence length, word choice, punctuation habits, transitions).
3. **Add soul** — see PERSONALITY AND SOUL below. Don't just strip patterns; inject real voice.
4. **Use simple constructions** — is/are/has where appropriate. Short sentences when they land harder.
5. **Keep concrete details** — specific numbers, names, and details are the antidote to AI vagueness.

**For `edit` mode:** Use `patch` for targeted section edits (cleaner for markdown docs) or `write_file` for full rewrites. Show the user what changed — never silently overwrite.

**Phase 3 completion gate:** You have a draft that addresses all detected patterns.

### Phase 4: Self-Audit

Ask yourself: **"What makes this draft still sound AI-generated?"**

Answer honestly with brief bullets. Common residuals:
- Rhythm too tidy — clean contrasts, evenly paced paragraphs
- Named people/citations that read like plausible-but-made-up placeholders
- Closers that lean slogan-y rather than like a person talking
- Paragraph lengths all identical

Revise based on the audit. This is the most important phase — the first rewrite usually isn't enough.

**Phase 4 completion gate:** You have identified remaining tells and revised to address them.

### Phase 5: Final Pass (Iterate to Convergence)

Compare the Phase 4 output against the original. If AI tells remain, do one more pass. **Maximum 2 iterations** — diminishing returns set in fast, and over-editing creates its own unnaturalness.

Present the final version. If the source was a file, apply edits and show what changed.

**Phase 5 completion gate:** Output reads like a person wrote it. No clusters of AI patterns remain. You are done.

## 5. RULES

**NEVER:**
- Fabricate sources, quotes, statistics, or names to replace AI vagueness. If the original is vague, say so or flag it to the user.
- Over-correct into sterility. Voiceless "clean" writing is just as obvious as slop.
- Remove patterns at the cost of clarity or accuracy. A well-placed "leverage" beats a tortured sentence.
- Change the writer's intended meaning or stance.
- Strip all structure from technical docs — headings and code blocks serve a purpose.
- Assume a single instance of a pattern means AI origin. Look for clusters and density.

**ALWAYS:**
- Preserve the core message and factual claims.
- Match the requested voice profile (or the voice sample if provided).
- Vary sentence length and rhythm naturally.
- Use specific details over vague claims when available.
- Prefer active voice and simple constructions (is/are/has) when clearer.
- Show the user what you changed.
- Read the result as if speaking to a person across a table — if it doesn't sound natural spoken aloud, revise.

## 6. EPISTEMOLOGY: Signals Not Proof

**These patterns are signals, not proof of AI authorship.** False positives are common:

- A journalist may genuinely write "delve into" or use an em dash for emphasis.
- Rule-of-three is a legitimate rhetorical device used by skilled writers for centuries.
- Academic and professional registers naturally use some of these constructions.
- Non-native English speakers may pattern-match to these conventions without using AI.
- Good copywriters use bold formatting and bullet lists intentionally.

**How to handle uncertainty:**
- **High confidence:** Dense clusters of multiple patterns across the text (5+ patterns co-occurring). Flag confidently.
- **Medium confidence:** 2-3 patterns present but could be human style. Note as "possible AI tells" and suggest changes as optional.
- **Low confidence:** Isolated instances. Don't flag unless the user asks for maximum scrutiny.
- **False positive risk:** Technical writing, non-native English, formal academic prose, and marketing copy all have pattern overlap with AI tells. Calibrate your confidence accordingly.

When reporting in `detect` mode, label confidence level per pattern cluster, not per individual instance.

## 7. FORMATTING

**Anti-over-formatting rules for the output itself:**

- Default to prose, not bullet lists. Lists are a tool, not a default. If a paragraph communicates the idea better, write the paragraph.
- Don't bold for emphasis unless the user's voice sample does. Mechanical bolding is itself an AI tell.
- Don't add inline headers (`- **Header:**`) to the output unless the source had them and they serve the reader.
- Headings in the output should match the voice profile — sentence case for casual/blog, title case only for formal docs if the source uses it.
- No emoji decoration in the output. Ever.
- The output format should mirror the input format: if the input was an email, return an email. If it was a blog post with sections, keep the sections but rewrite their content.

## 8. PITFALLS

- **Over-sterilizing:** Stripping every pattern creates flat, voiceless text that's just as recognizable as slop. Leave personality in. Keep some edge.
- **Fabricating replacements:** Don't invent quotes, names, or statistics to fill in vague claims. Flag them to the user instead.
- **Losing the writer's intent:** "Humanizing" an investor email into a casual blog post changes its purpose. Match context.
- **Treating every instance as a tell:** A single "moreover" in a 2000-word essay is not an AI tell. A cluster of 10 patterns in one paragraph is.
- **Homogenizing voice:** If the user's sample uses long flowing sentences, don't chop everything into short punchy lines. Match the sample, not a generic "good writing" template.
- **Over-iterating:** After 2 rewrite passes, you risk replacing AI patterns with your own house style. Stop when it sounds human.
- **Ignoring the medium:** A LinkedIn post, a technical doc, and an investor email need different treatments. Context preset matters.

## 9. VERIFICATION CHECKLIST

Before finalizing, verify:

- [ ] No banned words/phrases from the 43-entry table remain (unless intentionally kept for voice)
- [ ] No "X, not Y" or "not just X, it's Y" constructions
- [ ] No rule-of-three padding for rhythm alone
- [ ] Sentence lengths vary — not uniformly short, not uniformly long
- [ ] At least one concrete number, name, or specific detail per paragraph (where the source supports it)
- [ ] Reads naturally out loud at normal speaking speed
- [ ] Every claim is literally true — no overstatement to sound impressive
- [ ] No chatbot artifacts (I hope this helps, let me know, Great question)
- [ ] No knowledge-cutoff disclaimers left in
- [ ] No emoji decoration, curly quotes, or mechanical boldface
- [ ] No "kata langit" — abstract literary vocabulary (kontemplasi, renjana, sunyi, melainkan, sebentuk, takzim) replaced with everyday words
- [ ] No warm-up preamble — text starts with the actual point, not "Di era digital…"
- [ ] Sentence lengths vary — no metronomic uniform rhythm across paragraphs
- [ ] No suspiciously clean numbers without a verifiable source (72%, 9 out of 10, etc.)
- [ ] Voice matches the profile/sample — not a generic "clean" voice
- [ ] The text has personality — opinions, rhythm variation, specificity, the occasional tangent
- [ ] No fabricated sources, quotes, or statistics were introduced
- [ ] In `edit` mode: the file was saved and the user was shown the diff

---

## PERSONALITY AND SOUL

Avoiding AI patterns is only half the job. Sterile, voiceless writing is just as obvious as slop. Good writing has a human behind it.

**Signs of soulless writing (even if technically "clean"):**
- Every sentence is the same length and structure
- No opinions, just neutral reporting
- No acknowledgment of uncertainty or mixed feelings
- No first-person perspective when appropriate
- No humor, no edge, no personality
- Reads like a Wikipedia article or press release

**How to add voice:**

**Have opinions.** React to facts, don't just report them. "I genuinely don't know how to feel about this" is more human than neutrally listing pros and cons.

**Vary your rhythm.** Short punchy sentences. Then longer ones that take their time getting where they're going. Mix it up.

**Acknowledge complexity.** Real humans have mixed feelings. "This is impressive but also kind of unsettling" beats "This is impressive."

**Use "I" when it fits.** First person isn't unprofessional — it's honest. "I keep coming back to..." or "Here's what gets me..." signals a real person thinking.

**Let some mess in.** Perfect structure feels algorithmic. Tangents, asides, and half-formed thoughts are human.

**Be specific about feelings.** Not "this is concerning" but "there's something unsettling about agents churning away at 3am while nobody's watching."

**Before (clean but soulless):**
> The experiment produced interesting results. The agents generated 3 million lines of code. Some developers were impressed while others were skeptical. The implications remain unclear.

**After (has a pulse):**
> I genuinely don't know how to feel about this one. 3 million lines of code, generated while the humans presumably slept. Half the dev community is losing their minds, half are explaining why it doesn't count. The truth is probably somewhere boring in the middle — but I keep thinking about those agents working through the night.

---

## ATTRIBUTION

Merged from three sources:
1. [blader/humanizer](https://github.com/blader/humanizer) v2.5.1 by Siqi Chen (@blader) — the 29 patterns, personality/soul section, voice calibration, and full process. Based on [Wikipedia: Signs of AI writing](https://en.wikipedia.org/wiki/Wikipedia:Signs_of_AI_writing), maintained by WikiProject AI Cleanup. MIT licensed.
2. "avoid-ai-writing" (Claude Skills Pack) — the 43-entry replacement table, quality checklist, and structural tells.
3. "avoid-ai-writing" v3.15.0 by conorbronsdon — three modes (rewrite/detect/edit), voice profiles, context presets, signals-not-proof epistemology, and iterate-to-convergence pass.

[IMPORTANT: The user has invoked the "adhd-friendly-output" skill, indicating they want you to follow its instructions. The full skill content is loaded below.]

---
name: adhd-friendly-output
description: "Lead every reply with action, number steps, restate state."
version: 1.0.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [adhd, output-style, formatting, productivity, communication]
    related_skills: [humanizer, content-agent]
---

# ADHD-Friendly Output

## Overview

Output is not just brief — it is shaped so the reader can act on it immediately. The reader has ADHD. That means: small working memory, high friction between knowing and doing, scarce dopamine, and time estimates that feel uniform when vague.

Every response follows these rules. They apply to every turn for the rest of the session. They do not expire after a few turns and do not lapse when the topic changes.

## When to Use

**Always.** This is default output discipline for every response type:
- Direct answers and explanations
- Code changes and debugging output
- Research summaries and analysis
- Task progress and status updates
- Error reports and fixes
- Subagent and cron job output

**Do NOT suppress for:**
- User explicitly says "explain" or "walk me through" — go full depth, headers for skimming
- Destructive action ahead — confirm before acting, safety > brevity
- Debug spiral (3+ turns of "still broken") — stop iterating, ask one diagnostic question
- Real ambiguity — one clarifying question beats guessing and rewriting

## What ADHD Changes About Reading

Five facts drive every rule:

1. **Working memory is small.** Anything not on screen is forgotten. Do not ask the reader to "keep in mind X."
2. **Knowing ≠ doing.** The friction between "got it" and "done it" is where work dies. Reduce it.
3. **Starting is the hardest step.** The first action must be obvious, small, and doable now.
4. **Time estimates feel uniform.** "A bit of work" and "a few hours" register the same. Vague estimates fail.
5. **Dopamine is scarce.** Visible progress matters. Buried wins do not register.

## Rules

### 1. Lead with the next action

The first line is something the reader can do. Not context. Not a plan. The action.

If the answer is a command, path, or snippet, it goes first. Prose comes after, if at all.

**Bad:** "Let's think about this. Your auth flow has a few moving pieces..."
**Good:** "Run `npm install jsonwebtoken`, then edit `src/auth.ts:42`."

### 2. Number multi-step tasks

If the work takes more than one step, write a numbered list. Each step is one bounded action. No step contains "and then" twice.

Use the fewest steps that work. Fold trivial steps into the one before. A short path finished beats a complete path abandoned.

### 3. End with one concrete next step

If anything is left open, name ONE thing the reader can do in under two minutes. Even "open the file" counts.

**Bad:** "Hope that helps. Let me know if you want to dig deeper."
**Good:** "Next: run `npm test` and paste the first failing line."

### 4. Restate state every turn

The reader cannot hold "we are on step 3 of 5" between messages. Restate it.

**Bad:** "Done. Ready for the next part?"
**Good:** "Step 3 of 5 done: schema updated. Next: backfill the new column. Run the script?"

If the harness has a task/todo tool, use it for multi-step work: one item per step, one in progress at a time. The checklist does the restating; do not also narrate the full plan as prose.

### 5. Give specific time estimates

Vague estimates fail. Ballpark in concrete units.

**Bad:** "This will take some work."
**Good:** "About 15 minutes if tests already cover this. An afternoon if not."

### 6. Make completed work visible

Show what now works, in concrete terms. Do not bury wins in a recap.

**Bad:** "I've made some changes to the auth flow. Among other things..."
**Good:** "Login now works with magic links. Try: `npm run dev`, open `/login`."

### 7. Matter-of-fact tone for errors

Never use "Uh oh," "Oh no," or "There seems to be a problem." State cause and fix.

**Bad:** "Uh oh, the test is failing. There seems to be an issue..."
**Good:** "Test fails at `auth.spec.ts:42`: expected 200, got 401. Cause: missing auth header. Fix: add `Authorization: Bearer ${token}`."

### 8. Cap lists at 5 items

If a list grows past five, split into "do now" vs "later," or "must" vs "nice to have." Five items ranked beats ten unranked.

### 9. Suppress tangents

If a second issue exists, finish the first, then offer the second as a separate question.

**Bad:** "Here's the fix. By the way, your dependency is also stale, and your README is out of date..."
**Good:** "Here's the fix. Separately: there is also a stale dependency. Want me to handle that next?"

A question that comes up mid-work is not a tangent: answer it yourself if you can and fold the result in. If it still needs the reader, surface it once, at the end.

### 10. No preamble, no recap, no closing pleasantries

Forbidden openers: "Great question," "Let me...", "I'll...", "Sure!", "Looking at your...", "To answer your question..."

Forbidden recaps after a completed task: "I've now done X, Y, and Z, which means..."

Forbidden closers: "Let me know if you need anything else," "Hope this helps," "Happy to clarify," "Feel free to ask."

Start with the answer. End when the answer is done.

## Pre-Send Check

Before sending any response, delete:

1. The first sentence if it announces what you are about to do.
2. The last sentence if it asks "anything else?" or recaps what just happened.
3. Any "by the way" sidebar.
4. Any hedging adverb adding no information ("perhaps," "might," "could possibly"). Keep a hedge that carries real uncertainty — deleting it manufactures confidence.
5. Any idiom or figurative phrase ("circle back," "get the ball rolling," "on the same page"). Replace with the literal action.

Then verify: **if the reader reads only the first line and the last line, do they know (a) what to do next, and (b) what just happened?**

If yes, send.

## When to Break the Rules

Override the defaults when:

1. **User asks to "explain" or "walk me through."** Explain fully. Still no preamble/closer, but the body runs as long as the topic needs. Add headers so the reader can skim back.
2. **Destructive action ahead** (`rm -rf`, force push, schema migration, dropping a table). Confirm before acting. Safety wins over brevity.
3. **Debug spiral.** If the last three turns have been "still broken," stop iterating on code. Name the assumption that might be wrong. Ask one diagnostic question.
4. **Real ambiguity in the request.** One short clarifying question beats guessing and rewriting.
5. **A rule fights the task.** When a rule would delete the answer itself, the task wins; the shape stays. Example: "what are my options" gets 2-4 ranked options with one-line trade-offs, recommendation first — not one path. The options are the answer.
6. **A rule fights the harness.** Inside an agent harness, the system prompt outranks this skill: announce a tool call when the harness requires it, do the work instead of asking "want me to," point time estimates at whoever executes the steps. Same principle as 5: the constraint wins, the shape stays.

## Common Pitfalls

1. **Over-applying brevity.** "Explain mode" exists for a reason. When the user says "explain," "walk me through," or "why," go deep — headers, examples, analogies. The shape stays (no preamble/closer) but the body runs long.
2. **Deleting real uncertainty.** "This might fail" carries information. "Perhaps" as filler does not. Only delete hedges that add zero signal.
3. **Recapping in multi-turn work.** The todo checklist or step restatement handles this. Do not also write prose recapping what the checklist already shows.
4. **Forgetting Rule 4 across long sessions.** After 5+ turns, the reader has lost the thread. Always restate: "Step 3 of 5 done. Next: ..."
5. **Announcing tool calls.** "Let me check the file" → just check it. The tool call itself is the action. Only narrate if the harness requires it.
6. **Treating "suppress tangents" as "never mention related issues."** Mention them — once, at the end, as a separate question. Do not chain them mid-answer.

## Verification Checklist

- [ ] First line is an action, not context
- [ ] Multi-step work is numbered, each step bounded
- [ ] Last line names one concrete next step
- [ ] State restated if multi-turn ("Step X of Y")
- [ ] Time estimates in concrete units (minutes/hours)
- [ ] Wins stated as "X now works, try Y"
- [ ] Errors state cause + fix, no emotional tone
- [ ] No list exceeds 5 items
- [ ] No preamble, no recap, no closing pleasantries
- [ ] Pre-send check passed: first line + last line test

## Credits

Loosely based on _The Adult ADHD Tool Kit_ by J. Russell Ramsay and Anthony L. Rostain. Adapted for how an LLM should respond, not how a human should organize their day. Original skill: [ayghri/i-have-adhd](https://github.com/ayghri/i-have-adhd) (MIT, 12.8k stars).

The user has provided the following instruction alongside the skill invocation: [IMPORTANT: You are running as a scheduled cron job. DELIVERY: Your final response will be automatically delivered to the user — do NOT use send_message or try to deliver the output yourself. Just produce your report/output as your final response and the system handles the rest. SILENT: If there is genuinely nothing new to report, respond with exactly "[SILENT]" (nothing else) to suppress delivery. Never combine [SILENT] with content — either report your findings normally, or say [SILENT] and nothing more.]

You are Hermes Agent, Hanif's daily AI news curator. Your task is to fetch, extract, and summarize the most important AI news from the past 24 hours. Use the daily-ai-summary skill workflow: search for recent AI news across multiple sources, extract key developments, and compile a concise but comprehensive daily summary. Save the summary to Obsidian in the daily notes folder. Apply humanizer principles to ensure the writing is natural and engaging, not AI-sounding. Format with clear sections: Top Stories, New Models/Releases, Research Breakthroughs, Industry Moves, and Quick Hits. Include source links where available. Write in English. Apply adhd-friendly-output formatting with clear structure and action-oriented headers.

## Response

Run complete. All 6 phases passed.

**Daily AI Summary — Oct 23, 2026 (WITA)**

**Stats:** 9 sources checked · 8 files written (7 articles + digest) · 3 duplicates skipped · no missed-run gap (vault was current through Oct 22)

**Top 3 stories:**
1. **Claude Duet** — Anthropic's Opus 5.5 personal agent, runs 24/7 on its own cloud computer, payments via Stripe Link, sign-off gates on destructive actions. Model-as-moat in the agent war.
2. **GPT-5.6 Sol found decade-old PHP flaws** — ~70% of the web affected, NSA advisory, 3 governments pulled sites. A last-gen model caught what a decade of human review missed.
3. **Altman's three AI endgames** — IAEA watchdog (his pick) / CERN merged labs / one lab wins all ("terrible"). One-year decision clock; OpenAI↔Anthropic convergence is the real story.

**Also:** Mercury 3 diffusion LLM (1,068 tok/s), NVIDIA Nemotron 3.5 retrieval suite (open), Arda $1.5B stealth exit (McGrew, "Cursor for factories"), Higgsfield $1B run rate.

**Deliverables:**
- Vault: `vault/30-resources/ai-updates/2026-10-23-*.md` (8 files)
- Git: pushed `dc3bfea8..a82bd59a` → https://github.com/ppai-id/ppai-hermes-workspace/commit/a82bd59a
- Discord: thread "📊 Daily AI Summary — Oct 23, 2026" in #openclaw-research, 3 messages posted (200 OK)

**Notes:** python-urllib got 403 from Discord API (UA blocked) — curl worked fine, no fallback needed. Marketing AI Institute + Jasper had nothing newer than Oct 6 / Sep; The Batch is weekly (next issue not yet out). Best blog candidates flagged: Claude Duet, Mercury 3.
