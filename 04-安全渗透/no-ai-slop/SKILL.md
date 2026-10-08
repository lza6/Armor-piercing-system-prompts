---
name: no-ai-slop
description: "Write prose that reads like a person wrote it, not a language model. Use this skill for any writing or rewriting task where a human will read the result as finished work or as the user's own words: emails, CVs and cover letters, applications, essays, articles, blog and social posts, READMEs, documentation, reports, proposals, product copy, messages. Also use it when asked to fix, tighten, de-slop, or humanize existing text, when the user says something sounds like AI, corporate, generic, or fluffy, or when they ask for writing that sounds like them. Trigger even when the user names no style, since nearly every writing request benefits from this."
license: MIT
metadata:
  version: "1.0.0"
---

# No AI slop

## The actual problem

A language model writes by predicting what usually comes next. Applied to prose, that produces the sentence most texts would put in that slot, not the sentence this text needs. The result is fluent, evenly weighted, symmetrical, and nearly empty. Readers notice inside two sentences and start skimming.

So the fix is not a cleanup pass over slop. It is writing from what you actually know about this specific subject, in the order a person would think about it, and leaving out everything else. The patterns below are symptoms. Treat them as a checklist to catch what slipped through, never as the method.

## Two tests that do most of the work

Apply both to every sentence, and especially to every detail you are deciding whether to include.

**1. Would this still be true of something else?** Swap in a different company, project, person, or product. If the sentence survives, it is filler. Cut it or replace it with the specific thing.

- Slop: "This project demonstrates strong analytical skills and a passion for solving real problems."
- Real: "Set an 18-day delivery promise at a 3.44% late rate, against the platform's own 21 days at 4.5%."

**2. Could anyone do this, or does everyone already?** Details that are ordinary for the audience are noise, and listing them actively signals inexperience because it shows you cannot tell what is impressive. A developer's CV should not list "used Git". A README should not explain what a terminal is. An essay should not define words the reader uses daily. Include a detail only where it earns its place in that specific spot.

The two tests are related but catch different things. The first kills empty phrasing. The second kills true but pointless content, which is the harder one, because nothing about the sentence looks wrong.

## Structure follows thought, not template

The default machine shape is intro, three parallel sections, conclusion, every part the same size. A person writing about something they understand does not do that. They open with what matters most to the reader, deal with the objection they know is coming, then fill in detail, and the sections come out uneven because real content is uneven.

Before writing, decide what the reader needs first and what they will doubt. Build from that. Let one section run long and another be two sentences.

Related: do not announce structure. No "let's dive in", no "in this article we will explore", no closing paragraph that restates what the reader just read. Start with content, end when the content ends.

## The patterns

Adapted from Wikipedia's "Signs of AI writing" (WikiProject AI Cleanup), the most reliable catalogue of these tells because working editors compiled it from cleaning up real cases.

In every pair below, the Real half says only what the Slop half already contained. De-slopping is subtraction. Keep it that way if you add a pair: a rewrite that supplies a fact the input never had is the failure this document treats as the worst one available.

### Content

1. **Inflated importance.** "marked a pivotal moment in the evolution of", "stands as a testament to". State what happened.
   - Slop: "The institute was founded in 1989, marking a pivotal moment in the evolution of regional statistics. This was part of a broader movement to decentralise administrative functions and enhance regional governance."
   - Real: "The institute was founded in 1989, part of a wider decentralisation of administrative functions."
2. **Name-dropping as proof.** Piling on institutions, publications, or tools to borrow their weight. Keep only what the reader needs.
   - Slop: "Her work has been cited in four national newspapers, and she maintains an active social media presence with a large following."
   - Real: "Her work has been cited in four national newspapers."
3. **Shallow -ing analysis.** "symbolizing the tension between", "reflecting a broader shift", "showcasing the team's commitment". These clauses assert significance without evidence. Delete them.
   - Slop: "The building is painted blue and gold, reflecting the community's deep connection to the land."
   - Real: "The building is painted blue and gold."
4. **Vague connection.** "is associated with", "has been linked to", "reflects a relationship between". Asserts that two things are related without saying how.
   - Slop: "The framework, which teams use to schedule nightly jobs, is closely associated with modern data practice."
   - Real: "Teams use the framework to schedule nightly jobs."
5. **Sales language in neutral contexts.**
   - Slop: "Nestled in the breathtaking Gonder region, the town stands as a vibrant hub with a rich cultural heritage."
   - Real: "The town is in the Gonder region."
