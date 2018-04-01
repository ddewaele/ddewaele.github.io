---
layout: post
title: Running Jekyll in Docker
date: 2018-04-01 00:28:14 +0200
tags: [docker, jekyll]
excerpt_separator: <!--more-->
---
## Introduction

Jekyll is a great platform for publishing content, but it can be quite difficult to get up and running on a local environment due to its dependencies.
Jekyll is a blog-aware, static site generator in Ruby and in order to install it you need to ensure that you have the 

- correct version of Ruby installed
- RubyGems installed
- GCC / Make installed

If you're not familiar with these tools (ruby , gem, bundle, ....) then getting up and running can be time consuming and cumbersome. 

Depending on your OS, you might already have an existing version of Ruby, and you might need to upgrade or install other packages, potentially risking the sanity of other applications depending on those runtimes.

There must be a better way.... 

Enter Docker, the container technology that can help us encapsulate the jekyll specifics and its dependencies by keeping them `contained`.

![]({{ site.url }}/assets/images/jekyll_docker/logo.png)

<!--more-->

## What is Jekyll

Jekyll is a static site generator, typically used for blogs.
It processes a folder containing different items like posts, drafts, layouts and converts it into a fully fledged static html site that you can publish.
The folder is typically stored in version control, and a build pipeline takes care of processing that folder and publishing a new site.

In order to process that folder, you need to have the jekyll executable and all of its dependencies. 

- `jekyll new my-blog` (to create a new folder structure)
- `jekyll build` (to process the folder structure, and generate an html site)
- `jekyll serve` (to run an http server on your local environment)

However, as stated before, in order to run the `jekyll` executable on your environment, you'll need to have a lot of dependencies in place.

## Jekyll in docker

