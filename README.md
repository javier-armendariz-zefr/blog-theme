![engineering-blog](/readme/github-banner.jpg)

## βοΈ Environment

Before starting, make sure you have [Ruby](https://www.ruby-lang.org/en/documentation/installation/) and [NodeJS](https://nodejs.org/) installed.

Then install Jekyll:

```
$ gem install jekyll
```

And install Gulp client:

```
$ npm install gulp-cli -g
```

## π€ Installing the Blog

1. Clone the repo [Blog](https://github.com/javier-armendariz-zefr/blog-theme):

2. Access to the blog directory:

```
$ cd path/to/blog-theme
```

3. Install npm packages:

```
$ npm install
```

4. Install Ruby dependencies:

```
$ bundle install
```

5. Build Jekyll:

```
$ bundle exec jekyll build
```

6. Then run the blog:

```
$ gulp
```

or

```
$ bundle exec jekyll serve
```

7. Open the blog [http://127.0.0.1:4000/blog-theme/](http://127.0.0.1:4000/blog-theme/)

## π Drafts

Drafts are posts without a date in the filename. Theyβre posts youβre still working on and donβt want to publish yet. To get up and running with drafts, create a \__drafts_ folder in your siteβs root and create your first draft:

```
.
βββ _drafts
β   βββ a-draft-post.md
...
```

To preview the blog with drafts, run:

```
$ bundle exec jekyll serve --drafts
```

And open the blog at [http://127.0.0.1:4000/blog-theme/](http://127.0.0.1:4000/blog-theme/)

## πΌ Theme

### Hero

Anytime we add a new post or draft, we need to change the date in the **Hero Post** (\_posts/2021-04-14-hero.md) [Front Matter](https://jekyllrb.com/docs/front-matter/) block to today's date, so we can guarantee the Hero video animation post will persist as the first post in the blog:

![Hero post](/readme/hero-post.png)

## π Assets

### Post images

The post images should be under its own post folder (/assets/img/posts/{post-name}), and both the pictures and folder should be Kebab cased.

Example:

/assets/img/posts/a-post-directory-name-here/an-image-goes-here.png

```
assets
βββ img
|  βββ posts
|  |  βββ a-post-directory-name-here
|  |  |  βββ an-image-goes-here.png
|  |  |  βββ another-image-also-could-goes-here.jpg
...
```

## βοΈ Authors

To add a new author follow [this](https://jekyllrb.com/docs/step-by-step/09-collections/#add-authors).

### Author picture

You can place your picture right here:
(/assets/img/authors/{authorname.imageextension})

Example:
/assets/img/authors/zexuanzhou.jpeg

```
assets
βββ img
|  βββ authors
|  |  βββ javierarmendariz.jpg
|  |  βββ kellygajewski.png
|  |  βββ wesleytanner.jpeg
|  |  βββ zexuanzhou.jpeg
...
```

## π¦ ToDo

- Migrate this blog to the [engineering-blog](https://github.com/ZEFR-INC/engineering-blog) repo
- Add more authors
- To configure Jenkins or some script to update the Hero post date automatically after merging a new post (\_posts/2021-04-14-a-new-post-goes-here.md)
