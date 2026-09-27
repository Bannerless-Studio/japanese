# Japanese A1-B1 vocab trainer

A free vocabulary trainer for Japanese, A1 through B1: 2000 words with short
English glosses and kana readings, example sentences with kana lines, a
1590-unit kanji stage, and 60 graded reading passages.

**Live:** https://bannerless-studio.github.io/japanese/

**Scope note:** this app gives the vocabulary base for B1. A B1-level exam
(for example JLPT N3) also needs grammar, kanji, reading, listening and
speaking practice, which this app does not teach.

## Using the trainer

- **Script primer.** A "かな" stage before A1 teaches hiragana (108 units),
  then katakana (121 units), with symbol-to-sound, recognition and
  word-reading items. It's skippable ("I can read it") and reversible from
  Progress. Reading passages show the reading above every kanji until that
  kanji is mastered in the kanji stage, so a learner who hasn't finished it
  yet can still read a passage.
- **Kanji stage.** 1590 kanji units are taught after the A1-A2 words and
  before B1. Until a word's kanji are marked mastered in this stage, the
  trainer shows and drills it in kana; a "show written" tap reveals the
  kanji form early for anyone who wants it. Once a word's kanji are
  mastered, the word switches to its normal written form.
- **Today** runs one daily session: review, learn new words, listen, recall,
  sentence practice, and a reading passage when one is due. Each stage skips
  itself when there is not enough material for it yet.
- **Words** lets you browse and search the word list, and drill any set on
  demand.
- **Test** has a placement test (to skip words you already know) plus free
  tests.
- **Progress** shows your stats and lets you export, import, or reset your
  progress.
- Question types: hearing a word and picking its meaning, reading a word and
  picking its meaning, seeing a meaning and picking the word, typing the word
  from its meaning, and filling a gap in a sentence. Typing alternates two
  kinds: "type the reading" (kana, from your phone's kana keyboard;
  katakana and hiragana count as the same, half-width kana count as full
  width) and "type the characters" (plays the word first, and accepts the
  written form or any of its other accepted spellings).
- **Reading passages (Read tab, inside Today):** 60 passages, 20 per level,
  with tap-to-gloss on every word. A level's passages unlock once you've
  learned 70% of that level's words. Comprehension questions are
  machine-authored and went through two QA rounds, but have not been
  reviewed by a native Japanese speaker. A passage's spaced re-read (after 7
  days) becomes a listening pass when your device can play every sentence,
  with the text hidden and some questions audio-only.
- **Kana readings.** Every word shows its reading in kana, and every example
  sentence carries a kana line, so you can read even before you know a
  word's kanji.
- **Offline:** the app is a single page with a service worker, so once
  loaded it keeps working offline and loads instantly on repeat visits.
- **Speech:** there is no recorded audio for Japanese. The trainer speaks
  every word and sentence with the browser's `ja-JP` voice.
- **Progress export/import:** the Progress tab can export your progress as
  text and import it back (for example, to move to a new device). Progress
  is otherwise kept only in this browser's local storage.
- **Font:** the pack loads Noto Sans JP (400, 700) from Google Fonts, with
  Hiragino/Yu Gothic system fallbacks, so a Chinese system font never
  renders the Japanese kanji.

## Data

This is a static data pack for a language-agnostic vocab trainer (`key:
"ja"`). It's built from the shared
[`vocab-engine`](https://github.com/Bannerless-Studio/vocab-engine) (the UI
and drill logic, included here as a git submodule at `engine/`) plus this
repo's Japanese data and Japanese-specific pack-builder rules
(`engine/tools/packbuilder/langs/ja.py`; summarised in `tools/README.md`).

**Data quality.** Hand QA used stratified samples across three rounds (90-180
words and sentences at a time, seeds 51/52, 71/72, 81/82). In the final
samples every word has the right primary sense and reading, and every
sentence links the right words with the right kana line. Each wrong-link or
wrong-reading class found in QA was fixed by rule, not word by word (see
`tools/README.md` "Japanese rules"). Tatoeba's curated word index confirms
about half of all sentence links; Sudachi decides the rest. Levels are
frequency bands, not CEFR or JLPT levels. Full QA history and residuals are
in `tools/README.md` and `TODO.md`.

