# TODO (v2 candidates)

Residuals from the v1 build and QA rounds. The rules already in place are
described in `engine/tools/packbuilder/langs/ja.py` and summarised in the
README.

## Before shipping
- **Done (2026-09-25).** The engine submodule is bumped to vocab-engine
  350bdef, which contains `langs/ja.py` (tag version t22) and the shared-code
  additions it needs (`spoken_from_corpus`, `spoken_freq_label`,
  `use_simplemma`, `sentence_end_re` in `langs/base.py`; corpus-count
  substitution in `core/freq.py`; the `sentence_end_re` override in
  `core/sentences.py`; `spoken_freq_label` in `core/report.py`; the
  `note_words(words)` hook `core/sentences.py` calls before linking, used
  here to find kanji spellings that are another word, 寄る under よる). The
  `PACKBUILDER_PATH=../vocab-engine/tools` workaround is no longer needed.
- Reading passages: 60 authored (`tools/passages_src.json` -> `pack/passages.json`,
  see `tools/REPORT_passages.md`). The passage hooks are in vocab-engine
  `langs/ja.py`. Round-3 linking rules (2026-09-26, vocab-engine branch
  ja-passage-rules) cover てほしい, quotative という, 前 "ago", 後 read ご,
  numeral + つ, Xに and その後/一番 fragments, conjunction ただ, たった and
  位 read くらい; the report's Manual QA has the before/after table.
  `packbuilder passages --lang ja` leaves `.cache/derived/ja_groups_*.json.gz`
  untouched. Rebuild passages after the engine submodule moves past that branch.

## Engine
- Unspaced text has no spans. `sentences.json` carries no token spans, and
  with `spaced: false` the engine clozes and highlights by plain substring.
  Every conjugated form is therefore listed as an alt, and `pack.json`
  `compounds` (a list of longer surfaces) stops a short word from matching
  inside a longer one. Reading passages need real spans: the engine should
  accept per-sentence or per-passage character spans (start, end, word id)
  for unspaced languages and use them instead of substring search.
- The brief asked for `"compounds": false`. The pack ships `compounds` as a
  list because the engine schema types it as a list and the substring mode
  needs it.

## Lemmas and readings
- めくる and まくる are below the frequency cut (about 10 corpus tokens
  each), so めくった has no word. They were checked for cross-linking: neither
  is an alt of the other.
- Some kana headwords keep a kanji spelling Sudachi normalises separately
  when the senses agree (こと/事, つける/付ける, かける/掛ける, なくなる/亡く
  なる, あく/空く). The alt scan lists 16 such pairs; each is the headword's
  own spelling, not another word.
- Tatoeba furigana errors that name a real reading of the spelling are kept:
  入れる はいれる, 同じ どうじ, 暇 いとま, 博士 ひろし, 後から ごから. The
  furigana fix only replaces readings the word never has.
- Digits stay digits in kana lines. Six sentences read a native fused number
  and counter whole (１日 ついたち, ２人 ふたり, 10日 とおか, 110番).
- Sudachi gives one reading per spelling. Tatoeba's word index adds second
  readings when it spells them out, but a reading homograph under 20% of a
  word's uses stays folded in: 辛い is glossed and read つらい, and からい
  "spicy" is not taught. 訳 is わけ; やく "translation" links わけ.
- A hand table fixes three readings Sudachi gets wrong for learners
  (`READING_FIX`: 私 わたし, 明日 あした, 何 なに).
- Closed context tables in `langs/ja.py`: `COMPOUND_PARTICLE_VERBS`
  (において, につれて, にわたって), `IDIOM_UNLINK` (実を結ぶ, 主として,
  ある種, おいとま), `POTENTIAL_OF` (なれる after に is なる), `DIALECT_ONLY`, and the
  kana よる rule (因る only after に).
- `FIXED_READ` holds readings the corpus counts cannot show because Sudachi
  gives one reading per spelling: 何時 なんじ, 種 たね. Other folded reading
  homographs may need the same (種 was found by the seed-71 sample).
- The headword follows Tatoeba frequency, so some common verbs show in kana
  with kanji alts (とる with 取る; 撮る "to take a photo" is a word of its own).
