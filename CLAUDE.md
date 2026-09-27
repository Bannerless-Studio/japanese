# Japanese trainer — agent notes

```
kaikki (Wiktionary ja) ──┐
Tatoeba jpn/eng/index/furigana ┼─> tools/build_pack.py (packbuilder, langs/ja.py)
wordfreq ─────────────────┘        │
SudachiPy + SudachiDict-core (build-time only) │ gloss_overrides.json, gloss_display.json,
                                    │ forced_a1.txt, generated_sentences.tsv,
                                    │ passages_src.json, id_map_v1.json
                                    v
                          pack/*.json --jsonify_pack.py--> pack/*.js
                                    v
              build.sh (engine/build.sh: engine/app.html + engine/core.js + pack js)
                                    v
                    index.html + sw.js -> GitHub Pages
                    https://bannerless-studio.github.io/japanese/
progress lives in localStorage key vocab_ja on the shared origin
```

Why it is built this way: single-file site + service worker for offline; engine as a
git submodule so every language ships the same drills; pack ids frozen
(`tools/id_map_v1.json`) so learner progress survives rebuilds. Japanese is unspaced,
so words have both `alt` (other accepted spellings of the same word, e.g. 私/わたし)
and `forms` (conjugations and other surfaces the corpus links but that are never
accepted as a typed answer, e.g. 食べない for 食べる) — see tools/README.md "Japanese
rules" for the full alt/forms distinction. `sentences.json` carries no token spans
(`spaced: false`), so the engine clozes and highlights by substring match, and
`pack.json`'s `compounds` list stops a short word matching inside a longer one.

## IMPORTANT — in-flight work, do not disturb

- `pack/words.json` and `pack/words.js` are currently uncommitted (rebuilt by another
  worker mid-migration to the alt/forms schema split). **Never edit, stage, commit, or
  run build.sh/check.sh against pack/** while this is in flight — check `git status`
  first and leave any pending pack/ changes exactly as found.
- The README's "Alts are spellings; forms are conjugations" bullet (and its `words.json`
  field-list line in the layout diagram) reflect that in-progress schema change.
  Preserve that bullet's wording byte-for-byte wherever it appears (README.md or, after
  a docs restructure, tools/README.md) — don't paraphrase or "fix" it.

## Commands (pinned)

- Rebuild pack: `python3 tools/build_pack.py` (equivalent to
  `PYTHONPATH=engine/tools python3 -m packbuilder build --lang ja --repo .`), then
  `python3 engine/tools/jsonify_pack.py pack`
- Rebuild reading passages: `PYTHONPATH=engine/tools python3 -m packbuilder passages --lang ja .`
  then `python3 engine/tools/jsonify_pack.py pack`
- Build site: `./build.sh`
- Check (must pass before every commit of index.html): `./check.sh`
- Against a non-submodule vocab-engine checkout: set
  `PACKBUILDER_PATH=../vocab-engine/tools` for both `tools/build_pack.py` and `./check.sh`
- QA helpers: `PYTHONPATH=engine/tools python3 -m packbuilder {scan,sample} --lang ja --repo .`
- Engine tests live in vocab-engine (see its CLAUDE.md)

## Always

- Commit index.html and sw.js together; check.sh's stale-build guard runs post-commit.
- Bump the engine submodule only to a vocab-engine main sha; rebuild after every bump.
- Keep ids append-only; never renumber (`tools/id_map_v1.json`).
- Path-limited commits: engine, index.html, sw.js, pack/, tools/, README.md, TODO.md;
  never .venv or .cache.

## Never

- Edit pack/*.json by hand; change tools/gloss_overrides.json, tools/gloss_display.json
  or tools/forced_a1.txt and rebuild instead.
- Edit pack/*.js, index.html or sw.js by hand (generated).
- Delete sw.js (use engine/engine/sw.disable.js).
- Add comments that say what the code does; only why, or an external reference.
- Push to main without `git merge-base --is-ancestor origin/main HEAD`.

## Generated files

pack/*.js, pack/*.json, index.html, sw.js, tools/REPORT.md, tools/REPORT_passages.md,
tools/id_map_v1.json (frozen, hand-edit never).

## Where things are

README.md (end users), tools/README.md (builder inputs, file by file, incl. the
Japanese rules detail), TODO.md (residuals + engine notes), engine/ (submodule,
read-only here).
