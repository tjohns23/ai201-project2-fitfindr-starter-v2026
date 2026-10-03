# Acceptance criteria — FitFindr

Five criteria that say what "working" means for this agent, written in unit 3
**before** any results existed.

An acceptance criterion names a target: a number, a count, a rate, or something
a person could plainly observe. *"The agent handles errors"* is an opinion.
*"When search returns nothing, the agent stops before calling the second tool,
in 5 of 5 tries"* is a criterion.

Under each one, write a sentence or two on **why that target** and not a
stricter one. A reason that says something about your tools, your loop, or the
data earns credit; *"80% seemed reasonable"* does not.

> Missing your own targets next unit costs you nothing. Setting a target so
> easy you can't miss it does.

**Two are written for you. You write three.**

---

## 1. A matching query completes all three tools

Given a query that matches at least one listing, the agent completes all three
tool calls and returns a fit card — in at least 4 of 5 tries.

**Why this target:**
My search is a plain keyword match, so a phrasing that uses a word not in any
title, description, or tag (e.g. "shirt" for a listing that says "tee") scores
zero and stops the run. Two of the three tools also depend on a model call that
can fail.

---

## 2. An impossible query stops before the second tool

Given a query that matches no listings, the agent stops before calling
`suggest_outfit` and returns a message naming what to change — 5 of 5 tries.

**Why this target:**
This path never touches the model. It's a plain empty-list check in my own
code, so it should behave identically every time. There's no randomness to
excuse a miss.

---

## 3. The item search picked is the item every later tool received

For a matching query, `session["selected_item"]["id"]` equals
`session["search_results"][0]["id"]`, and the fit card names that item's exact
price — in 5 of 5 tries.

**Why this target:**
Passing the item from search to the next tools is my own code reading and
writing the session, not the model, so it should never drift. The price check
uses the model's output, but the exact price is in the prompt, so I expect
it to be copied correctly every time.

---

## 4. The fit card reads like a caption, not a product listing

For a matching query, the fit card is 2–4 sentences, mentions the item's price
and platform, and never contains the word "None" — in at least 4 of 5 tries.

**Why this target:**
The card comes from the model at temperature 0.9, so wording and length vary
from run to run. One overlong card or a dropped platform in five is plausible;
two would mean my prompt isn't constraining it enough.

---

## 5. Search respects the price ceiling and size

For the query "graphic tee under $30, size M", every item in
`session["search_results"]` has `price <= 30` and a size that matches M by my
token rule (e.g. "M" or "S/M", never "XL" or "US 9") — in 5 of 5 tries.

**Why this target:**
Price and size filtering are deterministic code with a rule written in my Tool
Inventory. The data has sizes like "XL" and "US 9" that a naive substring check
would wrongly match, so this criterion tests that rule directly. No model is
involved, so there's no reason to allow a miss.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     UNIT 4 — read this before you change anything above.

     If a criterion turns out to be BROKEN rather than merely unmet, you can
     revise it, and that earns credit. But never delete or edit the original
     line. Add the revision underneath it, like this:

         ## 4. Something about the fit card

         The fit card is different every time.

         **Why this target:** ...

         > **Revised in unit 4:** For 5 different items, the 5 fit cards share
         > no opening sentence.
         >
         > **Why revised:** "different" wasn't checkable — two cards that
         > differed by one word still counted. The new version is something I
         > can actually score.

     That's a revision because the criterion couldn't be MEASURED.

     Lowering a target because you missed it is not a revision, and it costs
     you the point:

         ✗ "I said the empty search stops it 5 of 5 times, but I got 3 of 5,
            so 3 of 5 is more realistic."

     A number you missed stays where it is, gets diagnosed, and gets a fix
     attempted. That's where the points are.
     ───────────────────────────────────────────────────────────────────────── -->
