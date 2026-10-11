# FitFindr

> ### 👋 Start here
>
> **New to this repo? Read [RUNNING.md](RUNNING.md) first** — setup, every
> command, and what to do when something breaks.
>
> Once `python test.py` passes:
>
> ```bash
> python app.py listings --full -n 6      # read the data (Milestone 1)
> python app.py fields                    # what you can filter on
> python app.py ask 'vintage graphic tee under $30'
> ```
>
> All three tools are stubs, so that last command will do nothing useful yet.
> That's the starting position.
>
> **The rest of this file is your submission.** Fill it in as you go.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     HOW TO USE THIS FILE

     This is your submission. Fill each section in as you finish the milestone
     it belongs to — don't leave it all to the end.

     Unit 3 asks for the first five sections. Unit 4 adds the five below them.
     Leave the unit 4 sections alone until then; they're here so you know
     what's coming.

     Everything is pasted as TEXT. No screenshots, no images, no video links.
     A typed block of output gets full credit; a picture of the same output
     gets none.
     ───────────────────────────────────────────────────────────────────────── -->

<!-- ═══════════════════════ UNIT 3 — THE BUILD ═══════════════════════ -->

## What This Does

<!-- Three or four sentences: what a user asks for, and what they get back. -->

FitFindr takes a plain-language request for a secondhand piece, like "vintage graphic tee under $30, size M", and searches resale listings from Depop, Poshmark, and thredUp for the best match within that price and size. It then pairs the top find with pieces already in the user's wardrobe and suggests one or two complete outfits. Last, it writes a short, post-ready caption (a "fit card") naming the item, its price, and where it was found. If nothing matches, it stops and tells the user whether to loosen the price, the size, or the keywords.


---

## Tool Inventory

<!-- Four lines per tool. This is worth 2 points and it's the single most
     common place students lose them.

     "Returns a list" earns NOTHING. The description has to say what is IN
     the list.

     The empty case isn't optional either — it's the thing your loop branches
     on, and if you don't decide it here you'll discover it as a crash in
     Milestone 5. -->

### `search_listings`

- **What it does:** Filters listings by price and size, then ranks them by keyword overlap with the description.
- **Inputs:** `description` (str), `size` (str | None), `max_price` (float | None)
- **Returns:** A list of up to 10 listing dicts, best match first, each with `id`, `title`, `description`, `category`, `style_tags`, `size`, `condition`, `price`, `colors`, `brand`, `platform`. A size matches when it equals a whole token of the listing's size, ignoring case (`"M"` matches `"S/M"`, not `"XL"`).
- **When it has nothing:** An empty list `[]`.

### `suggest_outfit`

- **What it does:** Asks the model for one or two outfits pairing the new item with pieces from the user's wardrobe.
- **Inputs:** `new_item` (dict), `wardrobe` (dict with an `items` list)
- **Returns:** A non-empty string of outfit suggestions naming specific wardrobe pieces.
- **When it has nothing:** If the wardrobe is empty, a string of general styling advice for the item.

### `create_fit_card`

- **What it does:** Asks the model for a short, post-style caption about the find.
- **Inputs:** `outfit` (str), `new_item` (dict)
- **Returns:** A 2–4 sentence caption string that mentions the item's title, price, and platform once each.
- **When it has nothing:** If `outfit` is empty, the string `"Can't write a fit card without an outfit suggestion."`

---

## Planning Loop

<!-- Your branch rule, stated as a rule — the condition AND both paths — plus
     the file and function that holds it.

     Like this:
       "If search_listings returns an empty list, put a message in the session
        and stop. Otherwise take the first result and go to suggest_outfit."
        — agent.py::run_agent

     The grader checks your code against what you claim here, so the file and
     function have to be real. -->

**Branch rule:** If `search_listings` returns an empty list, put a message in `session["error"]` telling the user what to loosen (price, size, or keywords) and stop. Otherwise, take the first result as `session["selected_item"]` and go to `suggest_outfit`.

**Where it lives:** `agent.py::run_agent`

**How the query is parsed:** Regex (`agent.py::parse_query`). "under $30" / "$30" becomes `max_price`, "size M" becomes `size`, and the rest of the text is the `description`.

**What moves through the session:** `query` → `parsed` → `search_results` → `selected_item` → `outfit_suggestion` → `fit_card`. On an empty search, `error` is set and the later fields stay `None`.

---

## Sample Run

