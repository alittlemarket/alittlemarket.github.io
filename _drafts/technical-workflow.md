---
layout: post
title:  "The technical workflow at Etsy France"
date:   2015-02-23 13:55:00
categories: workflow
tags: [workflow, jira, github, pr, review]
author: antoine
---

Here at Etsy France we’ve tried several management techniques such as everything and nothing. But for some time now, we’ve completely switched to a  Scrum agile development methodology.  In order to manage our workflow we interconnected some very useful tools such as [Jira][jira], [GitHub][github], [HipChat][hipchat] and [Travis CI][travis].  
This post will explain you what our daily workflow looks like from the early backlog to the production deployment.

## Ninja Teams
Last year, we decided to completely change our work habits. We went from one big team (basically the entire company) to 6 smaller ones, each composed of up to 7 people with tech and non-tech backgrounds. Why so many teams? Well, simply because as our website grew, our tech team did and regular tech meetings with 10+ people are just boring. But more importantly, it allowed each of our departments to have their projects in their hands and the means to move forward by managing their own timeline.

Each team works on its own weekly backlog on JIRA. Most of them are following Scrum development methodology, while some use Kanban.
In the Scrum method of Agile software development, work is confined to a regular, repeatable work cycle, known as a sprint or iteration.Each task in those iteration is going through 5 different states represented by 5 columns:: 

| To do | Waiting | In progress | Should be reviewed/tested | Done |

To briefly explain each column, this is how it works :

- To do: A task that will be done in the current sprint but yet to begin
- Waiting: More details are needed in order to begin/continue this task
- In progress: Development is in progress
- Should be reviewed/tested: Development is finished but the task should be reviewed and tested by another developer or project member
- Done: Task deployed and tested in production

## The tools

Git and GitHub are fully integrated to our workflow. The GitHub integration in Jira is very useful and offers an overview of our past/current/future developments in a single spot.  
To use this interaction between these tools, we use the "DVCS Accounts" component in Jira which allows you to link your GitHub account directly in Jira. You select the repositories you wish to link or choose future repositories to auto-link.

Then, a connection between Jira and GitHub is automatically created when:

 - a message containing the name of the Jira task is being committed (e.g.: CORE-542)
 - a branch having the same name as the Jira task is being created
 - a Pull Request with a title containing the name of the Jira task is being created

By doing so, you are able to find all the commits, branches and pull requests linked to your Jira task:

{:.text-center}
![PR, branches and commits displayed in Jira](/assets/technical-workflow/pr-branches-jira.jpg)

The connection between GitHub and Jira is definitely helpful to us. Here are a couple things we can do with it:
- We can easily access every pull request related to the current Jira ticket. No more wasting time looking around for pull requests on each repository, it can be really tedious at times.
- In case of bug (or what is considered as one) it's easier to find the responsible code for the issue, go back to the pull request and then to find the Jira task containing all the functional specifications.  
This trick is needed because we don't have the culture of small and atomic commits with explicit comments yet.


## Back to our workflow

When a task switches to "In progress", a git branch with the same name as the Jira task is created.  
Development begins.

{:.text-center}
![Wait, writing in progress](/assets/technical-workflow/loading.gif)

When the task has been done, it goes to the column "Should be reviewed/tested" in Jira and a pull request is created in GitHub.
Then Travis comes around ! And since we linked it to GitHub, each Push or Pull Request being created sends a build request. By doing so, we can quickly see if the development branch doesn’t pass some tests and automatically alerts us. All that without leaving GitHub.
**Very useful.**

{:.text-center}
![Travis integration in GitHub](/assets/technical-workflow/travis-ci.jpg)

As in Jira with the columns, we use labels in GitHub to reflect the PR status:

- WIP (Work In Progress) in red
- RFR (Ready For Review) in yellow
- RFM (Ready For Merge) in green

HipChat warns the others members of the team that the task is ready for review.

{:.text-center}
![Jira integration in HipChat](/assets/technical-workflow/hipchat.jpg)

At this time, the pull request should be reviewed by at least one other engineer before being merged into master.  
Each pull request must have one <img src="https://assets-cdn.github.com/images/icons/emoji/unicode/1f44d.png" alt="+1 in GitHub" style="height:20px" /> in order to be merged.

You did it !
When the pull request has been validated, it can safely be merged and deployed to a pre-production environment for a last safety check, and then to production.


[jira]: https://www.atlassian.com/software/jira
[github]: https://github.com/
[hipchat]: https://www.hipchat.com/
[travis]: https://travis-ci.com/
