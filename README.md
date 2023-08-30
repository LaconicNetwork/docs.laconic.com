# Laconic Documentation

## Local editing

The site is built with [Hugo](https://github.com/gohugoio/hugo/). To edit locally, first [install Hugo](https://gohugo.io/getting-started/installing/), and then start Hugo in server mode in the root of your checkout of this repository.

This site is currently built in production with the version set [in the CI file](.github/workflows/hugo.yml)

From the root of this repo, run:
```
hugo server
```

```
Started building sites ...
.
.
Serving pages from memory
Web Server is available at http://localhost:1313/ (bind address 127.0.0.1)
Press Ctrl+C to stop
```

Try:

```
hugo server -D
```

to see draft content.

## Adding content

Make a markdown file within the `content/section` directory. It's easiest to copy another file, and edit the `title:` and `weight:`.

## Create a new section

Make a directory within `content` that has an `_index.en.md`.

**Note:** for this additional section to show up in the sidebar, it requires another page in markdown within the new directory.
