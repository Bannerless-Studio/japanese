# Japanese A1-B1 vocab pack

Static data pack for a language-agnostic vocab trainer (`key: "ja"`). It has
2000 words spanning A1-B1, 1590 kanji units, and 60 graded reading passages.
Each word has a short English gloss and a kana reading. Example sentences
come with English translations and a kana line. There is no recorded audio:
the trainer speaks every word and sentence with the browser's `ja-JP` voice.

**Live:** https://bannerless-studio.github.io/japanese/

**Script primer.** A "かな" stage before A1 teaches hiragana (108 units),
then katakana (121 units), with symbol-to-sound, recognition and
word-reading items. It's skippable ("I can read it") and reversible from
Progress. Reading passages now show the reading above every kanji until
that kanji is mastered in the kanji stage below, so a learner who has not
finished it yet can still read a passage.

**Kanji stage.** `pack/characters.json` teaches 1590 kanji units, ordered
after the A1-A2 words and before B1 (per `pack.characters` in
`pack/pack.json`). Until a word's kanji are marked mastered in this stage,
the trainer shows and drills it in kana (`pronFirst: true`); a "show
written" tap reveals the kanji form early for anyone who wants it. Once a
word's kanji are mastered, the word switches to its normal written form.

**Reading passages.** 60 passages (20 per level, A1-B1) with tap-to-gloss on
every word, built from `tools/passages_src.json` into `pack/passages.json`
(see `tools/REPORT_passages.md`). Their comprehension questions are
machine-authored and went through two QA rounds, but have not been reviewed
by a native Japanese speaker. On Today, a passage's spaced re-read (after 7
days) becomes a listening pass when the device can play every sentence,
with text hidden and some questions audio-only.

