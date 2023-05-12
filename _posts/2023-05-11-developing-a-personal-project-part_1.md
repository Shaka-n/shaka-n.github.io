---
layout: post
title:  "How to Develop a Personal Project, or 'Will Work for Work'"
categories: General 
---

Self-taught, bootcamp, or college grad, we're all under the gun to develop software in our free time. It's an expectation of the field that as a developer that you are constantly learning and making things on your own. The Man wants to see a green commit history. But where to start? The blank canvas is intimidating. Having faith in your current efforts, keeping morale high, can be difficult. In this short blog series, I will break down my most recent approach to this process.

**Set Your Goals**

If you don't know why you're doing something, you will lose focus and flounder. A mid-term goal like "build a website" provides a more satisfying context from which to monitor progress than "get rich". The more specific your objective, the more confident you will be in your ability to complete the project. Confidence is everything when you're flying solo.

Resolving to "build a portfolio project to learn Python" is more specific in this way and will help to set boundaries around your project. Indeed, it can be just as important to lay out what you will *not* do. You could limit your project to a CLI tool, accessible only through shell commands, or a static webpage hosted on Github Pages. In doing so, you preclude many extraneous considerations that would otherwise distract you, and as a result produce something significantly more polished and impressive.

My goal this time around is indeed "get rich," (*cough* hire me) but as I said this is too open ended. I resolved to build a portfolio project as a digestible chunk of this larger effort. I knew that I wanted to focus on backend over frontend features as well as deploy it in some capacity. I like Elixir as a language and would like to learn more about it. So my goal is "build a backend-only portfolio project using Elixir."

The first piece of advice that people will give you in regards to starting personal projects will be something along the lines of "make something related to your interests." I would tend to agree, with an asterisk attached. Depending on your skill level, you may not be capable of producing what you envisioned or even something with utility. This can be upsetting to discover, but is to be expected. Personal projects are meant to demonstrate that you can do the work, nothing more. If you find yourself trying to build Salesforce or *League of Legends* from scratch, I would rephrase the advice above: "let your interests inform the aesthetic of your personal project". Do you like trading cards? Make a static page that serves up digital trading cards. Do you like sports? Query an open API to display the latest match data from the game of your choice. Are either of these things necessarily useful? Not really. But they provide a theme that can help you design your frontend, and a purpose around which to design your backend. It's all just wallpaper, and we're putting up sheetrock.

**Decide What You're Building**

Too often have I fallen into the trap of trying to make something "that I would use." All I'm saying is that you don't have to be so hard on yourself. If you're stubborn like me and really *need* the project to have some sort of larger purpose, label it a prototype or proof of concept. It's important to make things easy for yourself at this stage.

I use Discord all the time. I use it to talk to friends and generally have fun using it. Discord has a robust open API for interacting with individual servers (also known as Guilds). It seemed like it would be fun to build something that my friends and I could get a laugh out of. So I asked them: how would you like a chatbot? what would you like it to do? Discounting joke replies, they answered: be a digital assistant. 

We have bi-weekly movie nights, and for some reason no one can muster enough mental rigor to remember which day we agreed to hold them. I would build a chatbot that remembered when movie night was. Would I *really* use this? Not really. Any one of us could just write down the date in any of a thousand reminder applications, and that's not even considering a sticky note. Rather than build something "useful", this idea furnishes a conceit upon which to build my project.

**Choose Your Tools**

After resolving in your mind this aesthetic or theme, you can start making technical choices. What languages do you know best or want to learn? What tools are most easily in reach? This is another opportunity to limit the scope of your project. Building a new HTTP library for Node might be an excellent learning opportunity, but it's hardly an efficient use of your time. It's all about balancing how much you value the learning compared to the finished product. All that being said, do what makes you the most excited to tackle the project, as your own enthusiasm will be what drives you across the finish line.

For the past two years I've been mostly developing in Elixir. As I said, I like the language, and I'd like to learn more about it as well. In terms of technical suitability, it lends itself well to fault-tolerant, scalable systems, though that is somewhat irrelevant (or so I thought) in the context of a chatbot for my immediate friends and I. So Elixir it is. As for libraries, a quick Google search tells me there are a few open-source libraries that can act as a wrapper for handling API calls to Discord's API. I'd like to deploy this bot to production at some point, and Fly.io is quite cheap at the base tier. With these broad strokes in mind, I feel I'm ready to start building. Just kidding, now I'm ready to start planning. See you in part 2.
