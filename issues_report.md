### B1 — Mixed/Transliterated Language Detection (WRONG)
- `namma maneyalli current illa` → detected `en`, should be `kn` or `mixed`
- `comprehensive_test` MIXED bucket detected `it` (Italian!) instead of `kn`
- `kasa thagedilla 3 times complaint` → `en` (it is Kannada transliteration)
- **Root:** Language ID model has no transliteration-aware mode. Fix: add transliteration detection pre-pass or retrain lang-ID on romanised Kannada/Hindi.

### B2 — Transliteration→Script Pipeline Broken
- `ration card ge apply madbeku` → "Ratione Chard Gay Apply Madhwaku" (nonsense)
- `bescom ge call madidru response illa` → untranslated echo
- **Root:** Transliterator or translation prompt is not receiving correct language hint. Fix: pass detected language + `transliterated=true` flag to translation step.

### B3 — Escalation Overfiring on Politeness/Filler Words
These queries incorrectly trigger `emergency` or `escalate_complaint`:
- `"potholes on road help"` → emergency/DMER (should be report_issue/BBMP)
- `"where is the hospital? pls"` → emergency/DMER (should be seek_information)
- `"nobody is listening fast"` → distress escalation (should be escalate_complaint)
- `"ration card apply madbeku help"` → distress emotion triggered
- `"street light bad help"` → distress triggered
- **Root:** Keywords `help`, `fast`, `urgent`, `madam`, `sir` are weighted too aggressively in escalation scorer. Fix: treat these as polite suffixes unless co-occurring with genuine distress/emergency terms.

### B4 — Routing Inconsistency (Same Query, Different Departments)
Identical query across runs routes to different departments:
- `"garbage not cleared 4 times"` → BBMP / BBMP_HEALTH / KSRP / Waste Management / BBMP Garbage Department
- `"kasa thagedilla 3 times"` → BBMP / KSRP / BWSSB customer care
- `"strangers near house"` → Police / Police Helpline / DMER / None
- `"ration card apply"` → FOOD_CIVIL / Social Welfare / Public Distribution System / Health Dept
- **Root:** No canonical department mapping table; LLM/keyword layer freely invents department names. Fix: define a strict `intent→department` lookup with aliases; validate all routing output against it.

### B5 — Wrong Department for Intent
- `"house on fire"` → DMER (medical) instead of Fire Department
- `"strangers near house, very scared"` → DMER instead of Police
- `"garbage not picked"` → BWSSB customer care (water dept!)
- `"ಬೀದಿದೀಪ ಕೆಟ್ಟಿದೆ"` (street light) → unclear/None instead of BESCOM/BBMP
- **Root:** Intent→department mapping rules incomplete and not validated. Fix: add Police, Fire Dept, BBMP_ROADS, BESCOM explicitly to routing table.

### B6 — Emotion Classifier Miscalibrated
- `"house on fire help"` → dominant: **fear** (not distress), escalate: False — should escalate
- `"घर में आग लग गई है help"` → dominant: distress, escalate: **False** — missed escalation
- `"told 4 times garbage not cleared"` → neutral (should be anger/frustration)
- `"nobody is listening"` (repeat complaint) → neutral or distress, never anger
- `"ration card apply help"` → distress 0.556 — "help" is triggering false distress
- **Root:** Distress/anger thresholds wrong; `help` keyword inflates distress score; anger scoring misses frustration phrases. Fix: recalibrate per-emotion thresholds; remove `help` from distress keyword list; add frustration markers (repeat, 4th time, nobody listening).

### B7 — Confidence Miscalibration
- Same query `"no power in our house"` gets confidence 0.7, 0.9, and 1.0 across different runs
- `"unclear"` intent always gets confidence 0.0 (should reflect partial scoring)
- Obviously wrong routing (garbage→BWSSB) still shows confidence 0.8–0.9
- **Root:** Confidence is not tied to routing validity or language ambiguity. Fix: penalise confidence on (a) transliteration ambiguity, (b) mixed-language input, (c) no entity extracted, (d) department not in canonical list.

### B8 — Restatement Quality / Hallucination
- `"water problem hai BWSSB please come"` → restatement: "You said you have a water problem with BWSSB. Please come to help." — inverts meaning ("please come" is directed at BWSSB, not the agent)
- Some restatements invented locations or paraphrased wrong language
- **Root:** Restatement prompt has no constraint to preserve speaker direction. Fix: add prompt instruction: "rephrase only what the citizen said, do not invent context or change who is speaking to whom."

### B9 — Fear Queries Not Routed to Police
- `"I am very scared someone is following me"` → intent: unclear, routing: None, escalate: False
- `"strangers roaming near house, very scared"` → sometimes unclear/None
- These should always → Police, with escalation = True
- **Root:** Fear intent class exists in emotion but has no matching NLU intent. Fix: add `seek_safety` or map `fear+stranger` pattern → Police routing with auto-escalate.

### B10 — Restatement Language Mismatch
- For KN queries, restatement sometimes generated in EN (or vice versa)
- **Root:** Restatement prompt does not enforce `respond in detected_language`. Fix: pass `language=kn/hi/en` explicitly to restatement prompt.

---

## Summary Table
| B1 | Wrong language detection for transliteration | ❌ No | P1 |
| B2 | Transliteration→script pipeline broken | ❌ No | P1 |
| B3 | Escalation overfiring on help/fast/madam/sir | ❌ No | P1 |
| B4 | Routing inconsistency (no canonical dept map) | ❌ No | P1 |
| B5 | Wrong department for intent (fire→DMER etc.) | ❌ No | P1 |
| B6 | Emotion classifier miscalibrated | ❌ No | P2 |
| B7 | Confidence not reflecting ambiguity | ❌ No | P2 |
| B8 | Restatement hallucination / meaning inversion | ❌ No | P2 |
| B9 | Fear+safety intent not routed to Police | ❌ No | P1 |
| B10 | Restatement in wrong language | ❌ No | P3 |

---

## What to Fix First (Priority Order)

2. **Canonical department routing table** — fixes B4, B5, partially B3
3. **Escalation keyword weighting** — fix help/fast/urgent as suffix-only signals (B3)
4. **Lang detection for transliterated input** — B1, B2
5. **Fear→Police routing rule** — B9
6. **Emotion threshold recalibration** — B6
7. **Confidence penalty rules** — B7
8. **Restatement prompt constraints** — B8, B10
