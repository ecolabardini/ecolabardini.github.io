# Eduardo Colabardini's blog

Hi, this is the repository for my personal blog hosted with GitHub pages.

Check it at: https://ecolabardini.github.io

I developed my personal theme based on [jekyll/minima](https://github.com/jekyll/minima). Feel free to use it at your own website or blog!

# First steps

Clone this repository
```bash
git@github.com:ecolabardini/ecolabardini.github.io.git && cd ecolabardini.github.io
```

Install Jekyll and Bundler gems through RubyGems
```bash
gem install jekyll bundler
```

Build the site and preview it in an incremental way
```bash
bundle exec jekyll serve --incremental
```

# Features

### Tags

GitHub pages doesn't allow the execution of unsafe Jekyll plugins, thus [jekyll-tagging](https://github.com/pattex/jekyll-tagging) or other tag plugins are not allowed to be used.

This theme includes a [tag cloud](https://ecolabardini.github.io/tagcloud/) and [tag links](https://ecolabardini.github.io/2015/11/24/my-docker-whale-of-fortune/) in posts using page jumps in a static page instead of pre-generated pages for each tag.

### _config.yml configurations

`pagination`: the number of posts per page

`twitter/github/linkedin`: your username

`google_analytics`: your tracking id

# More info

Host your website free with GitHub Pages: https://pages.github.com/

What is Jekyll? https://jekyllrb.com/docs/home/

GitHub supported plugins: https://pages.github.com/versions/

About plugins on GitHub pages: https://jekyllrb.com/docs/plugins/

# Contributing

Please contact me if you have any ideas, suggestions or, even better, you want to collaborate on this project!

# License

This library is licensed under the [Apache License, Version 2.0](http://www.apache.org/licenses/LICENSE-2.0).
