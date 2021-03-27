---
layout: post
title: Build a static website with Jekyll and host it on GitHub
---

Had he known about [Jekyll](https://jekyllrb.com/) + [GitHub Pages](https://pages.github.com/) years ago, he wouldn't have had to cook HTML pasta and fight broken CI/CD so many times. These technologies save a good amount of time that he can now spend on doing more valuable things. For example, writing a post on his favorite blog using [markdown](https://daringfireball.net/projects/markdown/). Other use cases include:
- CV/Resume
- Business card
- Product landing page: to collect emails before product launch, to route visitors to mobile App Store/Play Store, to provide an overview of the product, etc.

Before jumping to conclusions, he gave a thought to this question:

> Why Jekyll + GitHub?

Especially when he already knows how to build a dynamic website using AWS hosting. While dynamic website covers the aforementioned use cases, he should think about the question from a different perspective. So he thought of this analogy: you can drive to work by both limousine and sedan car but would you spend an hour trying to find a place to park in the limousine or would you rather pick the sedan next time?

You can always rely on the big open source community when encountering any issues with Jekyll. There exists a lot of plugins such as analytics, comments, etc.

### Building a static website with Jekyll

Make sure you have all [prerequisites](https://jekyllrb.com/docs/installation/).

1. Install gems: `gem install jekyll bundler`

2. Create a new website: `jekyll new landing-page && cd landing-page`

3. Run your website: `bundle exec jekyll serve --livereload`

Now go to [`http://localhost:4000`](http://localhost:4000) and enjoy.

You can learn more about useful Jekyll features [here](https://jekyllrb.com/docs/step-by-step/01-setup/). Also plenty of Jekyll themes available at:
- [GitHub.com #jekyll-theme repos](https://github.com/topics/jekyll-theme)
- [jamstackthemes.dev](https://jamstackthemes.dev/ssg/jekyll/)
- [jekyllthemes.org](http://jekyllthemes.org/)
- [jekyllthemes.io](https://jekyllthemes.io/)
- [jekyll-themes.com](https://jekyll-themes.com/)

### Hosting the static website on GitHub

1. Go to your website folder and commit your changes

    ```bash
    cd landing-page
    git init
    git add .
    git commit -m "hosting my static website on GitHub"
    ```

2. Create [a new repo at GitHub](https://github.com/new) and follow instructions on how to push an existing repository from the command line

3. Go to the repository settings and scroll down to the "GitHub Pages" section. Then select the branch you would like to publish on GitHub Pages. Don't forget to save your changes.

In a couple of minutes, you should see the link to the new website under the "GitHub Pages" section.

You can learn more about configuring a custom domain for your website [here](https://docs.github.com/en/github/working-with-github-pages/configuring-a-custom-domain-for-your-github-pages-site). Pro tip: if your repository name is `<your-github-username>.github.io`, your website will be hosted at `<your-github-username>.github.io`.
