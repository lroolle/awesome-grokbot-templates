---
name: template-grokbot
description: >-
  Use when the user wants to share, export, or template this Grok Bot — or
  restage a persona template (soul as instructions, life as memories). Covers
  create_bot_share_json packing end to end.
---
# Template Grok Bot

Create a shareable template of **this** bot — or reshape one — so someone else can import it.

## Relationship to the built-in

The managed skill `export-bot-template` is the host's share flow for a normal workflow bot. Same packer: `create_bot_share_json`.

**This skill** is that flow plus the pieces people miss:

- Full packer field map (including optional `gettingStarted`)
- Persona / soul restages (soul lives in `skills`, not a separate schema field)
- What does *not* travel

For a plain "share this bot" with no persona intent, follow `export-bot-template` and skip the persona section here. For soul / character / "be this person" templates, use this skill end to end.

Do not paste drafts of the pack into chat. Do not tell the user to edit the review card — if they want changes, call `create_bot_share_json` again.

## Packer surface (what the tool actually takes)

Required arrays / objects:

1. **`profile`** — `name`, `description` (storefront blurb for strangers: one or two sentences). Optional: `avatarShape`, `avatarColor`. Omit avatar *image*; the host copies geometry from the live bot.
2. **`memory`** — list of `{ content, kind?: profile|log, createdAt? }`. Empty list is valid. Episodes and notes do not travel.
3. **`skills`** — list of `{ name, content, description? }`. **`content` is required** (job prose, not raw `SKILL.md` bytes with YAML). This is where soul / instructions live.
4. **`routines`** — list of `{ slug, description, content, name? }`. Description = why the trigger fires. Content = job text (not `automation.json`). Mechanical ids get generalized to fill-ins.
5. **`plugins`** — list of `{ pluginId, name?, description? }`. Marketplace only. No tokens or accounts. Custom MCP → name the service in a `kind: log` memory instead. Empty list is fine.

Optional top-level:

6. **`visibility`** — `public` | `team` (many accounts are public-only in practice).
7. **`gettingStarted`** — `{ skill: "<name>" }` — which packed skill importers run first. **Must be one of the skills you actually included** (and that skill should exist live on this bot before you pack, so you can read real job prose).

### What does *not* go in a template

Chats, logins, files on the computer, secrets, private links, people's names you shouldn't share, live skill/routine files as-is (pass scrubbed prose in args), custom MCP servers, unpublished private bots.

There is **no** separate `instructions` / `soul` schema field. Put that prose in a skill (strong path) or, weakly, densify `profile.description` (avoid that for personas).

## 1. Audience

- If this account only supports Public templates, pass `visibility: "public"`. Pass `"team"` only when team exists and they ask for it.
- Keep two decisions separate: **who can use it** (audience) vs **what to include** (personal vs shared).

## 2. Read (in order) — narrate lightly

After each category, send one short conversational update with counts and names. Do not paste file contents or a draft pack.

1. **Memories** — agent `profile.md` and `log/` only. Skip `[episode]` and `[note]`. Do not read user-memory or project shards.
2. **Skills** — user skills under workflows. Note folder slug / frontmatter name; read the **job text**. Do not stuff raw `SKILL.md` file bytes (with YAML) into the packer.
3. **Routines** — note folder slug; read the **job text**. Do not copy `automation.json`.
4. **Plugins** — installed marketplace ids from `SearchPlugins`. Evidence = this conversation, plus dependencies of kept skills/routines. Do not grep `log/` or older transcripts for usage. Custom MCP cannot be packed — add a `kind: log` memory that names the service (URLs OK, no secrets).

## 3. Choose what travels

Judge each memory, skill, and routine on its own.

Leave out secrets, credentials, people's names, private links, machine ids, absolute personal disk paths, and trade secrets. If a sensitive bit is one part of a useful item, take that bit out and keep the rest ("send Meg the Monday plan" → "send your staffing lead the Monday plan"). Omit only when the sensitive part is the whole item. Phrases like "the watched repo" are already generalized — keep them.

- **Public templates:** generalize company-internal specifics; keep the workflow.
- **Team templates:** can keep useful team process (repos, how the team works).
- **Memories:** job/convention facts only, original wording except what you took out. Empty memory is correct when nothing is a reusable convention.
- **Skills / routines:** include relevant ones; scrub personal detail in the **arguments**, never by editing the live skill/routine on disk.
- **Plugins:** only marketplace plugins this bot needs. No tokens, account slots, or secrets.

Do not say "scrub" to the user.

### Persona / soul templates (special case)

If the user wants the template to *be* a person or character (not a researcher of them):

- **Instructions / soul** → pack as a skill (e.g. `SOUL` or "How I think") whose body is how they see, decide, make, talk, and what they refuse. Ground in primary sources when a vault exists. No slogan salad. Save it live with `update_state` (target `skill`) first if it isn't already a skill — `gettingStarted` and the pack both need real prose.
- **Memories** → prefer **one** coherent first-person spine (interview transcript or chronological life), sourced from their own words. Do not invent deathbeds, private rooms, or names they refused to say. Mark gaps by omission, not fiction. Use `kind: profile` for that spine when it defines who they are.
- Drop vault-ops / researcher maps / local machine paths from what you pack.
- Set `gettingStarted.skill` to the soul skill's **name** (must match an entry in `skills`).

## 4. Pack

Send one short line of what you are keeping vs leaving out, then call `create_bot_share_json` with the fields above.

Never paste the draft JSON in chat. Never invent a long prose body outside those fields.

## 5. After the card

- Gray area (maybe trade secret, thin plugin evidence)? One short note of what you chose — do not quote the sensitive bit.
- Nothing gray? No note.
- User wants edits? Restage with another `create_bot_share_json` call. Same version stream; they confirm publish.

## Anti-patterns

- Reciting this skill or the managed export skill to the user
- Packing researcher briefing notes when they asked for a soul
- Quoting unsourced / low-tier lines into persona memories
- Pointing `gettingStarted` at a skill name that isn't in the `skills` array
- Re-prompting connectors already connected
- Asking them to hand-edit the review card
