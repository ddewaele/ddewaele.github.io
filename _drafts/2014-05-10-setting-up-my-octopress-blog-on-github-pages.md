---
layout: post
title: "Setting up my Octopress blog on Github pages"
date: 2014-05-10 13:28:14 +0200
comments: true
categories: 
author: Davy De Waele
categories: [Github, Jekyll, Octopress]
---

In this blog post I'll show you how to setup octopress on your blog. In this article we'll use Github pages to deploy our target. The goal of the article is to provide you with an overview on how Octopress can be setup and get you started very quickly.
<!-- more -->

## Follow instructions

The first thing you need to is ```install``` octopress. In order to do that you simply clone the octopress repository and execute a couple of commands on it.


- git clone git://github.com/imathis/octopress.git octopress
- cd octopress/
- gem install bundler
- rbenv rehash 
- bundle install
- rake install

At this point your blog is installed.

## Previewing

You can already preview your blog by executing the ```rake preview``` command. After executing the command you should see the following output 

Test Ruby

``` ruby Discover if a number is prime http://www.noulakaz.net/weblog/2007/03/18/a-regular-expression-to-check-for-prime-numbers/ Source Article linenos:false
class Fixnum
  def prime?
    ('1' * self) !~ /^1?$|^(11+?)\1+$/
  end
end
```

Test java

``` java
package com.mkyong.io;
 
import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;
 
public class BufferedReaderExample {
 
    public static void main(String[] args) {
 
        BufferedReader br = null;
 
        try {
 
            String sCurrentLine;
 
            br = new BufferedReader(new FileReader("C:\\testing.txt"));
 
            while ((sCurrentLine = br.readLine()) != null) {
                System.out.println(sCurrentLine);
            }
 
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                if (br != null)br.close();
            } catch (IOException ex) {
                ex.printStackTrace();
            }
        }
 
    }
}
```

Test gist

{% gist 28e219162e020c2a7126 %}


Test gist2

{% gist 1d14aabee41138e97079 %}



Test code

``` coffeescript Coffeescript http://www.google.com link Tricks start:1 mark:3,4
Starting to watch source with Jekyll and Compass. Starting Rack on port 4000
Configuration from /Users/ddewaele/octopress/octopress/_config.yml
[2014-05-17 11:34:54] INFO  WEBrick 1.3.1
[2014-05-17 11:34:54] INFO  ruby 1.9.3 (2013-06-27) [x86_64-darwin11.4.2]
[2014-05-17 11:34:54] INFO  WEBrick::HTTPServer#start: pid=14603 port=4000
Auto-regenerating enabled: source -> public
[2014-05-17 11:34:54] regeneration: 94 files changed
>>> Change detected at 11:34:54 to: screen.scss
identical public/stylesheets/screen.css 

Dear developers making use of FSSM in your projects,
FSSM is essentially dead at this point. Further development will
be taking place in the new shared guard/listen project. Please
let us know if you need help transitioning! ^_^b
- Travis Tilley

>>> Compass is watching for changes. Press Ctrl-C to Stop.
```

Test 2 

``` coffeescript Coffeescript Tricks start:51 mark:52,54-55
# Given an alphabet:
alphabet = 'abcdefghijklmnopqrstuvwxyz'

# Iterate over part of the alphabet:
console.log letter for letter in alphabet[4..8]
```


Test 3

<pre>
package com.mkyong.io;
 
import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;
 
public class BufferedReaderExample {
 
    public static void main(String[] args) {
 
        BufferedReader br = null;
 
        try {
 
            String sCurrentLine;
 
            br = new BufferedReader(new FileReader("C:\\testing.txt"));
 
            while ((sCurrentLine = br.readLine()) != null) {
                System.out.println(sCurrentLine);
            }
 
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                if (br != null)br.close();
            } catch (IOException ex) {
                ex.printStackTrace();
            }
        }
 
    }
}
</pre>

After that you can preview your blog at http://localhost:4000

# Deploying

I'm going to be using Github pages to deploy my blog so I followed the instructions and executed the ```rake setup_github_pages``` command.

    rake setup_github_pages
    Enter the read/write url for your repository
    (For example, 'git@github.com:your_username/your_username.github.io.git)
               or 'https://github.com/your_username/your_username.github.io')
    Repository url: git@github.com:ddewaele/ddewaele.github.com.git
    rm -rf _deploy
    mkdir _deploy
    cd _deploy
    Initialized empty Git repository in /Users/ddewaele/octopress/octopress/_deploy/.git/
    [master (root-commit) 3c52f91] Octopress init
     1 files changed, 1 insertions(+), 0 deletions(-)
     create mode 100644 index.html
    cd -
    
    ---
    ## Now you can deploy to git@github.com:ddewaele/ddewaele.github.com.git with `rake deploy` ##

