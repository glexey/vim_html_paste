vim_html_paste
==============

Automation recipe for pasting formatted code from gVIM to Outlook, blog post, etc.


PROLOGUE
========

Pasting HTML-formatted code from a text editor to e.g. e-mail is a pretty common task. For VIM users,
the editor provides 'TOhtml' command to create HTML code for the formatted text, but out-of-the-box this
is rather cumbersome to use. The sequence would be something like: select, :TOhtml, save to a file,
open in the browser, select/copy, paste into the e-mail. Even worse if you are trying to do same thing
from Linux session in the VNC window. Oh, and if you prefer dark color schemes for editing,
you probably want to switch to light one for pasting code to an e-mail.
After browsing for existing solutions I have created my own
simple automation that after some initial setup allows to perform the task in 3 keystrokes, and
works both from gVIM under Windows, as well as from gVIM in a VNC.


INSTALL/SETUP
=============

**Step 1**: Add the following code to your .vimrc:

   command! -range=% ClipHtml :call ConvertToClipHtml(<line1>, <line2>)
   function! ConvertToClipHtml(line1, line2)
      " Remember current colorscheme, then switch to the one with the
      " light background
      let l:prev_color = g:colors_name
      colorscheme emacs " viable options are: automation, bw, c, emacs, tolerable
      " Convert to HTML and copy to clipboard
      let g:html_font = "Courier New"
      let g:html_start_line = min([a:line1, a:line2])
      let g:html_end_line = max([a:line1, a:line2])
      runtime syntax/2html.vim
      normal gg"+yG
      " Close the buffer and clean up
      bdelete!
      unlet g:html_start_line
      unlet g:html_end_line
      unlet g:html_font
      " restore the original colorscheme
      exec "colorscheme " . l:prev_color
   endfunc

This function automates the 2html.vim call: generates HTML for the selected text in a new temporary
buffer, copies contents to the system clipboard as text, and closes the temporary buffer. Before generating HTML the color scheme is switched to light-backroung one, and after the task is complete,
restores the original color scheme. You can also add a shortcut key to call the above function on the
current selection. My personal choice is Shift-C (in visual mode only):

   vnoremap C :ClipHtml<CR>

**Step 2**: Copy clip2html.exe executable to your hard drive and create a shortcut for it either on
Windows Desktop or in Windows start menu (in any other location hotkeys won't work).
Right-click on the shortcut, select 'Properties', and assign a hotkey combination of your liking.
My choice is Ctrl-Shift-V. Alternatively, if you have python installed, you can use the python
source script clip2html.py instead and create a shortcut to execute it with python. This
script/executable takes clipboard contents as text, and marks it as HTML, so that when it's
pasted into HTML-aware context (e.g. Outlook e-mail), the formatted text is pasted.


USAGE
=====

With the above setup, the copy-paste process looks like this:

1. In gVim, select the text and press 'C'
2. Anywhere in Windows, press: Ctrl-Shift-V
3. In the e-mail editor, press: Ctrl-V


ACKNOWLEDGEMENTS
================

The solution is based on the code from thse sources:
http://code.activestate.com/recipes/474121
https://github.com/sgraham/sgraham/blob/master/vimfiles/plugin/cliphtml.vim

thanks guys!


NOTES
=====

Windows executable was created with [pyinstaller](http://www.pyinstaller.org) using the following command:

    pyinstaller clip2html.py -F -w --distpath .
