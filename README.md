[![License][license badge]][license]

# Neon
A system for working with language syntax.

Neon aims to provide facilities for highlighting, indenting, and querying the structure of language text in a performant way. It is based on [tree-sitter](https://tree-sitter.github.io/tree-sitter/), via [SwiftTreeSitter](https://github.com/ChimeHQ/SwiftTreeSitter).

The library is being extracted from the [Chime](https://www.chimehq.com) editor. It's a pretty complex system, and pulling it out is something we intend to do over time.

## TreeSitterClient

This class is an asynchronous interface to tree-sitter. It provides an UTF16 code-point (NSString-compatible) API for edits, invalidations, and queries. It can process edits of String objects, or raw bytes for even greater flexibility and performance. Invalidations are translated to the current content state, even if a queue of edits are still being processed.

TreeSitterClient requires a function that can translate UTF16 code points (ie `NSRange`.location) to a tree-sitter `Point` (line + offset).

```swift
import SwiftTreeSitter
import Neon

// step 1: setup

// construct the tree-sitter grammar for the language you are interested
// in working with
let language = Language(language: my_tree_sitter_grammar())

// construct your highlighting query
// this is a one-time cost, but can be expensive
let url = URL(fileURLWithPath: "/path/to/language/highlights.scm")!
let query = try! language.query(contentsOf: url)

// step 2: configure the client

// produce a function that can map UTF16 code points to Point (Line, Offset) structs
let locationToPoint = { Int -> Point? in ... }

let client = TreeSitterClient(language: language, locationToPoint: locationToPoint)

// this function will be called with a minimal set of text ranges
// that have become invalidated due to edits. These ranges
// always coorespond to the *current* state of the text content,
// even if TreeSitterClient is currently processing edits in the
// background.
client.invalidationHandler = { set in ... }

// step 3: inform it about content changes
// these APIs match up fairly closely with NSTextStorageDelegate,
// and are compatible with lazy evaluation of the text content

// call this *before* the content has been changed
client.willChangeContent(in: range)

// and call this *after*
client.didChangeContent(to: string, in: range, delta: delta, limit: limit)

// step 4: run queries
// you can execute these queries in the invalidationHandler

// produce a function that can read your text content
let provider = { contentRange -> Result<String, Error> in ... }

client.executeHighlightQuery(query, in: range, contentProvider: provider) { result in
    // TreeSitterClient.HighlightMatch objects will tell you about the
    // highlights.scm name and range in your text
}
```

### Suggestions or Feedback

We'd love to hear from you! Get in touch via [twitter](https://twitter.com/chimehq), an issue, or a pull request.

Please note that this project is released with a [Contributor Code of Conduct](CODE_OF_CONDUCT.md). By participating in this project you agree to abide by its terms.

[license]: https://opensource.org/licenses/BSD-3-Clause
[license badge]: https://img.shields.io/github/license/ChimeHQ/Neon