Tatoeba had fewer than two usable sentences for some words, so 22 simple
polite sentences were written for this pack (the build ships 20 of them,
count also in `pack/attribution.json`), each marked `"src": "gen"` in
`pack/sentences.json`. They are machine-written and reviewed, but not by a
native Japanese speaker. Examples prefer the polite register (です/ます).
Vulgar sentences and sentences with slurs are left out (mild words stay).
Sexual content and violence are kept out of A1/A2 sentences.

**Alts are spellings; forms are conjugations.** `alt` holds other spellings (the only extra typed answers); `forms` holds conjugations and other linked surfaces, found in text but never typed. Details: [tools/README.md](tools/README.md) "Japanese rules".

The 60 reading passages (`pack/passages.json`) were built from
`tools/passages_src.json` (see `tools/REPORT_passages.md`); their
comprehension questions are machine-authored and went through two QA
rounds, but have not been reviewed by a native Japanese speaker.

The pack links no audio and relies on TTS. No JLPT word list is used in the
build or shipped.

### Sources and licences

| Data | Source | Licence | Used for |
|---|---|---|---|
| Written/general frequency | [`wordfreq`](https://github.com/rspeer/wordfreq) Python package (`ja`, tokenized with MeCab + ipadic) | CC-BY-SA 4.0 (data), Apache-2.0 (code) | word ranking |
| Corpus frequency | lemma counts over the Sudachi-tagged Tatoeba sentences | CC-BY 2.0 FR | word ranking (stands in for a subtitle list) |
| Glosses, part of speech, readings | [kaikki.org](https://kaikki.org) Japanese Wiktionary extract | CC-BY-SA 3.0 / GFDL (Wiktionary) | English glosses, POS, kana readings, alternative spellings |
| Tokenizing, lemmas, readings (build time only) | [SudachiPy](https://github.com/WorksApplications/SudachiPy) with SudachiDict-core | Apache-2.0 | corpus lemmas, POS, readings and sentence links. The pack ships no dictionary files. |
| Example sentences | [Tatoeba](https://tatoeba.org) `jpn_sentences_detailed.tsv` | CC-BY 2.0 FR | sentence text (contributor usernames in `pack/attribution.json`) |
| Sentence translations | Tatoeba `eng_sentences.tsv` + `jpn-eng_links.tsv` | CC-BY 2.0 FR | English translations |
| Word index | Tatoeba `jpn_indices` | CC-BY 2.0 FR | confirms or corrects sentence links and readings |
| Furigana | Tatoeba `jpn_transcriptions.tsv` | CC-BY 2.0 FR | sentence kana lines |
| Generated sentences | written for this pack, `tools/generated_sentences.tsv` | CC-BY-SA 4.0 | sentences for words Tatoeba covers with fewer than 2 usable sentences, marked `"src": "gen"` |

Licence: code MIT, pack data CC BY-SA 4.0, see LICENSE.

## Level bands

Candidate words are ranked A1/A2/B1 by a blended frequency score across
`wordfreq` and the tagged Tatoeba corpus, with a forced A1 core (numbers,
days, months, greetings, pronouns, question words, demonstratives, core
particles and auxiliaries, common counters, time words, colours and family
words). This is a reproducible proxy for CEFR level, not an official CEFR
or JLPT classification. Full detail is in `tools/README.md`.

## Rebuild and publish

See `CLAUDE.md` for the pinned rebuild/check commands and `tools/README.md`
for what each file under `tools/` is, the full Japanese linking-rule
reference, and the from-clean-checkout rebuild steps. In short: `python3
tools/build_pack.py` rebuilds the pack, `./build.sh` builds `index.html`,
and `./check.sh` must pass before every commit that touches `index.html`.

**Note for agents:** `pack/words.json`/`pack/words.js` may be mid-migration
to an alt/forms schema split by another worker — check `git status` and see
`CLAUDE.md` before touching anything under `pack/`.
