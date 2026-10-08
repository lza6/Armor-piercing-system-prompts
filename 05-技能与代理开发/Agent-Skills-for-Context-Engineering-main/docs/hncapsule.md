---
name: karpathy-hn-time-capsule
description: Andrej Karpathy's project auto-grading decade-old Hacker News discussions using GPT 5.1 Thinking to identify prescient and incorrect predictions with hindsight analysis.
doc_type: blog
source_url: https://karpathy.bearblog.dev/auto-grade-hn/
---

https://karpathy.bearblog.dev/auto-grade-hn/
- A lot more detail in my blog post https://karpathy.bearblog.dev/auto-grade-hn/
- GitHub repo of the project if you'd like to play https://github.com/karpathy/hn-time-capsule
- The actual results pages for your reading pleasure https://karpathy.ai/hncapsule/


karpathy
Home Blog

Auto-grading decade-old Hacker News discussions with hindsight
10 Dec, 2025

hnhero

TLDR: https://karpathy.ai/hncapsule/

Yesterday I stumbled on this HN thread Show HN: Gemini Pro 3 hallucinates the HN front page 10 years from now, where Gemini 3 was hallucinating the frontpage of 10 years from now. One of the comments struck me a bit more though - Bjartr linked to the HN frontpage from exactly 10 years ago, i.e. December 2015. I was reading through the discussions of 10 years ago and mentally grading them for prescience when I realized that an LLM might actually be a lot better at this task. I copy pasted one of the article+comment threads manually into ChatGPT 5.1 Thinking and it gave me a beautiful analysis of what people thought + what actually happened in retrospect, even better and significantly more detailed than what I was doing manually. I realized that this task is actually a really good fit for LLMs and I was looking for excuses to vibe code something with the newly released Opus 4.5, so I got to work. I'm going to get all the front pages of December (31 days, 30 articles per day), get ChatGPT 5.1 Thinking to do the analysis, and present everything in a nice way for historical reading.

There are two macro reasons for why I think the exercise is interesting more generally:

I believe it is quite possible and desirable to train your forward future predictor given training and effort.
I was reminded again of my tweets that said "Be good, future LLMs are watching". You can take that in many directions, but here I want to focus on the idea that future LLMs are watching. Everything we do today might be scrutinized in great detail in the future because doing so will be "free". A lot of the ways people behave currently I think make an implicit "security by obscurity" assumption. But if intelligence really does become too cheap to meter, it will become possible to do a perfect reconstruction and synthesis of everything. LLMs are watching (or humans using them might be). Best to be good.
Vibe coding the actual project was relatively painless and took about 3 hours with Opus 4.5, with a few hickups but overall very impressive. The repository is on GitHub here: karpathy/hn-time-capsule. Here is the progression of what the code does:

Given a date, download the frontpage of 30 articles
For each article, download/parse the article itself and the full comment thread using Algolia API.
Package up everything into a markdown prompt asking for the analysis. Here is the prompt prefix I used:
The following is an article that appeared on Hacker News 10 years ago, and the discussion thread.

Let's use our benefit of hindsight now in 6 sections:

1. Give a brief summary of the article and the discussion thread.
2. What ended up happening to this topic? (research the topic briefly and write a summary)
3. Give out awards for "Most prescient" and "Most wrong" comments, considering what happened.
4. Mention any other fun or notable aspects of the article or discussion.
5. Give out grades to specific people for their comments, considering what happened.
6. At the end, give a final score (from 0-10) for how interesting this article and its retrospect analysis was.

As for the format of Section 5, use the header "Final grades" and follow it with simply an unordered list of people and their grades in the format of "name: grade (optional comment)". Here is an example:

Final grades
- speckx: A+ (excellent predictions on ...)
- tosh: A (correctly predicted this or that ...)
- keepamovin: A
- bgwalter: D
- fsflover: F (completely wrong on ...)

Your list may contain more people of course than just this toy example. Please follow the format exactly because I will be parsing it programmatically. The idea is that I will accumulate the grades for each account to identify the accounts that were over long periods of time the most prescient or the most wrong.

As for the format of Section 6, use the prefix "Article hindsight analysis interestingness score:" and then the score (0-10) as a number. Give high scores to articles/discussions that are prominent, notable, or interesting in retrospect. Give low scores in cases where few predictions are made, or the topic is very niche or obscure, or the discussion is not very interesting in retrospect.

Here is an example:
Article hindsight analysis interestingness score: 8
---
Submit prompt to GPT 5.1 Thinking via the OpenAI API
Collect and parse the results
Render the results into static HTML web pages for easy viewing
Host the html result pages on my website: https://karpathy.ai/hncapsule/
Host all the intermediate results of the data directory if someone else would like to play. It's the file data.zip under the exact same url prefix (intentionally avoiding a direct link).
I spent a few hours browsing around and found it to be very interesting. A few example threads just for fun:

December 3 2015 Swift went open source.
December 6 2015 Launch of Figma
December 11 2015 original announcement of OpenAI :').
December 16 2015 geohot is building Comma
December 22 2015 SpaceX launch webcast: Orbcomm-2 Mission
December 28 2015 Theranos struggles
And then when you navigate over to the Hall of Fame, you can find the top commenters of Hacker News in December 2015, sorted by imdb-style score of their grade point average. In particular, congratulations to pcwalton, tptacek, paulmd, cstross, greglindahl, moxie, hannob, 0xcde4c3db, Manishearth, johncolanduoni - GPT 5.1 Thinking found your comments very insightful and prescient. You can also scroll all the way down to find the noise of HN, which I think we're all familiar with too :)

My code (wait, Opus' code?) on GitHub can be used to reproduce or tweak the results. Running 31 days of 30 articles through GPT 5.1 Thinking meant 31 * 30 = 930 LLM queries and cost about $58 and somewhere around ~1 hour. The LLM megaminds of the future might find this kind of a thing a lot easier, a lot faster and a lot cheaper.


-------


Quick new post: Auto-grading decade-old Hacker News discussions with hindsight

I took all the 930 frontpage Hacker News article+discussion of December 2015 and asked the GPT 5.1 Thinking API to do an in-hindsight analysis to identify the most/least prescient comments. This took ~3 hours to vibe code and ~1 hour and $60 to run. The idea was sparked by the HN article yesterday where Gemini 3 was asked to hallucinate the HN front page one decade forward.

More generally: 

1. in-hindsight analysis has always fascinated me as a way to train your forward prediction model so reading the results is really interesting and
2. it's worth contemplating what it looks like when LLM megaminds of the future can do this kind of work a lot cheaper, faster and better. Every single bit of information you contribute to the internet can (and probably will be) scrutinized in great detail if it is "free". Hence also my earlier tweet from a while back - "be good, future LLMs are watching".

Congrats to the top 10 accounts pcwalton, tptacek, paulmd, cstross, greglindahl, moxie, hannob, 0xcde4c3db, Manishearth, and johncolanduoni - GPT 5.1 Thinking found your comments to be the most insightful and prescient of all comments of HN in December of 2015.
