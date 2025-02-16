+++
title = 'The Mythical Man Month'
date = 2025-02-10T11:39:42-03:00
draft = false
tags = ['tech', 'book', 'software engineering']
+++

After reading the book the [Goal]({{< relref toc-theory-of-constraints >}}), I went after the next reading, and I was recommended to read The Mythical Man-Month, from Fred Brooks. This book is a computer science class, which I didn't know yet, that describes the software development life cycle in the 70s.

The book's main subject is told at the title: the mythical man-month. Man-month is a productivity metric like the ones that are used today, such as points, t-shirts size or whatever that is being used. The argument is the metric is failed and number of tasks, line of codes or features developed by someone aren't constant between projects.

The author even quotes two projects: management/maintenance projects and compiler/translator projects. It is visible that the man-month are distorted by the task complexity. At the management/maintenance projects the metric was around at 550 words per man/year and at the other project, this number increases to be around 2250 words per man/year.

While searching for more productivity, the first impulse is to add more people to the team, because if we deliver 10 features per month with a 5 people team, we assume with 6 people we will deliver at least 12. In practice, it can cause less productivity because the cost of ensuring an efficient communication increases considerably, as the formula for the communication between nodes is given by **n * (n - 1) / 2**. We must also add the training cost for new people. This is the Brooks' law.

So, we can assume that communication was a challenge at that time, a challenge that isn't solved until today. At the 70s, the manuals were physical, which had extra costs for maintenance and update. Today we have much more technologies like Confluence, Jira, Trello or whatever work management tool. However, we still fail on how to document our projects.

A great tip from Brooks it that all software should have three kinds of documentation:
1. To use a program.
2. To believe a program.
3. To modify a program.

This first kind is the basic project documentation like its purpose, system requirements, allowed input types and returned types, which options are accepted and all the user needs to know for using your software without surprises. It is important because with it is the one that has the most value for your user, since it answers all their questions.

The documentation for believing a program are the examples, demonstrations, use cases and relevant tests that will show the software running. It also focuses on the user, giving credibility to what was written in the basic documentation.

The last kind is the most common to see: the project's technical documentation, such as, function responsibilities, the chosen language and its version, how to run the test suite, which libs are used. Its focus is on the people who work or will work at the project, helping to understand all the technical decisions and easing the maintenance over the years.

Brooks' view on projects is also interesting. He comments that the first project version will usually be discarded by being too slow, too complex, incomplete or any other problem. However, he also warns to be careful with the second version, since there is a tendency to included all the good ideas that were not part in the first version, creating the famous big ball of mud. The regression tests, documentation, and communication will avoid this concern. The author postulates it is better to have a small and concise project within its features than a huge one, with tons of messy features.

According to the author, there are two delay types in projects: the visible and the invisible. The visible are the ones that are not necessary to justify, such as disasters, global issues and else. The invisible ones we actually should concern: someone got sick, a feature took longer than expect or other team dos not deliver what was agreed. That is how a project delays by a year, by delaying one day each. It is necessary to have a full view on the project with clear and well-defined milestones. The author reinforces that and which techniques that can be used to ensure the deadlines, like having sheets with the initial estimation and with estimations by who works at the project, in order to control the delays.

The book also shows other valuable points like the need of clear roles on the team, like the technical leadership. It is also mentioned that the use of high level languages improved the productivity metrics and some programmer at that time complained about these languages being too slow, easy or whatever you already probably heard about any programming language. He also talks about organizing people on small teams.

To conclude, it is a great book to read since it shows how was the life of a software development team at the computer science beginnings. It is possible to draw comparisons with the challenges that exist nowadays. It is interesting to see which techniques were used to test software, to measure productivity, or even what people complained about. You will know other authors and researches from the same period because Brooks quotes a lot of them. You will not regret reading it!
