# Eduardo Colabardini's Jekyll Theme

This is my personal Jekyll theme. Feel free to use it at your own website or blog!

Check it at: https://ecolabardini.github.io/jekyll-theme

# First steps

Clone this repository
```bash
git@github.com:ecolabardini/jekyll-theme.git && cd jekyll-theme
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

### Building tags

GitHub pages doesn't allow the execution of unsafe Jekyll plugins, thus [jekyll-tagging](https://github.com/pattex/jekyll-tagging) is not allowed to be used.

This way you need to generate offline tags: `ruby _gentags.rb`

### _config.yml configurations

`pagination`: the number of posts per page

`twitter/github/linkedin`: your username

# More info

Host your website free with GitHub Pages: https://pages.github.com/

What is Jekyll? https://jekyllrb.com/docs/home/

Plugins on GitHub pages: https://jekyllrb.com/docs/plugins/

# Contributing

Please contact me if you have any ideas, suggestions or, even better, you want to collaborate on this project!

# License

This library is licensed under the [Apache License, Version 2.0](http://www.apache.org/licenses/LICENSE-2.0).