Note that I'm still using a ```github.com``` suffix for my pages. Github has recently switched to ```gitub.io```

I had also  already setup a "default" user page using github pages that contained some static html that was generated by th Github pages wizard.

So when I ran ```rake deploy``` it seems to run fine at first but i got the following error:

    rake deploy
    ## Deploying branch to Github Pages 
    ## Pulling any updates from Github Pages 
    cd _deploy
    warning: no common commits
    remote: Reusing existing pack: 17, done.
    remote: Total 17 (delta 0), reused 0 (delta 0)
    Unpacking objects: 100% (17/17), done.
    From github.com:ddewaele/ddewaele.github.com
     * [new branch]      master     -> origin/master
    You asked me to pull without telling me which branch you
    want to merge with, and 'branch.master.merge' in
    your configuration file does not tell me, either. Please
    specify which branch you want to use on the command line and
    try again (e.g. 'git pull <repository> <refspec>').
    See git-pull(1) for details.
    
    If you often merge with the same branch, you may want to
    use something like the following in your configuration file:
    
        [branch "master"]
        remote = <nickname>
        merge = <remote-ref>
    
        [remote "<nickname>"]
        url = <url>
        fetch = <refspec>
    
    See git-config(1) for details.
    cd -
    rm -rf _deploy/index.html
    
    ## Copying public to _deploy
    cp -r public/. _deploy
    cd _deploy
    
    ## Committing: Site updated at 2014-05-10 09:54:27 UTC
    [master de754e5] Site updated at 2014-05-10 09:54:27 UTC
     61 files changed, 1118 insertions(+), 1 deletions(-)
     create mode 100644 assets/jwplayer/glow/controlbar/background.png
     ...
     create mode 100644 images/search.png
     rewrite index.html (100%)
     create mode 100644 javascripts/github.js
     ...
     create mode 100644 stylesheets/screen.css
    
    ## Pushing generated _deploy website
    To git@github.com:ddewaele/ddewaele.github.com.git
     ! [rejected]        master -> master (non-fast-forward)
    error: failed to push some refs to 'git@github.com:ddewaele/ddewaele.github.com.git'
    To prevent you from losing history, non-fast-forward updates were rejected
    Merge the remote changes (e.g. 'git pull') before pushing again.  See the
    'Note about fast-forwards' section of 'git push --help' for details.
    
    ## Github Pages deploy complete
    cd -

The reason for this erro is because there was a conflict between waht was already on my github pages repository and what was generated by octopress

In order to solve this you need to ensure that your ```_deploy``` folder is in a state so that changes can be pushed.

What octopress was trying to do was this:

    cd _deploy/
    git push
    
    To git@github.com:ddewaele/ddewaele.github.com.git
     ! [rejected]        master -> master (non-fast-forward)
    error: failed to push some refs to 'git@github.com:ddewaele/ddewaele.github.com.git'
    To prevent you from losing history, non-fast-forward updates were rejected
    Merge the remote changes (e.g. 'git pull') before pushing again.  See the
    'Note about fast-forwards' section of 'git push --help' for details.

The reason why this push was failing is because we had ```non-fast-forward``` updates.

This becomes clear when you try to pull in the changes from the remote repository

    cd _deploy/
    git pull origin master
    
    warning: no common commits
    remote: Reusing existing pack: 17, done.
    remote: Total 17 (delta 0), reused 0 (delta 0)
    Unpacking objects: 100% (17/17), done.
    From github.com:ddewaele/ddewaele.github.com
     * branch            master     -> FETCH_HEAD
    Auto-merging index.html
    CONFLICT (add/add): Merge conflict in index.html
    Automatic merge failed; fix conflicts and then commit the result.

As you can see, a ```conflict``` was introduced on ```index.html```. This is why octopress wasn`t capable of deploying the site.


So we need to resolve these conflicts.


## Your blog source

You'll notice when doing a ```git status``` that octopress has created a git repository in your ```octopress`` folder that is currently checked out on the source branch

git status

    # On branch source
    # Changes not staged for commit:
    #   (use "git add <file>..." to update what will be committed)
    #   (use "git checkout -- <file>..." to discard changes in working directory)
    #
    #   modified:   Rakefile
    #   modified:   _config.yml
    #
    # Untracked files:
    #   (use "git add <file>..." to include in what will be committed)
    #
    #   sass/
    #   source/
    no changes added to commit (use "git add" and/or "git commit -a")

