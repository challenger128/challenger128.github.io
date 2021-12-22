---
title: Start your own blog on Github Pages 
date: 2021-12-22 13:00:00 +0500
categories: [Tutorial, Github Pages]
tags: [blog, github-pages, personal blog, jekyll]
pin: false
comments: true
---

![Github Pages & Jekyll](/assets/img/sample/gh-pages-blog/jekyll-github.jpg){: .shadow}

## Is there an easy way to create a blog?

I will definitely answer yes. You can make your own blog from scratch in almost couple of clicks, using Github Pages and Jekyll. 
Everyone may ask, "Why would I use it?". There are a lot of reasons to do it. Firstly, making something from scratch improves your technical skills. 
Secondly, it has no bloated code and target ads. Thirdly, you can modify and customize your blog as you want.

If I have convinced you, then follow these steps to do it.

## Step 1: Find a theme that you like

I recommend you to choose a free one from various Jekyll themes. You won't have to work with pages and styles, so that makes creating easier. 
Although, if you are experienced enough, you can make your own theme.

Browse these sites in search of themes.

- <http://jekyllthemes.org/>
- <https://jekyllthemes.io/>
- <https://jekyll-themes.com/>

> I've picked the [Chirpy Theme](https://github.com/cotes2020/jekyll-theme-chirpy), because it has a lot of features.

## Step 2: Setting up Github Pages 

Firstly, You need to create an Github Account if you don't have one. After you've chosen the theme, you need to host it. 
Usually themes contain a set of instruction in README and configure instructions may vary between different ones. 

[Chirpy Theme](https://github.com/cotes2020/jekyll-theme-chirpy) provides two ways to create repository for your site.

- [Using the Chirpy Starter](https://github.com/cotes2020/chirpy-starter/generate)
- [Forking on Github](https://github.com/cotes2020/jekyll-theme-chirpy/fork)

I've used first one and recommend the same to you, because it's easy to upgrade, isolates irrelevant project files so you can focus on writting.
Even if you need to update or change something in it, you can [override theme defaults](https://challenger128.me/posts/gh-pages-blog/#integrating-telegram-comments-to-your-website).

![Chirpy Starter](/assets/img/sample/gh-pages-blog/chirpy_starter.png){: .shadow}
_Filling in data on the Chirpy Starter web page_

You've followed the first link. Now name your repository as `<GH_USERNAME>.github.io`, where `<GH_USERNAME>` represents your Github username, and click on 
`Create repository from template`. 

After you've successfully created the repository, [clone](https://github.com/git-guides/git-clone) it locally.

Set `url` variable in `_config.yml` to `https://<GH_USERNAME>.github.io` and then [push](https://github.com/git-guides/git-push) changes to trigger Github Actions workflow.
Once the build is complete, a new remote branch named `gh-pages` will appear. Then go to your repository _Settings_ and click on _Pages_ in the left bar, and then
in the section **Source** of _Github Pages_ select the same parameters as in a screenshot and don't forget to click on `Save`.   

![Enable gh-pages](/assets/img/sample/gh-pages-blog/enable_gh-pages.png){: .shadow}
_Enable gh-pages_

Now wait a few minutes and check `https://<GH_USERNAME>.github.io/`. Your own website works!

## Step 3: Custom Domain

You can skip this step if you don't want to use a custom root domain. Firstly, you should purchase your domain on one of these sites.

- <https://www.godaddy.com/>
- <https://porkbun.com/>
- <https://www.namecheap.com/>

> I've purchased mine on Porkbun.

If you've already done it, then go into your domain management portal and add `A` type DNS records for Github Pages and `ALIAS` type record.

| Record Type | Host  | Answer  |
|-------------|-------|---------|
| A | your_domain.com | 185.199.108.153 |
| A | your_domain.com | 185.199.109.153 |
| A | your_domain.com | 185.199.110.153 |
| A | your_domain.com | 185.199.111.153 |
| ALIAS | www.your_domain.com | your_domain.com |

Head to your repository, then _Settings_ > _Pages_, where in custom domain section fill in it. Also make sure that you've marked `Enforce HTTPS`. 

![Enable gh-pages](/assets/img/sample/gh-pages-blog/custom_domain.png){: .shadow}
_Fill in custom domain_

Don't forget to update `url` variable in `_config.yml` and push changes. 

> DNS changes can take up a long time.

## Step 4: Enable comments to your posts.

[Chirpy Theme](https://github.com/cotes2020/jekyll-theme-chirpy) has _Disqus_ comments. 
However, there are a lot of disadvantages to using _Disqus_:

- Awful & shameful ads if you're using the free plan.
- Privacy problems.
- Increased page loading.

If you don't worry about these cons, you may sign up on _Disqus_ and add the site there.

There is another way to do it and it's the _Telegram Comments_.

Ups:

- Free.
- No ads.
- Users can upload multimedia attachments.
- Users can subscribe to comments.

Downs:

- There is no thread system.
- 100 comments per post is the limit.

### Integrating Telegram Comments to your website

Head to <https://comments.app/> and click on `CONNECT WEBSITE`, then set name as you want and enter your domain. 
Add these lines in `_config.yml`
```yaml
telegram:
  comments: true
  id: 'your site id from https://comments.app'
```
{: file='_config.yml' .nolineno}

Then create `_includes/telegram.html` in your website directory.

```html
<script async src="https://comments.app/js/widget.js?3" data-comments-app-website={{ site.telegram.id}} data-limit="100"></script>
```
{: file='_includes/telegram.html' .nolineno}

It the end, include `telegram.html` to your post layout. 

If you're using _Chirpy Theme_, copy [`_layouts/page.html`](https://github.com/challenger128/challenger128.github.io/blob/main/_layouts/page.html) and [`_layouts/post.html`](https://github.com/challenger128/challenger128.github.io/blob/main/_layouts/post.html) 
to your `repo/_layouts`. Don't forget to push changes.

## Tip: Running local server. 

Test your website before publication, you may do by running local server. 

```shell
bundle exec jekyll s
```
> The website will be available on `127.0.0.1:4000`

Make sure that you've installed [`Jekyll`](https://jekyllrb.com/docs/installation/). 