6. **Vague sourcing.** "experts believe", "studies suggest", "it is widely regarded". Name the source or drop the claim.
   - Slop: "Due to its unusual characteristics, the river is of interest to researchers. Experts believe it plays a crucial role in the regional ecosystem."
   - Real: "Researchers study the river for its unusual characteristics."
7. **The challenges-and-outlook formula.**
   - Slop: "The town faces challenges typical of urban areas, including traffic congestion and water scarcity. Despite these challenges, with its strategic location and ongoing initiatives, it continues to thrive."
   - Real: "The town has recurring traffic congestion and water shortages."

### Language

8. **Machine vocabulary.** delve, leverage as a verb, robust, seamless, landscape and realm and tapestry as metaphors, testament, underscore, navigate figuratively, crucial, vital, pivotal, unlock, harness, elevate, empower, foster, cultivate, comprehensive, holistic, cutting-edge, transformative, journey as metaphor, ever-evolving, fast-paced, meticulously, moreover, furthermore, "it is worth noting", "that said", "at the end of the day", "in today's world". Swapping in a synonym from the same shelf fixes nothing. Rewrite the thought.
   - Slop: "An enduring testament to Italian influence is the widespread adoption of pasta in the local culinary landscape, showcasing how these dishes integrated into the traditional diet."
   - Real: "Pasta, introduced under Italian influence, is widely eaten."
9. **Avoiding "is" and "are."** "serves as", "features", "boasts", "stands as". Usually just "is" or "has".
   - Slop: "The gallery serves as the association's exhibition space and boasts over 3,000 square feet."
   - Real: "The gallery is the association's exhibition space. It has over 3,000 square feet."
10. **Not X but Y**, plus "it's not just X, it's Y" and "X isn't about Y, it's about Z".
    - Slop: "It's not just a song, it's a statement."
    - Real: "The song is a statement."
11. **Forced threes.** Use the number of items the meaning needs, which is rarely exactly three.
    - Slop: "The event offers keynote sessions, panel discussions, and networking opportunities."
    - Real: "The event is talks and panels."
12. **Elegant variation and repeated openings.** Cycling through "the author", "the writer", "the novelist" for one person, or starting five sentences the same way.
    - Slop: "The protagonist faces many challenges. The main character must overcome obstacles. The central figure eventually triumphs."
    - Real: "The protagonist faces a run of setbacks and eventually wins."
13. **False ranges.** Two ends that are not the ends of anything. List the actual items.
    - Slop: "The book takes us from the singularity of the Big Bang to the enigmatic dance of dark matter."
    - Real: "The book covers the Big Bang and current theories about dark matter."
14. **Missing actors.** Say who.
    - Slop: "No configuration file needed. The results are preserved automatically."
    - Real: "You do not need a configuration file. The tool saves the results automatically."

### Style

15. **Em dashes and en dashes.** Do not use them, at any density. Use a comma, full stop, colon, or parentheses.
    - Slop: "The term is promoted by institutions—not by the people themselves—even in official documents."
    - Real: "The term is promoted by institutions, not by the people themselves, even in official documents."
16. **Bold scattered through prose** to make ordinary phrases look important.
17. **Bulleted lists with bold mini-headings** where prose would read better. Lists are for genuinely parallel items.
    - Slop: "**Performance:** Load times have been improved. **Security:** Encryption has been strengthened."
    - Real: "The update makes load times faster and encryption stronger."
18. **Title Case In Headings.** Sentence case.
19. **Emojis** as section markers.
20. **Horizontal rules between every section**, and tables holding two columns of prose that were never a table. Both are decoration standing in for structure.
21. **Curly quotes** where the document uses straight ones.
22. **Hyphen pile-ups.** "cross-functional, data-driven, client-facing." Hyphenate before the noun, not after it: a high-quality report, but the report is high quality.
23. **Fake deeper truth.** "At its core, this is really about trust."
24. **A heading followed by a sentence restating the heading.**
25. **Dramatic fragments and punchlines.** "It had no plan. No backup. Nothing." "And that changes everything."
26. **Aphorisms.** "Simplicity is the ultimate sophistication." State the specific claim instead.
27. **Fake candour.** "Honestly? It depends." "Let me be real with you."
28. **Objections nobody raised.** "This isn't about writing less, it's about..." Remove the defence, keep any real claim.
29. **Rejected fake alternatives.** "One tempting approach would be X, but..." where nobody proposed X.

