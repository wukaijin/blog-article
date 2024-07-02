# VSCode 中的 Vim

这两天通过 VSCode 的扩展插件 [_Learn Vim_](https://www.barbarianmeetscoding.com/boost-your-coding-fu-with-vscode-and-vim/) 算是系统学习了 `VSCode` 中的 `Vim`

相关网站 [boost-your-coding-fu-with-vscode-and-vim](https://www.barbarianmeetscoding.com/boost-your-coding-fu-with-vscode-and-vim/dedication)

本文是相关笔记，复习巩固用。

# mode

> - Insert Mode
> - Normal Mode
>   - Move around with the `hjkl` keys.
>   - Go into Insert mode with `i` where you can type stuff as usual.
>   - Go back to Normal mode with `<ESC>`, `<CTRL-C>` or `<CTRL-[>`.
> - Visual Mode

# move around in Vim

Vim 中的光标移动

## hjkl

```
           ↑
     ← h j k l →
         ↓
```

## Moving Horizontally Word By Word

光标 - 水平移动

- `w` to move word by word
- `b` to move backwards word by word
- `W` to move word by WORD
- `B` to move backwards WORD by WORD

大小写的区别是 `black-haired` 大写认为是一个 word， 小写认为是两个
光标会停在单词的首个字母

- `e` to jump to the end of a word
- `ge` to jup to the end of the previous word
- `E` is like `e` but operates on WORDS
- `gE` is like `ge` but operates on WORDS

大小写区别同上  
光标会停在单词的**最后一**个字母

## Move to a Specific Character

行内跳到某个字母

- Use `f{character}` (find) to move to the next occurrence of a character in a line.
- Use `F{character}` to find the previous occurrence of a character

大小写区别是方向  
以下是 `f` 和 `w` 光标跳转区别示意：

> ```
> type f(   ==> v                        v
>               const fireball = function(target){
> type wwww ==> ^     ^        ^ ^       ^
> ```

- Use `t{character}` to move the cursor just before the next occurrence of a character (think of `t{character}` of moving your cursor until that character).
- Again, you can use `T{character}` to do the same as `t{character}` but backwards

大小写区别是方向  
以下是 `f` 和 `t` 光标跳转示意：

> ```
> type t(   ==> v                       v
>               const fireball = function(target){
> type f(   ==> ^                        ^
> ```

After using `f{character}` you can type `;` to go to the next occurrence of the character or `,` to go to the previous one. You can see the `;` and `,` as commands for repeating the last character search.

- `;` 行内的重复搜索跳转
- `,` 向前重复搜素跳转

> ```
> type fdfdfd ==> v   v               v        v
>                 let damage = weapon.damage * d20();
>                 let damage = weapon.damage * d20();
> type fd;;   ==> v   v               v        v
> ```

## Moving Horizontally Extremely

- `0`: Moves to the first character of a line
- `^`: Moves to the first non-blank character of a line
- `$`: Moves to the end of a line
- `g_`: Moves to the non-blank character at the end of a line

## Moving Vertically

- `}` jumps entire paragraphs downwards
- `{` similarly but upwards
- `CTRL-D` lets you move down half a page by scrolling the page
- `CTRL-U` lets you move up half a page also by scrolling

`{}`目前在代码里的表现是在空行之间的代码块跳转  
`CTRL-D`和`CTRL-U`是滚动半屏

## High Precision Vertical Motions With Search Pattern

- `/{pattern}` to search forward
- `?{pattern}` to search backwards

搜索跳转，`/`向前搜索，`?`向后搜索，允许正则匹配

- `n` to go to the next match
- `N` to go to the previous match
  `n`是继续向后搜索，`N` 继续向前搜索

## Moving Faster With Counts

Counts are numbers which let you multiply the effect of a command:  
在命令前加数字，即是多次调用该命令了

```
{count}{command}
```

Try them yourself! Type `2w` to move two words ahead:

```
type wwww ==> v   v v   v   v
              word. are two words
              word. are two words
type 3w2w ==>       ^       ^
```

Try `5j` to jump file lines below`:

-x
0| <-- landing site
-x

Try finding an array within an array:

```
type f[;;  ==> vv    v
               [[1], [1, 2], [3]]
               [[1], [1, 2], [3]]
type 3f[  ==>        ^
```

Try jumping to the second cucumber with `2/cuc`:

```
--------------------------
---v--cucumber------v-----
-----------v---cucumber---
-----v-----------v--------
cucumber------------------
```

## Moving Semantically

- Use `gd` to **g**o to **d**efinition of whatever is under your cursor.
- Use `gf` to **g**o to a **f**ile in an import.

针对代码文件：  
`gd` 跳转到定义  
`gf` 跳转到 import 文件  
`gh` 显示提示

## And Some More Nifty Core Motions

- Type `gg` to go to the top of the file.
- Use `{line}gg` to go to a specific line.
- Use `G` to go to the end of the file.
- Type `%` jump to matching `({[]})`.

`%`值得一说，跳到对应的括号（左跳右，右跳左）

# Editing Like Magic With Vim Operators

Vim 中的 **Commands (命令)**，是一个会影响编辑器的动作。  
**Operators (操作符)** 允许执行操作以更改编辑器内容的命令。

```
   what to do (delete, change...)
      /
     /      how many times
    /         /
   v         v
{operator}{count}{motion}
                    ^
                   /
                  /
           where to perform
             the action
```

For example, take `d2w`. It tells Vim to **d**elete **2** **w**ords. Try it!

```
  start here
  /
 /
v
DO NOT ENTER!
```

假设普通模式下，光标停在 `NOT` 的 `D`上
输入 `d2w` 会删除两个单词，变成 ENTER!  
`dw` 命令，从光标开始向后删除单词，包括单词后面的**空格**。  
`de` 命令，从光标开始向后删除单词，**不**包括单词后面的**空格**。

## Undoing and redoing

Press `u` to undo and you last change will be reverted. You can continue pressing `u` if several things went sideways. If you want to redo, type `<CTRL-R>`.

## The d command

> Mini-refresher: The d command
>
> - d{motion} - delete text covered by motion
>
>   - d2w => deletes two words
>   - dt; => delete until ;
>   - d/he => delete until he
>
> - dd - delete line
> - D - delete from cursor until the end of the line

# The c command

The `c` **change command** deletes a piece of text and then sends you into **Insert mode** so that you can continue typing, changing the original text into something else. The change operator is like the `d` and `i` commands combined into one. **This duality makes it the most useful operator**.

# The Dot Operator

The dot operator allows you to **repeat your last change**. With one single keystroke `.` you can repeat a complete change that can be composed of as many keystrokes  
But **what is exactly a change?** Anything that changes the contents of your editor:

- using `d{motion}` constitutes a change,
- using `i{typeSomething}<ESC>` is another change,
- using `c{typeSomething}<ESC>` is yet another change

So by the very nature of `d` and `c`, the `c` command is more repeatable.

如上文提到，`c`命令类似 `d` + `i`, 把删除和写入内容合到一起，配合重复执行 `.`，可以提高效率。

以此可联想到很多 `d` 变种命令，在 `c`的执行效果

> Mini-refresher: The c command
>
> - c{motion} - delete text covered by motion then input
>
>   - c2w => deletes two words then input
>   - ct; => delete until ; then input
>   - c/he => delete until he then input
>
> - cc - delete line then input on this line of head, equivalent to S
> - C - delete from cursor until the end of the line then input

## Text Objects

```
            |- `a` means around
            |- `i` means inner
           /
          /
         /
        {a|i}{text-object}
                  /
                 /
                | w - word
                | s - sentence
                | p - paragraph
                | " - quotes
```

And then you combine them with an operator like so:

```
{operator}{a|i}{text-object}
```

Text objects with quotes `"`, `'` and backtick are special. They have a forward seeking behavior so that **you don't even need to be on top of the text object itself**.

So *i*nner means that it applies to the inner part of a text object, whereas *a*round means that it applies to the complete text object including delimiters (in case of `(`, `{`, `"`, etc) or whitespace in case of words, sentences and paragraphs.

以 `c` operator 举例

```typescript
const poem = "Roses are red";
const person = {
  name：`Nancy`, sex: 'female'
  age: 12, mother : { name: 'Moth' }
};
```

在 poem 这样，任意位置输入 `ci"` 可以直接删除字符串内的类容并输入， `ca"` 删除包括引号的内容并进入输入。  
`"` 和 `'` 属于特例，在该行的任意位置都能跳转。
其他如 `{}[]()` 只能在中间执行。

## Other Operators

In addition to `d` and `c` these are other useful operators:

- `y` (yank): Copy in Vim jargon
- `p` (put): Paste in Vim jargon
- `g~` (switch case): Changes letters from lowercase to uppercase and back. Alternatively, use `gu` to make something lowercase and `gU` to make something uppercase
- `>` (shift right): Adds indentation
- `<` (shift left): Removes indentation
- `=` (format code): Formats code

`y` and `p` is how you copy and paste things in Vim. Like `d` and `c`, `y` can be combined with motions and text objects to copy any text that you desire:

## More Short-hand Operators

- `x` is equivalent to `dl` and deletes the character under the cursor
- `X` is equivalent to `dh` and deletes the character before the cursor
- `s` is equivalent to `ch`, deletes the character under the cursor and puts you into Insert mode
- `r` allows you to replace one single character for another. Very handy to fix typos.
- `~` to switch case for a single character

A nice way use case for `x` is to swap a couple of characters when you make a typo. You remove (and cut) a character with `x` and immediately paste it after the cursor with `p`. Try it!

`xp`连用，交换下一个字母

`d`, `c`, `x` and `s` in addition to removing and changing also cut (so whatever you remove or change is available in your clipboard).

`d`, `c`, `x` 和 `s` 这些操作实际上是剪贴 (仅 vim 使用, 相当于 `y`， 寄存 `"`中， `p` 或者 `""p` 粘贴)

# Insert Text

- `i` lets you *i*nsert text before the cursor
- `a` lets you *a*ppend text after the cursor
- `I` lets you *i*nsert text at the beginning of a line
- `A` lets you *a*ppend text at the end of a line

```javascript

           you are here
               /
              /
             v
       const status = "I'm in awe"
      ^     ^ ^                   ^
      |     | |                   |
      I     i a                   A

```

- `o` lets you *o*pen a new line below the current line
- `O` lets you *o*pen a new line above the current line

```javascript

           you are here
               /
              /
   O ->      v
       const status = "I'm in awe"
   o ->

```

The next handy mapping to jump into _Insert mode_ is `gi`. `gi` let's you go back to the last place where you made a change.

`gi` 可以直接跳转到上次发生变化的位置，并进入输入模式

> VsCode-Vim 中的' gi '的行为与 Vim 中的不同。在 Vim 中，是回到上次离开插入模式的地方，在 VsCode-Vim 中，进入上次更改的插入模式。

# Remove stuff from Insert mode

Insert mode 中

- `CTRL-H` lets you remove the last character you typed (mnemonic _h_ which in _hjkl_ brings the cursor one space to the left)
- `CTRL-W` lets you remove the last word you typed (mnemonic *w*ord)
- `CTRL-U` you remove the last line you typed (mnemonic *u*ndo this line)

输入模式下，跟 linux 命令行有类似的快捷键

- `CTRL-H` 删除最近的字符 （ backspace ）
- `CTRL-W` 删除**向前**最近的单词（光标前）
- `CTRL-U` 删除**向前**整行

回到普通模式 （Normal mode）

- `<ESC>`
- `CTRL-C`
- `CTRL-[`

# Select Text

_Visual mode_ 相当于在 Vim 中拖动鼠标并选择任意文本位。

- `v` for character-wise visual mode
- `V` for line-wise visual mode
- `Ctrl-V` for block-wise visual mode

```bash
 action to perform
  /
 /
{operator}{count}{motion}
          ---------------
             /
            /
        bit of text over which
        to apply that action
```

In _Visual mode_ we apply an operator in the opposite way. First we select some text visually, then we apply the operator:

```bash
get into Visual Mode
    /
   /                  action to apply
  /                         /
---------                  /
{v|V|C-V}{count}{motion}{operator}
         ---------------
           /
          /
   bit of text over which
   to apply an action
```

- Type `I` to jump into _Insert mode_ and _prepend_ something to all lines in the beginning of the block.
- Type `A` to do the same but this time _append_ something to all lines at the end of the block.

```bash
  start here
  /
 /
v
- buy flour, salt and eggs
- mix two cups flour and some salt
- put a handful on flour on a flat surface
- Make sort of a volcano with the flour
- Break 3 eggs.
- The eggs are the lava in the volcano
- Slowly combine flour and eggs into a dough
- Knead the dough
- Let it rest
```

如以上，选择多行，`A` `I` 可以同时在**选中内容**的头尾操作

# Copying and Pasting

- The behavior of `p` and `P` depends on what you've copied with the `y` command:
  - If you copy characters (like `yaw`) when you use the paste commands you'll paste
    these characters before or after the cursor _within the same line_.
    **Copy characters and you'll paste character**.
  - If you copy lines (like `yy`) when you use the paste commands you'll paste lines
    below or above the current line. **Copy lines and you'll paste lines.**
- You can copy or duplicate entire lines by using `yyp`

# Vim Registers

**Vim Registers** 增强了复制和粘贴(以及其他事情)的工作方式。

- The **unnamed register** `"` is where you copy and cut stuff to, when you don’t explicitly specify a register. The default register if you will.
- The **named registers** `a-z` are registers you can use explicitly to copy and cut text at will
- The **yank register** `0` stores the last thing you have yanked (copied)
- The **cut registers** `1-9` store the last 9 things you cut by using either the delete or the change command

> The yank register (`0`) and the cut registers (`1-9`) are officially called **numbered registers**. But I think it is more useful to name them yank and cut registers. That way, remembering what they do becomes far easier.

For instance, `"ayas` yanks a sentence and stores it in register `a`. Now if you want to paste it somewhere else, you can type `"ap`. Alternatively, using the capitalized version a register (i.e. `A` instead of `a`) appends whatever you copy or cut into that register.

`ayas`提取一个句子并将其存储在寄存器`a`中。现在如果你想把它粘贴到其他地方，你可以输入 `"ap`。或者，使用寄存器的大写版本(即。`A`而不是`a`) 追加你复制或剪切到寄存器中的任何内容。

在 Vim 中，根据您粘贴的方式，您可以替换剪贴板中的任何内容(未命名的寄存器 `"`)。  
`yank register` (`0`)  
The **cut registers** (`1-9`) 您删除或更改的最后 9 个内容

> Did you know? At any point in time, you can use the `:reg` command to see what is in your registers. Or you can type `:reg {register}` to inspect the contents of a specific register.

**Insert mode** 下粘贴：

- `CTRL-R` + `a` pastes from the named register `a`
- `CTRL-R` + `0` pastes from the yank register `0`
- `CTRL-R` + `9` pastes from the cut register `9`

# Command-line mode

`Command-line mode` 是 Vim 中的另一种模式。它的定义特性是能够运行 **Ex commands** 命令 (以`:`开头的命令) 和搜索模式(以`/`和`?`开头)

```bash
:edit {relative-path-to-file}
:e {relative-path-to-file}
```

编辑某个相对路径的文件

- Use `:write` (shorthand `:w`) to save a file
- Use `:quit` (shorthand `:q`) to close a file
- Use `:wall` (shorthand `:wa`) to save all files
- Use `:qall` (shorthand `:qa`) to close all files
- Use `:wqall` (shorthand `:wqa`) to save and close all files
- Use `:qall!` (shorthand `:qa!`) to close all files without saving

```bash
:[range]command[options]
```

举例： `:10,12d a` 删除 10-12 行，并放入寄存器 a

- Using numbers (e.g. `:10,12d` to delete lines 10, 11 and 12)
- Using offsets (e.g. `:10,+2d` to delete lines 10, 11 and 12)
- Using the current line represented by . (e.g. `:.,+2d` to delete the current line and the next two ones)
- Using `%` to represent the whole file (e.g. `:%d` to delete the whole file)
- Using `0` to represent the beginning of the file (e.g. `:0,+10d` to delete the first 10 lines)
- Using `$` to represent the end of the file (e.g. `:.,$d` to delete from the current line to the end of the file)
- If you use Visual mode to make a text selection and then type : your command line area will be pre-populated with the following gobbledygook: `:'<,'>` which is a special range that represents the current visual text selection. (e.g. `:'<,'>d` means delete the current text selection)

- Using $ to represent the end of the file (e.g. `:.,$d` to delete from the current line to the end of the file)

倒数第二条比较有意思，`'<,'>` 可以表示 visual mode 下选中的内容，加上 `d` 之后，文中说删除选中，我试了下，是删除整行, linux vim 下同样如此。

## Repeat ex commands

输入`@:`重复上次的 ex 命令  
`@@` 之后再重复可以这么用。

## Substituting text

`:substitute` 将任意文本替换为您选择的其他文本

```bash
:[range]s/{pattern}/{substitute}/{flags}
```

- `range` defines the range in which we’ll apply the substitution
- `pattern` is a search pattern that describes the text we want to change. Like `/{pattern} it supports regular expressions.
- `substitute` is the text we want to substitute
- `flags` let us set options that configure the substitution

> 如果没 [range], 替换发生在当前行
> 如果 没有 flag `g` 只替换一个

比如 `:%s/^#//`

- `%` for the whole file
- `s` substitute
- `^#` any # at the beginning of a line (i.e. a header in markdown)
- `//` for an empty character

# 未完待续

marks commands
...
