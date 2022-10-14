---
layout: post
title:  "How to Open Source Your Project (II)"
date:   2022-07-26 10:00 +0100
tags: oss howto writeups
---

In part 2 of the OSS howto series, we take a look at Community, Collaboration, and Context (read [part 1]({% post_url 2022-07-25-stackrox-oss %}).

## Part II: Community, Collaboration, Context

### How To Enable Healthy Discussion?

As your audience grows, it is vital that you define clear rules to create a safe environment for everyone to participate. A common way of doing this is to define a code of conduct (CoC), which sets some basic guidelines on what kind of community interaction will not be tolerated. We decided to stick to well-established frameworks and based our CoC on the [Contributor Covenant](https://www.contributor-covenant.org/).


### How To Keep the Discussion Healthy?

The best CoC definition doesn’t help if there is no one there to enforce it. This meant we needed to find volunteers for a CoC committee and train them accordingly. The committee members should be publicly available and open for communication so they can be approached in case of any problems.  
Similar to handling responsible disclosure and CVEs discussed in part 1, I strongly recommend to not only have a CoC team in place, but also have them trained on CoC enforcement and communication in case they are newcomers.  

We did this by publishing the CoC and the committee members on our [community website](https://www.stackrox.io/code-conduct/).

> As your audience grows, it is vital that you define clear rules to create a safe environment for everyone to participate.  

It is especially important to clearly communicate and enforce the rules you set up, as a safe environment fosters collaboration from your community.


### Where To Communicate with the Community?

At this point, you should be aware of the goal of your open source go-live and the expected target audience. Answers to these questions shape how you interact with your community. Use all channels you have to reach out, but decide on one discussion medium out of the plethora available today, such as Slack, Discord, mailing list, forums or Matrix.

The StackRox community currently lives on the [CNCF Slack workspace in the channel #stackrox](https://www.stackrox.io/slack/).


### How To Accept Contributions?

Be clear and concise in what you accept from contributors. Is it only feedback in discussions? Do you accept issues or pull requests on GitHub? If so, it is recommended to provide guidelines in the form of a `CONTRIBUTING.md` document or Issue/PR templates to fill out.

If you decide to accept these, it also helps to give people a rough idea of your reaction times. Also, be sure to have processes in place to decide who keeps an eye on new items.

For example, we communicate that we aim to triage new issues and PRs within a week, with more detailed discussions and decisions communicated in our monthly meetings.  
That said, there is also the broader question whether you include community contributions in your downstream commercial verssion.  
There are many options how to do this. Check in with your Product Managers, Customer Support Team, and Legal department to make sure you don't violate any license requirements and have unified messaging for public facing communication, for community and paid customers alike.  

The StackRox team decided to accept community contributions, but clearly mark them as not supported by downstream Enterprise Support.  
While this adds a bit of management, as you need to keep track of two issue trackers (e.g. internal for downstream and GitHub for upstream), it fosters clear expectations for every involved party.


### How To Meet with the Community?

Regular public meetings lower the bar for participation and allow for issues to be raised efficiently. Any interested contributors can quickly stop by and get in touch with your project.

Currently, we run our StackRox Community meeting on the second Tuesday of each month at 9 a.m. PST, 12 p.m. EST, 5 p.m. GMT. You can subscribe to the events by adding the calendar community@stackrox.com to your calendar.

In these meetings, we discuss and show demos of upcoming features, talk about open issues, present guides and how-tos, and have an open forum for Q&A with the community.


### Conclusion

There is a lot to consider when opening a big commercial project to a broader audience, but tackling this in an organized manner is worth the payoff - especially for the Eng team.  
As a final recommendation, please take the time to celebrate the go live with your team and make sure to pass any positive feedback to them!
