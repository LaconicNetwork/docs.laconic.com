# Laconic Documentation

**NOTICE:** This repository is out of date and not maintained. To use the Laconic Stack, see [Stack Orchestrator](https://github.com/cerc-io/stack-orchestrator)

## Local editing

The site is built with [Hugo](https://github.com/gohugoio/hugo/). To edit locally, first [install Hugo](https://gohugo.io/getting-started/installing/), and then start Hugo in server mode in the root of your checkout of this repository.

This site is currently built with `hugo_extended_0.92.0`.

```
▶ hugo server -D

Started building sites ...
.
.
Serving pages from memory
Web Server is available at http://localhost:1313/ (bind address 127.0.0.1)
Press Ctrl+C to stop
```
The `-D` flag allows you to see draft content.

The deployment script is:

`hugo --minify`

If you notice differences in the styling between your local copy and the preview built on Fleek, try adding `--minify` to your local server command.

## Adding content

In general, to create a new page, open a terminal, cd to the repository's root directory, and type

```
hugo new content/<sectionName>/<newFile>.md
```

- Replace `<sectionName>` by the name of the directory right below `content/` that you want to add a page to.
- Replace `<newFile>` by the name of the file to create. In general, there are no rules for that name but it should relate to the content.

Example:

```
hugo new content/faq/watcher.md
```

would create a new file `watcher.md` in the FAQ section. The initial content is derived from a template file in `archetypes/` with the same name as the section (e.g., `faq.md` is the template for docuemnts created in the `faq` section.)

Then edit the new file as needed. If you have a local Hugo server running, it reflects all changes almost instantly.

### Adding FAQ entries

New FAQ entries contain this front matter section:

```
---
title:
weight: 999
category: laconic-network
---
```

- Add the question to the `title` field, without surrounding quotes.
- The `weight` entry specifies the sequence of FAQ entries. In other words, FAQ entries are sorted by weight in ascending order. Typically you would set this to the count of existing FAQ files (including the new one). If you want to move that item to a higher position, you would need to change the weights of all entries that shall appear after the new one. 
- `category` groups entries into given categories. The value for `category` must be one of:
  - laconic-network
  - laconic-token
  - validators-members
  - dapps-watchers
- Write the answer below the front matter. You can use Markdown formatting here.

Example:
```
---
title: How do I build a Watcher?
weight: 8
category: dapps-watchers 
---
It depends.
```

The categories can be edited in `data/categories/faq`. Each category is a YAML file with this structure:
```yaml
id: dapps-watchers
name: DApps & Watchers
weight: 4
```

On the FAQ page, the categories are sorted by their `weight` value.

### Adding glossary entries

Glossary entries come with this initial front matter:

```
---
title: A title
description: A new entry to the glossary
tags:
  - fundamental
---
```

- The `title` field contains the glossary term
- The `description` field contains a *short*  description
- The `tags` field is a list of tags (currently not evaluated but might be used for filtering the glossary when it gets too long for one page)
- Below the front matter, write a detailed explanation of the term. You can use Markdown formatting here.

Example:

```
---
title: diphthong
description: A phonetics term referring to an unsegmentable sound
tags:
  - fundamental
---

A phonetics term referring to an unsegmentable sound, like the OI in oil. It’s been described as the "[the best word ever](https://www.theatlantic.com/technology/archive/2012/09/here-it-is-the-best-word-ever/262348/)", and while that may or may not be true, the combination of dip, which can refer to a clueless individual, and thong, which is a style of underwear, creates auditory splendor. (Source: [dictionary.com](https://www.dictionary.com/e/s/funny-words/#diphthong))

```

### Cross-references 

To add cross-references to FAQ or glossary entries, use the following Hugo shortcodes:

```
{{< glossary "<file-base-name>" "<optional-link-text>" >}}
{{< faq "<file-base-name>" "<optional-link-text>" >}}
```

The link text can be omitted if it happens to be identical to `file-base-name`.

Examples:

```
A Validator is a {{< glossary "laconic-network" "Laconic Network" >}} member.
{{< faq "watcher" "Watchers" >}} query, cache, and transform blockchain data for DApps.
10 ways of querying a {{< glossary "blockchain" >}}.
```

