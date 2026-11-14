# Using the editor with a keyboard

Everything in a row, a column, an element or a piece of text can be done without
a mouse.

## While you are typing

| Key | What it does |
| --- | --- |
| `Tab` / `Shift+Tab` | Move to the next or previous column, then on to the next row |
| `Esc` | Leave the text and go to the tools for whatever you are in |
| `Alt+F10` | The same as that first `Esc` |

Each `Esc` after that steps out one level, from the thing you are in to the
thing that holds it, and a step past the last one brings you back to your text:

```
your text  →  text tools  →  image tools  →  element tools
           →  column tools  →  row tools  →  add a row  →  back to your text
```

`Shift+Esc` steps back in, so going one too far costs a single key.

You only ever see the levels that apply. Plain text in a one-column row is
text tools, column, row. An image gives you its own tools as well. The column
tools always belong to the column the cursor was in, whichever column that is.

## While you are on a tool strip

| Key | What it does |
| --- | --- |
| `←` `→` | Move along a strip laid out in a row |
| `↑` `↓` | Move along a strip laid out in a column |
| `Home` / `End` | First or last button on the strip |
| `Enter` or `Space` | Use the button |
| `Esc` | Step out one level |
| `Shift+Esc` | Step back in |

A panel that opens from a button, like Text Settings, takes the focus. `Tab`
and the arrows move around inside it, `Esc` closes it and puts you back on the
button you opened it from. After you use something in the panel the focus stays
where you used it, so you can apply another size or colour straight away.

Your text stays selected the whole time, so bold, a link, a size or an
alignment applies to what you had selected. Coming back to your text leaves the
selection as you left it.

## Adding a snippet

`Tab` from the content reaches the snippets panel: first the category, then the
list of snippets.

| Key | What it does |
| --- | --- |
| `←` `→` `↑` `↓` | Move between snippets |
| `Home` / `End` | First or last snippet |
| `Enter` or `Space` | Add the snippet below the row you were in |
| `Esc` | Leave the list, back to your text |

The snippet is added where the cursor was, so put the cursor where you want it
before going to the panel.

## Choosing a picture, a table or anything else

Put the cursor in the block that holds it, then press `Esc`. The editor selects
it, just as clicking it would, and its own tools join the walk:

```
your text  →  text tools  →  picture tools  →  element tools  →  column  →  row
```

So to change a picture: cursor in the paragraph that holds it, `Esc` twice, and
you are on **Change Image**, with Link, Select and Edit next to it.

To get from there to the text in another column, `Tab` leaves the tools for the
content, or `Esc` walks all the way back to where the cursor was and `Tab` moves
on from there.

## Worked examples

Each of these starts with the cursor in your text.

**Make text bold**

1. Select the text.
2. `Esc` to the text tools. The focus lands on **Bold**.
3. `Space` to apply it. The focus stays on Bold, so `→` then `Space` italicises it too.
4. `Esc` when you are done, until you are back in your text.

**Make text uppercase** (a button that lives inside a panel)

1. Select the text.
2. `Esc` to the text tools.
3. `→` until you reach **Formatting**, then `Enter`. The panel opens and the focus moves into it.
4. `→` until you reach **Uppercase**, then `Space`.
5. `Esc` closes the panel and puts you back on Formatting.
6. `Esc` again to carry on out, or `Shift+Esc` to go back into the panel.

**Change the font size**

1. Select the text.
2. `Esc` to the text tools.
3. `→` until **Text Settings**, then `Enter`.
4. `→` or `Tab` to the size you want, then `Space`. The focus stays there, so you can press it again.
5. `Esc` closes the panel.

**Add a link**

1. Select the text.
2. `Esc` to the text tools.
3. `→` until **Hyperlink**, then `Enter`. The dialog opens with the cursor in the address field.
4. Type the address, `Tab` to **OK**, then `Enter`.

**Change a picture**

1. Put the cursor in the paragraph that holds the picture.
2. `Esc` to the text tools.
3. `Esc` again to the picture tools. The focus lands on **Change Image**.
4. `→` for Link, Select and Edit, then `Enter`.
5. `Esc` carries on out, or `Tab` goes straight back to the content.

**Add a snippet**

1. Put the cursor in the row you want the snippet to go after.
2. `Tab` until you reach the snippets list.
3. Arrows to the snippet you want.
4. `Enter` to add it.
5. `Esc` puts the cursor back in your text. `Shift+Tab` goes to the category
   list instead.

**Move a row**

1. Put the cursor in the row.
2. `Esc` until you reach the row tools. The focus lands on **Move**.
3. `Space` to pick the row up.
4. `↑` or `↓` to move it, `Space` to drop it. `Esc` leaves it where it was.

**Add a column**

1. Put the cursor in the column you want the new one next to.
2. `Esc` until you reach the column tools. The focus lands on **Add column**.
3. `Enter`, then choose from the panel.

**Delete a row**

1. Put the cursor in the row.
2. `Esc` until you reach the row tools.
3. `↓` to **Delete**, then `Space`. The cursor moves to the next row.

## Canvas mode

Blocks float freely rather than sitting in rows, and a block is a stop in the
tab order the way a column is elsewhere. From a block:

| Key | What it does |
| --- | --- |
| `Tab` | The text inside the block, then on to the next block |
| `Shift+Tab` | Back the same way |
| `Esc` | The tools, then out one level at a time |

The text behaves as it does anywhere else once you are in it.

```
block  →  text tools  →  page tools  →  add a page  →  back to the block
```

The page tools move, duplicate or remove the whole page, and add another.

## Small screens

Below 970px the tools are hidden, because laying out a page on a narrow screen
is awkward. To keep them at every width, set `simpleEditingBreakpoint` to an
empty string:

```js
new ContentBuilder({ container: '.container', simpleEditingBreakpoint: '' });
```
