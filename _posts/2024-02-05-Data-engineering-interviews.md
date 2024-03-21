---
layout: post
date: 2024-02-05
title: "Data Engineering Interviews"
categories: [ python, programming ]
tags: 
description: "The field is now flooded"
featured: true
hidden: true
image: assets/images/interview.jpg
---

I recently started a new job (an exciting few months, as shortly after my last post my partner and I went to Japan.. got engaged! Then work began). So I've been a bit busy, or maybe just distracted and as yet unsolidified in my writing habit for it to carry over successfully. 

I noticed an interesting trend back when I was interviewing (being an interviewee rather than interviewer) which was the tendency of everyone to do live coding tests. They're a bit awkward as you get the problem sheet while sharing your screen and face-video, then the interviewer has to sit through the myriad of faces you inevitably make while working the problem out in your editor. I'll be honest, I don't like them - it feels like someone has their head of your shoulder and is breathing into your ear while you do something under time pressure. Not the most pleasant in any scenario. But I understand why they're there.

One of the recent things I've observed which has gobsmacked me explains the need for these quite a bit.

Sitting in WeWork you can see lots of different kinds of people working. (To be clear, yes, as I write this post WeWork is still open). I have the benefit of sitting with my coworkers and that makes for some nice socialising which is otherwise missing in such a venue. It also means I can spy on people more effectively because I have a large group of people to wander around between, dotted into the hot desks. 

My revelation: I've noticed a shocking amount of people using chatGPT for coding. Including people who I'm pretty much sure don't know how to code. 

What's perhaps ominous for me, is that a lot of these people were coding in python.

I don't really know how to express how depressing the problem feels. It was compounded when my colleague, who is learning programming, submitted a PR with a code snippet that was copy-pasted from somewhere. Having spent a while parsing it and making a review I realised it simply didn't do what he hoped it would. It means I spent time reviewing something under the assumption it was a solution instead of a faulty facsimile of one - Potentially more time than it would have taken for me to write an actual solution.

Obviously this is an expected problem occurring from tools like chatGPT. Which is that to some extent, what should be a "helper" or something to enable generation of boilerplate is instead being used blindly as a solution. The ability to output such stuff is obviously going to be high. Generating code is easy. Especially generating bad or inappropriate code. Reviewing code & suggesting better solutions on the other hand, is time consuming and feeling increasingly like it'll be a tiring and unrewarding job, especially given how we rarely if ever factor in such work in Agile sprint ticketing. 

The problem then comes back to hiring. What happens when you hire lots of people who _appear_ to be able to code, or appear to have learned to code very fast on the job? 

Unless you're the kind of manager who sits with your colleagues, reviews their PRs and pair programs occassionally then it's possible you're just never going to pick up on the actual skill sets you've hired. Possibly because people like me will be doing the above (reviewing, correcting) - and increasingly spending more of their time doing the above. Which to be clear, _is often time spent that doesn't count as effective work time by senior management_. Reviewing work, helping colleagues with their code, helping colleagues learn - In my experience none of this is ever ticketed or often understood by non-technical project management. So as a manager, you might well think that everything is great. Code is being merged in so it's fine right?

Ultimately it won't be fine and probably the outcome will be that people like me will leave areas where there is a high rate of this happening. Inevitably there may be programming languages where this is also more prevalent - my worry is that python will be one of them. In these scenarios, engineers like me should probably skill up more in other languages which are less popular or less easy to parse for lay people using chatGPT.

It comes down to learning, being interested, wanting what you make to be good and efficient. People who use LLMs but don't then ask it to explain the code it's splerged out might not learn simply because they don't want to. Those who use it as a tool might learn faster than those of us who figured it out ourselves 10-20 years ago. But testing for learning ability/resilience/design thoughtfulness is hard to do and so, instead, I predict we'll see longer and longer live coding tests. 