In [the following github project](https://github.com/envygeeks/jekyll-docker), you can find a containerized version of jekyll where all of these dependencies are available in the container.

As such, in order to get started, the only thing you need to do is this:

```bash
export JEKYLL_VERSION=3.5
mkdir -p ~/Projects/NewBlog ; cd ~/Projects/NewBlog
docker run --rm --volume="$PWD:/srv/jekyll" -it jekyll/jekyll:$JEKYLL_VERSION jekyll new .
```

you should see the following output 

```bash

Running bundle install in /srv/jekyll... 
  Bundler: The dependency tzinfo-data (>= 0) will be unused by any of the platforms Bundler is installing for. Bundler is installing for ruby but the dependency is only for x86-mingw32, x86-mswin32, x64-mingw32, java. To add those platforms to the bundle, run `bundle lock --add-platform x86-mingw32 x86-mswin32 x64-mingw32 java`.
  Bundler: Fetching gem metadata from https://rubygems.org/...........
  Bundler: Fetching version metadata from https://rubygems.org/..
  Bundler: Fetching dependency metadata from https://rubygems.org/.
  Bundler: Resolving dependencies...
  Bundler: Fetching public_suffix 3.0.2
  Bundler: Installing public_suffix 3.0.2
  Bundler: Using bundler 1.15.4
  Bundler: Using colorator 1.1.0
  Bundler: Fetching ffi 1.9.23
  Bundler: Installing ffi 1.9.23 with native extensions
  Bundler: Using forwardable-extended 2.6.0
  Bundler: Fetching rb-fsevent 0.10.3
  Bundler: Installing rb-fsevent 0.10.3
  Bundler: Fetching ruby_dep 1.5.0
  Bundler: Installing ruby_dep 1.5.0
  Bundler: Fetching kramdown 1.16.2
  Bundler: Installing kramdown 1.16.2
  Bundler: Using liquid 4.0.0
  Bundler: Using mercenary 0.3.6
  Bundler: Using rouge 1.11.1
  Bundler: Using safe_yaml 1.0.4
  Bundler: Using addressable 2.5.2
  Bundler: Using rb-inotify 0.9.10
  Bundler: Fetching pathutil 0.16.1
  Bundler: Installing pathutil 0.16.1
  Bundler: Using sass-listen 4.0.0
  Bundler: Fetching listen 3.1.5
  Bundler: Installing listen 3.1.5
  Bundler: Fetching sass 3.5.6
  Bundler: Installing sass 3.5.6
  Bundler: Fetching jekyll-watch 1.5.1
  Bundler: Installing jekyll-watch 1.5.1
  Bundler: Fetching jekyll-sass-converter 1.5.2
  Bundler: Installing jekyll-sass-converter 1.5.2
  Bundler: Using jekyll 3.5.2
  Bundler: Fetching jekyll-feed 0.9.3
  Bundler: Installing jekyll-feed 0.9.3
  Bundler: Fetching jekyll-seo-tag 2.4.0
  Bundler: Installing jekyll-seo-tag 2.4.0
  Bundler: Fetching minima 2.4.1
  Bundler: Installing minima 2.4.1
  Bundler: Bundle complete! 4 Gemfile dependencies, 24 gems now installed.
  Bundler: Bundled gems are installed into /usr/local/bundle.
New jekyll site installed in /srv/jekyll. 
```

What happened here ?

- We've started a docker jekyll container 
- We've mounted the current folder as `/srv/jekyll` in the container
- We've passed the `jekyll new` command
- A new site was created in the current folder by the container

Because the volume was mounted on our host, the generated files end up on the host as well.

At this point you have a very minimalistic set of files. These files are primarily configuration files and a sample post.
The site itself hasn't been `built` yet.

```
find .

.
./.gitignore
./404.html
./_config.yml
./_posts
./_posts/2018-04-01-welcome-to-jekyll.markdown
./about.md
./Gemfile
./Gemfile.lock
./index.md
```

To build it, we execute the following command:

```
docker run --rm --volume="$PWD:/srv/jekyll" -it jekyll/jekyll:$JEKYLL_VERSION jekyll build
```

You'll get the following output 

```
docker run --rm --volume="$PWD:/srv/jekyll" -it jekyll/jekyll:$JEKYLL_VERSION jekyll build

The following gems are missing
 * public_suffix (3.0.2)
 * ffi (1.9.23)
 * rb-fsevent (0.10.3)
 * sass (3.5.6)
 * jekyll-sass-converter (1.5.2)
 * ruby_dep (1.5.0)
 * listen (3.1.5)
 * jekyll-watch (1.5.1)
 * kramdown (1.16.2)
 * pathutil (0.16.1)
 * jekyll-feed (0.9.3)
 * jekyll-seo-tag (2.4.0)
 * minima (2.4.1)
Install missing gems with `bundle install`
Fetching gem metadata from https://rubygems.org/...........
Fetching version metadata from https://rubygems.org/..
Fetching dependency metadata from https://rubygems.org/.
Fetching public_suffix 3.0.2
Installing public_suffix 3.0.2
Using bundler 1.15.4
Using colorator 1.1.0
Fetching ffi 1.9.23
Installing ffi 1.9.23 with native extensions
Using forwardable-extended 2.6.0
Fetching rb-fsevent 0.10.3
Installing rb-fsevent 0.10.3
Fetching ruby_dep 1.5.0
Installing ruby_dep 1.5.0
Fetching kramdown 1.16.2
Installing kramdown 1.16.2
Using liquid 4.0.0
Using mercenary 0.3.6
Using rouge 1.11.1
Using safe_yaml 1.0.4
Using addressable 2.5.2
Using rb-inotify 0.9.10
Fetching pathutil 0.16.1
Installing pathutil 0.16.1
Using sass-listen 4.0.0
Fetching listen 3.1.5
Installing listen 3.1.5
Fetching sass 3.5.6
Installing sass 3.5.6
Fetching jekyll-watch 1.5.1
Installing jekyll-watch 1.5.1
Fetching jekyll-sass-converter 1.5.2
Installing jekyll-sass-converter 1.5.2
Using jekyll 3.5.2
Fetching jekyll-feed 0.9.3
Installing jekyll-feed 0.9.3
Fetching jekyll-seo-tag 2.4.0
Installing jekyll-seo-tag 2.4.0
Fetching minima 2.4.1
Installing minima 2.4.1
Bundle complete! 4 Gemfile dependencies, 24 gems now installed.
Bundled gems are installed into /usr/local/bundle.
Configuration file: /srv/jekyll/_config.yml
            Source: /srv/jekyll
       Destination: /srv/jekyll/_site
 Incremental build: disabled. Enable with --incremental
      Generating... 
                    done in 1.135 seconds.
 Auto-regeneration: disabled. Use --watch to enable.
 ```

Not how our output folder has grown a bit :

```
find .

.
./.gitignore
./.sass-cache
./.sass-cache/6f185a96dff45864ec708ea7d5aed927991ce39e
./.sass-cache/6f185a96dff45864ec708ea7d5aed927991ce39e/minima.scssc
./.sass-cache/bf0d24d96991851c620c912ab9fea583ebd85dad
./.sass-cache/bf0d24d96991851c620c912ab9fea583ebd85dad/_base.scssc
./.sass-cache/bf0d24d96991851c620c912ab9fea583ebd85dad/_layout.scssc
./.sass-cache/bf0d24d96991851c620c912ab9fea583ebd85dad/_syntax-highlighting.scssc
./404.html
./_config.yml
./_posts
./_posts/2018-04-01-welcome-to-jekyll.markdown
./_site
./_site/404.html
./_site/about
./_site/about/index.html
./_site/assets
./_site/assets/main.css
./_site/assets/minima-social-icons.svg
./_site/feed.xml
./_site/index.html
./_site/jekyll
./_site/jekyll/update
./_site/jekyll/update/2018
./_site/jekyll/update/2018/04
./_site/jekyll/update/2018/04/01
./_site/jekyll/update/2018/04/01/welcome-to-jekyll.html
./about.md
./Gemfile
./Gemfile.lock
./index.md
```

Notice also how a `_site` folder was created containing your actual site. (the generated html based on the markdown posts)

Now that you've got a site, you'll probably want to run it locally.

Execute following command :

```
docker run --name newblog --volume="$PWD:/srv/jekyll" -p 3000:4000 -it jekyll/jekyll:$JEKYLL_VERSION jekyll serve --watch --drafts
```

![]({{ site.url }}/assets/images/jekyll_docker/jekyll-site.png)

As you make changes to your blog, they will automatically be picked up the jekyll process running in the docker container.


## Themes

We're going to update our blog with [a new template](https://github.com/fongandrew/hydeout).

We'll need to add the following gems to our `Gemfile`.

```
group :jekyll_plugins do
   gem "jekyll-feed", "~> 0.6"
   gem "jekyll-gist", "~> 1.4"
   gem "jekyll-paginate", "~> 1.1"
   gem "jekyll-theme-hydeout", "~> 3.4"   
end
```

Next, in our `_config.yml` file, we'll enable pagination, set the theme to `jekyll-theme-hydeout` add add some plugins

```
paginate:         5

# Build settings
markdown: kramdown
theme: jekyll-theme-hydeout
plugins:
  - jekyll-feed
  - jekyll-gist
  - jekyll-paginate  
```

Finally, we'll need to remove the standard `index.md` with an `index.html` that contains this:

```
---
layout: index
title: Home
---
```

Simply running the jekyll build

```
docker run --name newblog --volume="$PWD:/srv/jekyll" -p 3000:4000 -it jekyll/jekyll:$JEKYLL_VERSION jekyll serve --watch --drafts
```

or restarting your docker container 

```
docker restart newblog
```

will get you your newly themes site:

![]({{ site.url }}/assets/images/jekyll_docker/themed-site.png)


## Commands

To recap, here our some commands you'll frequently use :

### Creating a new site
```
docker run --rm --volume="$PWD:/srv/jekyll" -it jekyll/jekyll:$JEKYLL_VERSION jekyll new newblog
```
### Building an existing site
```
docker run --rm --volume="$PWD:/srv/jekyll" -it jekyll/jekyll:$JEKYLL_VERSION jekyll build
```
### Serving the blog
```
docker run --name myblog --volume="$PWD:/srv/jekyll" -p 4000:4000 -it jekyll/jekyll:$JEKYLL_VERSION jekyll serve --watch --drafts
```
### Executing commands in the container
```
docker exec -ti myblog gem install "jekyll-theme-hydeout"
```

### Executing a shell in the container
```
docker exec -ti myblog /bin/sh
```

Removing container
```
docker rm -f myblog
```

