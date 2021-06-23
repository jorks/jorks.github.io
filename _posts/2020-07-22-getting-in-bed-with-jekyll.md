---
title:  "Getting in bed with Jekyll"
date:   2020-07-22 19:32:34 +1000
tags:
  - Jekyll
  - Laravel
---
Running Jekyll on macOS is reasonably simple thanks to Homebrew and the docs at [jekyllrb.com](https://jekyllrb.com/docs/installation/macos/). However, I already have a much-loved tool for my local dev playground, [Larvel Valet](https://laravel.com/docs/7.x/valet). Valet is always on, clean and lighting fast. If you're not familiar with Valet:

> Valet is a Laravel development environment for Mac minimalists. Laravel Valet configures your Mac to always run Nginx in the background when your machine starts. Then, using DnsMasq, Valet proxies all requests on the .test domain to point to sites installed on your local machine.

Valet serves your sites with nice domain names and https certificates ~ projects just feel more complete in your browser.

## The Building Blocks

As with everything fun on macOS, you will need [Homebrew](https://brew.sh/). You should review the official macOS guide at [jekyllrb.com](https://jekyllrb.com/docs/installation/macos/) before reading on. 

_This article is my real-world experience following the instructions, my style guide and my reference as I inevitably forget everything._

**Install Ruby, Bundler & Jekyll**

I only use Ruby for Jekyll, so I am not going to use `rbenv`, a more flexible approach to Ruby on macOS. Here is what I did.

- Install the latest version of Ruby (macOS ships with an older version of Ruby)<br>
	`brew install ruby`
- Add a custom path to your bash or zsh profile<br>
	`export PATH="/usr/local/opt/ruby/bin:$PATH"`
- Check you are running the correct Ruby, see instructions above<br>
	`which ruby && ruby -v`
- Install Bundller and Jekyll in user-land<br>
	`gem install --user-install bundler jekyll`
- Add another custom path<br>
	`export PATH="$HOME/.gem/ruby/2.7.0/bin:$PATH"`
- Run `gem env` And check that `GEM PATHS:` points to a path in your home directory.

**Create a new Jekyll site and serve it.**

- `mkdir ~/Sites/sup && cd ~/Sites/sup` make a project folder
- `jekyll new ~/Sites/sup` initiate a new Jekyll project
- `jekyll serve` will build your project into `_sites` folder serve it locally, visit [http://localhost:4000/](http://localhost:4000/).

At this point, you have a working Jekyll dev environment, but we can make this much nicer. Let's install valet and beautify our ugly localhost:port URL and give it a shiny SSL cert 🔐.

**Using Laravel Valet with Jekyll**

Install [Valet](https://laravel.com/docs/7.x/valet#installation) as per their instructions.

Valet has a few options for proxying your local dev workspaces to nice URLs (`park`, `proxy` and `link`), for our Jekyll project we are going to use `link` in combination with `jekyll build` (instead of `jekyll serve`). This will ensure our website always loads locally without an open Jekyll terminal.

I am not a Ruby expert, but I always run my `jekyll build` with `bundle exec`. As I understand, it ensures all dependencies, themes and plug-ins work.

- Make sure you are in the root of your new Jekyll project (`~/Sites/sup` in this example)
- Run `bundle exec jekyll build` to render your static site into the `_site` folder.
- `cd ~/Sites/sup/_site`
- Link the current working directory to Valet<br>
	`valet link sup`
- Secure the given domain with a trusted TLS certificate<br>
	`valet secure sup`

Check out your fancy new site in a browser: [https://sup.test](https://sup.test) 😎.

Finally, if you are actively working on your Jekyll project and want to render file saves in real-time, you can run the command with a watch flag:
	`bundle exec jekyll build --watch`

**Hot Tip** 💁‍♂️ Don't change your TLD to `.dev` or it cause issues with genuine websites that use this TLD - most probably long after you have forgotten you installed Valet.
{: .notice--warning}