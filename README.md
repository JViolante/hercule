# Hercule – Markdown Transclusion Tool

[![Join the chat at https://gitter.im/jamesramsay/hercule](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/jamesramsay/hercule?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

[![Build Status](https://travis-ci.org/jamesramsay/hercule.svg)](https://travis-ci.org/jamesramsay/hercule)
[![Coverage Status](https://coveralls.io/repos/jamesramsay/hercule/badge.svg)](https://coveralls.io/r/jamesramsay/hercule)
[![Dependency Status](https://david-dm.org/jamesramsay/hercule.svg)](https://david-dm.org/jamesramsay/hercule)

[![NPM](https://nodei.co/npm/hercule.png)](https://nodei.co/npm/hercule/)

Write large markdown documents, including API Blueprints, while keeping things DRY (don't repeat yourself).

Hercule is a command-line tool for transcluding markdown documents, including API documentation written in [API Blueprint](http://apiblueprint.org) format. With Hercule you can easily break complex documents into smaller logical documents, preventing repetition and improving consistency.

-----

[![Adslot](http://l.jwr.vc/1asBO+)](http://adslot.com/)

Hercule is used by [Adslot](http://adslot.com) to help document our APIs in [API Blueprint](http://apiblueprint.org) more efficiently and more accurately. We also use [Apiary](http://apiary.io) to distribute our API documentation and [DREDD](https://github.com/apiaryio/dredd) (a tool by [Apiary](http://apiary.io)) to validate the documentation against implementation.

## Installation

[Node.js](http://nodejs.org) and [NPM](http://npmjs.org) is required.

```
$ npm install -g hercule
```

## Example: API Blueprint

Borrowing from the Apiary [Gist Fox Tutorial example](http://apiary.io/blueprint), API Blueprint documents can be split into logical files and share common elements.

This example uses a different file for `gists` and `gists/{id}`, and shares a common file `gist.json`.

```
$ hercule gist-fox.apib -o apiary.apib
```

Hercule accepts a single source file, in this example `gist-fox.apib`.

```markdown
FORMAT: 1A

# Gist Fox API
Gist Fox API is a **pastes service** similar to [GitHub's Gist](http://gist.github.com).

# Group Gist

:[Gist](blueprint/gist.md)

:[Gists](blueprint/gists.md)
```

Each resource is being stored in its own file, `gist.md` and `gists.md` respectively.

```markdown
## Gist [/gists/{id}]
A single Gist object.
The Gist resource is the central resource in the Gist Fox API.
It represents one paste - a single text note.

+ Parameters
  + id (string) ... ID of the Gist in the form of a hash.

+ Model

    + Body

      ```
      :[](gist.json)
      ```

### Retrieve a Single Gist [GET]

+ Response 200

  [Gist][]
```

The common JSON body is stored in `gist.json`.

```json
{
  "id": "42",
  "created_at": "2014-04-14T02:15:15Z",
  "description": "Description of Gist",
  "content": "String contents"
}
```

See `examples/api-bluprint` for a full example.
Use `hercule gist-fox.apib | snowcrash` to validate.

## Example: Technical documents

Writing long documents, particularly technical documents, may require repetition of certain information.
Hercule helps reduce repetition in line with DRY–don't repeat yourself–principles.

```markdown
# Transclusions in Markdown

John Appleseed
(University of Technology)

## Abstract
:[Abstract](src/abstract.md)

...

```

Transcluding a document:

```bash
hercule src/transclusions-in-markdown.md -o final.md
```

Omitting the `output` argument allows the output to be piped into other text processing tools:

```bash
hercule src/transclusions-in-markdown.md | pandoc -o final.pdf
```

## Syntax

**Example:** `report-final.md` is generated by transcluding `apple.md` into `report.md`.

```bash
hercule src/report.md -o report-final.md
```

```markdown
:[Apple](apple.md)
```

**Example:** `src/apple.md` and `src/common/footer.md` are transcluded by parent reference into `fruit.md`,
which is transcluded into `src/report.md`.

```bash
hercule src/report.md -o report-final.md
```

```markdown
:[Fruit](fruit.md fruit:apple.md footer:common/footer.md)
```

### Transclusion

```markdown
An example document

:[](LINK)
```

A document may contain any number of transclusion links.
Links within linked documents, and their children are also transcluded.

Links are relative to the document the link is declared, in the typical `node.js` style.

### Transclusion by parent reference

```markdown
An example document

:[](LINK REFERENCE:PATH/TO/FILE.MD)
```

This allows a link to be provided from a parent file.

Strings are also supported and are denoted by double quotes.

```md
+ Response 200

    :[Gists](gist.json root:"api.example.com")
```

```json
{
  "links": {
    "self": "http://:[API Root](root)/gists"
  }
}
```

becomes

```md
+ Response 200

    {
      "links": {
        "self": "http://api.example.com/gists"
      }
    }
```

### Whitespace sensitivity

Leading whitespace is significant in Markdown.
Hercule preserves whitespace at the beginning of each line.

```markdown
Binary sort example:

  :[](snippet.c)

```

Each line of `snippet.c` will be indented with the whitespace preceding it.

This is also useful for nesting lists or including paragraphs within a list.

```markdown
- Apple
  :[Apple variety information](apple-varieties.md)
- Orange
  :[Orange variety information](orange-varieties.md)
- Pear
  :[Pear variety information](pear-varieties.md)
```
