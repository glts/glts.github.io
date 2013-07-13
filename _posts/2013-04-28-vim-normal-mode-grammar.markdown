---
layout: post
title: "\"a2d&lt;C-V&gt;3gE: Vim normal mode grammar"
tags: [vim, grammar, language]
---
# `"a2d<C-V>3gE`: Vim normal mode grammar

> *This is a purely academic exercise in doing a proper linguistic
> description of Vim's normal mode grammar. Read on if you're feeling
> adventurous!*

Much has been said of "nouns" and "verbs" in Vim's normal mode command
language. Unfortunately, a proper linguistic account has not been
forthcoming. In this article I will attempt to give such an account.


## Grammar outline

This grammar outline focuses on the syntax of Vim grammar. We will
examine all the constituents in the maximal grammatical sentence given
in the title, `"a2d<C-V>3gE`. We will walk through the syntax first,
covering all the constituents comprehensively, then talk about word
order, word classes, and some morphological phenomena in separate
sections.

### Syntax overview

#### Intransitive sentence

The minimal sentence in Vim consists of just a verb.

> `p` "put"  
> `$` "move to the end of the line"

These are Vim's intransitive sentences. Only a few **intransitive
verbs** like `J` `p` `~` are capable of this usage. Also capable of
functioning as verbs of intransitive sentences are all the **motion
verbs** (*motions*) `w` `^` `gE` etc. Motion verbs have a number of
properties which set them apart from the other intransitive verbs. We
will see more of them in the next and following sections.

Since all of Vim's normal mode sentences are commands, we will take them
as imperatives: "put!", "move!". A subject is never expressed.

#### Transitive sentence

Some verbs take an object argument to form basic transitive sentences.

> `yaw` "yank an entire word"

**Transitive verbs** (*operators*) include `y` `d` `gu`. An **object**
can be one of two kinds: a common noun (*text object*) as in `aw` "an
entire word" and `i(` "the material inside the brackets"; and a motion
verb in argument function.

Here is an example for a motion verb in argument function:

> `dw` "delete to the beginning of the next word"

Used as an argument, a motion verb comes to denote the part that would
be covered when executing the motion. So, `w` as a verb means "move the
cursor to the beginning of the next word", but `w` as an object means
"the part covered by moving from the current cursor position to the
beginning of the next word".

