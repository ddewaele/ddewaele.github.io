#Introduction

This is a clean blog template that can be used to start a jekyll based blog on github pages.

## Theme used

The Jekyll theme used on this blog is bsed on the [hpstr-jekyll-theme](https://github.com/mmistakes/hpstr-jekyll-theme/).

### Customizations

I've customized the theme a little bit :

- in `_sass/page.scss` I've increased the max-width from 800px to 1200px
- updated the license
- updated the config (change title, discuss, url , name, )
- removed adds from the footer
- updated the `about/index.md` page
- updated the `index.html` page to show `post.excerpt` instead of `post.content`
- Using Read More links in the post (for post excerpts)

If you want to clone this blog and run it locally (for whatever reason), simply run

```
bundle exec jekyll s  -c _config.yml,_localhost_config.yml --drafts
```

This blog is using [jekyll-compose](https://github.com/jekyll/jekyll-compose) (defined in the Gemfile), so to create pages/posts/drafts you can do

```
$ bundle exec jekyll page "My New Page"
$ bundle exec jekyll post "My New Post"
$ bundle exec jekyll draft "My new draft"
```