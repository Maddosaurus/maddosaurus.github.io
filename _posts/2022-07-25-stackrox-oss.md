---
layout: post
title:  "How to Open Source Your Project (I)"
date:   2022-07-25 10:35 +0100
tags: oss howto writeups
---
Transitioning a project from private to public development means more than just changing the visibility of the GitHub repositories. In part 1, we take a look at how your product and your team should guide your decisions.
<!--more-->
A version of this article was also published at [The New Stack](https://thenewstack.io/how-to-open-source-your-project-dont-just-open-your-repo/).

---

On April 1 2022, the release of the StackRox Community Platform was announced.  
This is the result of a great deal of work by our team to transition StackRox’s proprietary security platform into an open source one.  
I've been working behind the scenes and want to share a bit of insight into the challenges that bigger projects might face when opening up.  
Transitioning a project from private to public development means more than just changing the visibility of the GitHub repositories. It is essential to have a transition plan, especially if the goal is to build a thriving community where users can grow and leverage the platform.  
To have the best chance of success, the project’s goals and the community’s goals should be as aligned as possible.  
For the StackRox team, one of our top goals was to set the entry barrier as low as possible for contributors and community users. I’ve personally found this to be a significant challenge.  
It is one thing to tailor your environment to engineers, hoping to provide a thorough and guided onboarding experience. Creating a forum for a greater community of developers, operational and security folks poses an entirely different challenge.  


## Part I: Your Product and Your Engineering Team

Before you make any decisions, you should be aware of your product and your team.  
Obviously, you should know both rather well, but you should also be aware of the broader context of your product and its role compared to competing products.  
Last but not least, you should answer the question of what you want to achieve with opening your platform.  
- Are you interested in giving back to the community? 
- Do you want to grow trust in your product by exposing it to public scrutiny? 
- Are you interested in broader feedback or reaching a different user group?  

As soon as you have answered these questions, you can think about the next steps.

### What To Open Source?

If you look at the [StackRox GitHub organization](https://github.com/stackrox), you will find a multitude of repositories with the platform comprising many different components and features that could be kept private. However, we chose to be thorough and take the extra time. We decided to open the complete platform and all its dependencies, including our out-of-the-box policy ruleset that we ship on new installations, prebuilt Docker images, and Helm charts to make the open source deployment as easy as possible.

I've been a strong proponent of opening the platform as-is instead of artificially removing parts of it, like predefined rulesets and alerts.  
As Kubernetes deployments are highly personalized to the needs of customers, the OOTB rulesets are a good starting point but are usually adapted to the environments' needs sooner or later. As we wanted to lower the barrier of entry as much as possible, we wanted to ship the platform in its full state to give users a starting point right away.


### What License To Use?

When opening your source code, one of the first tasks should be to select a license that fits your use case. In most cases, it is advisable to include your legal department in this discussion, and [GitHub has many great resources](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/licensing-a-repository) to help you with this process. For StackRox, we oriented ourselves on similar Red Hat and popular open source projects and picked Apache 2.0 where possible.


### How Can People Access It?

After you’ve decided on what parts you open up and how you will open them, the next question is, how will you make this available?

Besides the source code itself, for StackRox, there are also Docker images. That means we also had to open the CI process to the public. For that to happen, I highly recommend you review your CI process. Assume that any insecure configuration will be used against you. Review common patterns for internal CI processes like credentials, service accounts, deployment keys or storage access.

Also, it should be abundantly clear who can trigger CI runs, as your CI credits/resources are usually quite limited, and CI integrations have been known to [run cryptominers](https://www.infoq.com/news/2021/04/GitHub-actions-cryptomining/) or other harmful software.


## How To Manage It?

StackRox functions as an upstream public build, whereas Red Hat Advanced Cluster Security (RHACS) is built on an internal Red Hat build system. Tending to two different build pipelines naturally brings some overhead, as the open source and commercial flavors of this project each have different needs.  
In general, I would recommend against duplicating your build infrastructure, but sometimes it is inevitable. If you plan to distinguish your OSS and commercial versions, a lot of options are available.  
We found build time feature flags rather practical to manage differences between Upstream and Downstream.

If all goes well and your CI succeeds, you will most likely end up with some artifact — a release binary, tgz file or Docker image, which raises the next qestion:


### How To Distribute It?

Making these artifacts publicly available to lower the barrier of entry is essential.

For StackRox, we decided to push built images to a [public organization at Quay](https://quay.io/stackrox-io). Alternatively, you can use GitHub’s release feature or other public distribution channels, depending on your release artifact type, such as NPM, PyPI, or Crates.  
I would recommend to use established channels as much as possible. If you are building a framework or library, language package managers should be your primary target. If you are building a product that needs deployment, consider publishing Docker images, as these are a well defined convenience that can be run everywhere - from local dev systems to enterprise-grade cloud Kubernetes deployments.  
In any case, you should not forget about potential external contributors. Your product most likely requires a very specific toolset to develop and build on. Consider migrating your dev environment in a container; not only for external contributors, but also for your Engineering team, to have a standardized dev environment.  
After distribution, the next step would be users downloading these artifacts and running your product. This brings us to the next important question.


### How To Document It?

The project documentation is your public representation of the project, inviting users and contributors alike. Potential users will often consult your documentation first to gauge whether your project fits their use case and understand how to use it most efficiently. Documentation is ideally written to convey information to the community while minimizing user confusion and clarifying issues in your GitHub repository.

> Remember that documentation for operators and developers are two very different things. 

For example:

Operators are more interested in deployment, configuration, platform maintenance, data preservation, updates and disaster recovery. 
Developers are interested in setting up a development environment (IDE, local deployment, debug builds, etc.) and getting access to detailed API descriptions.

Both target audiences profit heavily from “Getting Started” guides, be it how to get your first deployment up and running (operators), or how you accomplish everyday extension tasks in the codebase (developers).

Because StackRox is upstream of RHACS, we decided to focus our documentation efforts on developers, as quite a lot of user-tailored documentation is available. Open source-specific user documentation with StackRox branding is a project we’re planning right now, though.

Our developer-tailored documentation is being expanded in the [main projects’ README](https://github.com/stackrox/stackrox/blob/master/README.md) and [stackrox/dev-docs](https://github.com/stackrox/dev-docs/). The latter is a collection of Markdown guides that initially started as private Confluence articles. This collection keeps growing, especially as we get more feedback from contributors on which guides they would like to see. It is also a continued effort to migrate additional guides and how-tos that might have been missed in the first migration, or that might not have been published because they contain private information.


### How To Manage Privacy & NDA information?

Speaking of private information: Due to the nature of git, the complete history of your project will be public, starting with the first commit to the repository you publish. This also applies to any issues, discussions and pull requests that your project collected over time. While this is a non-issue for internal development, this can pose quite a problem when going public.

> It is heavily advised that you review all your issues and comments — on GitHub or other git repositories — and scrape the project’s git history, PRs, and Issues for any information or references not intended for public use.

This information does not need to be public to be considered open source, but it does add context for future users and your own devs, so you might want to keep as much of it intact as possible.

A quick and easy way is to start with a new project on GitHub and do away with your git history. This poses multiple problems, however.

Your engineering team loses the history of their work, problem-solving and discussions, which are valuable resources. Furthermore, this step has to be planned well, and the engineering team must be in the know — if one person pushes their old git history to your new project, the complete history will be accessible again.


### How To Handle CVEs?

Speaking of visibility: If you handle CVE/embargoed work, you will need a workflow in place. As your repository is public, you cannot simply use a public feature branch for this kind of work. For example, GitHub provides the option of [temporary private forks to resolve security issues](https://docs.github.com/en/code-security/repository-security-advisories/collaborating-in-a-temporary-private-fork-to-resolve-a-repository-security-vulnerability).  
Due to the time-sensitive nature of this topic, I would strongly recommend to have a process in place before you open your repository and, more importantly, make everyone aware of it. This includes Engineering as well as Engineering Managers and Management.


### How To Take Care of Our People?

Last but not least: As already mentioned, in all of these tasks, you should take care to keep your engineering team involved and in the know at all times.  
Community and Collaborators are one thing, but your Engineering team are the folks that work full time on your product, so you should be wary to keep them happy and motivated.

The Red Hat Advanced Cluster Security (RHACS) engineering team works in a design-document-driven process, where all major changes are discussed between the whole engineering team through shared documents and discussions, written (document comments) and spoken (review meetings).

In the months leading to our announcement, we conducted many discussions and tried to find solutions, workflows and approaches that the team was happy with.

> As the engineering team will still be the main driver for the project, they should know what changes in their daily work once you go open

Additionally, this is an excellent opportunity for initial external OSS contributions. If you maintain private forks with product-specific patches, this is the chance to shine by offering these changes to the original upstream projects. 

For our engineers, little changed. They still work primarily in the upstream repositories, with the main difference that all pull requests and comments are now publicly visible. This change means that teams need to be mindful of how they communicate, even internally, as all comments can be read by external people who might lack the context or shared humor your team has.


Continue reading in [part 2]({% post_url 2022-07-26-stackrox-oss-ii %}, where we take a look at the big C's: Community, Collaboration, and Context
