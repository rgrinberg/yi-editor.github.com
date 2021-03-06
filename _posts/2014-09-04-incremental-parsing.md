---
layout: post
title: Incremental parsing
---

In this post I will motivate Yi's incremental parsing library and describe the
main ideas behind it.

## Why another parsing library?

Why bothering developing another parsing framework while there exist plenty
already?

First, since we want to parse many languages, in many flavors, we want to be
able to reuse pieces of grammars. Since we are using Haskell, the easiest way to
achieve this is to through a parser-combinator library.

Second, we want to give timely feedback to users. Therefore, the parser has to
be efficient. In particular, responsivity should not depend on the length of
file. On the other hand, the input file will change incrementally (as the user
edits it), and the parser should take advantage of this. It should reuse
previous results to speed up the parse.

Third, the parser must gracefully handle errors in the input: while files are
being edited, they will inevitably contain syntactically incorrect programs at
several moments.

No parsing framework combining these characteristics exists.

## Approach

Hughes and Swierstra have shown how [online parsers][1] can be constructed.
An _online_ parser takes advantage of lazy evaluation: the input is analyzed
only as far as needed to produce the part of the result that is forced.

Effectively, this solves part of the incremental parsing problem: since the
user can see only one page of text at a time, the editor will only force so much
of the result tree, and only the corresponding part of the
input will be analyzed, making the parser response time independent of the
length of the input.

This does not completely solve the problem though. If the user edits the end of
the file, the whole input will have to be analyzed at each key-press.

## Caching results

The proposed solution is to cache intermediate parsing results.
For given positions in the input (say every half-page), we will store a partially
evaluated state of the parsing automaton. Whenever the input is modified,
the new parsing result will be computed by using the most relevant cached state,
and applying the new input to it. The cached states that became invalidated will
also be recomputed on the basis of the most relevant state.

Of course, the cached states will only be computed lazily, so that no cost is
paid for cached states that will be discarded.

## Conclusion

The strategy sketched above has several advantages:

* The design is relatively straightforward, and adds only a hundred
lines of code compared to the polish parsers of Hughes and Swierstra.

* There is no start-up cost. A non-online approach would need to parse
the whole file the first time it is loaded. In our approach loading is
instantaneous, and parsing proceeds as the user scrolls down the file.

* The caching strategy is independent of the underlying parsing
  automaton. We only require it to accept partial inputs.

Its notable that the design has a strong functional flavor:

* We never update a parse tree (no references, no zipper) and still achieve
incremental parsing.

* We take advantage of lazy evaluation in a cool way.

The main drawback is that the user code must use the parse
tree lazily, and there is no way to enforce this in any current implementation
of Haskell.

[1]: http://www.cs.uu.nl/groups/ST/Software/UU_Parsing/p224-swierstra.pdf
