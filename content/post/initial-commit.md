+++
date = "2017-02-12T00:54:28-08:00"
title = "Initial Commit: Setup Blog with Hugo + Netlify"
draft = false
+++

![Hugo Logo](/hugo-logo.png)

This is my first post for my brand new tech blog and I'm going to talk about how I finally made my initial commit to the Github repo containing this very blog after the umpteenth time of starting over.  Retrospectively speaking, I find it pretty amusing now (after finally figuring it out) how many times I had to blow away the repo contents to finally get it right and have the blog actually render properly, CSS included as well as setup the hosting platform to run the blog.

The two technologies I used to build and host this blog are:

- Hugo: Go-based static site generator
- Netlify: hosting platform using Free tier plan

## Why Hugo?
I've been around the block before when it comes to utilizing static site generators ([Jekyll](https://jekyllrb.com/) and [Middleman](https://middlemanapp.com/)).  Hugo appealed to me most because of its superior page build speed over the other solutions.  The dev server is extremely fast on rebuilding the site when you make your local changes and is setup to watch for these file changes by default using LiveReload.

Many people knock on Hugo with regards to it's lack of a built-in asset pipeline to provide additional frameworks, libraries and technologies to the developer such as SASS, ES6, bundling, etc.  To me, this is actually a great plus.  I like working with a smaller set of tools which get the job done really well.  Only when I've reached the limitation of the tools I'm using or want to do some fancier things will I start looking at more feature rich tools or libraries or technologies.

It's also written in Go, something I've been inkling to learn for a while now!

For a more comprehensive comparision review of the other alternative solutions, you can check out this [article](https://www.smashingmagazine.com/2015/11/static-website-generators-jekyll-middleman-roots-hugo-review/).

## Why Netlify?
Netlify has a lot of great features and benefits out of the box which made it a good choice for me.

- Easy to setup
- Automated deployment on Git pushes
- Fast page loads, no cold starts waiting for server to spin up
- Built-in support for Hugo

## Setup
Getting everything setup was for the most part straightforward.  I followed the [quickstart guide]((https://gohugo.io/overview/quickstart/)) provided by Hugo to help me walk through the steps for creating the blog, creating a new post, adding a theme, running a dev server to preview the blog and finally deploy the site.

I followed the Hugo Quickstart guide until step 12.  For step 12, I moved over to the [Netlify guide](https://www-redirects.netlify.com/blog/2015/10/06/a-step-by-step-guide-hugo-on-netlify) to figure out how to deploy my blog onto the Netlify platform.  Deploying to Netlify was as simple as picking my blog's repo from Github and entering the start-up command to execute (hugo) to build the site once the repo was checked out onto the Netlify server.

There were a few "gotchas" I had to figure out through the setup process which I wish I had found out earlier to save me the few hours of head scratching and digging I had to do.

## Gotchas
#### #1 Different themes read the content folder differently
This one didn't take too long to figure out.  I was trying out 2 different themes provided from [themes.gohugo.io](themes.gohugo.io), cocoa and hemingway.  I started working with the hemingway theme which expects the posts to live in the `/content/post` folder.  After switching to the cocoa theme, my posts suddenly disappeared!  It turned out the cocoa theme expected the posts to be in the `/content/blog` folder.

There are many approaches you can take to fix this problem.  One way is to modify the templates to use the path where your posts live in.  Another way is to check to see if your `config.toml` file has a parameter which defines where the templates will look for the posts.  I took the more lazy approach and simply renamed the folder name to match with the theme I was using. (I eventually settled for the hemingway theme so I renamed the folder back to `post`)

#### #2 `baseurl` config parameter can break link refs
So, I had my blog working in my local dev environment fine using the cocoa theme.  I then switched to the hemingway theme and suddenly my blog was naked.  All the CSS disappeared.  What happened?!  Interestingly enough, when I took a look at the header of the `index.html` file, I saw that the reference to the main css file looked like: `//css/style.css`.

OK! Problem found.  But where is this extra `/` being added?  So, for background, the extra `/` is actually the value that the `baseurl` config parameter needs to be set to when deploying to Netlify.  The next place I looked at was the `header.html` partial file contained in the hemingway theme.  If the CSS isn't being loaded properly, then this should definitely be the place where the problem is happening.  And sure enough, it was!

Can you see the problem in the snippet?

```
<link rel="stylesheet" href="{{ .Site.BaseURL }}/css/style.css">
```

The problem is that the theme makes the assumption that the `.Site.BaseURL` template variable will not provide a trailing `/`.  The href will work if this assumption is true.  But, in my case since `baseurl` is already equal to `/` which we can agree is pretty much a trailing `/`, then the assumption can't be upheld anymore.  

To fix this, I simply removed the `/` after `{{ .Site.BaseURL }}` and left it up to me to include it in the event I wanted to add a different `baseurl` path in the config.  

And Voila!  My blog was able to display the hemingway theme!

#### #3 Hugo themes cloned into `themes` directory can break Netlify deployment
This problem eluded me the most.  After following all the steps in both step-by-step guides for setting up Hugo and Netlify, I still kept opening a Chrome tab to an empty page.  What was going wrong?

Strangely, when I was looking at my blog's Github repo page I couldn't navigate into the themes folder.  So with a little Google-fu and StackOverflow perusing, I came across something new that I learned about git!

[Git Submodules!](https://github.com/blog/2104-working-with-submodules)

From the article I referenced above, [@jaw6](https://github.com/jaw6) says:

> Submodules allow you to include or embed one or more repositories as a sub-folder inside another repository.

Another gotcha with submodules is that the contents of the submodule repo will not automatically be populated in the target repo unless explicitly told to do so!

Back to the deployment issue, Netlify would log that no theme was found when I tried several times to push a new repo containing different themes up to Github.

So with a little deductive reasoning, I figured out that the cloned themes in the themes directory worked fine locally because the contents were available, however, when the I pushed the blog repo up to Github, it was actually **not including the contents of the themes since they were identified as submodules** since they contained .git versioning metadata.

My solution was to download the repo as a .zip and place it as a non-versioned theme so that I didn't have to worry about the theme changing in the future and also configuring it to explicitly push the contents of the theme submodule to my blog repo.  After this, I was finally able to get my blog up and running and here we are!

## Resources
- Hugo Quickstart Guide: [https://gohugo.io/overview/quickstart/](https://gohugo.io/overview/quickstart/)
- Hugo Themes: [https://themes.gohugo.io/cocoa/](https://themes.gohugo.io/cocoa)
- A Step-by-Step Guide: Hugo on Netlify: [https://www-redirects.netlify.com/blog/2015/10/06/a-step-by-step-guide-hugo-on-netlify](https://www-redirects.netlify.com/blog/2015/10/06/a-step-by-step-guide-hugo-on-netlify) (This one is an older version of the current guide, but I used it because it just uses the Hugo CLI.  The newer guide uses Victor Hugo which has extra stuff I didn't need.)
- Static Website Generators Reviewed: Jekyll, Middleman, Roots, Hugo: [https://www.smashingmagazine.com/2015/11/static-website-generators-jekyll-middleman-roots-hugo-review/](https://www.smashingmagazine.com/2015/11/static-website-generators-jekyll-middleman-roots-hugo-review/)
- Working with submodules: [https://github.com/blog/2104-working-with-submodules](https://github.com/blog/2104-working-with-submodules)