# book_bake

Write a book in Markdown, keep it in Git, and let pandoc build it.

Two outputs, from the same chapters:

* an **EPUB** people can read
* a **PDF in [Shunn manuscript format](https://www.shunn.net/format/story/)**,
  the layout agents and editors expect for submissions

Push a version tag and GitHub Actions builds both and attaches them to a
Release. There is also a small download page, served straight from your repo.

The same book can live here in more than one language — see
[Another language](#another-language).

## Start

Click **Use this template**, clone your new repo, then:

```bash
./setup.sh
```

> No **Use this template** button? Then this repo's *Template repository* box
> is unticked — Settings → General, for whoever owns it. Meanwhile you can just
> clone it and run `rm -rf .git && git init`, which gets you the same thing.

It asks for your title, your name, the language you write in and your GitHub
details, writes them into `book.yaml`, generates a unique EPUB identifier, and
removes itself.

Then write. Your chapters go in `manuscript/<language>/`, one file per chapter —
`manuscript/en/` for English:

```text
manuscript/en/front/     title page, dedication — EPUB only
manuscript/en/01_*.md    your chapters, in number order
manuscript/en/back/      acknowledgments, appendix
manuscript/en/draft/     never built
```

Sample files are included so the build works before you have written anything.
Delete them as you go.

## Build

```bash
epub/bake_book_epub.sh --version=v0.1.0
pdf/bake_book_pdf.sh   --version=v0.1.0
```

Run both from the repo root. They write to `versions/`, which is gitignored —
build output belongs on Releases, not in the repo. File names end in the
language code: `Your_Book_Title_v0.1.0_en.epub`.

Both take `--lang=nl` to build a language other than the default. The PDF also
takes `--a4`, `--classic`, `--title-page`, `--with-extras` and `--font=`. Run it
with `--help`, or see [`wiki/shunn_pdf_format.md`](wiki/shunn_pdf_format.md).

**You need:** pandoc, XeLaTeX (for the PDF), and Node 20 (the EPUB pulls your
release history from GitHub).

## Publish

Tag it. That is the whole process.

```bash
git tag -a v0.1.0 -m "First release"
git push origin v0.1.0
```

Both workflows build every language, and attach the EPUBs and PDFs to the
Release. To get a build without tagging, run either workflow from the Actions
tab and download the artifact.

To put the download page online, turn on GitHub Pages in Settings and point it
at the `main` branch, folder `/`. There is no site build step — `index.html` is
served as-is, so edit it and push.

## Everything is in `book.yaml`

Title, author, contact block, cover path, GitHub details, and two switches for
how the PDF filter treats ambiguous markdown. The build scripts read it every
time, so there is one place to change and nothing to keep in sync.

`index.html` is the exception: Pages serves it as a static file, so it cannot
read config. `setup.sh` fills it in once.

## Another language

Every folder in `manuscript/` is a language, and **the folder name is the
language code**: `en`, `nl`, `de`. To add Dutch next to English:

```bash
cp -r manuscript/en manuscript/nl
uuidgen | tr 'A-Z' 'a-z'
```

Then create `manuscript/nl/book.yaml`, with the id you just generated, and
translate the chapters:

```yaml
title:          "Jouw Boektitel"
shorttitle:     "Boektitel"
slug:           "Jouw_Boektitel"
description:    "Eén zin over je boek."
identifier:     "urn:uuid:paste-the-uuid-here"
wordcount-text: "ongeveer %s woorden"
edition-text:   "Huidige editie"
```

This file only holds what differs. Anything it leaves out — author, contact
block, cover, font — comes from the root `book.yaml`. Add `cover:` if the Dutch
edition has its own cover.

The `identifier` is not optional: a translation is a different EPUB, and the
build refuses to reuse the original's id, because e-readers would treat the
two as one book.

Build it with `--lang=nl`. On a tag, CI builds every language on its own.

## Getting template updates

A repo made with **Use this template** is a copy with no link back, so GitHub
never updates it for you. Two commands do.

Once per book, tell Git where the template lives:

```bash
cd path/to/your-book
git remote add template https://github.com/fluxpiada/book_bake.git
```

Each time the template changes:

```bash
git fetch template
git checkout template/main -- lib epub pdf .github wiki .gitignore
```

That replaces only the tooling. Your chapters, `book.yaml`, `index.html` and
this README are never touched. Build once to check, then commit.

## Writing conventions

| You write | You get |
| --- | --- |
| `# Chapter I - The Fjord` | a chapter opening, and a table-of-contents entry |
| `## ~ * ~` | a scene break, centred |
| `*emphasis*` | italic, or underlined with `--classic` |
| `<!-- a note -->` | nothing — pandoc drops comments from every format |

Chapter files sort by filename, so keep the numbers padded: `01_`, `02_`, …
`10_`. That order *is* the order of your book.

## Checking your prose

[readability-stats](https://github.com/fluxpiada/readability-stats) reads a
folder of Markdown chapters and reports readability, pacing and vocabulary,
with a report you can diff between drafts. Point it at one language folder:

```bash
./run.sh 8 /path/to/your-book/manuscript/en
```

It has an English and a Dutch implementation.

## Licence

The tooling here is GPL-3 — see [LICENSE](LICENSE). Use it for anything.

**What you write is yours.** This licence makes no claim on your manuscript,
your cover, or the files you build from them.