- ほとんど is labelled a noun (Sudachi and Wiktionary agree) but glossed with
  its adverb use too.
- Other hand tables: `PHRASES` (greetings, compound particles, こういう),
  `SUFFIX_WORDS` (taught suffixes such as 〜さん), `AUX_SURFACES`,
  `DIALECT_ONLY` (よう as Kansai よく), `FUSED_UNLINK` (何もかも, かどうか),
  `SUFFIX_BAD_PREV` and `HONORIFIC_AFTER_NAME` (the suffix guard),
  `JUU_HEADS` / `CHUU_HEADS` (中 じゅう / ちゅう), `WORD_PREFIXES` (貴).

## Glosses
- About 170 hand overrides in `tools/gloss_overrides.json` cover the top 300
  and the QA misses. Other glosses come from the Wiktionary sense ranking and
  can lead with a secondary or definitional sense in lower B1 (the seed-71
  sample needed 撃つ "to shoot" and ふう "way, manner").
- Katakana words are glossed from their own Wiktionary entries only. That
  fixes homophone glosses (ビル, ジム) but can pick an onomatopoeia entry
  over a kanji word's katakana spelling. ガン has the hand gloss "cancer
  (癌)", so it merges into 癌 instead of reading "thud". A sweep of katakana
  glosses for sound or mimetic senses found only ガン.
- Two overrides are reported unused. ガン and 無し are still needed: each
  is the primary sense that makes its spelling merge (ガン into 癌, 無し into
  なし), and the merged key then no longer exists.
- A few する-nouns read "to order (+ suru)" where Wiktionary has no short
  noun sense.
- Fragment glosses (2026-09-25 audit): 箱 read "small" because its 4th
  Wiktionary sense is "small ライブハウス (music venue)". The build kept the
  English before the Japanese word. An audit of all 2000 glosses found only
  that one. The fix is the override 箱 "box". A rule should reject a sense
  whose English is only a modifier of a Japanese word.
- Neighbouring-word gloss: おっ (A1 noun, "man") is Sudachi's split of
  おっさん and おっと, glossed from おっさん. It is not a word. Dropping it
  changes sentences, so it waits for the next rules round.
- `tools/gloss_display.json` holds display-only glosses: 高い, and counters
  whose gloss hid the unit (〜キロ kg/km, 〜年間, 〜秒, 〜名, 〜組, 〜点).

## Sentences
- Known wrong-link residuals from the seed-52 sample: 一杯 "one cup" links
  いっぱい "full"; literary にして links する; 恥ずかしがりや links the
  particle や; the imperative 取れ links 取れる. (金 "gold" is fixed.)
- 方 ほう / 方（かた）: the link and the kana line can disagree, because each
  source errs somewhere. Tatoeba's furigana reads 気が強い方 as かた, while
  the word index labels 違う方に as かた. No context rule separates 行きたい方
  "anyone who wants to go" (かた) from 強い方 "on the strong side" (ほう). In
  the final build 3 of about 20 方 sentences are affected.
- Borderline links kept: という links 言う, お目にかかる links 目 and
  かかる, and ペルーへ立つ links 立つ glossed "to stand".
- Kana homophones (`_homophone` in `langs/ja.py`) are judged from the first
  Wiktionary gloss or the hand gloss, so a sense missing there is not
  confirmed: 事故にあいました "had an accident" (遭う, an alt of 会う) links
  nothing. もう一枚とってください "take another one" keeps the index's 撮る,
  since "take" fits both とる and 撮る; 撮る has no kana alt, so the form is
  not bolded.
- 冴えない and 忍びない are not index-attested as words: they link the verb
  (冴える, 忍ぶ) and the ない.
- 後 lost its kana alt あと: 多少あとが残る ("scars", 跡) is now unlinked, and
  an alt that is exactly a token of another linked word is pruned. 洗ったあと
  and そのあと link 後 without bolding it. 
- Kana-line residuals: 夏休み中家 reads 家 as か (Sudachi); 身体中 reads
  しんたいじゅう (からだじゅう is more usual); 来週中 is ちゅう by rule.
- Three compound-substring occurrences stay unshielded in `pack.json`
  `compounds`, because a shield would also hide a true link.
