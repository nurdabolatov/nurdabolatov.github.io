---
layout: post
title: Why tests are important?
---

"Writing a test is time-consuming" one might think. It is indeed so. On top of that, there is a maintenance cost for tests and it rarely justifies the effort. Because of this naive thinking, a developer might skip test implementation and focus on building things that matter. However, there is another side of the coin which I am going to tell you about in this post.

Tests keep your application stable. You will rarely notice that it does prevent potential bugs.

For example, let's say you've implemented the force update feature in your mobile application. Time passes by and a new member of your team decides to modernize the feature but breaks it for particular operating systems. People are usually lazy when it comes to testing every single edge case manually so I wouldn't blame the developer who broke your feature. But nobody notices it until there is an emergency and you have to upgrade everyone to the newest version of the app. Guess what? The force update feature is not working.

Now let's consider the case when you've decided to spend additional time for tests. It would have been almost impossible to break your feature. Anyone who is maintaining the feature would acknowledge their mistake when tests start failing and have a chance to fix the bug before anything major happens. And you would never notice this happened which is good because tests are saving time by doing the verification work for you.

Tests accelerate development. This might sound controversial depending on the perspective you have. For short-term goals, it does indeed take a huge portion of the effort. But if you think long-term, tests facilitate development and increase productivity. 

For example, consider you joined a team that built the authentication layer. And you have interesting projects in mind that you would like to implement in the layer. But there are no tests in place. It would hold you off because there is a high chance to break the entire application. So you would end up spending a lot of time making sure you didn't break the legacy code. By the way, [never fear the bugs]({% post_url 2021-04-17-dont-fight-bugs-welcome-them %}).

Wouldn't it be great if the authentication layer was already covered by tests? You could have build not one but many projects on top of it with high efficiency. 

Of course, it is hard to predict the long-term plans for any project but you should keep your changes covered by tests whenever possible. So that posterity can enjoy the fruits of the plant you seeded. 

Let me know if you have other compelling reasons to/not to write tests.