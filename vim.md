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
