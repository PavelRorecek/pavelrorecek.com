---
layout: post
title: "Integrating Jekyll into GitHub Pages"
---
This article will show you how to integrate [Jekyll](https://jekyllrb.com/)
into your GitHub Pages.

Tutorial is based on
[GitHub docs](https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/creating-a-github-pages-site-with-jekyll)
so reference that if you find something unclear. 

### Prerequisites:
- Github Pages properly setup up.
- [Ruby](https://www.ruby-lang.org/en/documentation/installation/) installed.
- [Bundler](https://bundler.io/) installed.

### Integration steps:
1. Open terminal and `cd` into folder with GitHub pages repository.
2. Run `jekyll new --skip-bundle --force .` to setup `Jekyll`. `--force` is necessary because the folder most
likely contains some files that will be overwritten by the command.
3. Open the `Gemfile`.
   1. Delete line containing `gem "jekyll"`.
   2. Add `gem "github-pages", "~> <version>", group: :jekyll_plugins`. Replace
`<version>` with the latest version of `github-pages` gem found
[here](https://pages.github.com/versions/).
   3. Optional: Delete all lines starting with `#` to keep the file tidy.
   4. Save and close.
4. Run `bundle install` to install dependencies.
5. Rename `index.markdown` to `blog.markdown`.
6. Run `bundle exec jekyll serve` to test that Jekyll works locally. Append `--livereload`
to the command if you want to automatic rebuild and browser refresh.
7. In your browser navigate to the address server is running on. Most likely
[http://127.0.0.1:4000/blog](http://127.0.0.1:4000/blog).
