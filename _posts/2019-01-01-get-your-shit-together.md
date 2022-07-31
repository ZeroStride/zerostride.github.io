---
layout: post
title:  "Get Your Shit Together"
date:   2019-01-01
categories: blog
tags: drunken-rant
---


Many years ago there was #AltDevBlogADay which [Mike Acton](https://twitter.com/mike_acton) started, and asked me to contribute. I agreed, under the condition that I could write about the business of programming, and not write about code things.

The rest of you can write about code things and, let’s be honest here, you’re probably better at it than I am. Even if you’re not, you’re certainly louder.

It is now, with some reluctance...and some nice rye whisky, that I weigh in on the C++20 thing.

# So What’s the Problem?

The story so far: In the beginning, the C++ standards committee was created. This has made a lot of people very angry and been widely regarded as a bad move.

Yes, the syntax is bullshit. The compile time of “modern C++” vs C is bullshit too.

The errors suck, it’s nearly unreadable, it’s miserably debuggable, and it’s certainly not desirable.

That is the entirety of the fucks that I will devote to that part of the problem.

# The Game Industry Does Not Use C++

The game industry does not use C++. We use, I guess, C with “namespaces”, also some vtables, because sometimes-that-makes sense-right-up-until-it-doesn’t-oh-god-why.

So, with that in mind...

Why the fuck are you yelling at the C++ standards committee?

The game industry does not use C++, so it’s not the job of people who teach C++, or people who determine the direction of C++ to cater to the whims of the game industry.

# The Job to be Done in the Game Industry

Put down the pitchforks, and get your shit together.

You need to hire. You need to grow your team, and that means junior developers.

You can make including Boost a fire-able offense (not...totally unreasonable).

You can rant about the C++ standards committee (again, not unreasonable).

But you need junior developers, and you need to grow your junior developers into senior developers, so they can step up when the time comes.

# Re-stating the Problem

Let me show you a different perspective:

> We can’t effectively run our engine development teams, unless a standards committee we have ignored for 30 years does what we want it to do,” is a problem so large that the SEC would make angry noises about it not being disclosed to investors.

And as it stands right now, your CEO would be trying to bullshit a statement to shareholders that wasn’t, “Our most experienced developers are handling this by getting into flame wars on Twitter.”

# Get Your Shit Together

1. Join the standards committee.
    It’s unlikely to solve anything right now, but if you want a seat at the table...literally sit down at the table.

2. Communicate expectations clearly to your teams.
    If the only way a developer can find out not to use Boost is an old joke about getting fired for using Boost, your communication sucks.

3. Automate checking those expectations.
    Run a CI server, compile with `-std=c++98`. Grep for `<boost>` or maybe even do code reviews!

4. Try some technical leadership.
    It’s this crazy thing where a senior developer stops programming all the time, and instead uses their knowledge and experience to make everyone on the team more effective.

5. Mentorship. Just do it.
    Yeah it sucks when people you spent a year or more training just churn out and go somewhere else. Want people to not quit? Maybe try a legit incentives structure, opportunities for advancement, and invest time into helping them grow their professional goals (and give them space for their personal goals as well).

    [Read](https://hbr.org/2018/01/why-people-really-quit-their-jobs). [Some](https://www.forbes.com/sites/valleyvoices/2017/02/22/dont-be-surprised-when-your-employees-quit/#31657bd6325e). [Fucking](https://medium.com/@checkli/why-employees-quit-20-stats-employers-need-to-know-b921c253f767). [Literature](https://www.inc.com/marcel-schwantes/why-are-your-employees-quitting-a-study-says-it-comes-down-to-any-of-these-6-reasons.html).

# For Advanced Players Only...

Want to really make a difference? Get together and talk about the core knowledge required, with the goal of putting together some kind of open coursework on unlearning “modern C++” and learning game engine C(++).

You know what that would mean?

You could **tell applicants what they need to know**, and be at least reasonably proficient with the material in that coursework before applying.

You could have a **template for skill development** once someone is hired.

You could have a solid **path for advancement** within companies, for employees to move into engine code from other disciplines.

You could use it as a **guideline for acceptable coding practices** for your teams.

You could **create opportunities for people**, anywhere, to learn the basics and *create new pools of talent* in places where there currently exists none.

And you want to know what is even better?

You could actually talk to universities in a language they understand, and provide a framework for them to not just teach what the game industry needs, but to understand why we need it.

# We Can Fix It

The problems that people are voicing about C++20 are legitimate. The difficulty of hiring, training and retaining developers is real. It’s ok to feel frustrated and angry.

The good news is that the solutions for keeping the code clean, on-boarding new hires, and *not needing to care what’s in C++20* are all things which will make life better for everyone.

You will make better tech, make better games, make a better workplace, and feel better about it.

These are your teams.

These are your code bases.

These are your companies.

This is our industry.

## Get your shit together.

*Special thanks to: Alex Scarborough, Ben Garney, Robert Blanchette, Tom Forsyth, Mike Acton, and Kat Miller.*
