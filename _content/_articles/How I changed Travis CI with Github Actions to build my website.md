---
layout: post
title: How I use changed Travis CI with Github Actions to build my website
author: Stefan Mitev
comments: false
date: 19.05.2022
permalink: "/articles/how-i-use-changed-travis-ci-with-github-actions-to-build-my-website"
tags:
- CI
- jekyll
---

It's been a while since I posted last. In fact I was meaning to post something else first, but I came to the realization that I no longer can use my old Travis CI setup without making some changes. However, instead of fixing it, I took the opportunity to learn something new and decided to give [GitHub Actions](https://github.com/features/actions) a go!

To learn about the setup of the website and repository, please read my old article "[How I use Travis CI to build my website](https://mitevstefan.com/articles/how-i-use-travis-ci-to-build-my-website)".

Without changing the structure of the repository, I only had to create a definition for my new GitHub workflow and commit it on my main branch (`release` in this case). GitHub then validated the workflow and made it appear in my [Actions](https://github.com/mrmitew/mrmitew.github.io/actions).

I called the workflow "Deploy site" and I stored it in `.github/workflows/deploy-site.yml`. 

You can choose to re-use existing actions, do everything manually or do like me - mix of both.

The actions I re-used are:
- `actions/checkout` for checking-out my repository
- `ruby/setup-ruby` for downloading and setting up ruby
- `peaceiris/actions-gh-pages` to deploy my static files to the `master` branch

Once my static files end up in `master`,  GitHub's `pages-build-deployment` workflow would be automatically triggered and will deploy the website to GitHub Pages.

Here is the definition I used

```yaml
# .github/workflows/deploy-site.yml
name: Deploy site

on:
  push:
    branches: release

env:
  RUBY_VERSION: 2.7

jobs:
  build-deploy:
    name: "Build and deploy"
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v3
      
      - name: Set up Ruby
        uses: ruby/setup-ruby@v1
        with:
          ruby-version: ${{ env.RUBY_VERSION }}
          bundler-cache: true

      - name: Set up dependencies
        run: |
          gem update --system --no-document
          gem update bundler --no-document
          bundle config set path vendor/bundle
          bundle install --jobs 4 --retry 3
          bundle clean

      - name: Build
        run: JEKYLL_ENV=production bundle exec jekyll build --verbose --trace

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./_site
          publish_branch: master
          commit_message: ${{ github.event.head_commit.message }}
          user_name: 'github-actions[bot]'
          user_email: 'github-actions[bot]@users.noreply.github.com'
          force_orphan: false
          destination_dir: ./
          cname: mitevstefan.com
```

## References
- https://github.com/actions/checkout
- https://github.com/ruby/setup-ruby
- https://github.com/peaceiris/actions-gh-pages
- https://github.com/jekyll/jekyll