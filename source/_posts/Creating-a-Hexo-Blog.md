---
title: Creating a Hexo Blog with Heroku
date: 2017-05-09 15:04:01
tags: tech
---

This blog was a learning experience––firstly, because this was my first website all my own outside of GitHub Pages, secondly, because it's with a CMS (Hexo, obviously) other than WordPress and hosted on Heroku. This isn't my first experience with Heroku, but it is as a from scratch enterprise.

## Reasons I chose Hexo

No reason really. Looked over the docs, looked simple enough and I liked being able to create blog posts with [markdown](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet) and the simple looking deploys.

## The Process

I originally intended to doing something like this with GitHub Pages, I created a repo, set up Hexo, found this nifty [theme](https://github.com/probberechts/cactus-dark), and was all set... then I found out Jekyll is pretty picky about themes when you have a `config.yml` with something like:
```
theme:cactus-dark
```
You think you're set and you're served a build error:
>The page build completed successfully, but returned the following warning for the `master` branch:
<br />
>You are attempting to use a Jekyll theme, "cactus-dark", which is not supported by GitHub Pages. Please visit https://pages.github.com/themes/ for a list of supported themes

And alas Jekyll themes are circa 2012 and look like it. So new plan, I'll host with something else.

## Heroku, and how to deploy Hexo to it in 7 easy steps

Hexo doesn't do a great job explaining the deployment process to Heroku, it gives you the deployment config documentation and then nothing. And pouring through the Heroku documentation can be a bit of a slog, and obviously has no Hexo-centric info. So lets talk about it. Once you have a repo with your blog and create a Node.js Heroku app (which is super simple from Heroku's UI) follow these steps:
1. login in to your Heroku account from your terminal `$ heroku login`
2. remove `public/` from the .gitignore
  + when you run `$ hexo generate` Hexo generates static files of your blog and puts them in the `public/` directory
  + once `public/` is no longer ignored, Heroku can see the files it needs to host your site
3. add `engines` to your `package.json`, it seems odd they're not there out of the box with Hexo. It should look like:
```
"engines": {
  "node": "7.1.0",
  "npm": "3.10.9"
}
```
4. add Hexo's offical hexo-deployer-heroku plugin
  + `$ npm install hexo-deployer-heroku --save`
5. <span id="number5"></span>create a public SSH key so you can deploy to Heroku
  + `$ ssh-keygen -t rsa` will create the key
  + then `$ heroku keys:add` adds that key to your Heroku account
  + you can then use `$ ssh -v git@heroku.com` to validate your connection
6. add your configuration settings to your `_config.yml` file, it should look something like:
```
deploy:
  type: heroku
  repo: https://git.heroku.com/YOUR-APP-NAME-HERE.git
  message: Deployment of Hexo blog to Heroku
```
7. commit your changes and then why not try out a deploy
  + commit your stuff
  + then `$ hexo generate -deploy` to generate static files and deploy to your Heroku app.

![](/images/brad-dance2.gif)

There's other local environment things to set up as well, which can be found at this [helpful post](http://www.graymatterdeveloper.com/2016/01/05/setting-up/).

***

# Toubleshooting Tips I Learned Along the Way
## The Error:
>remote: -----> Failed to detect app matching https://codon-buildpacks.s3.amazonaws.com/buildpacks/heroku/nodejs.tgz buildpack
remote:        More info: https://devcenter.heroku.com/articles/buildpacks#detection-failure

## The Fix:
`$ heroku buildpacks:remove heroku/nodejs -a rooks-books-blog`
Yup, that's it, and Heroku will give you this nice little response:
"Buildpack removed. Next release on YOUR-APP-NAME-HERE will detect buildpack normally." ...as if the buildpack was unnatural that whole time.

Also found [here](https://github.com/hexojs/hexo-deployer-heroku/issues/2)
<br />
## The Error:
>'Permission denied (publickey). fatal: Could not read from remote repository.'

## The Fix:
Please see [^number 5 above^](#number5)
<br />
## The Error:
>The page build failed with the following error:
"The submodule `EXAMPLE_SUBMODULE` was not properly initialized with a `.gitmodules` file.

## The Fix:
GitHub says:
>In the submodule's directory, run `$ git submodule init`, then `$ git submodule update`.
Commit and push your changes to trigger another build on the server.

That didn't work for me.

The bottom line is if you install a theme make sure you install it as [submodule](https://git-scm.com/book/en/v2/Git-Tools-Submodules). A lot of themes' `README.md`s say to just:
```
$ git clone https://github.com/probberechts/cactus-dark.git themes/cactus-dark
```
and update the `config.yml` theme.

I would recommend first forking the theme you want, so you can make changes to it of your own. Once you've done that, run:
```
$ git submodule add https://github.com/YOUR-GITHUB-USERNAME-HERE/cactus-dark.git themes/cactus-dark
```
Now whenever you make a change within the submodule you commit it seperately, but hell, it's worth it for the freedom of owning the theme.