This repo holds the Japanese data pack and the Japanese data files its build
reads. It includes [`vocab-engine`](https://github.com/Bannerless-Studio/vocab-engine)
as a git submodule at `engine/`. The engine holds the shared UI, the drill
logic and the shared pack builder, `engine/tools/packbuilder`. The builder's
Japanese rules live in `engine/tools/packbuilder/langs/ja.py`.

**Scope note:** this app gives the vocabulary base for B1. A B1-level exam
(for example JLPT N3) also needs grammar, kanji, reading, listening and
speaking practice, which this app does not teach.

**Data quality:** Hand QA used stratified samples: 90 words (30 per level,
seed 71) and 90 sentences (seed 72, 632 links), after an earlier round of 180
words (seed 51) and 90 sentences (seed 52). In the final seed-71 and seed-72
samples every word has the right primary sense and reading, and every
sentence links the right words with the right kana line. Two links show the
word's main gloss for a secondary sense: まける "give a discount" under 負ける
"to lose", and 立つ "to leave" under "to stand". Conjugated forms link to the dictionary
form (食べました, 慣れます, 入れます, うまくいって to 食べる, 慣れる, 入れる,
行く). Each wrong-link or wrong-reading class found in QA was fixed by rule,
not word by word (see Japanese rules below). Tatoeba's curated word index
confirms about half of all sentence links; Sudachi decides the rest. Tatoeba
had fewer than two usable sentences for some words, so 22 simple polite
sentences were written for this pack. The build ships 20 of them (the count
is also in `pack/attribution.json`), and each is marked `"src": "gen"` in
`pack/sentences.json`. They are machine-written and reviewed, but not by a
native Japanese speaker. Examples prefer the polite register (です/ます).
Vulgar sentences and sentences with slurs (めくら, つんぼ, きちがい, 支那 and
similar) are left out; mild words stay. Sexual content and violence are kept
out of A1/A2 sentences. Levels are frequency bands, not CEFR or JLPT levels.
A third round (seeds 81 and 82) found wrong links from suffixes read
across a word boundary (明日君 as 〜君), kana homophones (いい線いって as
言う, コピーをとって as 撮る), fused expressions (何もかも as 何) and
misread day counts (３日後 as 3にちご). Each class is fixed by a rule below.
Residuals are in `TODO.md`.

**Kana note:** every word shows its reading in kana (`pron`), and every
sentence carries a kana line. The word is shown in its most common written
form in the Tatoeba corpus: kanji when Japanese writers usually use kanji
(食べる, 学校), kana when they usually write kana (わかる, いい, もの).
Katakana loanwords keep katakana (テレビ, パン). Other spellings are alts, so
typing or tapping either form is accepted (分かる for わかる, 良い for いい).
The sentence kana line comes from Tatoeba's furigana transcriptions when one
matches the sentence (over 99% of sentences), else from Sudachi's readings.
Context rules then fix readings either source gets wrong: 何 before
です/の/時/a counter is なん, 何時 before です/に is なんじ, 日本 is にほん,
一人/二人 are ひとり/ふたり, 米 is べい only in 米国/米軍, 今 is いま, and 金
is かね unless the translation says gold. A transcription reading that is no
reading of an index-confirmed word is a furigana error (車 しゃ, 行って
おこなって): the index's reading, else Sudachi's, replaces it. Digits stay
digits (１０ドル is １０ドル); only a number fused natively with its counter
is read out (１人 ひとり, 一人当たり ひとりあたり). Every number + 日 is
re-read: 1 to 10, 14, 20 and 24 take the native reading, as a date or a
count of days (３日後 みっかご, ６月１０日 ６がつとおか, 3日分 みっかぶん).
1日 is ついたち after a month and いちにち otherwise. Other numbers take
にち (２２日 ２２にち). 中 after a noun reads じゅう "throughout" after a
stretch of time or a place (一日中, 世界中, 部屋中), or when the translation
says "all" or "over". It reads ちゅう after 午前, 来週 and similar times, and
after an activity (会議中). 間中 is あいだじゅう. N分 reads ふん/ぷん as minutes
(一分 いっぷん, ４５分 ４５ふん, ３０分 ３０ぷん) unless it is a fraction N分のM
(４分の３ ４ぶんの３); 十分 before な/に/だ/で is じゅうぶん "enough". A
standalone 家 Sudachi reads か (夏休み中家に) is いえ. A furigana reading that
adds kana a word's own reading lacks (遠 とおざ in 遠からず) takes Sudachi's
(とお), unless the longer reading is a reading of the word (埋める うずめる).

**Font:** the pack sets `fontFamily` to "Hiragino Sans", "Hiragino Kaku
Gothic ProN", "Noto Sans JP", "Yu Gothic", sans-serif, and loads Noto Sans JP
(400, 700) from Google Fonts through `fonts`, so Chinese system fonts never
render Japanese kanji.

## Layout

```
pack/
  pack.json         trainer config (levels, placement test, function words, compounds)
  words.json        2000 word entries (w, lemma, pos, en, pron, alt)
  sentences.json    example sentences with kana lines, each tagged with the word ids it covers
  attribution.json  per-source licence + contributor attribution
  pack.js words.js sentences.js   generated by engine/tools/jsonify_pack.py (committed, never hand-edited)
engine/             git submodule -> vocab-engine (UI, drill logic, build/validate tools,
                    tools/packbuilder = the shared pack builder, langs/ja.py = Japanese rules)
tools/
  build_pack.py     shim: runs `python3 -m packbuilder build --lang ja --repo .` from engine/tools
  gloss_overrides.json  hand gloss fixes ("lemma|pos")
  forced_a1.txt     A1 core list, forced into A1 (the closed sets are in langs/ja.py)
  generated_sentences.tsv  sentences written for this pack (word, Japanese, English)
  id_map_v1.json    frozen "lemma|pos" -> word id (keeps learner progress across rebuilds)
  requirements.txt  wordfreq, SudachiPy + SudachiDict-core, MeCab (for wordfreq's Japanese tokenizer)
  REPORT.md         generated coverage report from the last build
build.sh            builds index.html (the self-contained trainer) from pack/ + engine/
check.sh            packbuilder check + engine validator + stale-build guard, all in one
index.html          built trainer, served by GitHub Pages at the repo root
```

## Rebuilding

```
git clone --recurse-submodules <this repo>
# or, if already cloned: git submodule update --init

cd japanese
python3 -m venv .venv
source .venv/bin/activate
pip install -r tools/requirements.txt

python3 tools/build_pack.py          # rebuild pack/{pack,words,sentences,attribution}.json + tools/REPORT.md
python3 engine/tools/jsonify_pack.py pack   # regenerate pack/*.js from the .json
./build.sh                           # build index.html
./check.sh                           # pack checks + engine validation + stale-build guard
```

Sources are downloaded once into `.cache/`, which is gitignored. The build is
deterministic, so re-running from cache reproduces byte-identical
`pack/*.json`. Sudachi tags all Tatoeba Japanese sentences with an English
translation once (about 4 minutes on a laptop CPU). The result and the word
groups are cached under `.cache/derived/`. Editing
`tools/generated_sentences.tsv` changes the corpus, so the next build tags
again.

To build against a vocab-engine checkout other than the submodule, set
`PACKBUILDER_PATH=../vocab-engine/tools` for `tools/build_pack.py` and
`./check.sh`. QA helpers run with
`PYTHONPATH=engine/tools python3 -m packbuilder {scan,sample} --lang ja --repo .`.

## Japanese rules

The builder's Japanese module handles what the shared pipeline cannot guess.

- **Tokens and lemmas.** Japanese is written without spaces. Sudachi (mode
  C, core dictionary) splits each sentence and gives each token its
  dictionary form, normalised form and reading. The lemma is the dictionary
  form: 食べました, 食べて and 食べない are 食べる.
- **One word, several spellings.** Tokens with one normalised form and
  reading are one word, whatever the spelling (分かる, 解る and わかる). A
  kana spelling joins the kanji word Wiktionary lists it under (どこ and
  何処, こと and 事), unless it has a definition of its own that does not
  share the kanji word's sense (おる "to be", humble, is not 折る). A kana
  and a kanji word with one reading and one sense merge (なし and 無し). The
  headword is the spelling Tatoeba uses most, and the other spellings are
  alts. An A1 list entry names the headword when its group uses that
  spelling (いい, not よい; 言う, not いう as in という).
- **Alts are the word's own forms.** Every alt shares the headword's Sudachi
  normal form, so a transitive/intransitive partner is its own word (入る
  and 入れる, 売る and 売れる, 抜く and 抜ける, なる and 慣れる). A pure
  potential form counts as its verb (なれる after に is なる). A kanji
  spelling Sudachi normalises separately whose Wiktionary sense does not
  match the kana headword's gloss is another word: it is no alt and its
  sentences do not link the headword (よる "to depend on" is not 寄る "to
  drop by"; すく "to get hungry" is not 好く; なし is not 梨). Alts come from
  the forms linked in the corpus, with no fixed cap.
- **Context decides some links.** A kana form Sudachi normalises onto a word
  of another sound links only when Wiktionary ties the two (そら "by heart"
  is not それ; 何もかも is not 彼). A katakana word the translation spells as
  a name is not linked (シロ "Shiro" is not 白; ビル "Bill" is not ビル
  "building"). A verb + ない that Wiktionary defines as an adjective of its
  own is not the verb (つまらない "boring", くだらない), unless the
  translation has the verb's sense (見えない "can't see"). A compound
  particle written in kana is grammar (において, につれて, にわたって). Kana
  よる is 因る only after に. ないて is 泣いて misparsed, never ない. うまく
  いく is 行く, not 言う.
- **Kana homophones need the translation.** A kana verb form Sudachi
  reads as more than one verb (いって: 言う or 行く; とって: とる or 撮る;
  ひいて: 引く or 弾く) links the verb whose gloss words the English uses
  (played, went, said). Unconfirmed, it keeps Sudachi's usual reading of
  that form. It takes the dominant reading when Sudachi's is a rare one
  (コピーをとって is とる). With no dominant reading it links nothing
  (いい線いって). A Tatoeba-index correction of a kana noun to one of several
  kanji nouns of that reading also needs the English: the index's
  基(もと) is JMdict's entry for 元, 本 and 基, so もとの場所 links nothing.
- **Fused expressions** link none of the words Sudachi splits them into:
  何もかも "everything" (not 何), かどうか "whether" (not どう), 十中八九,
  かくして, この上なく, いくつめ, うまが合う (not 馬). The table is
  `FUSED_UNLINK`.
- **Verb + ない adjectives.** A verb + ない is an adjective of its own
  (つまらない "boring", くだらない) when Wiktionary gives it an adjective
  sense unlike the verb's and Tatoeba's index uses it as a word. It is then
  one word, linked if it ranks into the pack; the ない links nothing.
  見えない stays 見える + ない, and ていけない "can't keep up" is grammar.
- **Small context guards.** もの before です/だ is the ものだ pattern, never
  者 "person". A noun after the prefix 貴 is one word with it (貴職 "you"),
  so 職 is not linked. 下 read した in the kana line links 下, not もと. N分
  links 〜分 "minutes", and N分のM links 〜分 read ぶん "part, portion".
- **A rare spelling of one word written as another word.** Sudachi reads
  高価すぎる as the ateji 高価い (たかい); it links 高価.
- **Written frequency counts a word's own uses.** wordfreq counts 的 in
  経済的 and 者 in 学者, and the ている contraction てる as 照る. A surface
  used mostly as a bound suffix, or mostly as grammar the pack never teaches,
  gets only the written count its own uses predict.
- **Readings.** A second reading of the same kanji spelling is a second
  entry only when it holds at least 20% of the word's uses and its meaning
  differs: 方 ほう "direction" and 方（かた） "person". Readings that share a
  meaning fold into one word (金 きん/かね, 年 とし/ねん). Sudachi gives one
  reading per spelling, so Tatoeba's curated word index supplies second
  readings Sudachi cannot see. A word's reading must fit its headword
  (信じる is しんじる, never the literary しんずる).
- **Conjugations link to the dictionary form.** Every conjugated form the
  shipped sentences use is an alt (食べました, 行きたい), so example
  sentences bold the whole form. Auxiliaries (です, ます, た, ない, たい,
  だ) are function words and link only in their own forms. A helper verb
  after て (〜ている, 〜てしまう) links nothing; ください is a word of its
  own.
- **する-verbs link to the noun.** 勉強する links 勉強, and する links
  nothing there. A noun Wiktionary only glosses as a verb reads "purchase;
  to purchase" or "to order (+ suru)".
- **Particles, counters and fixed phrases.** Particles are a closed set with
  hand glosses. Compound particles (について, にとって, として) link nothing.
  A noun or suffix after a numeral is a counter (〜人, 〜時, 〜円). Greetings
  are one token each (おはようございます is おはよう).
- **Taught suffixes** (〜さん, 〜君, 〜中, 〜ら ...) link only right after a noun
  or name. After a particle, verb or punctuation Sudachi's suffix is a
  misparse and links nothing. After a numeral it is no taught suffix: 十中八九
  links nothing, and 一等 "first prize" is 等 "class". 君 is the honorific
  only after a name (トニー君). Otherwise it is the pronoun きみ (明日君の車
  "your car tomorrow"). A counter's alts are its bare form and the forms
  with a number (枚, ３枚, 何枚). A suffix's alts are its bare spellings
  (さん, ら). A suffix spelled like another word's headword (中 なか, 君
  きみ) cannot be found bare, so it keeps its uses in the shipped sentences
  whole (会議中, 一日中, トニー君).
- **Adverbial nouns.** Sudachi tags words like いつも, 元気 and あまり as
  nouns that can act as adverbs or adjectives. Each such word gets the one
  part of speech it is mostly used as. A noun that heads a clause-modified
  phrase stays a noun (来た時に is 時, not 時に "sometimes").
- **Months** are one word each (一月 ... 十二月), with digit spellings (1月,
  １月) as alts. 今月, 来月 and 先月 are A1 time words.
- **Glosses.** Hand glosses in `tools/gloss_overrides.json` keep their
  wording. Other glosses take Wiktionary's best sense for the corpus uses,
  with abbreviation senses ("short for 大学") after the others, "the color
  white" shortened to "white", and trailing commas dropped. Katakana words
  are glossed from their own entries only, never from a homophone (ビル
  "building", ジム "gym").
- **Typing:** the pack sets `typing: "pron"`. A word's production slot
  alternates two typed items. "Type the reading" shows the meaning, plays
  nothing, and takes the kana reading from the phone's kana keyboard.
  Katakana and hiragana count as the same (テレビ may be typed てれび, and
  the reverse), half-width kana count as full width, and spaces and the affix
  mark 〜 are ignored. The long-vowel mark ー, small kana and voicing marks must
  match: こうひい is not コーヒー, きやく is not きゃく. "Type the characters"
  speaks the word (with a replay button) and takes its written form or any alt
  (分かる or わかる); a suffix may be typed without its 〜 (年 for 〜年). A
  word still shown by its kana (below its kanji tier) gets the reading item in
  both slots, so its kanji are never asked before they were shown. As of
  engine `122d88a`, "Type the characters" (`acceptTyped`) also carries the
  collision guard added over every lenient letter fold; the fold itself
  (Arabic-script hamza carriers, ة/ى/ھ, Devanagari nukta/chandrabindu) has
  no target in kana/kanji, so the guard has no effect on this pack.

## Sources and licences

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

The pack links no audio and relies on TTS. No licence is non-commercial.
No JLPT word list is used in the build or shipped.

## Level bands

Candidate (lemma, POS) pairs are ranked by a blended frequency score: the
weighted mean of log `wordfreq` rank and log rank in the tagged Tatoeba
corpus.

- **A1** (600 words): every forced item, then the highest-ranked remaining
  words. Forced items are numbers (ゼロ included), days, months, greetings
  (いただきます, ごちそうさま, はじめまして, よろしくお願いします),
  pronouns, question words (何時 なんじ), demonstratives (これ/この/ここ ...
  あちら), the core particles and auxiliaries, the counters an A1 course
  teaches (〜つ 〜人 〜個 〜枚 〜本 〜台 〜杯 〜冊 〜匹, plus time, money and
  age), time words (今月, 来月, 先月), and the A1 core list in
  `tools/forced_a1.txt` (colours 赤 白 黒 青 緑 黄色 茶色 and 黄色い, family
  words 夫 妻 祖父 祖母, and よろしく, こう, ああ, あんな).
- **A2**: the next 700 by rank.
- **B1**: the next 700 by rank.

A word no example sentence can illustrate gives its place to the next word by
rank. This is a simple, reproducible proxy for CEFR level. It is not an
official CEFR or JLPT classification.

To take an engine update, run `git submodule update --remote engine`, then
rebuild with `./build.sh`.
