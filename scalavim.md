# Programming Scala in Vim
- Michael Schmitz
- schmmd
- 2011/10/11
- Scala
- published

I moved from programming Java in Eclipse to programming Scala in Eclipse.  I
didn't need any convincing that Scala was a surperior language than Java, but
the fact that Scala tools were years behind Java tools smacked me in the
face.  <a title="ScalaIDE" href="http://www.cala-ide.org/">ScalaIDE</a>, the
Eclipse plugin for Scala, would have unexpected bugs (right clicking browse to
source works, but F3 doesn't), unimplemented features (can't watch variables),
and huge performance issues.  An Eclipse plugin should never grind Eclipse to
a halt and issue `OutOfMemoryException`s on a 4 core 32 GB machine
(admittedly it was a 32-bit OS).  The plugin has improved greatly in the past
months, but it still causes many headaches. Besides, Eclipse doesn't work over
ssh.

First, you will need syntax highlighting in vim.  To set this up, go to 
<a href="https://github.com/scala/scala-dist/tree/master/tool-support/src/vim">this GitHub repository</a>. 
You will to download three of the files and put them in `$HOME/.vim/`

* `~/.vim/indent/scala.vim` (indentation rules)
* `~/.vim/ftdetect/scala.vi`m (detect scala files)
* `~/.vim/syntax/scala.vi`m (syntax highlighting)

