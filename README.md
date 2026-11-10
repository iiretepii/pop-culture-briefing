# Automated Daily Pop Culture Podcast

A daily pop culture podcast for an audience of one, written by Claude while everyone sensible is asleep, so that its listener never again has to piece together what happened in the world from six apps, four feeds and a comment section.

Every morning at about 4:12am Eastern, a scheduled Claude routine reads the last 24 hours of entertainment news, checks the charts, verifies the numbers, remembers what it said yesterday, and writes one briefing to Google Drive. That briefing is the only source document for a Gemini Notebook (formerly NotebookLM) Audio Overview, so by breakfast there are two cheerful podcast hosts ready to explain the weekend box office, the week's breakout song, the Formula 1 strategy story and whether the Bears found a new way to make it interesting, in roughly the time it takes to make coffee.

## Origin Story

All anecdotes in this section are, at best, lightly true and, at worst, entirely made up. The prompt is real. The legends are not.

It began with a simple question: what actually happened today? Answering it took the author about forty minutes and six apps. One news app led with a story another had apparently decided did not exist. A third covered it as a triumph, a fourth as a catastrophe, and both were very sure. In between sat the feeds, where every update arrived wrapped in someone's opinion, a discount code, or both: an influencer reviewing a trailer while plugging a supplement, a hot take on the hot take, and a video titled "you won't believe what happened" that, it turned out, you would. Somewhere underneath all of that, on a good day, was the news. Occasionally even good news, usually just below an ad for a mattress.

The author's first attempt at a fix was a sticky note on the fridge that said "ask a well-informed friend." The note was effective but did not scale, because the friend eventually wanted to talk about other things, such as their own life.

So the friend was replaced with a prompt. The first version was three sentences long. The current version is about fifty-eight thousand characters, which is what happens when you let a person who enjoys rules write rules for a robot. Most of those characters exist to do what the feeds never would: say where a fact came from, call a rumor a rumor, and skip the part where someone tries to sell you something. The rest exist because something, at some point, went hilariously wrong.

The algorithm took the news badly. It has since been sending increasingly personal "we miss you" notifications, and at last report had worked through denial and bargaining and was somewhere around offering a free trial of something nobody asked for. We wish it well. We are not coming back.

## What You Get Each Morning

The briefing runs in a fixed order, because the audio hosts read the document top to bottom and treat it as their run-of-show:

