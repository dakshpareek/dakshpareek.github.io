---
title: "Improving German Search: Splitting The Compound Word"
date: 2026-09-25
tags: ["search", "full-text-search", "german", "postgresql"]
draft: false
description: "Our full-text search failed on German compound words like Heißluftballonfahrt. Here's how splitting compounds fixed it and lifted top-5 accuracy from 87% to 98% for compound queries."
---

# Improving German Search - Part 1: Splitting The Compound Word

We have multi locale data with us and we use full text search(FTS) to query data and return results to user.

## Problem
English user who comes to our gifting platform searches for `Hot air balloon ride` and FTS queries as `hot + air + balloon + ride`. Products title stored in database also uses same seperate words like `Hot Air Balloon Ride`. So, FTS works well for English users.

But German joins the same concept into one word and it can join it in different ways. For example, the German translation of `Hot air balloon ride` is `Heißluftballonfahrt`. If a German user searches for `Heißluftballonfahrt`, the FTS query will be `heißluftballonfahrt`, which may not match the product title stored in the database as `Heißluftballon Fahrt` or `Heißluft Ballon Fahrt`.

## Solution
As we have seen `Heißluftballonfahrt` is actually a compound word made up of three words: `Heißluft`, `Ballon`, and `Fahrt`. So, we can split the compound word into its constituent words and then perform FTS query on those words.
Using this we would able to match the product title stored in the database as `Heißluftballon Fahrt` or `Heißluft Ballon Fahrt` which means same as `Heißluftballonfahrt`.

## Some German language points to know

1. German joins words into compounds (Komposita)
   - Example: `Heißluftballonfahrt` is "hot air balloon ride" in one word.
   - Why it matters: FTS compares whole words and it cannot find a word inside a longer word.

2. The last part carries the main meaning (the head)
   In a compound, the last part names the thing and the parts before it describes it.
   - Example: `Paarmassage` is a type of massage and `Ballonfahrt` is a type of ride ("Fahrt").
   - Why it matters: The important word is at the end. A prefix match (ballon:*) finds "Ballonfahrt" but it cannot find "Heißluftballonfahrt".

3. Compounds can contain compounds
   - Example: "Heißluftballonfahrt" is made up of "Heißluft" and "Ballonfahrt", and "Ballonfahrt" is made up of "Ballon" and "Fahrt".
   - Why it matters: A compound can be made up of other compounds, so we need to split recursively.

4. Joining letters (Fugenelemente)
   German sometimes puts a few letters between the parts of a compound: s, es, er, en, e, n, and so on.
   - Example: Eule + n + Motiv = Eulenmotiv. Tag + es + Zeit = Tageszeit.
   - Why it matters: The splitter must skip these letters otherwise "eule | nmotiv" fails.

5. One concept, many spellings
   The same product can use a compound word, a hyphen or a phrase.
   - Example: "Ballonfahrt", "Heißluftballonfahrt", "Fahrt im Heißluftballon" or "Dinnershow" and "Dinner-Show".
   - Why it matters: A visitor and a product title often use different forms. Splitting brings them to the same parts.

6. Some words only look like compounds
   A word can have parts that are real words but it is not a compound of them.
   - City names: "Regensburg" is not "Regen" (rain) + "Burg" (castle). "Düsseldorf" is not "Düssel" + "Dorf". "Frankfurt" is not "Frank" + "Furt".
   - Other words: "Wahrzeichen" is not "wahr" (true) + "Zeichen" (sign).
   - English words: "Workshop" is not "work" + "shop".
   - Why it matters: A computer cannot tell these words from real compounds and that's why we have to build compound-stopwords.json

## Implementation flow

### Step 0: Fn tokenize(text)
- toLocaleLowerCase('de'): first we will lowercase with the German language rules.
- then any character that is not a letter ends a word like space, hyphens, digits, brackets, &, - all separate words and we would not include those characters.
- return the list of words.

### Step 1: Iterating over text
- calling tokenize(text) to get the list of words.
- we will do count of each word and store it in a map.
- after getting that map, we will iterate over that word and if its short or its frequency is less than 2, we will skip it. We will only consider words which are 4 or more letters and have frequency 2 or more times.

### Step 2: Creating STOPWORDS
- we will iterate over all the cities present in database.
- for each city we will be calling tokenize(city) to get the list of words.
- if word is 8 or more letters characters, we will add it to the STOPWORDS list. This is because we don't want to split city names like "Regensburg" or "Düsseldorf" into parts.
- this will give us stopwords json file which we will use in the next step to split the compound words.

### Step 3: Building Compound Splitter function

```javascript
splitter = new CompoundSplitter(lexicon, stopwords)
splitter.split("Heißluftballonfahrt") // returns ["Heißluft", "Ballon", "Fahrt"]
```
- CompoundSplitter(lexicon, stopwords): The splitter loads the word list and the stop list one time when service starts.
- split(word) returns the parts of one word, or nothing. It uses these rules:
  - If the word is in the stop list, it returns nothing. "Regensburg" stays whole.
  - If the word has fewer than 8 letters, it returns nothing. A shorter word is not long enough to have two parts of 4 letters.
  - It tries each cut position. The left part must have 4 or more letters, and it must be in the word list.
  - After the left part, it skips a joining letter if there is one: s, es, n, en or e. "eule | n | motiv" becomes "eule" + "motiv".
  - The right part must have 4 or more letters. If the right part is in the word list, the cut is valid.
  - If the right part is not in the word list, the splitter calls split on the right part again (recursion). "sonne | n | untergang" becomes "sonne" + "unter" + "gang".
  - If more than one cut is valid, it picks the best cut:
    i. It picks the cut with the fewest parts.
    ii. If two cuts have the same number of parts, it picks the cut whose least common part occurs more often.
  - It keeps each result in a cache, so the same word is split only one time.

#### Three examples

```
heißluftballonfahrt
  heißluft | ballonfahrt         2 parts, least common part 3
  heißluftballon | fahrt         2 parts, least common part 26  ✓
  → heißluftballon + fahrt

eulenmotiv
  eule | nmotiv                  "nmotiv" not in list ✗
  eule | n | motiv               joining letter "n" ✓
  → eule + motiv

weinprobe
  wein | probe                   "probe" not in list ✗
  → no parts (a synonym handles this word)
```

### Step 4: Where the splitter runs
The splitter runs at two different times:
- When a product is saved: SearchVectorService uses allParts(word) on products.
- When a visitor searches (the query side): The query builder uses only split(word), which is the first level.

### Step 5: Building the query
- Compound words: Each German query word with parts become `(word | (part1 & part2))`.
- Why AND and OR: All query words stay required, which is the same as today. The OR is only between two spellings of the same word, so it never makes the query wider.

## Results that we see
- Our top 5 result accuracy went from 84% to 89%. This is the percentage of correct products among the first 5 results
- **Top-5 accuracy for queries with compound words:** 87% → 98%.
- No query got worse.
- 3 of the 30 real queries now show the correct product first. The other 27 did not change.

## What splitting does not solve
- Words with no shared part such as "Weinprobe" and "Weinverkostung" need synonyms.
- Spelling mistakes still needs to go with fuzzy match.
- The splitter can take a real letter as a joining letter. The fix is if we find that we add that to the stop list.
