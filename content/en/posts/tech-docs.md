+++
title = 'The importance of tech docs'
date = 2024-09-12T19:11:16-03:00
draft = false
tags = ['tech', 'documentation']
+++

When I chose to pursue a career in computer science when I was 15 years old, it was basically on liking math and physics. I want to distance myself from writing posts, essays, etc. Time goes by, and now, one of the things that I value at a mature company or team is the existence of technical documentation.

Docs help us to gain historical knowledge of our company, reevaluate decisions, and comprehend the trade-offs between all the decisions.
It is also beneficial for the newcomers because it tells them a story. The richer in detail it is, the better it is.

There are several ways and patterns to write great tech docs. The ones that I like most are Design Docs, ADRs, and RFCs. Let's talk about them!

# Design Docs

Design docs give us the details about the following solution. The goals are to document the decisions, improve the visibility around them, foment the study of several options, and share knowledge.

A good design doc must have a header with all info about the author and the discussion, context about the issue,  the goals, what is out of scope, all the discussed options, the solution, and even some general concerns and next steps.

To go deeper, I recommend visiting [https://www.designdocs.dev/](https://www.designdocs.dev/). There, you can find templates and tips on how to build a great design doc.

# ADRs

ADR, or Architecture Decision Records, aims to document the decisions taken during the development process or at a project. A great ADR gives us the history of all decisions and the whys associated with them.

When building an ADR, it must have a title and context about the issue, which are the options, the decision itself, and why it was the one.

For those who want to learn more about ADRs, I recommend visiting [https://adr.github.io/](https://adr.github.io/). 

# RFCs

RFC is an acronym for `Request for Comments`. The goal of a RFC is to specify every aspect of a solution, pattern, or project. It is a very rigorous and structured process, with many details, that can take some relevant time to conclude. But, given the depth and the detail, it doesn't make room for doubts about the project.

The [IETF](https://datatracker.ietf.org/) (Internet Engineering Task Force) maintains almost every RFC of big projects and patterns, and it is a great guide on how to write them.	It also has a [guide](https://www.ietf.org/process/rfcs/).

# Considerações finais

I hope that with all this information and links about different ways of writing tech docs, you notice that doesn't matter which model you or your team pick. The most important thing is to have some form of documentation. All of them have their strengths and weaknesses, and it is your responsibility to understand all their trade-offs and choose which suits you the best. It is also important to not be too rigid in its process because we know that docs usually don't age well.

For example, here at PicPay, decisions that affect the whole company use RFCs. They are formal documents, with reviewers and a very structured process. For decisions that may affect a squad or a small portion of the company, design docs are usually used because they are simpler.

Also, when you evolve within your career, it is expected that you can build impact. The higher your role, the higher the impact expected. And I'm sure that writing great tech docs can make you reach higher levels in your career path. I hope you enjoy this post, and tell me: Which standard for tech docs do you use?
