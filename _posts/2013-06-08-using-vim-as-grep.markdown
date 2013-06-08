---
layout: post
title: Using Vim as a grep
tags: [vim, bash, shell]
---
# Using Vim as a grep

I spent a morning poking at Vim from the command-line with various
arguments, option flags, and redirections. At some point I discovered
this clever contraption. I name it 'vgrep':

{% highlight bash %}
vgrep() {
    vim -N -i NONE -n -e -s <<<"argdo g/${1////\\/}/p" -- "${@:2}"
}
{% endhighlight %}

See what it does:

{% highlight console %}
$ pwd
/home/glts/code/vim-src
$ vgrep '\(http\)\@<!://\S\+' runtime/doc/usr_*.txt
        :Explore ftp://somehost/path/to/dir/
        :e scp://somehost/path/to/dir/
                ftp://ftp.vim.org/pub/vim/README ~
        ftp://ftp.vim.org/pub/vim/MIRRORS ~
        ftp://ftp.vim.org/pub/vim/MIRRORS ~
{% endhighlight %}

It's a grep. With Vim regular expressions. And an adorable little piece
of code that gives us the opportunity to explore some interesting
features of Vim, and Bash.


## Silent Vim

Vim has a number of command-line options that enable special modes of
operation. With `-y` Vim runs in "easy mode", with `-Z` it runs in
"restricted mode", and with `-g` it runs with a GUI, among others. The
most exciting option we have though, is
[`-s-ex`](http://vimdoc.sourceforge.net/htmldoc/starting.html#-s-ex).

The `-e` `-s` flags enable the "silent" or "batch" mode of operation. If
Vim is like GVim without a GUI, then batch mode Vim is like Vim without
any graphical interface at all: it becomes a pure Ex command
interpreter.

Silent Vim consumes input from stdin and executes it as Ex commands. A
pipe is thus the most obvious feeding mechanism, but we can be terser
and more efficient by using Bash's here strings.

{% highlight console %}
$ echo "set bomb?" | vim -e -s
nobomb
$ vim -e -s <<<"set compatible?"
  compatible
{% endhighlight %}

It's a good thing that the `'bomb'` isn't armed, but that Vim is running
in `'compatible'` mode is a bad thing indeed.

We need to add the `-N`ocompatible flag, and while we're at it why not
add all the powerful customizations we have in our vimrc as well, such
as custom filetype detection and the lot? This needs to be done
explicitly because silent Vim skips initializations by default.

And nothing prevents us from passing an argument, too: this will be the
file on which Vim executes the Ex commands.

{% highlight console %}
$ vim -Nu ~/.vimrc -es <<<"set filetype?" 2013-06-08-using-vim-as-grep.markdown
  filetype=liquid
{% endhighlight %}


## Vgrep's parameters expanded

Here's vgrep again.

{% highlight bash %}
vgrep() {
    vim -N -i NONE -n -e -s <<<"argdo g/${1////\\/}/p" -- "${@:2}"
}
{% endhighlight %}

Now it should be clear how it works. It's a Bash function that accepts
two or more arguments, the first of which is a Vim regular expression
that gets injected into this command:

    :g/re/p

This familiar and entirely pronounceable Ex command then forms the
argument to `:argdo` which, you have guessed it, executes `:g/re/p` over
all the file arguments following the double minus.

The two [*parameter expansions*](http://linux.die.net/man/1/bash) in
vgrep certainly add to its magic.

In a nutshell, Bash's parameter expansion performs various string
operations on the contents of a variable, using a highly imaginative
syntax. One of them is "pattern substitution". The pattern substitution
parameter expansion in

<code>${var/<span style="font-weight:normal;">teh</span>/<span
style="font-weight:normal;">the</span>}</code>

yields the contents of variable `var` with the first instance of `teh`
replaced with `the`. Doubling the first slash <code>${var//<span
style="font-weight:normal;">teh</span>/<span
style="font-weight:normal;">the</span>}</code> makes the pattern
substitution global, thus changing all occurrences of `teh` into `the`.

In vgrep, the pattern substitution on `${1}` serves to escape all
forward slashes in the Vim regexp, because they would otherwise
terminate the `:g/re/p` command early. It is these lucky circumstances
plus an escaped backslash that eventually create the satisfying pattern
in <code>${1//<span style="font-weight:normal;">/</span>/<span
style="font-weight:normal;">\\/</span>}</code>.

The second parameter, `"${@:2}"`, expands to the arguments passed to
vgrep minus the first one, it's an array slicing expansion of sorts.
Note the quotes: they are the vital elixir against the [demon inside
Bash](http://mywiki.wooledge.org/BashGuide/Practices#Quoting) that is
*word splitting*.


## A slightly better vgrep

The original `vgrep()` is cute, but it could use a little robustness.
And let's add this grep feature as well:

{% highlight console %}
$ echo "What is love" > faq
$ egrep '' faq
What is love
{% endhighlight %}

The empty regexp matches on every line. This feature is easy to
implement once you realize that the regexp `^` matches on every input
line. Thus, using `^` as the default regexp works well, and there's a
parameter expansion for that.

{% highlight bash %}
vgrep() {
    re=${1:-^}
    vim -N -i NONE -n -e -s <<<"argdo g/${re////\\/}/p" -- "${@:2}"
}
{% endhighlight %}

{% highlight console %}
$ vgrep '' faq
What is love
{% endhighlight %}

Vim opens files that are directories with its netrw plugin. Unless *all*
plugins are disabled via `--noplugin` Vim will open all directories in
the arguments as netrw buffers. This is a problem. A command like

{% highlight console %}
$ vgrep 'Netrw Directory Listing' ~/*
{% endhighlight %}

will match at least once for every directory in `$HOME`. They need to be
filtered out. And the variables should be `local` to the function, too.

{% highlight bash %}
vgrep() {
    local re=${1:-^}
    local files=()
    for arg in "${@:2}"; do
        [ -f "${arg}" ] && files+=( "${arg}" )
    done
    vim -N -i NONE -n -e -s <<<"argdo g/${re////\\/}/p" -- "${files[@]}"
}
{% endhighlight %}


## Silent mode is unfinished business

Perhaps we'd like to implement more features of grep just to see how far
we can push this. The file names aren't listed yet, perhaps the
"only-matching" `-o` switch could be useful, and the match count `-c`
for a quick overview. Can we do all this?

The answer is yes, but it gets ugly fast. `:h -s-ex` states silent mode

> switches off most prompts and informative messages. Also warnings and
> error messages.

Silent mode is actually silent, we can't output anything normally.
`:print` in `:g/re/p` is the exception together with `:set`, `:list`,
and `:number`. And the output of `:print` is the current line and this
can't be changed.

There is help in `:verbose` which gives `:echo` back its voice even in
silent mode but this method has some quirks. First consider this:

{% highlight console %}
$ echo "ho
> hum" | tee sounds | xxd
0000000: 686f 0a68 756d 0a                        ho.hum.
$ vgrep '^h' sounds | xxd
0000000: 686f 0a68 756d 0a                        ho.hum.
{% endhighlight %}

Vgrep `:print`s lines just like the shell's `echo` or *cat* do, with
linefeed-terminated lines. Good. Now consider what happens with
`:verbose echo`:

{% highlight console %}
$ vim -es <<<"verb echo 'ho'|verb echo 'hum'" | xxd
$ vim -es <<<"verb echo 'ho'|verb echo 'hum'" 2>&1 | xxd
0000000: 686f 0d0a 6875 6d                        ho..hum
{% endhighlight %}

In general, unix utilities like outputting to stdout and terminate every
line with newline. It doesn't work like that with `:verbose echo` --
line endings look like CRLF, the final terminator is missing, and output
is to stderr.

The thing is, the silent mode output facilities for `:print` and friends
aren't there for `:echo`. The CRLF-like line endings are in fact
artifacts of cursor movement key codes Vim sends to the terminal while
in raw mode. That is inconvenient and the reason why I feel silent mode
is a feature that could still use some work.

Wrapping it up, here is a vgrep that can show the names of the files it
searches.

{% highlight bash %}
vgrep() {
    local re=${1:-^}
    local files=()
    for arg in "${@:2}"; do
        [ -f "${arg}" ] && files+=( "${arg}" )
    done
    if [ -z "${files[*]:1}" ]; then
        vim -Ni NONE -nes <<<"argdo g/${re////\\/}/p" -- "${files[@]}"
    else
        vim -Ni NONE -nes <<END -- "${files[@]}" 2>&1 | sed -e '1d' -e 's/\r$//'
        argdo g/${re////\\/}/verbose echo expand('%').':'.getline('.')
        verbose echo "\n"
END
    fi
}
{% endhighlight %}

{% highlight console %}
$ diff <(egrep 'http://' ~/*) <(vgrep 'http://' ~/*)
$
{% endhighlight %}

Time to make a [gist](https://gist.github.com/glts/5693322) and call it
a day.

## Further reading

*   *bash*(1)
*   Wikibooks article ['Learning the vi Editor: Modes;
Ex-mode'](http://en.wikibooks.org/wiki/Learning_the_vi_Editor/Vim/Modes#Ex-mode)
(2006)
*   Masi, Super User question ['VIM: What is the EX-mode for batch
processing for?'](http://superuser.com/q/22455) (2009-08-13)
*   kana, vspec ['Driver script to test Vim
script'](https://github.com/kana/vim-vspec/blob/master/bin/vspec)
(2009--2012)
*   Vim tips wiki, ['Vim as a system interpreter for
vimscript'](http://vim.wikia.com/wiki/Vim_as_a_system_interpreter_for_vimscript)
(Nov 2012)

<br />

*First published by glts on June 8, 2013. I appreciate any feedback.*

---