<!-- Two things go here.

     1. One FULL query and its output, pasted as text.
     2. Your three per-tool terminal tests — the command and what it printed. -->

**One full query**

```
$ python app.py ask 'vintage graphic tee under $30'
  Found:    Y2K Baby Tee — Butterfly Print — $18.0 on depop

  Outfit:   **Outfit 1: Casual Y2K Streetwear**
Pair the butterfly baby tee with Baggy straight-leg jeans and Chunky white sneakers. Layer the Black cropped zip hoodie over your shoulders or wear it open, and accessorize with the Black crossbody bag for an effortless, nostalgic look.

**Outfit 2: Soft Contrast**
Tuck the baby tee into the Wide-leg khaki trousers, cinched with the Brown leather belt. Throw on the Vintage black denim jacket and finish with Chunky white sneakers for a balanced mix of vintage edge and sweet Y2K charm.

  Fit card: Obsessed with this butterfly Y2K baby tee I just scored on Depop for only $18! It's giving total early 2000s nostalgia, and I'm styling it two ways—either super casual with baggy denim or mixed with khakis for a sweet, retro vibe. Which look is your favorite?

```

**The three tools, tested one at a time**

```
$ python -c "from tools import search_listings; print([(l['title'], l['price'], l['size']) for l in search_listings('graphic tee', max_price=30)])"
[('Y2K Baby Tee — Butterfly Print', 18.0, 'S/M'), ('Graphic Tee — 2003 Tour Bootleg Style', 24.0, 'L'), ('Mesh Long-Sleeve Top — Black', 15.0, 'S/M'), ('Vintage Band Tee — Faded Grey', 19.0, 'L'), ('Low-Rise Cargo Pants — Khaki', 27.0, 'W29'), ('Vintage Graphic Hoodie — Faded Black', 26.0, 'L')]
```

```
$ python -c "from tools import suggest_outfit; from utils.data_loader import get_example_wardrobe, load_listings; print(suggest_outfit(load_listings()[0], get_example_wardrobe()))"
**Outfit 1: Effortless Streetwear**
Pair your Vintage Levi's 501 Jeans with the White ribbed tank top and the Vintage black denim jacket. Accessorize with the Black crossbody bag and finish the look with Chunky white sneakers.

**Outfit 2: Casual Grunge**
Combine the Vintage Levi's 501 Jeans with the Oversized grey crewneck sweatshirt and the Brown leather belt. Ground the outfit using the Black combat boots.
```

```
$ python -c "from tools import create_fit_card; from utils.data_loader import load_listings; print(create_fit_card('jeans and white sneakers', load_listings()[0]))"
Scored these vintage Levi's 501s on Depop for just $38 and I am never taking them off. They have that 90s streetwear slouch that is impossible to fake. Just keeping it simple with fresh white sneakers for the ultimate off-duty look.
```

---

## How I Used AI

<!-- Two specific moments. What you asked, what came back, what you changed.

     "I used Claude to help me code" is not enough.

     "I gave Claude my search_listings spec. It returned None on no match
     instead of an empty list, so I changed it" is the level we want. -->

**Moment 1**

- *What I asked for:* A step-by-step plan for building the three tools in `tools.py`, one at a time, with a terminal test for each before moving on to the next.
- *What came back:* A plan for each tool. `search_listings` would filter by `max_price` and by whole-token size ("M" matches "S/M" but not "XL" or "US 9"), drop filler words so every listing doesn't score at least 1, score keyword overlap, and return `[]` when nothing is left. `suggest_outfit` would leave out a `None` brand and switch to general styling advice when the wardrobe is empty. `create_fit_card` would return a fixed message for an empty outfit without calling the model. It also gave tests for each tool, like `search_listings('ballgown', max_price=5)` should return `[]`.
- *What I changed:* Nothing. I followed the plan as written and tested each tool from the terminal before starting the next one.

**Moment 2**

- *What I asked for:*
- *What came back:*
- *What I changed:*

<!-- ═══════════════════════ UNIT 4 — THE TEST ═══════════════════════

     Don't fill these in during unit 3.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Run Log — Before

<!-- Five criteria, five tries each, in this exact format.

     Five, because your criteria are written out of five. Mark each try PASS
     or FAIL, count the passes, and read that count against your target — a
     row targeting 4 of 5 with three PASS cells is MISSED (3/5).

     `python run_eval.py --label before` runs everything and writes the table
     into results/. Paste it here and fill in the verdicts. -->

