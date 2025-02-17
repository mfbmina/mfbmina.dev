+++
title = 'The theory of constraints'
date = 2024-11-11T18:19:03-03:00
draft = false
tags = ['tech', 'theory', 'continous improvement', 'book']
+++

At the end of 2023, my squad started its most important project. I assumed the project leadership role, organizing the project, coordinating the meetings, the release process, etc. At the first months, the project went well, as we took some hard decisions, proved a couple of concepts, and started the rollout for the first phase.

However, I always felt that I was holding the team back and being a bottleneck in the whole team. At a 1x1 with the head of the area, I told him about my frustration and got this answer back:

> Yes, you are being a bottleneck for the team, but that isn't necessarily bad.

I tried to argue with him because I couldn't understand how being a bottleneck wasn't bad, and then he introduced me to the theory of constraints. In the end, he recommended a book by Eliyahu M. Goldratt called The Goal: A Process of Ongoing Improvement. Hence, I want to share what I took from it here.

## Metrics

Before we understand the theory of constraints, we need to know what is the company main goal. We can argue that companies try to improve the world, or they provide better services, but that isn't their main goal. Their focus is earning money, and if this is the final goal for every company, all our focus should be on achieving this objective.

Because making money is a generic goal, the author gives us three goals to focus on. They are:
- production: how the company makes money;
- inventory: the money the company has invested to make more money;
- operational expense: the money spent on turning inventory into production;

In that sense, we should aim to increase the production, while reducing inventory and operational expenses. Also, all metrics must have an impact on the company's goals.

Extrapolating to the world of software engineering, we have:
- production: features, products, and everything that can be sold to a client;
- inventory: servers, cloud, hardware, and everything that is part of the product;
- operational expenses: programmers, P.Os, heads;

So, we need to deliver more features and products while reducing the amount of hardware and people.

## Concept

Now that we have a clear goal, we can talk about the ToC. In short, it is a continuous improvement process that aims to find the system's constraints and work around them. It is important to define that a system without restrictions doesn't exist. They will always be there, and most importantly, they regulate the delivery flow.

Let's move back to the initial point that made me read the book, constraints aren't essentially good or bad! Restrictions exist, and sometimes, they are necessary. A great example is a semaphore since they regulate the flow of cars and the speed in certain streets. Again, they aren't bad, they simply exist. During the software development process, we also have some constraints that are considered essential, like code review and writing tests. Both slow us down, but they ensure a higher quality.

However, it is worth mentioning that the only capacity that matters is the constraints because the system will never be able to deliver more than what the constraint can. The ToC aims to understand which constraints you have and how you can work around, or even remove, them by following a systematic approach.
1. Identify the constraint or constraints of your system;
2. Explore how you can work with them;
3. Avoid producing more than the constraint can handle, or in other words, work around the constraint;
4. Elevate the constraints of your system or expand its capacity with new investments;
5. If the constraint has been eliminated, move back to phase 1 and start over. If the process shows improvements, be prepared for the constraint to move to another point or if new constraints appear;

As I said, you can also add constraints to your system to ensure a continuous delivery flow. For example, in software engineering, you can start doing code reviews. But they are not the only constraints! I can also list databases, rate-limiting on APIs, team capacity, pair programming, and other infinity of constraints that we can deal with while building software.

## Conclusion

Despite the book being a romance and happening in an industry, it is possible to draw a parallel to what we live daily in a software development team. It is important to note that our job is part of a larger system, and you should understand it and its restrictions to build an effective and constant delivery flow. Before you make any optimization, be sure that you validate all hypotheses. Make sure that you are being productive and if you are, in fact, increasing the performance of a bottleneck.

Summarizing what I learn from the book:
- Productivity and occupation aren't synonymous. Work is only productive if it brings you closer to your goals. If not, maybe you're working on something that isn't so important;
- Bottlenecks are the system's main part! To increase the overall efficiency, you must increase the efficiency on the constraints;
- Each improvement at a non-bottleneck is a premature optimization that doesn't improve the overall efficiency. Having local optimus doesn't bring you closer to an optimized system;
- To identify your constraints, apply the scientific method. Suggest a hypothesis, and if it turns true, something should happen. If not, you may discard it and validate a new idea;
- The optimization is always continuous. When new bottlenecks appear, apply the systematic approach again. It will always emerge new restrictions and new improvement opportunities;

Lastly, I learned more about my role at the squad and the project. Now, I have a better understanding on how I work and how I can reduce the impact of being a bottleneck. That is the Theory of Constraints, and for those who enjoyed it, I recommend reading the book!