A special property of a motion verb, its [motion
type](http://vimdoc.sourceforge.net/htmldoc/motion.html#linewise),
becomes apparent when the motion verb is in argument function: some
motion verbs yield a linewise object whereas others yield a
characterwise one.

#### Quantifiers

Both the verb and the object can be quantified by preceding them with a
**quantifier** (*count*):

> `5gUe` "five times upcase the text covered by moving the cursor to the
> end of the next word"  
> `c2k` "change the lines covered by moving the cursor up two lines".

Both quantifiers can be given at the same time, as in our title
sentence.

> `2d3gE` "twice delete to the end of three whitespace-separated words
> backwards"

The verbal and the object quantifier form an intimate relationship. They
are multiplied and their product quantifies the whole predication. In
fact, Vim does not remember the quantifiers individually, only the
product. When the `.` command is used with a count the entire predicate
quantification is replaced with the new count. It is therefore not
implausible to argue that the verb and object quantifiers represent one
discontinuous syntactical constituent.

We will call the constituents covered so far -- verb, object, and
optional quantifiers -- the "core predication". For our title sentence
the core predication is `2d3gE`.

#### "Goal"

Any core predication can be preceded by a **Goal**[^1] (*register*). The
register specifies where the result of the predication should go or
where it should draw from.

> `"adE` "into register `a` delete what's covered by moving to the
> end of the next whitespace-separated word"  
> `"xp` "put what's in register `x`"

When no register is given, the default register `"` is understood.

It should be mentioned that the default register isn't the only thing
that is understood. Place and time -- the current cursor position and
"now" -- are also part of the default context, as is a count of 1. Thus,
the simplest sentence `p` actually means "put (it) (here) (once)",
i.e. `""1p`. "Now" is always implicit, all sentences are immediate.

#### Adverb

A less well known part of Vim's grammar is the **adverb** (called
"motion force" in the source; the Vim documentation [doesn't have a
specific term](http://vimdoc.sourceforge.net/htmldoc/motion.html#o_v)
for this). There are three adverbs `v` `V` `<C-V>`, which specify
characterwise, linewise, and blockwise operation. An adverb is only
acceptable in transitive sentences. When present, it changes the way in
which the verb applies to the object.

Consider these sentences:

> `g?}` "rot-13-encode (characterwise) to the end of the paragraph"  
> `g?V}` "rot-13-encode linewise to the end of the paragraph"

Both sentences make the same simple predication, but the second has the
linewise `V` adverb. The motion verb `}` has characterwise motion
type when used as an argument. Its meaning is "the part covered by
moving from the cursor position to the end of the paragraph".

With the adverb `V` the "part covered" is to apply linewise, so the
command `g?V}` will rot-13-encode all lines from whatever line the
cursor was on to the line at the end of the paragraph. The adverb has
essentially changed how the core predication must be understood.

An alternative analysis for `v` `V` `<C-V>` could seem possible. Since
their ultimate effect is to modify the extent of the text operated upon
we might view them as adjectives, modifiers of the object.

There is evidence against that analysis: the dependence of the adverbs
on the verb, the fact that they can combine with Ex command movement,
and their relative syntactic independence from the object (`d2vj` ~
`dv2j`) provide clues. The decisive evidence, however, comes from their
interplay with the quantifiers. `2yV3w`, executed on the buffer shown
below, yanks three lines, not four: the adverb has scope over the whole
core predication with both verbal and object quantifiers. Its scope is
not limited to just the quantified object.

{% highlight text %}
[a]b cd
ef gh
ij kl mn
op qr
{% endhighlight %}

This concludes our overview of the syntax. We return to our title
sentence `"a2d<C-V>3gE` in the next section.

### Word order

Typologically speaking, Vim's normal mode syntax is
[VO](http://en.wikipedia.org/wiki/VO_language) with fixed word order
(slots). The only exception to the fixed word order is in the order of
adverb and object quantifier: these can be ordered freely with no
difference in meaning.[^2]

As explained above, the verbal and the object quantifier have an
intimate relationship and may be considered one discontinuous
constituent.

Here are examples and syntax schemas for intransitive and transitive
sentences.

<table>
  <tr>
    <th rowspan="3" valign="top">Intransitive sentence</th>
    <td align="center">[<code>"x</code>]</td>
    <td align="center">[<code>9</code>]</td>
    <td align="center"><code>gp</code></td>
  </tr>
  <tr>
    <td align="center"><em>register</em></td>
    <td align="center"><em>count</em></td>
    <td align="center">"command"/<em>motion</em></td>
  </tr>
  <tr>
    <td align="center">Goal</td>
    <td align="center">quantifier</td>
    <td align="center">intransitive/motion verb</td>
  </tr>
</table>

<br />

<table>
  <tr>
    <th rowspan="3" valign="top">Transitive sentence</th>
    <td align="center">[<code>"a</code>]</td>
    <td align="center">[<code>2</code>]</td>
    <td align="center"><code>d</code></td>
    <td align="center">[<code>&lt;C-V&gt;</code>] ~ [<code>3</code>]</td>
    <td align="center"><code>gE</code></td>
  </tr>
  <tr>
    <td align="center"><em>register</em></td>
    <td align="center"><em>count</em></td>
    <td align="center"><em>operator</em></td>
    <td align="center">"force" ~ <em>count</em></td>
    <td align="center"><em>motion</em>/<em>text object</em></td>
  </tr>
  <tr>
    <td align="center">Goal</td>
    <td align="center">quantifier</td>
    <td align="center">transitive verb</td>
    <td align="center">adverb ~ quantifier</td>
    <td align="center">object</td>
  </tr>
</table>


So, coming back to the title sentence `"a2d<C-V>3gE`, we may call this a
maximal transitive sentence in Vim's normal mode language. In a maximal
sentence every constituent slot is filled.

Even in this situation the language retains its human-language
character, and we may translate approximately into English: "Into
register `a` twice delete blockwise to the end of three
whitespace-separated words backwards".

### Word classes

We are now well prepared to identify Vim's parts of speech, or word
classes.

The major word classes are large. They are extensible through mappings
and user-provided language elements. They are referential in that they
refer to actions on the text, or to parts of the text.

As major word classes we can identify the following:

*   intransitive verbs, e.g. `p` `J` `~`

*   transitive verbs, e.g. `y` `d` `gu`

*   motion verbs, e.g. `G` `w` `0` `;`

    *   complex motion verbs, { `f` `F` `t` `T` }

*   common nouns, e.g. `iw` `as` `a[`

The minor word classes are small and closed, that is not extensible. Its
members are more or less enumerable. The minor word classes are:

*   quantifiers, { `1`, `2`, `3`, ... }

*   Goal, { `[-a-zA-Z0-9"*+~_/]` }

*   adverb, { `v`, `V`, `<C-V>` }

### Morphological phenomena

We are not primarily concerned with form in this article, so a few
remarks on the morphology of Vim's words should suffice. The form of
verbs and objects is generally guided by some kind of mnemotechnic
principle: they are supposed to be easy to remember. Most verbs are just
one character in size, but there are exceptions like `gp` (intransitive
verb), `g?` (transitive verb), `gE` (motion verb). Common nouns are two
characters long, and start with `a` or `i`.

There is one interesting phenomenon touching on both morphology and
syntax. Consider these sentences:

> `dd` "delete one line"  
> `cc` "change one line"  
> `yy` "yank one line"

These are exactly equivalent to `d_` `c_` `y_`. These last sentences are
perfectly regular transitive sentences: verb plus the object `_` "the
current line". They can be extended with optional constituents, e.g.
`"xd4_` "into register `x` delete four lines under the cursor".

Now, the object `_` is actually a rather obscure motion verb meaning
"move *count−1* lines downward". So, to make a very common utterance
like `d_` easy to type, the object has been assimilated to the verb:
`dd`. But despite the change in form, the second "d" remains an ordinary
object, so it is possible to say `dv3d` meaning "delete characterwise
three lines under the cursor". Since this is not well known, `dd` is
often interpreted as a single intransitive verb by users and is firmly
idiomatic. What we are dealing with here is an instance of
**[grammaticalization](http://en.wikipedia.org/wiki/Grammaticalization)**.

A final point are intransitive verbs with internalized objects:

> `D` "delete to the end of the line"  
> `x` "delete the character under the cursor"

These are synonymous with the verb-object predicates `d$` and `dl`.
Internally, Vim does in fact translate them to their verb-object
equivalent, and `.` repeats the verb-object sentence.

The interesting case of `S` takes part in both morphological phenomena
shown here: it is translated to `cc` which in turn is the
grammaticalized version of `c_`.

## Outlook

Vim's normal mode grammar, the topic of the above grammar outline, is
the part of Vim that lends itself best to a description in linguistic
terms. It is the part that most directly gives an impression of natural
language.

But Vim is much bigger than that. There are commands -- intransitive
verbs -- which switch into a different but related grammar:

> `v` "go to Visual mode and select the character under the cursor"

After selecting something in Visual mode and pressing an operator -- a
transitive verb -- the verb is applied to the selection, the object:
verb and object have switched places to
[OV](http://en.wikipedia.org/wiki/OV_language).

A clause of an entirely different grammar can even be embedded as the
object of a transitive sentence. Consider this single sentence:

> `d:call setpos(".",[0,3,4,0])<Enter>` "delete to line 3, column 4"

In this example, the object starts with `:` which makes it possible to
use Ex commands to specify the object. The Ex language has its own,
unrelated grammar.

Finally, since to be frank we are not dealing with an actual natural
language, there are a fair number of verbs which do not fit well in a
language interpretation in the terms used in this outline: `q` "record
keystrokes", `m` "drop a mark", `&` "repeat last substitution", and
so on.

## Further reading

*   kana (2008): ["operator, the true power of Vim"][operator]
*   Jim Dennis (2009): [Answer to "What is your most productive shortcut with Vim?"][Jim Dennis]
*   Yan Pritzker (2011): ["Learn to speak vim – verbs, nouns, and modifiers!"][Yan Pritzker]
*   Rafe Colburn (2012): ["The grammar of Vim"][Rafe Colburn]
*   takac (2013): ["Vim Grammar"][takac]
*   AshleyF (2013): *[VimSpeak][]*

[operator]: http://whileimautomaton.net/2008/11/vimm3/operator
[Jim Dennis]: http://stackoverflow.com/a/1220118/329063
[Yan Pritzker]: http://yanpritzker.com/2011/12/16/learn-to-speak-vim-verbs-nouns-and-modifiers/
[Rafe Colburn]: http://rc3.org/2012/05/12/the-grammar-of-vim/
[takac]: http://takac.github.io/2013/01/30/vim-grammar/
[VimSpeak]: https://github.com/AshleyF/VimSpeak

<br />

*First published by glts on April 28, 2013, amended on July 13, 2013.*


[^1]: This may not be the best term. A register can have the semantic
role of recipient or beneficiary with `d` `c` `y`. But it can also be
the source with `p`. I am not aware of a grammatical category that uses
the same marking for both of these, so I chose an ad-hoc term (vaguely
reminding of dative).

[^2]: As a matter of fact, adverb and object quantifier can be
alternated arbitrarily many times, e.g. `dv2V3v2E`. I consider this a
bug, even though the BDFL of Vim [has not been
willing](https://groups.google.com/d/msg/vim_dev/JTJ7qcgqZMg/WUaJJiDXT6cJ)
to acknowledge it as such.