Produced by `python run_eval.py --label before` (`run_eval.py::main`), caching
off, temperature 0.9. Full output: `results/run_2026-10-10_1914_before.md`.

| Criterion | Target | Try 1 | Try 2 | Try 3 | Try 4 | Try 5 | Verdict |
|---|---|---|---|---|---|---|---|
| 1. A matching query completes all three tools | 4 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 2. An impossible query stops before the second tool | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 3. The item search picked is the item every later tool received | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 4. The fit card reads like a caption, not a product listing | 4 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 5. Search respects the price ceiling and size | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 1 (revised). Everyday phrasings complete all three tools | 4 of 5 phrasings | PASS | PASS | FAIL | FAIL | FAIL | MISSED (2/5) |

**Criterion 1 was revised in unit 4** (see `criteria.md`): the original row is
kept above, and the revised one is underneath. For the revised row each Try
column is one phrasing, run 5 times: Try 1 `band shirt under $25`, Try 2
`jean jacket under $50`, Try 3 `trainers size 8`, Try 4 `purse under $40`,
Try 5 `slacks size M`. Search is deterministic, so each phrasing gave the
same result on all 5 runs. Output: `results/run_2026-10-10_2101_before_criterion1_revised.md`.

```
[2] search_listings (via MCP)
      in:  description='trainers', size='8', max_price=None
      out: [] (empty)
      →    branch: empty, stopping before suggest_outfit

stopped early: yes — Nothing matched your search. You could drop the size 8 filter, or try broader keywords than "trainers".
```
— `agent.py::run_agent`, search by `tools.py::search_listings` via MCP. Platform Sneakers (US 8) is in the data.

Scenarios (`scenarios.py`): C1 `'vintage graphic tee under $30'`, C2
`'designer ballgown size XXS under $5'`, C3 `'90s track jacket in size M'`,
C4 `'denim jacket under $50'`, C5 `'graphic tee under $30, size M'`, all with
the example wardrobe. A sixth, diagnostic scenario (empty wardrobe) also
completed 5 of 5 but isn't one of the five criteria.

**Real output from one try**, pasted as text, naming the file and function
that produced it:

**Criterion 1, Try 1** — `agent.py::run_agent`; trace from `trace.py::step`,
fit card from `tools.py::create_fit_card`

```
[1] parse_query
      in:  vintage graphic tee under $30
      out: description='vintage graphic tee', size=None, max_price=30.0
[2] search_listings (via MCP)
      out: 10 items: Y2K Baby Tee — Butterfly Print, Graphic Tee — 2003 Tour Bootleg Style, Vintage Band Tee — Faded Grey … +7 more
      →    branch: picked Y2K Baby Tee — Butterfly Print
[3] suggest_outfit
[4] create_fit_card

Fit card: Obsessed with this little butterfly tee I just scored on Depop for only $18! It’s giving total Y2K princess, but I’ve been loving styling it with baggy jeans for that casual street vibe or toughening it up with combat boots. Such a good vintage find.
```

**Criterion 2, Try 1** — `agent.py::run_agent`, message from
`agent.py::_no_results_message`

```
[1] parse_query
      in:  designer ballgown size XXS under $5
      out: description='designer ballgown', size='XXS', max_price=5.0
      →    regex
[2] search_listings (via MCP)
      in:  description='designer ballgown', size='XXS', max_price=5.0
      out: [] (empty)
      →    branch: empty, stopping before suggest_outfit

stopped early: yes — Nothing matched your search. You could raise your max price above $5, or drop the size XXS filter, or try broader keywords than "designer ballgown".
```

The trace ends at step 2, so `suggest_outfit` and `create_fit_card` were never called.

**Criterion 3, Try 1** — `agent.py::run_agent` (session), fit card from
`tools.py::create_fit_card`

```
[2] search_listings (via MCP)
      in:  description='90s track jacket', size='M', max_price=None
      →    branch: picked 90s Track Jacket — Navy/White Stripe
[3] suggest_outfit
      in:  new_item='90s Track Jacket — Navy/White Stripe', wardrobe_items=10
[4] create_fit_card
      in:  item='90s Track Jacket — Navy/White Stripe', …

selected_item: 90s Track Jacket — Navy/White Stripe ($45.0, poshmark)
Fit card: Score this 90s Champion track jacket for just $45 on Poshmark and I am obsessed. It has that ultimate vintage athletic energy whether I'm dressing it down with baggy denim or mixing it with prepier trousers. Such a good find!
```

