---
layout: post
title: How I use Travis CI to build my website
author: Stefan Mitev
comments: false
date: 21.04.2019
permalink: "/articles/how-i-use-travis-ci-to-build-my-website"
tags:
- CI
- jekyll
---

## Contents:
- How was this website built
  - What is Jekyll
  - What is Github Pages
  - What is Travis CI
- How is the website repository organized
  - Workflow
- Setup
  - Github repo
  - Travis CI
  
> **Disclaimer:** This is not a general how-to article. Its more of an explanatory article for how I setup my website and everything else needed for having automatic deployments. 

# How was this website built

This website is powered by [Jekyll](https://jekyllrb.com/) which utilizes the [Hydeout](https://github.com/fongandrew/hydeout) theme that I modified to my liking.
 
> Jekyll is a powerful and extendable static site generator. Content is written in [markdown](TODO) and pages are generated using templates.
 
The website and blog are hosted on [Github Pages](https://pages.github.com/) and are being automatically deployed by [Travis CI](https://travis-ci.org/) whenever I make a change. A little further you will read the implementation details.

> Github Pages is a free hosting, provided by Github for personal and/or project websites. You can learn more about it [here](https://pages.github.com/).

If you are curious to see the source code of the website, you can check it out on my [Github](https://github.com/mrmitew/mrmitew.github.io/). In fact, if you were to clone the repository, you'll be able to run it locally with no effort at all.

```bash
git clone https://github.com/mrmitew/mrmitew.github.io.git mitevstefan
cd mitevstefan
bundle exec jekyll serve
```

`jekyll serve` builds the website any time a source file changes and serves it locally. To find out the address you should visit, look for the "Server address" output line in the terminal. By default it should be `http://127.0.0.1:4000/`.

# How is the website repository organized

There are three active branches, namely `master`, `release` and `develop`.

- `master` branch contains the generated static site content (the one you will find in the `_site` folder)
- `release` branch contains the jekyll code, including configs, layouts, sass, markdown files etc (everything needed to generate the website) 
- `develop` branch is a working copy of the `release` branch. It contains code and content that is not yet in production.

## Workflow

New changes are done in `develop`. When those are merged to `release`, Travis CI would trigger a new build and deploy the newly generated static content onto `master`. As soon that is done, the changes can be observed by visiting `mitevstefan.com` or `mrmitew.github.io`.

# Setup

## Git + Github

- Created a new repository `mrmitew.github.io` - its important to use your github username for the subdomain.
- Created `release` and `develop` branches.
- From the repository settings I chose `release` as a default branch, so that it will be the 'landing' branch when someone would open the repository. 
- Ran `bundle exec jekyll build` on my local machine to generate the static content and pushed it to `master`.
- Switched to `release`, which at this point was still empty*.
- Created a `.gitignore` file to exclude some files and directories I did not want to commit:
```
.idea/*
_site/*
Gemfile.lock
.sass-cache/
```
- Added the jekyll code in the root directory
- Pushed to the remote `release` branch.
- Since I wanted to use my own domain and not `mrmitew.github.io`, I created `A` records through my DNS provider that pointed to GitHub's IP addresses. Click 
- [here](https://help.github.com/en/articles/using-a-custom-domain-with-github-pages) for more info.
- Entered my custom domain name from the _"GitHub Pages"_ section in the repository settings and also ticked the box for enforcing HTTPS.

_* In fact, that's what I was aiming to do, but in reality `release` was created after pushing to `master` with the static content already in it, so I had to delete it first._

## Travis CI

- Headed over to [Travis CI](https://travis-ci.org/) and created a new account, which took less than a minute. After syncing my GitHub repositories, I enabled the Travis CI integration for `mrmitew.github.io` repository.
- From [GitHub's Developer settings](https://github.com/settings/tokens) I created a new personal access token and gave it a custom name. From there, I selected `repo:status`, `repo_deployment` and `public_repo` scopes.
- Generated the new access token and copied it for later usage in the Travis CI dashboard. It is important to copy it before closing the page because there is no possible way to retrieve it again.
- In Travis CI dashboard, I created a new environment variable `GITHUB_TOKEN` with value equals to the token, generated earlier.
- Finally, I committed on the `release` branch a new file, called `.travis.yml` that contained instructions for Travis CI:

```ruby
language: ruby
cache: bundler
branches:
  only:
    - release
script:
  - JEKYLL_ENV=production bundle exec jekyll build --destination _site
deploy:
  fqdn: mitevstefan.com
  provider: pages
  local-dir: ./_site
  target-branch: master
  email: deploy@travis-ci.org
  name: Deployment Bot
  skip-cleanup: true
  github-token: $GITHUB_TOKEN
  keep-history: true
  on:
    branch: release
```
- Pushing it to `release`, triggered the first successful build and deployment by Travis CI :)

# Further reading
- [Using a custom domain with GitHub Pages](https://help.github.com/en/articles/using-a-custom-domain-with-github-pages)
- [.travis.yml configuration for GitHub Pages](https://docs.travis-ci.com/user/deployment/pages/)