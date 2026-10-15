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
- Reading passages: 80 authored (p0061-p0080 added 2026-10-14: 10 A2 + 10 B1) (`tools/passages_src.json` -> `pack/passages.json`,
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
Republish a1a290b live proof KEEP (2026-10-08): live md5 equals local (index 663a79791c0a943d0c67441ec30bdebe, sw 24f06d3fa9587125e23cbb98042d4800). Seed = 31 sessions on the previous build with the kana primer learned; byte-equal after boot, reload and Progress; leaving Progress adds only pv; one Today session writes the new fields and three unit records (f, wm, ws; pl absent); that record boots on the previous build byte-equal with no backup keys and a session runs there; back on live byte-equal; 0 errors. Today shows "≈ 340 sessions".
- Migration proof on engine 8745de1 (placement covers kana/characters and stops early, placed level drives reading/patterns/estimates, pronunciation until primer, session estimates recalibrated): rollback hash b3882f01bc4a871868f6e542f767140d1783d3a7 (engine a1a290b). Live md5 before: index.html 663a79791c0a943d0c67441ec30bdebe, sw.js 24f06d3fa9587125e23cbb98042d4800. New build: index.html 76fa3c129ae3bb005e8b9450a4de247e, sw.js 140f63a6276269b5f8dbbbdbe7765ffc. Pack diff: pack.json gains placedKnown, placedRead, placementChars, placementEarlyStop, placementWhole, pronUntilPrimer (+ eta; goal 1 null by the queue verdict, so fresh Today shows no "≈" on goal 1, goals 2/3 and the gate carry estimates). Storage: new field chars.c[unit].prov and prog.script choice on a placement on this build; boot writes nothing. 12-session seed (primer learned) from the previous build boots byte-equal on the new build (boot, reload, Progress; only pv added on leaving Progress), after one session boots on the previous build byte-equal with no backup keys and runs a session, and back on the new build byte-equal (scratch 13/13). Placed records: placement on the previous build boots unchanged on the new build with no prov seeded; placement on the new build (316 prov units) boots on the previous build byte-equal (boot, reload), a session runs there, back on the new build byte-equal (12/12, KEEP).
- Migration proof on engine 81b8907 (notranslate meta, pron-first stems with the show-written tap, placement mix of recall and typed items; collapsed flag keys dropped from the pack): rollback hash bf042ccb982d68a27530e41928b4aa69fbe116a9 (engine 8745de1). Live md5 before: index.html 76fa3c129ae3bb005e8b9450a4de247e, sw.js 140f63a6276269b5f8dbbbdbe7765ffc. New build: index.html 2a1eabd65c4139367152cd2b7de11c74, sw.js 7f59c02d91f08be0d0f3833f55a12498. Pack diff: pack.json loses the 21 collapsed flag keys (appView, dayAware, freqTiers, glossFocus, glossStyle, helpClose, levelGate, listenQuestions, optsMix, pairs, pauseNew, placedKnown, placedRead, placementChars, placementEarlyStop, placementWhole, progressView, pronUntilPrimer, readAnswerBlock, readRotation, wordsBy are engine defaults now); every other pack file equal. Storage: boot writes nothing; leaving Progress adds only pv. 12-session seed from the previous build boots byte-equal on the new build (boot, reload, Progress), one session writes sessions/script/sn/day/pm only, the record boots on the previous build byte-equal with no backup keys and runs a session, back on the new build byte-equal (13/13, 0 errors). Placed records: placement on the previous build boots unchanged on the new build (no prov seeded); placement on the new build (316 prov units) boots on the previous build byte-equal, a session runs there, back on the new build byte-equal (12/12). ETA reused from 8745de1 (tools/eta.json unchanged; goal 1 stays null, known seed variance). Feature check (360 and 412, scratch): notranslate meta present; ja pron-first stems are kana text without ruby, so no .hasruby box exists here, and the show-written tap sits inline after the text with nothing to overlap; fresh placement items 1-6 are type, read, recall, recall, type, read.
- Migration proof on engine 232dbc5 (adaptive placement, level exam with critical set, cloze and pattern options never show a distractor from their sentence): rollback hash c62283017cdd1303f0e82447cc8c26f8f4d59d58 (engine 81b8907). Live md5 before: index.html 2a1eabd65c4139367152cd2b7de11c74, sw.js 7f59c02d91f08be0d0f3833f55a12498. New build: index.html 948d2b32e31afd4917858bb4d5ddfd2f, sw.js 39c8bae982a3116b9343735163a9c4e6. Pack unchanged (guard ok); ETA reused from 8745de1 (ETA_MODE external, no recalibration). Storage: new fields prog.crit, prog.exam and prog.exam.seen, placement prov records; boot writes nothing. 12-session seed from the previous build boots byte-equal on the new build (boot, reload; leaving Progress adds only pv); one session writes sessions/script/sn/day/pm; the record boots on the previous build byte-equal with no backup keys and runs a session; back on the new build byte-equal. Exam record (3 misses) boots on the previous build unchanged, crit and exam kept through a session there; back byte-equal. Placed record (17 items) boots both ways byte-equal (26/26, 0 errors). Feature check (360x780 touch, light and dark, scratch): placement finishes in 17 items, result "Start at A2, set 36" matches Today's Learn line; "Level exam ready: A1" and the Test row show; 20 bare exam items with no ruby; 3 misses give "Exam misses 3 left", 3 words flagged on Words, a Review session drew critical words; 5 exam and 3 session gap items, no option in its stimulus; notranslate present; 0 console errors, 0 failed requests. Verdict KEEP.

- Migration proof on engine d2876d8 (round d2876d8, 2026-10-13): PASS 30/30 KEEP vs e50d0e1 both directions: 12-session seed boots/reloads/progress byte-equal on prev, session changes it, new build boots byte-equal and back; exam record (20 items, 3 word misses, prog.exam + crit x3) boots byte-equal on e50d0e1, crit and exam kept through a session there. Browser: placement 17 items landing A2 set 36 = Learn set; placed-word Review miss sets prov 0 and w 1 (visible ETA unchanged at 210, eta kept live); exam "Exam misses 3 left"; Words flags 3 of 3; notranslate; 0 console errors. Screenshots .cache/live/round-d2876d8.
- Migration proof on engine d6fe859 (round d6fe859, 2026-10-14): PASS 18/18 KEEP vs d48eef9 both directions: 12-session seed (on d48eef9) boots/reloads/progress byte-equal on the new build, a session changes it, the record boots byte-equal on d48eef9 and back; exam-2 record (20 items, 3 word misses, prog.exam + crit) boots byte-equal on d48eef9 and back. Browser (working tree, 360x780 touch, dark): check 18/18 PASS, 0 console, 0 failed requests; placement 17 items landing A2 set 36; Level exam A1 20 items 0 reading aids; cloze 9 gaps 0 leaks. Kanji hints: 10 hint lines seen on a teach card, 0 on stimuli or options (72 items); EDRDG and JmdictFurigana credits footer with links. Goals: Today Goal 2 of 3 ≈ 210 sessions (placed+exam record), Goal 1 of 3 ≈ 540 (12-session record). Controls (run from vocab-engine main 88e4706): pack_flags 39/0, port_sites japanese 34/0, flagoff --check 11 fail: pack_drift japanese/korean/urdu/hindi/persian/indonesian/arabic, pure_arabic, boot_arabic_seed3, boot_hindi_seed3, boot_urdu_seed3 (goldens recaptured after the round). Build: index 24679a12ea07f4e243f2eeaa3adf7e5e, sw 65b580313cfbe6fedb29db1c7d9c72c4.
- Migration proof on engine 7d47e32 (round E, 2026-10-14): PASS 18/18 KEEP vs 8b5b9a7 both directions: 12-session seed (on 8b5b9a7) boots, reloads and keeps progress byte-equal on the new build, a session changes it, and the record boots byte-equal on 8b5b9a7 and back; exam-2 record boots byte-equal on 8b5b9a7 and back; 0 console errors, 0 failed requests. Browser (360x780, light and dark): furigana over the kanji only (食べる: た over 食, べる plain), bare tier hides only the rt with the width kept, option 行く shows い over 行 only, kanji hint line on the teach card (私), credits footer links EDRDG and JmdictFurigana. Gloss pass: 詳細 renders as "details (detailed)" in Words.
- Migration proof on engine 1fd9546 (round G, 2026-10-14, 20 more A2/B1 passages, session estimates kept live): PASS 42/42 KEEP vs a3abf2b (live) both directions: 12-session seed (on a3abf2b, Read unlocked) boots, reloads and keeps progress byte-equal on the new build, a session changes it, and the record boots byte-equal on a3abf2b and back; read.done keeps p0001 done and p0061 absent on boot; placed record boots byte-equal on the new build, runs a session on the previous build and boots on the new build. Browser (360x780 touch, light and dark): Today "Goal 1 of 3", "≈ 540 sessions"; Read lists 80 passages (A2 30, B1 30); p0061 renders, a word tap glosses, Done reading runs 5 questions and the results show; furigana over kanji on the title (2 rt) and the sentences (56 rt) once the kanji units are learned, light and dark; a learner with no kanji units yet sees kana with no furigana (pron-first), as on p0001; 0 console errors, 0 failed requests. Scratch .cache/live/round-1fd9546.
- Migration proof on engine aa76f69 (2026-10-14, scratch, live to follow): rollback hash bf74a36 (republish PREV; step-0 commit, index.html and sw.js unchanged there; previous live md5 index 45b03f3dad10d5c59c78317b40e089a2, sw 764f858a52f8b8ea5f351f44737bc788). Republish 34a92ed. Linker fixes (まもなく fragment, ので, 起きる/起こる, 見る after te, conditional と) change links on 52 sentences and move 759 ranks. KEEP: proof 42/42 on 360x780 touch (the 12-session seed from the previous build boots on the new build byte-equal after boot, reload and Progress, and back both ways; placed and read.done records boot unchanged and back; p0001 done, p0061 absent). Feature check: 80 passages listed, p0061 opens, word tap glosses, questions run and results show, furigana on passage title and sentences in light and dark, Today estimate shows numbers; 0 console errors, tatoeba audio aborts 0 (as before).
- Migration proof on engine a53b8c8 (round zh-ja-hooks, 2026-10-15, furigana bare for known words + kanji compound hooks): PASS. Rollback e004871 (live md5 index fa764417, sw c95d03c4). 16-session seed on e004871 boots, reloads and round-trips byte-equal on 3168c54; Progress text equal except goal 1 estimate (490 to 440 sessions, recalibrated); a session changes it; record boots back. Known 来 furigana hidden; 学校 hook last under hints; Words count equal. Visible unknown-kanji furigana not observed in a session (pron-first shows the reading). New index 8f7ed2b2, sw e655e814.
- Migration proof on engine 6a6f55a (round ja2, 2026-10-15, near-done level typed asks first, session estimates recalibrated on 6a6f55a): PASS 14/14 strict vs 270d42b (live before: index 8f7ed2b2, sw e655e814). Rollback 270d42b. 16-session seed (round-zhjako record, 100 word records) on 270d42b boots, reloads and round-trips byte-equal on 8715fb6 (live index 3e27d68b, sw de906e1f); a session changes it; the record boots back on 270d42b; 0 console errors, 0 failed requests. Today goal 1 on this record 440 to 150 sessions (fresh record 490 to 200; tools/eta.json cal and oos PASS at 600/600). Known 来 furigana hidden on both builds; 学校 and 学習 hooks last in unitHints on both builds, rendered teach card not seen in 8 sessions. Words row count equal (10 shown on both). NOT verified: visible furigana for an unknown kanji word (no visible ruby in 437 sampled). Scratch .cache/live/round-6a6f55a.
