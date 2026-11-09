# prompt-check

A Claude skill that makes Claude ask before it answers. When a request is missing something that would change the answer, or rests on a premise that isn't true, Claude flags it and asks instead of guessing. When it does answer, it maps every claim to a source, labels what's verified and what isn't, and ends with what it couldn't confirm.

## Why I built it

I think the things in this skill are super valuable, and more people should be doing them. Asking for sources, checking what's verified, making sure the request is clear before you start. I don't always remember to do them either.

Having Claude ask me questions first cuts down on waste, and it gives me more confidence in what I'm creating with AI. It also makes me feel more like a human in the loop, because the questions show me more of its reasoning.

It's mutually beneficial. I get better quality answers, and I understand more of what's happening.

## What it does

Before answering, the skill checks the request for:

- **Gaps that would change the answer**, such as audience, purpose, scope, country or plan
- **Wrong premises**, like "why did X shut down?" when X didn't

If it finds either, Claude asks short, specific questions and stops. It doesn't add a partial answer or background "in the meantime." It only asks about gaps that pass three tests: the answer would change the content, the request doesn't already answer it, and it's about substance rather than formatting.

When it answers, it:

- maps each claim to the source that supports it
- labels claims as verified, reported, inferred or unverified
- ranks sources, preferring primary ones, and names weak sources and conflicts of interest
- dates sources on fast-changing topics like prices
- shows conflicting sources side by side and says which is more authoritative
- ends with what it didn't cover or couldn't confirm
- keeps decisions about people, money and legal terms with the human

## Example

Prompt: *Why did Google discontinue Google Docs?*

**Without the skill**, Claude corrected the premise in one line, then wrote a history of Google changes that might have caused the confusion, with no sources for any of the dates.

**With the skill:**

> I couldn't find any sign that Google has discontinued Google Docs. Its own Docs API release notes have entries as recent as October 12, 2026.
>
> What prompted the question? The answer depends on which of these it is:
> 1. You saw an announcement or article saying Docs is shutting down
> 2. Docs stopped working or your documents are missing
> 3. You're thinking of a different change, like the 2012 move to Google Drive
> 4. Something else

## How I tested it

I wrote nine test prompts, each targeting one behavior: a vague request, a fully specified one, a false premise, a statistics question, the skip phrase, a link that can't be opened, a hiring decision, several questions in one sentence, and "your call." I ran them in fresh chats, scored each as pass, partial or fail, and changed one thing at a time between rounds so I could tell which change made the difference.

| Behavior | Final result |
|---|---|
| Asks before answering open-ended requests | Pass |
| Catches hidden gaps (country, plan) | Pass |
| Asks only about gaps that matter | Pass, after a fix |
| Catches false premises | Pass, with occasional extra background |
| Refuses to summarize a file it couldn't open | Pass every run |
| Keeps a hiring decision with the human | Pass |

## What I found and fixed

**1. Claude flagged gaps, then answered anyway.** The skill said to ask and wait, but when Claude already had related information, it would note the gap and keep going. I added a "hard stop" section that lists every way it was leaking (partial answers, background, troubleshooting for a situation I never described) and example replies that end with the question.

**2. The skill wasn't loading at all on some prompts.** This was the big one. When I turned off my personal preferences to test the skill on its own, it stopped working on simple-looking questions. I rewrote the description, the part Claude reads to decide whether a skill is relevant, and it started loading on open-ended requests. It still didn't load on simple-looking questions.

I first guessed the failures were about questions on Anthropic's own products, since all three failing tests were about Claude. I tested that by swapping in Google and Netflix versions of the same traps. They failed too, so that theory was wrong. Then I forced the skill to load with `/prompt-check`, and both passed cleanly. That proved the instructions worked and the problem was triggering.

**3. A bug in my own rules.** The skill told Claude to skip its checks for "simple facts," which also skipped the premise check. But false premises almost always look like simple questions. Now two quick checks (premise and answer-changing gaps) always run.

**4. The fix for #1 made it over-ask.** On a request that was already complete, it asked four questions when only one mattered. I added three tests a question has to pass before Claude asks it. On the rerun it asked one question, the right one, and it still asked which country on the Netflix test.

## Known limitations

- **The skill doesn't load by itself on simple-looking questions.** Claude decides when to use a skill, and questions like "why did X happen" look like easy lookups. A skill can't guarantee it runs on every message. My workaround is one line in my Claude profile preferences: *"For factual, research, analysis, advice or drafting requests, use the prompt-check skill before answering."* With that line on, it caught both the false premise and the hidden country gap without being forced. For anything high-stakes, typing `/prompt-check` guarantees it.
- **The hard stop isn't perfect.** In one run it asked the right questions but added a paragraph of background first.
- **It's guidance, not enforcement.** Even always-loaded instructions are followed very consistently but not mechanically. The only truly automatic option is a hook in Claude Code, which isn't available in the regular Claude app.
- **Not yet fixed:** answers after the "quick:" skip phrase are longer than they should be, and secondary sources are occasionally labeled "verified."
- **My test design had a flaw.** Three of my nine tests were about the same topic, which hid the real cause of the failures for a while.

## How to use it

1. Add the `prompt-check` folder to Claude as a skill.
2. Add the preference line above to your Claude profile preferences.
3. For anything important, start your message with `/prompt-check`.
4. For a fast answer without questions, start with `quick:`.

## Prior art

Asking clarifying questions first isn't new. The "Lyra" prompt on Reddit made it popular, and there are Claude Code prompt improvers on GitHub that run automatically through hooks. What I wanted was both halves together: asking before answering, plus sourcing and verification standards for the answer itself, built for the regular Claude app rather than for coding.

## How I built it

I built this with Claude. I set the requirements and made the design calls: flag gaps instead of filling them, keep the new rules conditional so answers don't get cluttered, and change one thing at a time while testing. Claude drafted the skill text. I ran every test and decided what each result meant and what to change next.

## What I learned

The instructions were never the main problem. The skill worked every time it loaded; it just wasn't loading on the questions that needed it most. I spent two rounds improving wording before testing whether Claude was even reading it. Next time, the first thing I'll check is whether the thing I'm fixing is actually running.

## License and credit

Licensed under [your choice of license]. If you use or adapt this skill, please credit:
**prompt-check by Sarah Vine** (https://github.com/vinesc1234/prompt-check-skill)
