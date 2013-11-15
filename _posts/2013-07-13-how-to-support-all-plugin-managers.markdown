---
layout: post
title: How to support all plugin managers
tags: [vim, plugins]
---
# How to support all plugin managers

One of our interests as responsible Vim plugin authors is to let our
users have the freedom to choose, where it makes sense to do so. We
might design our plugin so that it is platform-independent, or we might
provide a configuration option for one of the more contentious defaults.

One choice that should lie firmly with the user is the choice of plugin
manager. Our plugin should have no say in this matter -- instead it
should support them all.

In this article, I'll show how to support all plugin managers.

Or rather less catchily, how to support seven popular plugin managers,
namely [pathogen.vim][], [Vundle][], [neobundle.vim][],
[vim-addon-manager][], [GetLatestVimScripts][], [Vimana][], and
[vim-flavor][].

[pathogen.vim]: http://www.vim.org/scripts/script.php?script_id=2332
[Vundle]: https://github.com/gmarik/vundle
[neobundle.vim]: https://github.com/Shougo/neobundle.vim
[vim-addon-manager]: https://github.com/MarcWeber/vim-addon-manager
[GetLatestVimScripts]: http://www.vim.org/scripts/script.php?script_id=642
[Vimana]: http://search.cpan.org/dist/Vimana
[vim-flavor]: https://www.relishapp.com/kana/vim-flavor


## Two checklists

This article isn't about what these plugin managers can do. Some of them
are simple, some very capable. Here we are interested in what the
minimal requirements are for a plugin that aims to support easy
installation with all of them.

What needs to be done depends primarily on whether the plugin is a
**stand-alone plugin** (the normal case), or whether it is a **plugin
that has dependencies**.

### Stand-alone plugins

This is the shopping list needed for a simple stand-alone plugin.

*   Follow canonical runtime directory structure
*   Host the plugin in a public repository on [GitHub.com][]
*   Add a "GetLatestVimScripts" line to the plugin script
*   Tag a release with Git tags in version number format
*   Upload an archive of the release to [vim.org][]

[GitHub.com]: https://github.com
[vim.org]: http://www.vim.org

If you follow these guidelines then you will be able to sleep soundly in
the knowledge that your plugin will be installable without fuss on all
plugin managers in popular use.

### Plugins with dependencies

A plugin might depend on another plugin to function properly. In that
situation, the obvious minimum is to declare any dependencies in the
Readme file or the docs. Some plugin managers can automate dependency
installs, and to support this, four additional steps are necessary.

*   Add additional "GetLatestVimScripts" lines for the dependencies
*   Add an `addon-info.json` file to the repository
*   Add `:NeoBundleDepends` statements in your plugin script
*   Add a "Flavorfile" to the repository

That is all. Now that we have these checklists ready, let's go over them
in detail.


## Stand-alone plugins

We will go over each step, explaining why it is necessary and which
plugin managers require it.

### Follow canonical runtime directory structure

<span style="background-color:#ffcc00;">Required by more or less all
plugin managers</span>

This should be a given. Vim runtime files are organized in a number of
directories like `plugin`, `indent`, `autoload`, `doc`, etc. Make sure
to place all script files in appropriate runtime directories, and
nowhere else.

Don't have script files lying around homeless.

Don't have directories hanging around that are unknown or inaccessible
to Vim's runtime system.

The relevant documentation is at [`:h 'rtp'`]['rtp'] and [`:h
usr_41`][usr_41].

['rtp']: http://vimdoc.sourceforge.net/htmldoc/options.html#%27rtp%27
[usr_41]: http://vimdoc.sourceforge.net/htmldoc/usr_41.html

### Host the plugin in a GitHub.com repository

&#8203;<span style="background-color:#ffcc00;">Required by
<em>vim-flavor</em></span>  
<span style="background-color:yellow;">Recommended for
<em>pathogen.vim</em>, <em>Vundle</em>, <em>neobundle.vim</em>,
<em>vim-addon-manager</em></span>

