---
id: filesystem
title: The virtual filesystem
---

Web browsers can't access the user's filesystem directly because of security 
considerations. Therefore Vorple uses a virtual filesystem to handle save games,
[JavaScript communication](in-depth.html) and any external files that the Inform
story file might want to read and write.

The virtual filesystem lives in the browser's 
[localStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage).
LocalStorage persists between sessions so anything saved to the filesystem is
available when the player runs the game again, even if they closed the browser in
the meanwhile (assuming they didn't clear the browser data manually.)


## Accessing the filesystem with JavaScript

Vorple provides methods for some of the most common actions like reading, writing
and deleting files and directories. To get access to more methods, `vorple.file.getFS()`
returns the [BrowserFS](https://jvilk.com/browserfs/) object that can be used to
access the full API. 

The Vorple filesystem methods are documented in the [JavaScript API](js-api.html#file)
and BrowserFS API methods can be found 
[here](https://jvilk.com/browserfs/1.4.1/classes/_backend_localstorage_.localstoragefilesystem.html). 
Note that Vorple's methods are slightly different from BrowserFS methods,
so read the documentation carefully before using them.


## Directory structure

The directory structure created by Vorple looks like this:

```text
inform/
savegames/
transcripts/
vorple/
├── VpHndshk
├── VpJSEval
├── VpJSRtrn
└── VpJSType
```

The *inform* directory is what contains the files Inform handles. When you say for example...

```inform7
The file of Magic Tricks is called "tricks.txt".
```

...Vorple will actually try to read a file called `/inform/tricks.txt`. Note that this
applies only to Vorple, so on a standard interpreter the location would be different,
depending on the interpreter.

The `vorple.file` methods default to using *inform* as the current working directory (cwd)
so `vorple.file.read("foo")` is equivalent to `vorple.file.read("/inform/foo")`
which is equivalent to `vorple.file.read("foo", { cwd: "/inform" })`.

The files in the *vorple* directory are used by Vorple to tell the Inform story that it's
running in the Vorple interpreter (VpHndshk), pass JavaScript code from Inform to the
interpreter for evaluation (VpJSEval) and pass the results of the evaluation back to 
Inform (VpJSRtrn and VpJSType).

While it shouldn't ever be necessary to touch the files in the *vorple* directory, it 
should be noted that using `vorple.file.read()` to read the handshake file (VpHndshk)
will always return the same standard response, regardless of the file's actual contents.
By getting different contents than what it just wrote to the file is what allows Inform
to determine that it's running on the Vorple interpreter.

The *savegames* and *transcripts* directories contain the files that Vorple creates when
the player saves the game or starts a transcript. While reading these files is safe, it's
easy to break things by trying to edit them.

To avoid name clashes, it's recommended to create a new uniquely named directory to store
your files if possible (files read by Inform must be in the *inform* directory.)


## Exchanging files with Inform 7

Inform 7 writes files with a special header that tells which project owns the file.
The header also has information about whether the file is "ready to read", which 
signifies that the file's owner has finished handling it and it's ready to be handled by
another project. See [chapter 23.15.](http://inform7.com/book/WI_23_15.html) in *Writing with Inform* for details.

If we want Inform 7 to be able to read the files we write, we'll have to name our
project and use the same name in the Inform code. 

```inform7
The file of Best Recipes (owned by project "VORPLE") is called "recipes".
```

Vorple detects whether it's running an Inform 6 or 7 game and automatically adds the
correct header to Inform 7 files when using `vorple.file.write()` and doesn't include
it to the return value of `vorple.file.read()`.

The default project name is "VORPLE" but it can be changed when writing the files:

```js
vorple.file.write("recipes", "Mushroom soup: ...", { project: "COOKING" });
```

Contents of the created file:

```text
* //COOKING// recipes
Mushroom soup: ...
```

The information in the header (and other information) can be retrieved with
`vorple.file.info()`:

```js
vorple.file.info("recipes");

// example result:
{
    contents: "Mushroom soup: ...",
    directory: "/inform",   
    header: {
        project: "COOKING",
        ready: true         
    },
    name: "recipes",
    path: "/inform/recipes",
}
```

The method `vorple.file.markReady()` can be used to toggle a file's ready status 
(see [the API](js-api.html#file) for details.) The `vorple.file.write()` method
automatically marks the file as ready to read after it has finished.

Inform 6 doesn't have a similar system and can read and write files without any 
special headers. The only caveat is the edge case that if we would happen to have 
two Vorple projects sharing the same filesystem, the other written in Inform 6 and
the other in Inform 7, then some extra effort would be needed to handle the headers
that Inform 7 requires but Inform 6 doesn't understand. The easiest approach would be
to use Inform 7 to create the files and in Inform 6 ignore the header which is always
the first line of the file.


## Example



## Other considerations

* For simpicity's sake, Vorple's filesystem methods return just true or false to
  indicate if the operation succeeded or failed. If the actual reason for the
  failure is required, the BrowserFS methods can be used instead and they will
  throw an error that contains more details.
* The filesystem is not immediately available when the page loads. It's possible
  to call `vorple.file.init()` manually to get access to the filesystem before
  starting the game, see [the API](js-api.html#file) for details.
* `vorple.file.read()` and `vorple.file.write()` read and write text files.
  To handle binary files, BrowserFS methods need to be used. Vorple doesn't 
  directly support binary files.
* Vorple uses the synchronous versions of the filesystem methods and it's a good
  idea to do so even when accessing the filesystem directly. LocalStorage itself 
  is synchronous so there's no benefit to using the asynchronous versions.
* While the filesystem fully supports Unicode, Inform reads and writes files in
  ASCII Latin-1. Unicode characters outside Latin-1 won't be read correctly by Inform.
* LocalStorage is domain-specific. That is, all Vorple games played on example.com
  use the same virtual filesystem and share the same files. Conversely, games on 
  example1.com can't access the filesystem of games played on example2.com.
  Subdomains are considered different domains, so foo.example.com has a separate
  virtual filesystem from bar.example.com.
* LocalStorage persists until the user clears it manually in browser settings
  ("clear browsing data" or similar.) In private browsing / incognito mode
  localStorage is cleared after the user closes the last private browsing window.
* While BrowserFS supports file permissions (chmod), they don't make much sense
  in this context and are best left unused.