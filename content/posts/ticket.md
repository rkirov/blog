---
title: "Ticket to Ride: First Journey simulation authored with AI"
date: 2025-01-18T11:52:17-08:00
draft: false
---

## Ticket to Ride: First Journey simulation authored with AI

After playing [Ticket to Ride: First Journey](https://www.daysofwonder.com/ticket-to-ride/kids/first-journey/)
with my family recently, I got the urge
to analyze some statistical properties of the game.
This seemed like an excellent opportunity to experiment with Cursor AI and explore more automated approaches to coding. Here's what I discovered.

## Results

After a couple of hours the AI and I produced the following analyses:

1. The 32 tickets in the game can be categorized by their optimal paths:

- 3 tickets (best) requiring 3 trains and 1 intermediate city
- 11 tickets requiring 4 trains and 1 intermediate city
- 5 tickets requiring 4 trains and 2 intermediate cities
- 6 tickets requiring 5 trains and 1 intermediate city
- 7 tickets (worst) requiring 5 trains and 2 intermediate cities

1. The probability distribution for game completion time in single-player mode (assuming reasonably good strategy) is roughly:
    - 13% of games finish < 21 turns
    - 15% in 21 turns
    - 27% in 22 turns
    - 27% in 23 turns
    - 13% in 24 turns
    - 5% in > 24 turns

The code for this can be found [in my github repo](https://github.com/rkirov/ticket_to_ride).

## Experience of using AI

I approached this project by maximizing AI involvement through Cursor AI's chat/composer interface, rather than relying on simpler autocompletion features. This led me to reflect on my role in the development process and the areas where I still performed human intervention:

1. High-level strategic direction - while AI could have contributed to planning, I had a clear vision for the analyses I wanted to perform.
1. Low-level data input - I typed in the description of the routes graph
and tickets by hand. That took a good 10-15min and in the future I should try to take a picture
and as ChatGPT.
1. Code Review - I entered a basic description of the game rules for the 
simulation for the AI to produce that code. However, the poorly spec'ed parts resulted in bugs where the simulation didn't match how the game is really played. For example, that happened around the decision about what to do when after a ticket completion a new tickets is drawn that happens to also be already completed. Once I identified the bug through reading of the code, I just had to describe it in chat to get a proper fix in.
1. Validating code outputs - Another bug in the code was spotted by seeing the simulation going into an infinite loop on occasions. I know that is impossible during play (technically not a proof, but probably can be easily proven), so there must be a bug. After describing the issue, the AI
could identify the underlying bug and resolve it without a problem.

## Efficiency analysis

I am not 100% sure, but my gut feeling is that it was in totality faster, especially
considering this is a tool that I am unfamiliar with and with more usage it is likely
that I will get better at prompting and so on.

Maybe I need to wait a few weeks and try to write code to achieve the same results (listed above)
but with no AI or only with Copilot autocompletion only and compare the
timing for a true comparison.

## Code quality

The code is of medium quality, the overall structure can be improved, the style is a bit inconsistent, but there are no glaring issues. That said, I could have spent more cycles iterating with AI on prompts around that, but my goal was to finish a correct implementation in minimal amount of time. The code quality was good enough for me to perform the correctness read through of the code.

## Will the AI workflow get faster in the future?

Most of my time was spent on:

1. Game implementation
1. Bug detection and validation of their resolution

For 1) my hope is that
a photo of the map could be turned into the abstract definition of routes.
While of course, that feels within the reach for AI, it is substantially different task
from writing code. There is a lot of visual stuff on the map that could make this
hard. Note that verifying the correctness of this task, is almost as hard as the task, so
if AI is mostly correct, it won't be of much use as I still have to go through each route to check.

For 2) I think bugs crept in mostly due to sloppy specification on my part and inability
of the AI to self-identify that where those were. I even tried to ask it the
high-level question, find problems with this code as a whole, but it then
started suggesting irrelevant (to this project's goals) improvements around
performance and readability. Finding the bugs necessitated me reading the code and
spotting those jumps from the high-level ambiguous game description to a
particular implementation that might disagree with my unstated expectations.

In retrospect, I should have tried to input directly the game rules as written by
the game makers, but as board game players know those are still ambiguous ([see](https://www.reddit.com/r/tickettoride/comments/1d02ygh/rule_question_about_ticket_to_ride_first_journey/)
for example).

## AI is too accommodating

This appears to be a general theme with AI assistant tools - they still are too accommodating. There are times where they should push back against
bad or incomplete specifications, but currently they don't.

One example is for the ticket analysis the AI knew immediately to use Dijkstra. However, in the game there is a subtle point, while distance / number of cards needed is the main resource, the number of intermediate cities is a secondary resource that also matters, because controls the number of turns needed to build the in between connections, even if you have all the required cards. So the classic Dijsktra's needs to be
modified to minimize the number of hops when distance is equal when deciding on the best path.

My initial prompt didn't account for that and of course the produced code didn't either. Moreover, the AI didn't stop to ask - does the number of hops matter? Once I realized this is important to add to the analysis, a simple high-level description like the paragraph above was enough for the
 AI to update the code ([sha](https://github.com/rkirov/ticket_to_ride/commit/1db7ceefd2752ad6a91d59b319e5e67bda7aa176).)
 
## Future of Programming

At the end this exploration lead me to the timeless truth that coding is not just telling
the computer what to do, but also a process of refining our own sloppy and ill-defined ideas.

Paradoxically, if we want to take ourselves completely out of coding tasks,
we need to getter better at specifying them some other medium. But currently there is no better medium of specification than the code itself.

As Minsky remarked in his 1967 [paper](https://rafal.io/static/papers/why_programming_is_minsky.pdf)
programming is a good medium for poorly formulated ideas, and as I am arguing the best medium to turn them into a better formulated ones.

Given that, my expectation is that coding will remain relevant,
but the mode of engagement will shift from authoring directly, to reading and discussing it with the AI that authored it.

## Consequences

If the future of programming looks like that, technical interviews and software engineering education will have to also adapt and answer:

- How do we assess candidates' ability to guide and refine AI-generated code? Why do we still ban AI usage in all tech hiring?
- Should programming education still begin with manual coding, or should we develop new pedagogical approaches that include AI earlier? Should there be an underpowered version of AI that grows in power as one progresses in their education?