The first search result, the item passed to `suggest_outfit`, and the item passed to `create_fit_card` are the same listing, and the card states its exact price, $45.

**Criterion 4, Try 5** — `tools.py::create_fit_card`

```
selected_item: Denim Jacket — Light Wash, Cropped ($42.0, poshmark)
Fit card: Score this cropped vintage Wrangler denim jacket on Poshmark for just $42! Obsessed with the light wash, so I styled it two ways for that perfect 90s streetwear vibe. Which look is your favorite?
```

3 sentences, price ($42), platform (Poshmark), and no "None".

**Criterion 5, Try 1** — `tools.py::search_listings`, called through
`mcp_server.py::search_listings`

```
[2] search_listings (via MCP)
      in:  description='graphic tee', size='M', max_price=30.0
      out: 2 items: Y2K Baby Tee — Butterfly Print, Mesh Long-Sleeve Top — Black
      →    branch: picked Y2K Baby Tee — Butterfly Print
```

Y2K Baby Tee is $18, size `S/M`, and Mesh Long-Sleeve Top is $15, size `S/M`. Both are ≤ $30 and match M.
The data's other tees, Graphic Tee — 2003 Tour Bootleg Style ($24) and Vintage
Band Tee ($19), are size `L` and were correctly excluded. All 5 tries returned
the same two items.

---

## Verdicts and Diagnoses

<!-- MET or MISSED per criterion against LAST UNIT's target, plus a sentence on
     how you decided.

     Then, for every miss: which of the four places it happened — a tool, the
     loop's branch, the session, or the model's output — AND the mechanism.

     Not a diagnosis:  "The fit card was bad."
     A diagnosis:      "The fit card criterion missed on 2 of 5 items. Both had
                        an empty brand field. My prompt puts the brand in the
                        first sentence, so the card opened with a blank and read
                        like a fragment. The tool worked; the prompt assumed a
                        field that isn't always there."

     Look for a pattern. Three misses on the same tool is one problem, not
     three. -->

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 | A matching query completes all three tools | 4 of 5 | MET (5/5) | Every try had `session["error"] is None` and a non-empty `fit_card`; each trace shows all four steps. |
| 2 | An impossible query stops before the second tool | 5 of 5 | MET (5/5) | Every trace ends at `[2] search_listings (via MCP)` with `[] (empty)`; `suggest_outfit` never appears as a step, and the message names price, size, and keywords. |
| 3 | The item search picked is the item every later tool received | 5 of 5 | MET (5/5) | In each trace the first search result, the `new_item` passed to `suggest_outfit`, and the `item` passed to `create_fit_card` are the same listing, and every card contains "$45" exactly. |
| 4 | The fit card reads like a caption, not a product listing | 4 of 5 | MET (5/5) | Every card was 3 sentences, named "$42" and "Poshmark", and never contained "None". |
| 5 | Search respects the price ceiling and size | 5 of 5 | MET (5/5) | All 5 tries returned the same 2 items, both `S/M` and ≤ $30, checked against `data/listings.json`. The two size-`L` tees were correctly excluded. |
| 1 (revised) | Everyday phrasings complete all three tools | 4 of 5 phrasings | **MISSED (2/5)** | `trainers`, `purse` and `slacks` returned `[]` on all 5 runs and stopped at step 2. `band shirt` and `jean jacket` completed, which counts as a pass, but both picked the wrong item (see diagnosis). |

**Diagnoses**

**Original criteria: nothing missed.** All five passed 5 of 5, which says as much
about the targets as about the agent (see the pattern below).

**Revised criterion 1: MISSED, 2 of 5 phrasings. Place: the tool,
`tools.py::search_listings`.** Mechanism: the score is the number of query words
that appear *literally* among a listing's title, description, category, and tag
tokens. There is no notion of synonyms. "trainers", "purse", and "slacks" don't
appear anywhere in `data/listings.json`, even though Platform Sneakers (US 8),
the Mini Shoulder Bag ($38), and the Wide-Leg Linen Trousers (M/L) all fit those
filters. Every listing scored 0, was dropped, and the loop's branch correctly
stopped on `[]`. The branch and the session did what they should; the tool
returned empty when it shouldn't have.