- Casual sentences are ranked after polite ones but can still appear at B1.
- 20 generated sentences (`src: "gen"`) have not been reviewed by a native
  speaker.
- Policy: sentences with slurs (めくら, つんぼ, きちがい, 支那, ジャップ and
  similar; `VULGAR_JA` in `langs/ja.py`) are dropped. Mild words stay.

## Levels
- Levels are frequency bands over wordfreq and the Tatoeba corpus. Tatoeba
  is a translation corpus, so a few topics (crime, war) rank higher than in
  everyday speech.

## Round 3 (seeds 81/82) notes
- Words that left the pack when the rules above removed inflated counts:
  自殺 (earlier), 折る (humble おる no longer counts as 折る), 基 (the
  index's 基(もと) corrections are no longer taken unchecked), 無くなる (its
  kana uses link なくなる when the English says "run out"), 〜等 (一等, ２等
  now link 等 "class"), いえる, カモ, 塔. Entered: 等, もと, 党, 覚める, 依頼,
  つまらない, 修正 (ids w2004 to w2010, appended to `tools/id_map_v1.json`).
- Final polish round: `〜分（ぶん）` (w0964) is now "part, portion, share"
  read ぶん (fractions: ４分の３). もと (w2005) left the pack once 下 read した
  stopped counting for it; しばしば (w2011) entered. 者 moved to A2 once
  ものです stopped counting as 者; 着く is forced to A1.
- Suffix alts: the brief asked for bare or number + suffix only. 〜中 and 〜君
  also keep their noun + suffix uses (会議中, トニー君), because their bare
  spellings are other words' headwords (中, 君) and are pruned; without them
  those links could not be bolded. The suffix guard runs first, so these are
  real uses.

## Passage rules round 3 (2026-09-26) follow-ups
- The rules are passage-only (`passage_post_resolve`, `passage_retag`). The
  sentence build still links conjunction ただ to ただ "ordinary" (cross-POS
  link to the noun), 一つ as 一 (いち) + つ, 1ヶ月後 to 後 (あと), and
  quotative という to 言う. Porting them to `post_resolve` changes
  `sentences.json` and needs its own QA round.
- ただ has no "but, however" sense in its gloss; conjunction ただ、 links the
  adverb "only, simply". A gloss override would fix the display.
- 前 after a time amount links the noun 前 ("front; before, ago"). Its gloss
  leads with "front"; a display gloss could lead with the time sense.
- The mc verbatim self-check no longer exempts 一つ keys (飲み物を一つ, p0007
  q3): 一つ is now one 〜つ token, not a NUM token.

Republish 09e90bc: sentence spans (23349/23349 linked words placed, 0 unspanned WARN); inflected forms now cloze targets. words.json unchanged: no word level or gloss moved by stab narrowing.
Republish aa00571: no word/gloss moved, pack byte-identical; 0 dead override keys (無し|noun, ガン|noun kept); set-counter and no-voice planner fixes.

Republish on engine a1a290b (2026-10-08): typed modes, day-aware scheduling, reading rotation, goals, pairs, frequency tiers, Progress v2, redesigned tabs, session estimates, characters lag set + levelExam. Rollback hash 45a0f1b264a4e9450fa957b1f8206ec0c075f2fb; previous live md5 index 174cb2a8c808b515122058e3362a3b15, sw cc2cfc5fef1b48fac6c5405a731a119a; new local md5 index 663a79791c0a943d0c67441ec30bdebe, sw 24f06d3fa9587125e23cbb98042d4800. Pack diff vs 45a0f1b: every word and character unit gains `ft`; pack.json gains the flag block, the characters lag/bareBy block, levelExam and eta; nothing else. Storage: new fields day/sn/t/u/f/p/pm/pv/pause/read.done s,ls/today.tw and unit wm/ws on first use; boot writes nothing; previous build ef44c6e/aa00571 carries them (migration [port] ja unit records boot on both, byte-equal and back). eta (calibrate 400 sessions, 85%, seeds 5/6/7): goal 1 curve 364 sessions fresh, goals 2 and 3 do not reach 0.9 within 400 sessions and ship null; gate curves A1 and A2 pass.