Now when you start vim, you should have syntax highlighting (although you might need to add `:syntax on` to your `vimrc`.

![syntax](posts/scalavim-syntax.png)

While syntax highlighting is a must, it alone probably won't pull you away from
a full-featured IDE.  However, with continuous compilation using the Fast Scala
Compiler (fsc), one can code effectively in Scala using `vim`.

It's helpful to have two panes so you can see the output of the compiler while
you are editing the code. I like doing this in a `screen` instance. `screen`
has the added benefit that, if you are using an ssh terminal, you can
disconnect and reconnect without losing your place. `screen` has the downside
that now you are able to have windows inside `vim`, inside `screen`, and
(maybe) inside your window manager.  The fact that each (necessarily) has a
separate key to jump between windows can be frustrating (note<a
href="http://www.vim.org/sponsor/vote_results.php">vim may allow terminals in
buffers</a> at some time). `screen` also does not do a great job of managing
multiple windows, and it overrides `CTRL+A`.

Regardlesss, start up `screen`. To create a split window, hit `CTRL+S SHIFT+S`
(control S followed by shift S). Next, switch down to the new pane with `CTRL+A
TAB` and create a new screen window with `CTRL+A C`. You can move between these
two panes with `CTRL+A TAB`, as before.

Before you do jump back up to the top pane, however, you might as well start
continuous compilation in the lower pane. If you are using maven, start `mvn
scala:cc` and if you are using sbt, start `sbt ~compile`. Now switch back to
your code pane, open a file with `vim`, make an edit, save your changes, and
watch!

![split](posts/scalavim-split.png)

If you were to make a mistake and save, you get fast feedback.

![error](posts/scalavim-error.png)

As a side note, if you are not interested in real time compilation but would
rather compile with the built-in make command (`:make`) you can set your
compiler to maven and use this <a href="https://gist.github.com/1320359">maven
compiler plugin for vim</a>. You may even find this helpful in addition,
because when you compile with `:make`, you can <a
href="http://vimdoc.sourceforge.net/htmldoc/quickfix.html#:make">jump to the
errors with QuickFix</a>.

By this point we have syntax highlighting and real-time feedback from the
compiler. This is great, but you'll probably still miss a few key IDE
features. For one, it's hard to jump between files.

Vim has a few commands that can help you switch between files. `find` will
search for exact matches and open them in the editor. You can use wildcards and
search recursively.  

    :find **/Hello*scala

Or, if you are more comfortable doing so, you can edit the results of an
external command.  

    :e `find -name Hello*scala`

Vim also supports `:grep` (external grep) and `:vimgrep`
(internal grep) command. The latter supports a lot of vim features (for
example, you can navigate through multiple results by opening the quicklist
window :copen), but is slower because each file is loaded into vim. <a
href="http://vim.wikia.com/wiki/Find_in_files_within_Vim">This vim tip</a>
contains more information on finding files using vim commands.

While useful, I find these tools rather limited. I prefer the <a
href="https://wincent.com/products/command-t">Command-T plugin</a>, which adds
excellent search functionality. From any vim project, you can hit
`<LEADER>+T` (the leader key is '\' by default) and then type to
interactively find a file. While not worlds different than the built-in
commands, I find Command-T much more effective and easier to use.

![commandt](posts/scalavim-commandt.png)

If you are looking for more control when jumping between files, you can add navigation to definitions using <a href="http://ctags.sourceforge.net/">exuberant ctags</a>.  The `ctags` project was created to support this functionality in vi with C. `exuberant ctags` has replaced the original with the ability to extend to many languages.

First, you will download the following gist to `$HOME/.ctags` in order to support scala.

    --langdef=scala
    --langmap=scala:.scala
    --regex-scala=/^[ \t]*((abstract|final|sealed|implicit|lazy)[ \t]*)*(private|protected)?[ \t]*class[ \t]+([a-zA-Z0-9_]+)/\4/c,classes/
    --regex-scala=/^[ \t]*((abstract|final|sealed|implicit|lazy)[ \t]*)*(private|protected)?[ \t]*object[ \t]+([a-zA-Z0-9_]+)/\4/c,objects/
    --regex-scala=/^[ \t]*((abstract|final|sealed|implicit|lazy)[ \t]*)*(private|protected)?[ \t]*case class[ \t]+([a-zA-Z0-9_]+)/\4/c,case classes/
    --regex-scala=/^[ \t]*((abstract|final|sealed|implicit|lazy)[ \t]*)*(private|protected)?[ \t]*case object[ \t]+([a-zA-Z0-9_]+)/\4/c,case objects/
    --regex-scala=/^[ \t]*((abstract|final|sealed|implicit|lazy)[ \t]*)*(private|protected)?[ \t]*trait[ \t]+([a-zA-Z0-9_]+)/\4/t,traits/
    --regex-scala=/^[ \t]*type[ \t]+([a-zA-Z0-9_]+)/\1/T,types/
    --regex-scala=/^[ \t]*((abstract|final|sealed|implicit|lazy)[ \t]*)*def[ \t]+([a-zA-Z0-9_]+)/\3/m,methods/
    --regex-scala=/^[ \t]*((abstract|final|sealed|implicit|lazy)[ \t]*)*val[ \t]+([a-zA-Z0-9_]+)/\3/l,constants/
    --regex-scala=/^[ \t]*((abstract|final|sealed|implicit|lazy)[ \t]*)*var[ \t]+([a-zA-Z0-9_]+)/\3/l,variables/
    --regex-scala=/^[ \t]*package[ \t]+([a-zA-Z0-9_.]+)/\1/p,packages/ 

You can generate ctags for scala (and any other supported language, which include Java by default) by running `ctags -R`. If you want to create tags for dependencies, the following command will unpack all the available sources for dependencies in `target/dependency`.

    mvn dependency:unpack-dependencies
      -Dclassifier=sources
      -Dmdep.failOnMissingClassifierArtifact=false

You could generate tags once globally for each project, but I find it easier to
create tags on a per-project basis. You may still want to have a seperate ctags
file for OpenJDK sources. In this case, you can have a secondary tags file.

Vim has excellent support for ctags.  First, you can set the tag file by adding
the following to your vimrc (looks for tags file in the directory of the open
file, then the tags file in the current directory).

    set tags=./tags,tags

You can chain more tags files if you have more than one.

    set tags=./tags,tags,/my/other/tags

For a taste, I'll introduce a few common commands. Jump to a tag inside vim
using `:ta TagName`. Or better yet, jump to the tag under the cursor with
`CTRL+]`. You can return to where you were with `CTRL+T`. To learn more about
using ctags in vim, read <a
href="http://vimdoc.sourceforge.net/htmldoc/tagsrch.html#ctags">`:h ctags`</a>
or this <a href="http://vim.wikia.com/wiki/Browsing_programs_with_tags">vim
wiki</a>.

If you want, check out my <a href="https://gist.github.com/1279915">vimrc</a>.

Enjoy your vim IDE! Happy vimming!

---

Some more useful tools.

* <a title="scalariform" href="http://mdr.github.com/scalariform/">scalariform</a>. Code formatting for scala.
