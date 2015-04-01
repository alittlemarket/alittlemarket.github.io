---
layout: post
title:  "The technical workflow of Etsy France"
date:   2015-02-23 13:55:00
categories: workflow
tags: [workflow, jira, github, pr, review]
author: antoine
---

Here at Etsy France we tried several organisations.  
For some years now we fixed our organisation in scrum. To manage our workflow we use some useful tools that are interconnected: [Jira][jira], [Github][github], [HipChat][hipchat] and [Travis CI][travis].  
This post will explain to you what is our daily workflow from the backlog to the deployment.

## Our organisation

In fact we do not have only one backlog, we have several backlogs because we do not have only one technical team but many.  
Many because last year we decided to split our technical team in smaller and more specialized teams composed of technical and non technical people.

Each team has its own backlog in Jira and most of them are organized in scrum (or Kanban).  
Each current sprint is cutting in several columns approaching these : 

| To do | Waiting | In progress | Should be reviewed/tested | Done |

To briefly explain each column, this is how it works :

- To do: A task that will be done in the current sprint but that is not yet began
- Waiting: More information is needed to begin/continue this task
- In progress: Development is in progress
- Should be reviewed/tested: Development is finished but the task should be reviewed and tested by another developer or project member
- Done: Task deployed and tested in production

## The tools

Git and github are fully integrated to our workflow. The github integration in Jira is very useful and offer an overview of our development in one place.  
To use this interaction between these tools, we use the "DVCS Accounts" component in Jira which allow you to link your github account with Jira. You can choose the repositories to link or choose to autolink new repositories.

Then the link between Jira and github is automatic when:

 - a message containing the name of the Jira task is being committed (CORE-542 for example)
 - a branch having the same name as the Jira task is being created
 - a Pull Request with a title containing the name of the Jira task is being created

When you do that, you can find all the commits, branches and pull requests linked to your Jira task:

{:.text-center}
![PR, branches and commits displayed in Jira](/assets/technical-workflow/pr-branches-jira.jpg)

The connection between github and jira is usually helpful for us.  
First, in our Jira task, we can easily access to the pull requests. So, time lost to look for the pull requests on each repository.  
Second, in case of bug (or what is considered as a bug) it's easy to find the code responsible for the issue, to go back to the Pull Request and then to find the Jira task with the functionnal specifications.  
This trick is needed because we don't have the culture of small and atomic commits with explicit comments yet.

Well, back to our workflow.

## Our workflow

When a task passes to "In progress", a git branch with the same name as the Jira task is created.  
Development begins.

{:.text-center}
![Wait, writing in progress](/assets/technical-workflow/loading.gif)

When it's done the task goes to the column "Should be reviewed/tested" in Jira and a pull request is created in github.  
Travis is linked to github and each push or Pull Request creation send a build request. So, we quickly see if the development branch breaks the tests or not and without leaving github. **Very useful.**

{:.text-center}
![Travis integration in github](/assets/technical-workflow/travis-ci.jpg)

As in Jira with the columns, we use labels in github to reflect the PR status:

- WIP (Work In Progress) in red
- RFR (Ready For Review) in yellow
- RFM (Ready For Merge) in green

HipChat warns the others members of the team that the task is ready for review.

{:.text-center}
![Jira integration in HipChat](/assets/technical-workflow/hipchat.jpg)

At this time, the pull request should be reviewed by at least one other engineer to be merged in master.  
Each pull request must have one <img src="https://assets-cdn.github.com/images/icons/emoji/unicode/1f44d.png" alt="+1 in github" style="height:20px" /> to be merge.

You did it !
When the pull request is validated, it can be merged and deployed to preprod and then to production.


[jira]: https://www.atlassian.com/software/jira
[github]: https://github.com/
[hipchat]: https://www.hipchat.com/
[travis]: https://travis-ci.com/