Hosting a plugin in a Git repo on GitHub is widespread practice
nowadays. Pathogen.vim advertises the `git clone git://github.com/...`
installation procedure in its Readme file, and many modern plugin
managers such as Vundle, neobundle.vim, and VAM seem to regard a GitHub
repo as the principal source for a Vim plugin.

Vim-flavor goes one step further: A public repo on GitHub is required
for a plugin to be installable with vim-flavor. While there may be good
reason to avoid GitHub, in order to support vim-flavor such a repo is
(currently) strictly required.

### Add a "GetLatestVimScripts" line in the plugin script

<span style="background-color:yellow;">Recommended for
<em>GetLatestVimScripts</em></span>

This step isn't strictly required for a stand-alone plugin. It is a fair
thing to do though, as it enables GetLatestVimScripts to recognize an
installed plugin after the fact, for example if it has been installed
without going through the normal installation procedure of adding an
entry to `GetLatestVimScripts.dat`.

Add a line like

    " GetLatestVimScripts: 6789 1 :AutoInstall: shebang.vim

to your `plugin/shebang.vim` script, where "6789" is the script ID on
vim.org (the one in the plugin URL), and "shebang.vim" is the name of
your plugin (as it appears in the title of the plugin page).

GetLatestVimScripts will see the line and register the plugin for
automatic updating.

But note: GetLatestVimScripts looks for such lines only inside
`plugin/**/*.vim` files. It is not a clever plugin. If your plugin
consists of `autoload` scripts only, then your best bet is to create a
`plugin/shebang.vim` file containing only the "GetLatestVimScripts"
line. If this is your situation and you decide not to support
GetLatestVimScripts you have my sympathy. (Alternatively, file a bug
with the GetLatestVimScripts author.)

### Tag a release with version number tags

<span style="background-color:#ffcc00;">Required by
<em>vim-flavor</em></span>

Using Git tags to properly version a release is good practice in any
serious development effort.

Vim-flavor has a rigid versioning and dependency policy which it
enforces using Git tags. Not any old Git tags, mind you: The tags must
be in a parsable version number format, for instance, `0.3.0`, `v2.45`,
or `7.2`. Use annotated tags with `git tag`:

    git tag -a v1.0.0

Plugins not conforming to this requirement will be refused installation
by vim-flavor.

### Upload an archive of the release to vim.org

&#8203;<span style="background-color:#ffcc00;">Required by
<em>GetLatestVimScripts</em></span>  
<span style="background-color:yellow;">Recommended for <em>Vimana</em>,
also recommended by <a
href="http://vimdoc.sourceforge.net/htmldoc/usr_41.html#distribute-script"><code>:h
distribute-script</code></a></span>

