---
layout: post
date: 2026-07-03
title: "Does company growth push you into having more repos?"
categories: [ tech ]
tags: 
description: "Monorepos vs Polyrepos"
featured: false
hidden: false
image: assets/images/polyrepos.png
---

I've been thinking about the problems associated with having hundreds (thousands) of repositories in my current job leading the data function at Time Out. Now, I know this problem is actually called Polyrepos (not ManyRepos, MaxiRepos, or RepoMaxxing. Probably because Repomaxxing would be making your repo look very attractive rather than having hundreds of them) and that having many repositories is not a problem, really (yes, it's really _not a problem_.. I repeat).

My theory at present is that the reason some companies have absolutely stonking amounts of repos is either 
1. Because the company is big or
2. Because somebody fucked up

Now this is maybe a controversial opinion and polarising for a reason - That reason probably being because both approaches have big upsides and big downsides.

Monorepos in many ways make sweeping changes immensely easier. They allow you to switch out libraries or modules across the board, without dependencies breaking or needing to coordinate other teams to update their tooling. They allow greater oversight across code (meaning you can find shared functions much easier) and standardised enforcement (easy when it's one thing to enforce) meaning you can share checks, tests and build steps. Commits are thereby atomic - Changing two related or inter-tied services is in one commit, not in two, and so with that you don't need to worry about your deployment timing being bang-on, nor worry about one service breaking and the other being fine with the upgrade. If they do break - you can see all the history in one place and you already know what related services exist because you've probably structured them in a sensible way via directories. Overall it's a fairly easy setup when there aren't too many people futzing with it.

On the other hand, they make contributions by lots of teams difficult (blocked by deployments, other teams, PR priority ordering), increase complexity (potentially hundreds of branches), make ownership more fuzzy, increase the blast radius (change one thing, affect many teams) which can cause big issues if your testing isn't thorough and can easily leak into high dependency or coupling between services and general purpose code without anyone noticing.

Polyrepos on the other hand do actually allow a lot of individuality in how your code is structured, checked, tested and built. Different teams or members within teams can have their own repos, meaning they can deploy on different schedules and to different degrees of security/checks/scopes (meaning you might deploy a few quick changes to your batch data pipelines and run those within a day, but you might be testing out your API repo for a longer timeframe). With that setup, the batch engineers don't need to worry about the API being stuck in staging, nor about a build being broken and holding them up. They also don't have to worry about constraints applying to them that don't make sense. They equally don't have to worry about someone updating (breaking) a shared function which breaks code unrelated to a specific PR as much since other repos will be using pinned versions of shared code (ideally with good semver). Overall it means the security feels a bit tighter, the ownership clearer and the blast radius of any changes is shorter. It means you can hand your contracting team a fresh repo and not worry too much. 

However, it also means that you're at great risk of
1. Exponentially diverging tooling (everyone picks their favourite!)
2. Lack of key standards across repos
3. A lot of potential issues keeping everything in sync (update one repo -> bump the other 1000.. update 40 repos -> Bump everything? Test everything?)
4. OR an alternative to 3: you don't bump your dependencies/repos because you're busy / you think you don't need to -- stuff gradually gets staler (version rot), and then changing code in those lagging repos becomes difficult / slow / painful

I think (assuming you wish to sensibly update repos i.e. not item 4) the 'testing everything' is probably the very problematic bit here, or so it seems for me. Having bots bump everything automatically in your 1000 repos to the new minor version of a shared private package is doable, and having individual tests per repo (your unit tests) for the service is a good safety net - But what about across cross-services? What about your integration or E2E tests? Do you need to spin up your 300 services all together to check your application plus your auth plus users plus page recomm plus person recomm plus (...) all works together? Is that very expensive? Do you run every service in a staging environment? Equally, what happens when you have to bump the major version and expect lots of breaks?

If you make one change and have one repo that's used by 10 others - then those 10 repos are actually used by 10 others .. you have 100 repos to test. This doesn't seem like an unrealistic scenario to me for a company with thousands of services. If you decide to run them all in staging but you're doing a gradual roll-out - Are you testing all 'newest' 100 repos? In which case - can your deployment process efficiently deploy all 100 services in a timely manner (timely enough for a company of thousands, where changes are potentially desirable on a high velocity)? Or do you get stuck in a 'partial' staging environment, testing version A or repo 1, version B (the desirable change) of repo 2 and version C (because someone has a hot fix) of repo 3 - all at the same time?

A risky things feels like 
- The feedback loop being too slow, unless you have a beast of a deployment / CI system
- High overhead for spinning all these things up - Potentially many versions at once in many environments? (10 teams each testing their own 100 repos)
- Not catching your actual issues anyway because staging schemas, data or traffic being different to production

What I can imagine working in a polyrepo setup is maybe something like repo criticality levels being used for testing tiers. By this I mean:
1. All your 'essential' services are Tier 1 and always deployed beside any change to test. With something like X (twitter) that might be content feeds, user service and auth - so if you're deploying 'Service-fun-profile-pic-personalisation' then you'd test it alongside those three Tier 1 services
2. The other services then go down in Tier levels e.g. Profile-service might be Tier 2 whereas Recommended-Users-To-Follow-service might be Tier 5. 
3. Testing maybe only happens end to end for the critical tiers, and maybe integration or contract testing for the rest

If services are quick to roll-back, then you might be OK with 'my critical functionality works' and be fine to roll-back or tweak breakages happening in production against your less important services. This kind of model would likely work reasonably well when you're doing a slow rollout to 1% of your traffic to start with and when your roll-back is relatively fast.

Now I think a flaw with everything discussed above is actually the conflation of polyrepos and microservices. I often see these coupled together because
- It follows the deployment cycle
- Access control / the same 'teams' arguments as before
- Independence / ownership
When actually it doesn't necessarily make a lot of sense to keep multiplying your repos after you get to e.g. 50+ because you then suffer from the problems I detailed above. 

The one-repo-per-service convention isn't a great one, and this gets me on to a real-world example - Monzo. Monzo put up some good blog posts a couple years ago such as [this one](https://monzo.com/blog/how-we-run-migrations-across-2800-microservices) as well as doing a great talk last year (2025) at the GCP Summit on their backup system they created in GCP (a set of core services they launch in GCP in the event AWS goes down and reroute traffic/data to). Super interesting and thoughtful stuff and I recommend reading their post if you find this problem a worthwhile one.

I thought it was quite pointed that in their post from 2024 they detail that:
1. They have 2800 microservices
2. They have invested an absolute mammoth effort into standardising everything (tackling the potential exponential drift issue I mentioned) and clearly have a strong engineering culture for it
3. They've abstracted core tooling/tech around their own SDK meaning they can control the interfacing cleanly and add custom useful bits
4. For migrating services they literally have a whole _team_ for it
5. To handle the 'testing' bit they've built their own 'mass deployment tool' which means they can launch at huge scale in async batch jobs - and importantly here, have clear prioritisation for repos and use what sounds like good (fast) automated rollbacks
6. Rollback sounds like it works via config
7. The rather pointed aspect for me - Even with all this, they _still_ use a centralised monorepo for *all their services code*, even if it sounds like they might have a polyrepo structure generally (from their talk, it sounds like they separate out their core highly reused internal libs + infra etc)

I've personally been struggling with this problem because I've worked in both setups and am in the poly camp (legacy reasons) at present. Specifically from personal experience, what I've learned is:
- Creating a way to update many repos when that's not already baked into the system is hard
- The version rot issue is real and causes big drops in velocity as engineers find older, unknown code written in different styles
- Observability and governance become very critical

The observability and governance are obvious yet the problems you experience when they're lacking can feel almost unimaginable. As an example: It turns out you can literally miss finding a repo relevant to your work when the repo itself is also badly described / inconsistently named for a job such as migrating an integration between an old partner and a new one (with many repos handling different aspects of the integration). What happens then is that you might assume the missing functionality was a bug or issue in the previous integration ... and start writing a new one (in a new repo) from scratch. I can't describe how painful it is to discover this weeks later.

This has been a bit roundabout but I'd like to come back to my original thesis, that polyrepos tend to occur either because of large organisation structures or because of mistakes. 

Conway's law (via Wikipedia) states
> Organizations which design systems (in the broad sense used here) are constrained to produce designs which are copies of the communication structures of these organizations.

So if you have 20 teams with independent plans, velocities, roadmaps, you might grow into a code structure with akin to 20 repository clusters. It makes intuitive sense because the organisational structure you have reflects your ability to govern, know and update the system you've made. If you're a small team of 4-10 people however, then it's unclear how you can govern 100+ repositories without a significant amount of overhead or initial input to create the governance and systems you need. In these scenarios, my original premise holds.

In a company of 1 small team of engineers, 100+ repositories would likely be ungoverned and suffer from the issues listed: That's then a mistake, whether it's from lack of direction from a senior figure or from deliberate intention to separate out all services. In a company 10+ teams, the polyrepos approach likely works because there's enough people to share the knowledge and burden of maintenance/governance. Org structure might push you to polyrepos, but Monzo shows that it doesn't need to push you there all the way - & that it can actually be beneficial to have a centralised backbone team who helps and maintains other teams through a monorepo source, sitting in a polyrepo structure. For me this feels like the perfect Goldilocks balance - Enough repos so that we're not stepping on each others toes, but a core function who keeps everyone aligned and singing the same tune.