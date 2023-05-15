---
title: 'Developing a Personal Project, Part 3: Planning'
category: General
---

*In the last part of this series I discussed in brief my thoughts on the research process. Upon reflection, I realize that that topic deserves a more in-depth discussion, but hopefully it at least gave you an idea of my approach and thought process. In this latest post I will be discussing the last "administrative" step before actually getting into the code: planning.*

After much research and deliberation you feel it's time to start coding. You can see your finished project, gleaming in your mind's eye. But while that complete idea is still fresh it would behoove you to enshrine its likeness in written form: a project overview document. As you knuckle down and start cranking out PRs, your high-level perspective will fade as you become concerned with nitty gritty details. You're going to switch contexts, wearing the hat of the programmer rather than the project manager. If you don't write down your plan now, you may find it shifting over time, possibly in very unproductive directions. As you struggle with practical challenges, you may become discouraged or lose sight of your goal. A written roadmap will help guide you, keep you on track, and ultimately tell you when it's time to stop working.

There are many formal methodologies for organizing a project, and you can choose any or none of them. You can organize your project however you like so long as it provides structure. It may be that you wish to work for a company that uses the Agile method. It could be good practice to organize your project in that style. Maybe you just need a "To Do" list. No matter how you choose to structure your plan, I would hold on to the document. Though uncommon, I have seen some interviewers ask for samples of technical writing. For my part, I am most recently familiar with the ShapeUp method, and so that is my current preference, although as you'll see I use it mostly as inspiration.

Below you will see the planning document I wrote for my Discord Bot. I begin with a problem statement, followed by potential solutions. I know that I want to build a bot regardless of utility, so this initial part is mostly a mental exercise. Still, I think it's useful to lay out the problem as it is, along with it's potential solutions and their pros and cons. In a professional setting, this can help assure you that your plan is sound, or point out flaws in your reasoning.

The following section outlines what features the finished project must have, what it would be nice for it to have, and what it will not have. Here I define in concrete terms how the application will behave, but also what it will not do. As I mentioned in previous posts, be ruthless with scope. Cut away anything that isn't weight bearing for your central idea. You'll be grateful for it later.

The last section is where I lay out specific tasks to be completed. In ShapeUp, tasks are organized by "Vertical Slice", such that when a Slice is completed some increment of value is delivered. Seeing as I'm just one guy, and this is sort of a "draw the rest of the owl" scenario, I opted for horizontal slicing instead. Is this a glorified "To Do" list? Certainly, but it's no less useful for that fact. At this level, some people like to use project trackers like Trello or a GitHub project, so if I were so inclined I would assign each bullet point to a card.

As I progressed through these tasks, I would update them with notes to keep track of important info like test keys and test user UUIDs. Occasionally, when some aspect of the API did not behave as I expected, I would tweak this document to reflect this new technical reality. For example, when I first laid out my plan I expected to be using the REST API, but then I realized that Nostrum uses the Gateway API and had to adjust accordingly. When I became frustrated with config or a stubborn error, I would refer back to the overview and note how far I had come. 

Maybe you're a 10x sorcerer who doesn't need this kind of thing. But for a dope like me, staying organized and tracking my progress is critical to maintaining both my productivity and equanimity. See you in part 4, where I actually will show you some code.



---

**Discord Bot - Movie Scheduler - Project Overview**

*Problem*: My friends and I have trouble remembering when our bi-weekly movie night is scheduled for.

*Potential Solutions*:

Become more mindful of our commitments as a group.\
Pros: Humanist. Cons: Hard, unreliable.

Send out google calendar invites when a day is decided on.\
Pros: Reliable. Easy. Cons: Boring, tedious. Resembles work too much

One person write down an automated reminder to themselves.\
Pros: Very Easy. Cons: unreliable, boring

Create a discord bot that has functionality for recording movie night dates and functionality for sending reminders about the chosen date.\
Pros: Fun, good resume project, has scaling potential for other projects. Cons: high effort, overcomplicated

**Accepted Solution: Discord Bot**

Must Haves:
- an application that connects to a discord bot user, deployed in some way that does not rely on a local machine.
- A slash command that accepts a string representing a movie night date and persists it as a date. (i.e. "/movienight schedule 3/6/23")
- A cron job that monitors the stored dates and sends a message to the discord channel when the chosen date approaches. (i.e. "There's a movie night this week on Thursday!")
- proper error messages for misconfigured commands (e.g. "/movienight thrudsay" produces "I don't understand that command")

Nice To Haves:
- Accept multiple date formats (e.g. "/movienight thursday" == <this coming thursday>. "/movienight next thursday" == <the thursday after the coming sunday>, "/movienight Thursday, March 6th")
- Accepts the movie name as part of the initial scheduling string and returns it as part of the reminder. (e.g. "There's movie night this week on Thursday! We're watching Scream 2!")
- Reschedule command that accepts 2 dates, a previously scheduled movie night and the new chosen date (i.e. /movienight 3/6/23 3/12/23)
- Cancel command: accepts a date and cancels the movie for that date. (e.g. /movienight cancel 3/6/23 -> "Movie canceled for 3/6/23")

Won't Haves:
- Will not process unformatted strings (e.g. /movienight 2 weeks from now. /movienight schedule a movie for next friday)
- Will not accept mispelled days
- Will not play spotify songs

(Horizontal Slicing due to manpower issues)

Slices:
- Slice 1 (CRUD Stuff):
    - create database to store movie night data
	    - for dev purposes, this is handled already. We'll need to figure out the deploy situation later though.
    - create migration for movienight table
	    - what fields does a movienight need? datetime, movie title, creator id (discord user id?). Can't think of any others
    - create schema for movienight
	    - date must be unique, spit out error when creating a movie night when one is already booked for that date
    - create context for CRUD movienights
        - required functions:
            - create 
                - collision logic for existing movie nights?
            - get movie night by date
            - get next movie night
            - update movie night with new date (and/or movie)
            - cancel/delete movie night
    - create functions for parsing movienight input strings into dates
    - write tests
- Slice 2 (The Cron Stuff):
    - import Oban (or other cron scheduler?)
    - create cron job to monitor movie queue
    - create worker to trigger reminders
    - write tests
- Slice 3 (The Discord Stuff):
    - create module for sending reminder messages to the channel
    - register a new slash command for scheduling movie night
    - register slash command for checking movie night
    - register slash command for rescheduling movie night
    - register slash command for canceling movie night
    - create gateway event handler consumer to accept data from slash command
    - persist data received by consumer
    - write tests (includes integration)

Useful Info:
- Test Server Guild Id: '**************'
- Application Id: '**************'