---
layout: post
published: true
---

I thought it would be nice to describe how I actually post to this blog, meaning which technologies and services I use and how did I got to a state I'm comfortable with, for now.

I don't pay anything to keep this blog running, thanks to the great service that is GitHub and to the great community that is open source. I also write all the content in Markdown, which is IMHO a lot better than having to deal with a WYSIWYG editor.

For example, all my posts are sourced from [this directory](https://github.com/rodolfo42/rodolfo42.github.io/tree/master/_posts)

<!-- more -->

## The stack

Basically I'll describe how to work with these things to create your blog:

- **Poole** for the blog style
- **GitHub Pages** for hosting
- **Prose.io** for online realtime editing
- **A DNS Registrar** for hosting it under your own domain *(optional)*

### Poole

To get to Poole I'll first introduce your next best friend: [Jekyll](http://jekyllrb.com/).

If you already know Jekyll, you can skip this next paragraph.

**Jekyll** is a static site generator in Ruby. It's basically a way for you to have dynamic content but without any runtime dependencies to serve it, other than any simple web server. You write your pages in Markdown, define layouts in plain HTML/CSS/JS and you're good to go. Have a look at their [docs](http://jekyllrb.com/docs/home/).

**[Poole](http://getpoole.com)** is basically a starting point for a given Jekyll site. As such, it just provides a nice foundation for writing your pages with minimal configuration (or none at all) rather than being a tool.

It also gives you two themes you can freely use. The one you see in this blog is the [Lanyon](http://lanyon.getpoole.com/) theme with some minor styling changes I've made (e.g. the avatar you see when you draw the sidebar on your left).

To get Poole and Jekyll working, follow [these instructions](https://github.com/poole/poole#usage).

### GitHub Pages

**GitHub Pages** is a free and hosted service for serving static files as HTML/CSS/JS from the awesome GitHub. The content of the site must be entirely available on a Github repo. There are two types of sites:

- **Project sites**:  
These are sites used to publish a homepage/documentation for a project. The static files must be placed under a branch named 'gh-pages' in the project's repo.

- **User sites**:  
This is the one you'll use for a blog. You may only have one user site per Github username, and all the content is to be under a repo named `<username>.github.io`. You can see I have my own [here](https://github.com/rodolfo42/rodolfo42.github.io).

> The `<username>` must match the owner of the repo - you won't be able to have GitHub publish a repo named `john.github.io` under a user named `johnny`.

To get GitHub Pages to publish your page, all you need to do is to create a repository named as above in GitHub, `git init` inside the directory and `git push` all your files. After that, it will be available at `http://<username>.github.io/`

The [Github Pages homepage](https://pages.github.com/) provides a quick and excellent guide to publishing from scratch.

### Prose.io

[**Prose.io**](http://prose.io/) is a free online service that lets you edit any text file in any GitHub repository you own and commit the changes, all without any dependency from your machine.

It means you can create, edit and publish your posts while waiting for a doctor's appointment, at your grandmother's house or in the bus. All you need is a browser, go to Prose.io and authorize the app with Github to publish the changes under your name.

> You can edit files online in Github as well, but I prefer Prose.io as it's cleaner and does this one thing well.

### Publishing under your domain (optional)

If don't already have a domain name, get one. Then come back here.

To have your blog published under your own domain, you have two options, depending on the type of domain you want your blog to be hosted:

-  `subdomain.yourdomain.com`  
  You need a `CNAME` record pointing to `<username>.github.io`

- `yourdomain.com` (no subdomain)  
  Insert an A entry pointing to [these IPs](https://help.github.com/articles/setting-up-a-custom-domain-with-github-pages#apex-domains) from GitHub

You can read about that in more detail [here](https://help.github.com/articles/setting-up-a-custom-domain-with-github-pages)

Hope it helps!

There's another great post on this [here](http://parezcoydigo.wordpress.com/2013/08/26/getting-started-with-github-and-prose-io/)