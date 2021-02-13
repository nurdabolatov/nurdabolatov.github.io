---
layout: post
title: Can I trust open source?
---

He started to work on yet another billion-dollar startup. The importance of time cannot let him lose a minute of reinventing the wheel. Therefore importing existing modules from an open repository sounded like the best option to move fast. Tons of questions arise in his mind. The important one being: 

> Can I trust this repository?

Anyone can contribute to an open-source project. He cannot trust strangers, especially, on the internet. On the other hand, he cannot spend hours implementing authentication, animations, debugging tools, etc. He has to find the right balance between risk and speed. A guardrail of rules played a vital role in finding the balance and the trustworthy repositories.

**The first rule.** He decided to count the number of stars the repository has and set a threshold of 100 stars. The threshold indicated the reputation of the repository in his eyes. The higher reputation easily gains his trust.

**The second rule.** He checked out contributors because a wide range of maintainers usually means more code reviewers. The more eyes the better.

**The third rule.** He took a look at the identity of the module author. He has to rule out any repository with a sketchy creator. To determine the sketchiness, he looked up the author in Google and made sure they don't have a vague identity.

**The fourth rule.** As a last resort, he can always review the implementation. Of course, reviewing a large repository cannot be accomplished in a short time. Therefore he decided to take a look at dependencies, core logic, and the part he will be using in his project. Exploring the new codebase excited him as well as building a startup.

These rules are not set in stone and each developer should use their own judgment.

It is worth noting that [GitHub puts huge effort into making sure the repositories are safe](https://github.com/security). They offer a nice feature to [scan your codebase for any vulnerabilities](https://docs.github.com/en/github/finding-security-vulnerabilities-and-errors-in-your-code/automatically-scanning-your-code-for-vulnerabilities-and-errors).