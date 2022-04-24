# Blog

This repo hosts the source of https://mirzak.github.io/.

## Setup environment to run site locally

Install RVM: https://wiki.archlinux.org/title/RVM#Installing_RVM (btw I use Arch)

The current stable versions of Ruby is >3.x, but
[Github Pages uses Jekyll 3.9.0](https://pages.github.com/versions/) which
is only compatible with Ruby 2.7.x, hence the dance with RVM.

Install Ruby 2.7 and set as default:

```sh
rvm install 2.7 && rvm use 2.7 --default
```

In `mirzak.github.io` directory, run:

```sh
bundle install
```

Serve site:

```sh
bundle exec jekyll serve --livereload
```

Site is served at: http://127.0.0.1:4000/

## License

Shield: [![CC BY-SA 4.0][cc-by-sa-shield]][cc-by-sa]

This work is licensed under a
[Creative Commons Attribution-ShareAlike 4.0 International License][cc-by-sa].

[![CC BY-SA 4.0][cc-by-sa-image]][cc-by-sa]

[cc-by-sa]: http://creativecommons.org/licenses/by-sa/4.0/
[cc-by-sa-image]: https://licensebuttons.net/l/by-sa/4.0/88x31.png
[cc-by-sa-shield]: https://img.shields.io/badge/License-CC%20BY--SA%204.0-lightgrey.svg