The same mechanism explains the two "passes". Each listing's score is a count
of distinct matching words, so most items tie at 1, and `scored.sort` is stable,
so **ties are broken by order in the data file**. `band shirt` matched both "band"
(Vintage Band Tee, `lst_033`) and "shirt" (Oversized Flannel Shirt, `lst_003`) at
score 1, and the flannel came first. `jean jacket` matched only "jacket", so 90s
Track Jacket (`lst_004`) beat Denim Jacket (`lst_007`). These completed, so they
pass as written, but neither user got what they asked for. This is the same
fault as `'black leather boots'` → Mary Janes below: one problem in one tool,
not three.

**A knock-on effect in the branch's message:** `_no_results_message` suggests
loosening whatever filters were set ("drop the size 8 filter", "raise your max
price above $40"). Here those filters weren't the cause; the word was. So the
message points the user at the wrong fix.

**The pattern: my criteria test what can't fail.** Criteria 2, 3 (the ID half),
and 5 check deterministic code in `agent.py::run_agent` and
`tools.py::search_listings`. They have no randomness, so they can only fail if the
code is broken, and 5 of 5 was guaranteed once the code was right. That's a
reasonable regression check, but it isn't a measurement. The two criteria that
touch the model passed easily for reasons I can name:

- **Criterion 4 restates my own prompt.** `tools.py::create_fit_card` tells the
  model to "Write a 2-4 sentence caption … Mention the price and the platform
  once each". The criterion checks the same three things, so it measures whether
  Gemini follows a direct instruction, which it did 25 of 25 times. It doesn't
  check whether the card reads like a caption rather than a product listing. One
  card (criterion 5's scenario, Try 3) was written in all lowercase with emoji
  run-ons; it would still pass the sentence check, and nothing in the criterion
  could flag it.
- **Criterion 1 can't see the failure its own reason names.** The "why" says the
  likely miss is a phrasing like "shirt" for a listing that says "tee", which
  scores zero in `search_listings`. But the criterion only counts queries that
  *already* match a listing, so that failure is outside what it tests. The only
  miss it can still catch is a model outage.

**Were my targets low? Yes.** The 4-of-5 targets on criteria 1 and 4 allowed a
miss in exactly the places that turned out to be the most reliable. Every test
also used one fixed query per criterion, so five tries of the same input mostly
re-tested the same path.

**The criterion I'd tighten: 4.** Run it on five *different* items, including
ones with `brand: None`, and require 5 of 5. Add a check the prompt doesn't
hand to the model, such as the card naming at least one wardrobe piece from
the outfit. That would test whether the card reflects the styling rather than
just the listing.

**Seen but not measured:** `'black leather boots under $60'` picked Platform
Mary Janes as the best match. Keyword overlap in `tools.py::search_listings`
ranks on any shared word ("black", "leather"), not on the item type. None of
the five criteria checks whether the top result is the kind of item asked for.



---

## Loop Trace

<!-- One full run, printed step by step, with the MCP call visible in it.

     `python app.py ask '...' --trace` once you've added the trace.step()
     calls in Milestone 2.

     Worth pasting BOTH the happy path and the empty-search path. The empty
     one should be visibly shorter, because it stops. If your two traces are
     the same length, your branch isn't working — and this is the fastest way
     anyone will ever find that out. -->

**Happy path** — `python app.py ask 'vintage graphic tee under $30' --trace`

```
[1] parse_query
      in:  vintage graphic tee under $30
      out: description='vintage graphic tee', size=None, max_price=30.0
      →    regex
[2] search_listings (via MCP)
      in:  description='vintage graphic tee', size=None, max_price=30.0
      out: 10 items: Y2K Baby Tee — Butterfly Print, Graphic Tee — 2003 Tour Bootleg Style, Vintage Band Tee — Faded Grey … +7 more
      →    branch: picked Y2K Baby Tee — Butterfly Print
[3] suggest_outfit
      in:  new_item='Y2K Baby Tee — Butterfly Print', wardrobe_items=10
      out: **Outfit 1: Casual Y2K Streetwear** Pair the butterfly baby tee with Baggy straight-leg jeans and Chunky white…
[4] create_fit_card
      in:  item='Y2K Baby Tee — Butterfly Print', outfit='**Outfit 1: Casual Y2K Streetwear**
Pair the butterfly baby te…
      out: Obsessed with this butterfly Y2K baby tee I just scored on Depop for only $18! It’s giving total early 2000s n…
```

**Empty search** — `python app.py ask 'designer ballgown size XXS under $5' --trace`

```
[1] parse_query
      in:  designer ballgown size XXS under $5
      out: description='designer ballgown', size='XXS', max_price=5.0
      →    regex
[2] search_listings (via MCP)
      in:  description='designer ballgown', size='XXS', max_price=5.0
      out: [] (empty)
      →    branch: empty, stopping before suggest_outfit

  Nothing matched your search. You could raise your max price above $5, or drop the size XXS filter, or try broader keywords than "designer ballgown".
```

The empty run stops after two steps; the happy path runs four.

**On the MCP move:** `search_listings` is registered in `mcp_server.py` with
FastMCP, and `agent.py::run_agent` now calls it through
`mcp_client.call_tool("search_listings", {...})` instead of importing it from
`tools.py`. The registered input types (`description: str`,
`size: str | None`, `max_price: float | None`) already matched the Tool
Inventory. Nothing behaved differently: I ran the direct call and the MCP call
side by side on three queries (10 results, 4 results with a size filter, and an
empty search) and the lists were identical. That's because the tool already
returned plain JSON-safe values (strings, floats, lists, `None`). Writing the
description did surface two size rules I had never stated: "One Size" listings
match any size, and only a single size token matches (`"9"` works, `"US 9"`
doesn't).

After the improvement, the synonym change in `tools.py` reached the MCP tool
with no edit to `mcp_server.py`: the server imports the real function, and
`call_tool` starts a fresh server on every call. Direct and MCP results still
matched (`purse`, `graphic tee size M`). What *didn't* update itself was the
published description, which still said "keywords" with no mention of
synonyms. A caller reading only the description would have been wrong about
what matches, so I added one clause to it. Code changes reach the server on
their own; the description has to be updated by hand.

### Failure Modes

| Failure | How I triggered it | What the agent said | Handled before? |
|---|---|---|---|
| Empty search | `python app.py ask 'sequined astronaut suit size XXS under $3'` | "Nothing matched your search. You could raise your max price above $3, or drop the size XXS filter, or try broader keywords than "sequined astronaut suit"." | Yes — the search branch stops before `suggest_outfit`. |
| Empty wardrobe | `python app.py ask 'denim jacket under $50' --empty-wardrobe` | Two general outfits for the Wrangler cropped denim jacket (cargo pants + ribbed tank; slip dress + ankle boots), then a fit card. No crash, no empty string. | Yes — `suggest_outfit` switches to general advice when `wardrobe["items"]` is empty. |
| Model unavailable | Broken `GEMINI_API_KEY`, new query `'black leather boots under $60'`, cache off | Before: `ModelUnavailable: The model rejected your API key…` printed by `app.py`'s catch-all, and the run died with exit code 1. After: "Found Platform Mary Janes — Black Patent ($55.0 on depop), but couldn't reach the model to run suggest_outfit: The model rejected your API key. Check GEMINI_API_KEY in your .env file, or create a fresh key at aistudio.google.com. Your search worked; try again once the model is reachable." | No — added a `ModelUnavailable` handler in `agent.py::run_agent` around `suggest_outfit` and `create_fit_card`. It puts the message in `session["error"]` and ends the loop, so `serve.py` and `run_eval.py` get a session back instead of an exception. |

While I was testing the empty wardrobe, the real service also returned
`503 UNAVAILABLE` partway through a run. That raised the same
`ModelUnavailable`, so the handler covers an outage as well as a bad key.


---

## The Improvement

<!-- What you changed, why your diagnosis pointed at it, and the after-run in
     the same table format. One change, measured properly.

     `python run_eval.py --label after` -->

**What I changed:** one thing, in `tools.py::search_listings`. A query word now
scores if the listing contains that word *or any word in its synonym group*
(`_SYNONYM_GROUPS`: tee/shirt, sneakers/trainers/kicks, bag/purse/handbag,
trousers/pants/slacks, denim/jean/jeans, sweatshirt/jumper/crewneck). It is still
one point per query word, so a synonym can't double-count. I left the loop, the
branch, the prompts, and the tie-breaking alone, and the tool's inputs, return
shape, and empty case are all unchanged.

```python
score = sum(1 for w in words if _SYNONYMS.get(w, {w}) & tokens)   # was: if w in tokens
```

**Which failure it was meant to fix:** revised criterion 1, which MISSED at 2 of 5
phrasings. The diagnosis put it in the tool: `search_listings` only matched
literal words, so "trainers", "purse", and "slacks" scored 0 against listings
that fit. I didn't change the tie-breaker separately. Once "shirt" also matches
"tee" and "jean" matches "denim", the right items score 2 instead of tying at 1.

### Run Log — Before / After

Same scenarios, same agent apart from the one change, 5 tries each, caching off.
Before: `results/run_2026-10-10_1914_before.md` (criteria 2–5) and
`results/run_2026-10-10_2101_before_criterion1_revised.md` (revised criterion 1).
After: `results/run_2026-10-10_2117_after.md`. For criterion 1 each Try column is
one phrasing (band shirt, jean jacket, trainers, purse, slacks), and a phrasing
passes only if all 5 of its runs completed.

**Before**

| Criterion | Target | Try 1 | Try 2 | Try 3 | Try 4 | Try 5 | Verdict |
|---|---|---|---|---|---|---|---|
| 1. Everyday phrasings complete all three tools (revised) | 4 of 5 | PASS | PASS | FAIL | FAIL | FAIL | MISSED (2/5) |
| 2. An impossible query stops before the second tool | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 3. The item search picked is the item every later tool received | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 4. The fit card reads like a caption, not a product listing | 4 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 5. Search respects the price ceiling and size | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |

### Run Log — After

| Criterion | Target | Try 1 | Try 2 | Try 3 | Try 4 | Try 5 | Verdict |
|---|---|---|---|---|---|---|---|
| 1. Everyday phrasings complete all three tools (revised) | 4 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 2. An impossible query stops before the second tool | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 3. The item search picked is the item every later tool received | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 4. The fit card reads like a caption, not a product listing | 4 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |
| 5. Search respects the price ceiling and size | 5 of 5 | PASS | PASS | PASS | PASS | PASS | MET (5/5) |

The original criterion 1 query (`vintage graphic tee under $30`), now a
diagnostic, still completed 5 of 5 with the same item (Y2K Baby Tee).

**Real output from the after run** (`agent.py::run_agent`, search by
`tools.py::search_listings` via MCP):

```
[2] search_listings (via MCP)
      in:  description='trainers', size='8', max_price=None
      out: 1 items: Platform Sneakers — White Chunky Sole
      →    branch: picked Platform Sneakers — White Chunky Sole
[3] suggest_outfit
[4] create_fit_card
```

What each phrasing selected, the same on all 5 runs:

| Phrasing | Before | After |
|---|---|---|
| `band shirt under $25` | Oversized Flannel Shirt | Graphic Tee — 2003 Tour Bootleg Style |
| `jean jacket under $50` | 90s Track Jacket | Denim Jacket — Light Wash, Cropped ✓ |
| `trainers size 8` | `[]`, stopped | Platform Sneakers — White Chunky Sole ✓ |
| `purse under $40` | `[]`, stopped | Mini Shoulder Bag — Tan Leather ✓ |
| `slacks size M` | `[]`, stopped | Wide-Leg Linen Trousers — Natural ✓ |

**Did it help, and how do I know:** Yes, for the failure it targeted. Revised
criterion 1 went from MISSED (2/5) to MET (5/5): the three phrasings that
returned `[]` now complete on all 5 runs, and all three find the exact listing
I named in `criteria.md`. Criteria 2–5 held at 5/5, so the change didn't break
the branch, the session hand-off, or the price and size filter. Search is
deterministic, so the before/after difference is the code change and not run
to run noise.

It is not a clean win, though, and three caveats matter:

1. **The map was written with the test words in view.** I added exactly the
   synonyms my five phrasings needed. A 5/5 here mostly shows the map contains
   those five words. A fair test would use phrasings chosen *after* the change,
   ones that aren't in the map. "runners" for sneakers and "tote" for bag both
   still return `[]` (checked with a direct `search_listings` call after the
   change).
2. **`band shirt` still doesn't pick the Vintage Band Tee.** It now picks the
   Graphic Tee — 2003 Tour Bootleg Style, which arguably *is* a band shirt, but
   it isn't the item I named. Both score 2 and the tie still goes to data-file
   order. Criterion 1 counts completion, so it passes, but the tie-break
   problem from the diagnosis isn't gone; it's just hidden in more cases.
3. **Recall went up and precision went down.** Treating "shirt" as "tee" pulls
   in shirts that aren't tees: `graphic tee under $30, size M` now returns
   3 items instead of 2, adding the Vintage Polo Shirt. Criterion 5 still
   passes because it checks only price and size, but a polo is not a graphic
   tee, and nothing in my criteria would catch that.

---

## What's Still Broken

<!-- For each criterion still missed: what you'd do, and why you stopped where
     you did. "I ran out of time" is fine if it's true. Pretending nothing is
     left is not. -->

**No criterion is still missed.** All five, with criterion 1 revised, are MET (5/5) in
`results/run_2026-10-10_2117_after.md`. That doesn't mean the agent is right.
These are the problems I know about that my criteria don't catch:

**1. The synonym fix only covers the words I tested.** `_SYNONYM_GROUPS` holds
exactly what my five phrasings needed. "runners" for sneakers and "tote" for
bag still return `[]` after the change. *What I'd do:* rerun criterion 1 with
five new phrasings chosen without looking at the map; if they miss, replace
the hand-written map with a model call that rewrites the query into words the
listings actually use. *Why I stopped:* the unit allows one measured change,
and growing the map until a test passes would be tuning to the test.

**2. Ties are still broken by data-file order.** `band shirt` scores 2 on both
the Graphic Tee (2003 Tour) and the Vintage Band Tee, and the Graphic Tee wins
only because it comes first in `listings.json`. *What I'd do:* add a
tie-breaker that weights title matches above description matches, and add a
criterion that checks the top result is the named item, not just that the run
completes. *Why I stopped:* this was a second change, and making it in the
same run would have hidden which one caused the improvement.

**3. Search got less precise.** Treating "shirt" as "tee" adds the Vintage Polo
Shirt to `graphic tee under $30, size M`, and earlier `'black leather boots'`
picked Platform Mary Janes. Criterion 5 checks only price and size, so it
passes either way. *What I'd do:* add a criterion on result relevance, e.g. the
top result's `category` matches the item type asked for, in 5 of 5. *Why I
stopped:* none of my five criteria measured it, so I had no before number to
improve against.

**4. Criterion 4 still mostly checks that the model follows my prompt.** It
asks for the same three things `create_fit_card`'s prompt demands, so 5/5 is
close to guaranteed. *What I'd do:* run it on five different items, including
ones with `brand: None`, and require the card to name a wardrobe piece from
the outfit. *Why I stopped:* I named it as the criterion to tighten but chose
to revise criterion 1, because criterion 1 was measuring the wrong thing and
criterion 4 is only measuring too little.

**5. The empty-search message can point at the wrong fix.** Before the
change, `trainers size 8` told the user to "drop the size 8 filter", which
wouldn't have helped because the word was the problem. Words outside the
synonym map still get that advice. *What I'd do:* when a search is empty,
re-run it without the filters, and only suggest loosening a filter if that
second search finds something. *Why I stopped:* the branch passed its own
criterion, and I kept this unit's changes to the one in `tools.py`.



<!-- ═════════════════════════════════════════════════════════════════════

     SUBMISSION CHECKLIST — unit 3

       [ ] criteria.md has five numbered criteria, each with a target
       [ ] Each criterion has a reason underneath it
       [ ] All five unit 3 sections above have real content
       [ ] Tool Inventory: all three tools, inputs WITH TYPES, a specific
           return value, and the empty case
       [ ] Planning Loop names the branch rule and agent.py::run_agent
       [ ] Sample Run: one full query plus the three per-tool tests, as text
       [ ] At least four new commits
       [ ] Repository URL submitted — WRITE IT DOWN, you submit the same one
           next unit

     SUBMISSION CHECKLIST — unit 4

       [ ] mcp_server.py exists with one tool registered
           (or a written record of exactly where the rewire broke)
       [ ] Run Log — Before, five criteria, five tries each
       [ ] Real output pasted underneath, naming file and function
       [ ] A verdict on every criterion
       [ ] A diagnosis for every miss, naming a place AND a mechanism
       [ ] Loop Trace, with the MCP call visible in it
       [ ] All three failure modes triggered and handled
       [ ] One improvement, with Run Log — After in the same format
       [ ] What's Still Broken
       [ ] At least four new commits
       [ ] The SAME repository URL as last unit

     Do not delete and recreate this repository. Your commit history is what
     shows your criteria existed before your results did.
     ═════════════════════════════════════════════════════════════════════ -->

---

📖 **How to run this project: [RUNNING.md](RUNNING.md)**
