# webcomplete.vim

A Vim plugin that completes words from the currently open web page in your
browser.

![demo](./demo.gif)

# Limitations

## Chrome 59 and AppleScript

The old approach of this extension was to interact with Chrome (on macOS) was via AppleScript. Starting with Chrome 59, Google is disallowing access via AppleScript. The workaround is to open Chrome with remote debugging

```
open /Applications/Google\ Chrome.app/ --args --remote-debugging-port=9222'
```

You can test if the port is open with `telnet localhost 9222`

## Empty results

Sometimes I get an empty result back from `sh/webcomplete`. This is because the actual response, doesn't
contain any data, but this:

```
{u'id': 0, u'result': {u'result': {u'type': u'string', u'value': u''}}}
```

I highly suspect this has something to do with the  Chrome Remote Debugging Protocol
and which target it sets.

This is not reproducible and so I thought the python client is broken and turned to `pcyrus-and/chrome-remote-interface`. With the default usage I got also not the forefront tab back but almost always some extension.

```
npm install -g chrome-remote-interface
chrome-remote-interface inspect
>>> Runtime.evaluate({expression: 'window.location.toString()'})
{ result:
   { result:
      { type: 'string',
        value: 'chrome-extension://bikioccmkafdpakkkcpdbppfkghcmihk/background.html' } } }
>>> Runtime.evaluate({expression: 'window.location.toString()'})
{ result:
   { result:
      { type: 'string',
        value: 'chrome-extension://bikioccmkafdpakkkcpdbppfkghcmihk/background.html' } } }
```

I filed a bug, see https://github.com/cyrus-and/chrome-remote-interface/issues/91

> What happens when you do not specify the target (tab option) is that the first entry of CDP.List() (or chrome-remote-interface list) having the webSocketDebuggerUrl property set is used (note that this field is not available if the target is already being inspected).

The recommendation is to filter for `page` objects in the result, but I can't identify what the current page is!

I browsed the documentation for a while and can't find out how to do that. All examples also only point tout how to navigate to a page, and then get data but don't assume that there is a user interacting with the browser.

Let's hope my questions gets answered via the mailing list

https://groups.google.com/forum/#!topic/chrome-debugging-protocol/6jIRM6KSRcY

# Requirements

After that you need to install some python packages

```
git clone git@github.com:max-weller/chrome_remote_shell.git
cd chrome_remote_shell
pip install --user greenlet
pip install websocket-client
python setup.py install
```

This solution was not my idea, but came from here

https://gist.github.com/baongoc124/ab7fab56f9f95363c75ff79b19257b60

# Installation

With [vim-plug](https://github.com/junegunn/vim-plug):

```
Plug 'thalesmello/webcomplete.vim'
```

## Using with deoplete

[`deoplete`](https://github.com/Shougo/deoplete.nvim/) is an awesome asynchronous
completion engine for Neovim. `webcomplete` works with `deoplete` out of the box.
Just start typing to see suggestions of words comming from your browser.

## Using with `completefunc` or `omnifunc`

Vim allows you to define a `completefunc` or an `omnifunc` to give you
completions during insert mode. `webcomplete` provides you with a function that
you can plug into these built in features.

To set it up, use either of the two lines below:
```
" Use <C-X><C-U> in insert mode to get completions
set completefunc=webcomplete#complete

" Use <C-X><C-O> in insert mode to get completions
set omnifunc=webcomplete#complete
```

# Contributing

If you would like to contribute to the project by supporting your browser or
operating system, I would be happy to accept pull requests.

# Inspiration

The project was only possible with the [help of Reddit user 18252](https://www.reddit.com/r/commandline/comments/4j73um/any_way_of_getting_the_text_of_open_chrome_pages/d34ftzx)
and by looking at [tmux-complete.vim](https://github.com/wellle/tmux-complete.vim) as reference when implementing this plugin.