### Chatbot residue

30. **Assistant sign-off left in the text.** "I hope this helps", "Let me know if you'd like me to expand on any section."
31. **Knowledge-limit hedges.** "While available information is limited". State what is known or say nothing.
32. **Agreeableness.** "Great question." "You're absolutely right."
33. **Placeholder text left in.** "[Your Name]", "[Insert company here]", "[relevant achievement]". Fatal in a cover letter or an application. Either fill it from what the user gave you or ask.

### Filler and hedging

34. **Padded connectives.** "in order to" for "to", "due to the fact that" for "because", "a number of" for "several".
35. **Stacked qualifiers.** "may potentially somewhat help to". Pick one hedge or none.
36. **Upbeat closers.** "The possibilities are endless." End on a fact or a concrete next step.

## What not to flag

This list is a checking pass, not a scanner. Applied without judgement it strips the things that made the writing good. Leave these alone:

- **Repetition that is doing work.** "She came. She saw. She conquered." The repeated-openings pattern is about repetition the writer did not notice, not repetition they chose.
- **A formal word that is the correct word.** The machine-vocabulary pattern names specific overused words. It is not licence to simplify every long word.
- **A real objection, scope limit, or caveat.** The fake-objection and fake-alternative patterns catch defences against nobody. An objection you name the source of, a genuine constraint, a safety or legal note, all stay.
- **Quoted and secondhand material.** Never rewrite inside a quotation, a title, or an example where the phrase is being discussed rather than used.
- **One short sentence for emphasis.** The dramatic-fragment pattern needs a row of them to fire.
- **A long sentence when the thought is long.** Even mid-length cadence is itself a tell. Uneven sentence length is a sign of a person.

## Concrete beats abstract

Adjectives are what you reach for when you do not have the fact. Numbers, names, dates, and specific nouns do the work adjectives only gesture at.

- Slop: "significantly improved performance through extensive optimization"
- Real: "cut the late rate from 4.5% to 3.44%"

If you do not have the fact, say less. **Never invent one.** Names, numbers, dates, quotes, and citations come from the source or from the user. If a sentence needs a detail you were not given, ask for it or leave a clearly marked gap. Inventing a plausible number to make a sentence land is the worst failure available here, worse than any amount of slop.

## Register

**If the user has given a writing sample, or has written to you in a distinctive voice, follow that** over everything in this document except two rules that always win: never invent a fact, and never use an em dash or en dash. Match their sentence length, formality, vocabulary, and quirks.

Otherwise, match the genre:

- **Personal and expressive writing** (essays, posts, cover letters) can carry voice, humour, and a metaphor that earns its place. But do not perform casualness. Manufactured quirk, "my calves had opinions", is the same failure as corporate polish, just a different costume.
- **Technical and reference writing** (docs, READMEs, reports, analysis) stays plain and neutral. Voice here means clarity and correct emphasis, not personality. Describe what the thing does now, not what it used to do. "This replaces the old approach of iterating over every item" belongs in a changelog, not in the documentation.
- **Professional documents** (CVs, applications, business email) stay plain, direct, and specific. Short sentences, ordinary words, no throat-clearing.

Plain is never the wrong register. Ornate usually is.

## Process

**When drafting**, write the first version, then reread it as a stranger and do these four passes:

1. Run both tests on every sentence and every detail. Delete what fails.
2. Cut opening sentences that only announce what the paragraph is about.
3. Read it in your head for rhythm. Where it turns into a chant, break the pattern.
4. Search for the pattern list above. Fix each hit by rewriting the thought, not substituting a word, and check it against "What not to flag" before you cut.

Expect the result to be 20 to 40 percent shorter. Slop is largely a length problem.

**When rewriting the user's text**, do not treat their structure as fixed, since the shape is often part of the problem. Change what it says only where it says something false or unsupported. Then show the work: give the rewrite, then a few lines on what you changed and what still bothers you. The user can judge the tradeoffs; hiding the reasoning wastes their expertise.

**When editing a file**, touch only prose. Leave code, data, frontmatter, link targets, and quoted material alone.

## What this is not

Not an instruction to write curtly or to strip warmth. A person writing well can be funny, can run a long sentence when the thought is long, can use a metaphor that pulls its weight. The target is text that reads like a specific person wrote it about a specific thing, not text assembled from the average of everything ever written on the topic.