It has setup a gitignore file to ensure that your deploy folder doesn't get deployed to github pages.
 
    cat .gitignore 

    .bundle
    .DS_Store
    .sass-cache
    .gist-cache
    .pygments-cache
    _deploy
    public
    sass.old
    source.old
    source/_stash
    source/stylesheets/screen.css
    vendor
    node_modules


And it has the following git config.

    cat .git/config 
    [core]
        repositoryformatversion = 0
        filemode = true
        bare = false
        logallrefupdates = true
        ignorecase = true
    [remote "octopress"]
        url = git://github.com/imathis/octopress.git
        fetch = +refs/heads/*:refs/remotes/octopress/*
    [branch "source"]
        remote = origin
        merge = refs/heads/master
    [remote "origin"]
        url = git@github.com:ddewaele/ddewaele.github.com.git
        fetch = +refs/heads/*:refs/remotes/origin/*

In order to commit the source code to your blog

    git add .
    git commit -m 'your message'
    git push origin source


# Errors occured

    ---
    ## Now you can deploy to git@github.com:ddewaele/ddewaele.github.com.git with `rake deploy` ##
    Davys-MacBook-Air:octopress ddewaele$ rake deploy
    ## Deploying branch to Github Pages 
    ## Pulling any updates from Github Pages 
    cd _deploy
    warning: no common commits
    remote: Reusing existing pack: 17, done.
    remote: Total 17 (delta 0), reused 0 (delta 0)
    Unpacking objects: 100% (17/17), done.
    From github.com:ddewaele/ddewaele.github.com
     * [new branch]      master     -> origin/master
    You asked me to pull without telling me which branch you
    want to merge with, and 'branch.master.merge' in
    your configuration file does not tell me, either. Please
    specify which branch you want to use on the command line and
    try again (e.g. 'git pull <repository> <refspec>').
    See git-pull(1) for details.

    If you often merge with the same branch, you may want to
    use something like the following in your configuration file:

        [branch "master"]
        remote = <nickname>
        merge = <remote-ref>

        [remote "<nickname>"]
        url = <url>
        fetch = <refspec>

    See git-config(1) for details.
    cd -
    rm -rf _deploy/index.html

    ## Copying public to _deploy
    cp -r public/. _deploy
    cd _deploy

    ## Committing: Site updated at 2014-05-10 09:38:40 UTC
    [master 4eb09be] Site updated at 2014-05-10 09:38:40 UTC
     1 files changed, 0 insertions(+), 1 deletions(-)
     delete mode 100644 index.html

    ## Pushing generated _deploy website
    To git@github.com:ddewaele/ddewaele.github.com.git
     ! [rejected]        master -> master (non-fast-forward)
    error: failed to push some refs to 'git@github.com:ddewaele/ddewaele.github.com.git'
    To prevent you from losing history, non-fast-forward updates were rejected
    Merge the remote changes (e.g. 'git pull') before pushing again.  See the
    'Note about fast-forwards' section of 'git push --help' for details.

## Pages

### Creating new pages

In order to create a new page we execute the ```rake new_page``` command

``` text
rake new_page["about"]

mkdir -p source/about
Creating new page: source/about/index.markdown
```


source/_includes/custom/navigation.html

## Themes

If you want to install a theme you can use the ```rake install [theme]``` command.

For example, to re-install the ```classic``` theme you can execute ```rake install classic```.

``` text
    A theme is already installed, proceeding will overwrite existing files. Are you sure? [y/n] y
    ## Copying classic theme into ./source and ./sass
    mkdir -p source
    cp -r .themes/classic/source/. source
    mkdir -p sass
    cp -r .themes/classic/sass/. sass
    mkdir -p source/_posts
    mkdir -p public
    rake aborted!
    Don't know how to build task 'classic'

    (See full trace by running task with --trace)
```

rake install['readify']

## Updating a theme

``` text
Davys-MacBook-Air:readify ddewaele$ git pull origin master
From https://github.com/vladigleba/readify
 * branch            master     -> FETCH_HEAD
Updating dd00471..e1792cc
Fast-forward
 sass/custom/_styles.scss          |   20 +++++++++++++-------
 source/_includes/custom/head.html |    4 ++++
 source/favicon.png                |  Bin 400 -> 0 bytes
 3 files changed, 17 insertions(+), 7 deletions(-)
 create mode 100644 source/_includes/custom/head.html
 delete mode 100644 source/favicon.png
```

# References

- http://octopress.org/docs/setup/
- http://octopress.org/docs/deploying/github/
- http://octopress.org/docs/configuring/

- https://github.com/imathis/octopress/wiki/3rd-Party-Octopress-Themes

http://devspade.com/blog/2013/08/06/fixing-gist-embeds-in-octopress/
https://github.com/vladigleba/readify