| Section | What's in it |
|---|---|
| EPISODE_OVERVIEW and DIRECTOR_NOTES | A one-line steering header (under 500 characters, short enough to paste into Gemini Notebook's customize box) and five labelled notes for the hosts on tone, continuity, story priority, segment coverage and guardrails |
| Opening note | Timing, and the as-of dates of the charts used |
| The Culture in Sixty Seconds | A quick preview of the day and its connecting thread |
| The Big Stories | The four or five stories that matter most today |
| On Screen: Film & TV | Box office, streaming rankings, reviews, premieres, renewals and cancellations |
| On Air: Music | Chart number ones, streaming leaders, the week's breakout song or artist |
| Celebrity, Culture & Gaming | Confirmed celebrity news, viral moments that reached real outlets, fashion, books and games |
| The Business of Entertainment | Deals, earnings, labor, AI in entertainment, and a short entertainment-stocks note |
| Pit Wall: Formula 1 | Results, strategy, regulations and FIA decisions, pitched at a listener who knows what an undercut is |
| Chicago Scoreboard | Final scores, a one-line recap and the standings for the Bears, Bulls, Blackhawks, Cubs and White Sox (omitted on days nobody played) |
| What to Watch & Listen | Three to five new recommendations, with where to find them and what the critics thought |
| Coming Up | The next seven days of releases, premieres, award shows, live events and the next Grand Prix |

A normal day runs 2,400 to 3,400 words. The days of the week change the shape: Monday leads with the full weekend box office and a Weekend Recap, Wednesday picks up Netflix's weekly Top 10 (which lands Tuesday afternoon, after the briefing has already gone to bed), Thursday previews the weekend's openings, Friday turns recommendations into a Weekend Watchlist, Saturday is a Week in Review and Sunday is a Week Ahead.

Formula 1 gets its own race-week logic: a full race report the day after a Grand Prix, session-by-session coverage on race weekends, a preview midweek, and a polite two or three sentences in a quiet week.

Awards season gets the full red carpet. On Oscars, Grammys and Emmys ceremony days there is an Awards Preview, the morning after there is an extended Awards Edition of 4,500 to 6,000 words with every major winner checked against the official results page, and the morning after nominations there is an expanded Nominations section. Winners are never reported from predictions or social media, a rule written in the full knowledge of how tempting that is at 4am.

## How It Works

The routine is a single long prompt, [`prompts/daily-pop-culture-briefing.md`](prompts/daily-pop-culture-briefing.md), run unattended on a schedule. It never asks questions, which makes it the only colleague in history to never schedule a quick sync.

First, it loads its memory. Before any research, it reads the latest story tracker from Drive and the two most recent briefings, so continuing stories (a film's box office run, a song's weeks at number one, the awards race, the F1 title fight) are told as updates rather than reintroduced every morning like a goldfish meeting its castle.

Then it researches in parallel. The lead agent launches seven subagents in a single message (eight on awards days): Film & Box Office, TV & Streaming, Music, Celebrity, Culture & Gaming, Business of Entertainment & Coming Soon, Formula 1, and Chicago Sports. It runs a front-page sweep of the trades itself, so nothing big slips through. Each subagent gets a self-contained brief with the date window, its sources, its search caps and the storylines it is tracking, and returns ranked, dated, sourced findings ending in a self-count of its tool use.

Searching follows a strict order: the built-in WebSearch first, the anonymous Parallel web search as the backup, and Perplexity as the last resort, each with hard per-agent caps. Every news search must name the date, because an undated search for "box office" will cheerfully return a story from 2019 and present it as breaking. Data comes from connectors where possible: Viberate for music charts and artist data, Financial Modeling Prep for the stocks note (Netflix, Disney and Sony), and the chart pages themselves for the rest (The Numbers, Netflix's Tudum Top 10, FlixPatrol's dated daily pages for the other streamers, kworb, Rotten Tomatoes, Metacritic, formula1.com and the FIA's decision documents).

The source list is long and opinionated for a reason. In testing, one fetch tool served Deadline's homepage as it looked in 2017, and the briefing briefly had the confidence of a man who has just woken from a very long nap. Hence the stale-cache warning, the rule that every page and every figure must carry a date inside the window, and a short list of sites the routine is told never to fetch at all.

Next it consolidates and verifies. Findings are merged, deduplicated, matched to tracked storylines and ranked by cultural significance rather than by how loudly a publicist shouted. Every top story and every number is confirmed against a primary source or two independent outlets. Rumors stay labelled "reportedly," and the routine checks every candidate against an Aired Items Log of the last fourteen days so the hosts don't announce the same casting news three mornings running with fresh astonishment each time.

Then the quality gate. The draft is checked for word count, attribution, section order, spoilers (none for anything released in the last two weeks unless the spoiler is the news, with a warning), and the audio rules below. It is revised until it passes.

Finally it saves and remembers. The briefing is saved to the "Pop Culture Briefings" folder in Drive as `YYYY-MM-DD - Pop Culture Briefing`, and a brand-new tracker goes into the "Pop Culture Story Tracker" subfolder as `YYYY-MM-DD - Pop Culture Story Tracker`. Earlier trackers are never edited or deleted, which gives the system a better memory than most people and a more honest one than the internet. The tracker has ten parts: active and dormant storylines, charts and figures, the release calendar, Formula 1 standings, the Chicago teams, the awards calendar, everything already recommended in the last three weeks, a metaphor log and the aired items log. The run ends with one push notification naming the day's biggest story, with links to both documents, and a final report of run time, search usage and anything that broke.

## The Audio Rules, or Why the Hosts Sound Like Friends and Not a Spreadsheet

Whatever is written gets said out loud, so the prompt has a section on audio hygiene, each rule a small monument to a past episode.

The Rule of Two allows at most two figures per film, show, song or race. Before it, the hosts read box office numbers like a lottery drawing. Say each thing once: a story told in The Big Stories is only referred back to later, never retold. No pipeline talk, ever: nobody wants two podcast hosts discussing a chart site's caching behavior, and the briefing never mentions tools, agents or fetches. If Anthropic shows up in a story, the briefing discloses exactly once that Anthropic makes the AI that compiles it, like a commentator mentioning they used to play for the team.

There is also a connecting thread instead of a running metaphor. An image is allowed only in the first two sentences of the preview and the last sentence of Coming Up, it may never come from film, TV, music, showbiz or motor racing, and nothing playful goes anywhere near sad news. A metaphor log blocks any image family used in the last fourteen days. Legend says this rule was born the week every single story was compared to a soufflé. Legend may be exaggerating. The soufflé declined to comment.

The briefing used to end with an EPISODE_FOCUS paragraph. It has been retired in favor of the EPISODE_OVERVIEW and DIRECTOR_NOTES at the top, on the principle that you give the director their notes before the show, not after the credits.

## Set It Up Yourself

You will need a Claude account that can run scheduled routines, and these connectors available to the routine: Google Drive (memory and saving), Viberate (music data), Financial Modeling Prep (the stocks note), Parallel Search in both its anonymous and authenticated forms, and Perplexity (the last-resort search). The built-in WebSearch and WebFetch tools do the rest.

1. Create a folder in Google Drive called "Pop Culture Briefings," and inside it a subfolder called "Pop Culture Story Tracker." Copy both folder IDs from their URLs.
2. Open [`prompts/daily-pop-culture-briefing.md`](prompts/daily-pop-culture-briefing.md) and fill in the two folder IDs listed at the top. If you don't root for Chicago, swap in your own teams, their CBS Sports schedule URLs and your local outlets in the Chicago sports paragraphs, and accept that the routine will judge you quietly.
3. If you are not on Eastern time, change the time zone and the timing paragraph in the prompt to match.
4. Create a scheduled routine with everything below the line in that file as its prompt, set to run daily before you wake up. The original runs at 4:12am Eastern, a time chosen because nothing good has ever happened on the internet at 4:12am, so there is nothing to miss.
5. When the first briefing lands in Drive, add it as the only source in a Gemini Notebook, paste the EPISODE_OVERVIEW line into the Audio Overview customize box, and generate the episode.

The first run will find no tracker, say so, and build one from scratch. From the second morning on, it remembers.

## File Map

| Path | What it is |
|---|---|
| `README.md` | This file, which you are reading instead of doomscrolling. Well done. |
| `prompts/daily-pop-culture-briefing.md` | The full routine prompt, sanitized, with a placeholder table at the top |

The six apps, for the record, are still on the phone. They have been moved to the last screen, into a folder called Later, where they are learning patience.