Unbeknownst to many a GitHub hipster, the ["scripts" section on
vim.org](http://www.vim.org/scripts/index.php) remains the official
distribution channel and is in active use.

The GetLatestVimScripts plugin, which is included in the runtime files
distributed with Vim, downloads plugins from vim.org, as does Vimana
(though it handles other sources as well). In order to support
GetLatestVimScripts uploading an archive of the plugin to vim.org is
required.

I recommend packaging the plugin as a zip archive for best
compatibility. The official recommendation is to make a
[Vimball](http://www.vim.org/scripts/script.php?script_id=1502), but
this format has never been widely accepted, and personally I try to
avoid them as best I can.

**Very important:** the archive must be flat. All runtime directories
must be contained directly (at the top level) inside the zip. There must
*not* be any additional files lying around like a Readme file or VCS
files. For example, this would be a good upload for the fictional
*shebang.vim* plugin, uploaded to vim.org as `shebang-1.0.0.zip`:

    autoload/
        shebang/
            shebang.vim
            util.vim
    doc/
        shebang.txt
    plugin/
        shebang.vim

As a bonus, this style also makes it possible to install without any
plugin manager by simply honouring the ancient installation tradition
that goes: "Unzip in `~/.vim`".


## Plugins with dependencies

A few plugin managers have the ability to retrieve and install
dependencies automatically: VAM, neobundle.vim, GetLatestVimScripts, and
vim-flavor can do this given proper support by the plugin.

Unfortunately, every plugin manager has its own way of doing it so we
must support them all separately. It isn't hard to do though.

### Add additional "GetLatestVimScripts" lines for the dependencies

<span style="background-color:#ffcc00;">Required by
<em>GetLatestVimScripts</em></span>

This is straightforward. Adding additional "GetLatestVimScripts" lines
of the same format makes GetLatestVimScripts retrieve the dependencies,
too. My experience has been better when I placed the dependency lines
*above* the line for the plugin itself.

Assuming the *shebang.vim* plugin depends on the *basebang.vim* plugin
(which also Supports All Plugin Managers), the following declaration
inside `plugin/shebang.vim` would enable automatic installation with
GetLatestVimScripts:

    " GetLatestVimScripts: 3456 1 :AutoInstall: basebang.vim
    " GetLatestVimScripts: 6789 1 :AutoInstall: shebang.vim

### Add an addon-info.json file to the repository

<span style="background-color:#ffcc00;">Required by
<em>vim-addon-manager</em></span>

VAM takes a different approach. An `addon-info.json` file at the root of
the repository contains meta information. Here is a simple example of
such a file:

    {
        "name": "shebang.vim",
        "author": "glts <676c7473@gmail.com>",
        "dependencies": {
            "basebang.vim": {}
        }
    }

The crucial bit here is the `"dependencies"` entry. Take special care to
use the right names, both for the plugin name as well as for the
dependencies. The correct names are as they were registered on vim.org
and as they appear on the plugin's page.

### Add `:NeoBundleDepends` statements in your plugin script

<span style="background-color:#ffcc00;">Required by
<em>neobundle.vim</em></span>

Neobundle.vim is pretty feature-rich, and as such it has not one but two
different ways of declaring plugin dependencies. The simpler one uses
`:NeoBundleDepends`. This is a command that you can put in your plugin
script. Surrounded with a little boilerplate it looks like this:

    if exists(':NeoBundleDepends') == 2
      NeoBundleDepends 'glts/vim-basebang'
    endif

Put this anywhere in `plugin/shebang.vim` and neobundle will
automatically pull in the *basebang.vim* dependency from its GitHub
repository.

(For the second way look up `:h neobundle-recipe`.)

### Add a Flavorfile to the repository

<span style="background-color:#ffcc00;">Required by
<em>vim-flavor</em></span>

Like VAM, vim-flavor uses a special file with dependency information at
the root of the repository. This is called a "Flavorfile"; the file name
must be `VimFlavor`. The format is more Ruby-like and rather simple.
Here is the complete Flavorfile for *shebang.vim*:

    flavor 'glts/vim-basebang', '~> 1.6'

This declares a dependency on the `1.6` version tag of the
`glts/vim-basebang` repository.

This also clears up why vim-flavor is so strict in its requirements for
a GitHub repo and version tags: It's so that any plugin can be declared
a dependency in Flavorfiles.


## Summary

This checklist has grown rather long. But this shouldn't mislead anyone
into thinking it's much work. Supporting All Plugin Managers is actually
quite easy. For very little effort you can give the power to choose a
plugin manager back to your users.

Another point to keep in mind is that plugin managers for Vim are still
a very busy area. Until there is a "winner", if there should ever be
one, I recommend to support them all.

For an example of a stand-alone plugin that heeds the advice in this
article, take a look at my *[cottidie.vim][]* plugin.

For an example of a plugin with dependencies, check out
*[textobj-comment][]*. It relies on *[textobj-user][]* to do its job.
(Unfortunately, *textobj-user* does not support GetLatestVimScripts
itself, so that auto-installation isn't possible in this case.)

[cottidie.vim]: https://github.com/glts/vim-cottidie
[textobj-comment]: https://github.com/glts/vim-textobj-comment
[textobj-user]: https://github.com/kana/vim-textobj-user

<br />

*First published by glts on July 13, 2013, updated on November 15, 2013.
Your feedback is welcome.*

*[VCS]: version control system
*[VAM]: vim-addon-manager

---
