phantomjs-prebuilt-that-works
==================

An NPM installer for [PhantomJS](http://phantomjs.org/), headless webkit with JS API.

__Automatically falls back to other CDNs if the main one is down or oversaturated. Hence the name of this fork.__

[![Build Status](https://travis-ci.org/Medium/phantomjs.svg?branch=master)](https://travis-ci.org/Medium/phantomjs)

Installing
-----------------------

As a library:

```shell
npm install phantomjs-prebuilt-that-works
```

As a cli command:

```shell
npm install -g phantomjs-prebuilt-that-works
```

CLI
---

```shell
phantomjs [phantom arguments]
```

Running via node
----------------

The package exports a `path` string that contains the path to the
phantomjs binary/executable.

Below is an example of using this package via node.

```javascript
var path = require('path')
var childProcess = require('child_process')
var phantomjs = require('phantomjs-prebuilt-that-works')
var binPath = phantomjs.path

var childArgs = [
  path.join(__dirname, 'phantomjs-script.js'),
  'some other argument (passed to phantomjs script)'
]

childProcess.execFile(binPath, childArgs, function(err, stdout, stderr) {
  // handle results
})

```

Or `exec()` method is also provided for convenience:

```javascript
var phantomjs = require('phantomjs-prebuilt-that-works')
var program = phantomjs.exec('phantomjs-script.js', 'arg1', 'arg2')
program.stdout.pipe(process.stdout)
program.stderr.pipe(process.stderr)
program.on('exit', code => {
  // do something on end
})
```

Note: [childProcess.spawn()](https://nodejs.org/api/child_process.html#child_process_child_process_spawn_command_args_options) is called inside `exec()`.

Running with WebDriver
----------------------

`run()` method detects when PhantomJS gets ready. It's handy to use with WebDriver (Selenium).

```javascript
var phantomjs = require('phantomjs-prebuilt-that-works')
var webdriverio = require('webdriverio')
var wdOpts = { desiredCapabilities: { browserName: 'phantomjs' } }

phantomjs.run('--webdriver=4444').then(program => {
  webdriverio.remote(wdOpts).init()
    .url('https://developer.mozilla.org/en-US/')
    .getTitle().then(title => {
      console.log(title) // 'Mozilla Developer Network'
      program.kill() // quits PhantomJS
    })
})
```

##### Using PhantomJS from disk

If you plan to install phantomjs many times on a single machine, you can
install the `phantomjs` binary on PATH. The installer will automatically detect
and use that for non-global installs.

A Note on PhantomJS
-------------------

PhantomJS is not a library for NodeJS.  It's a separate environment and code
written for node is unlikely to be compatible.  In particular PhantomJS does
not expose a Common JS package loader.

This is an _NPM wrapper_ and can be used to conveniently make Phantom available.
It is not a Node JS wrapper.

I have had reasonable experiences writing standalone Phantom scripts which I
then drive from within a node program by spawning phantom in a child process.

Read the PhantomJS FAQ for more details: http://phantomjs.org/faq.html

### Linux Note

An extra note on Linux usage, from the PhantomJS download page:

 > There is no requirement to install Qt, WebKit, or any other libraries. It
 > however still relies on Fontconfig (the package fontconfig or libfontconfig,
 > depending on the distribution).

Troubleshooting
---------------

##### Installation fails with `spawn ENOENT`

This is NPM's way of telling you that it was not able to start a process. It usually means:

- `node` is not on your PATH, or otherwise not correctly installed.
- `tar` is not on your PATH. This package expects `tar` on your PATH on Linux-based platforms.

Check your specific error message for more information.

##### Installation fails with `Error: EPERM` or `operation not permitted` or `permission denied`

This error means that NPM was not able to install phantomjs to the file system. There are three
major reasons why this could happen:

- You don't have write access to the installation directory.
- The permissions in the NPM cache got messed up, and you need to run `npm cache clean` to fix them.
- You have over-zealous anti-virus software installed, and it's blocking file system writes.

##### Installation fails with `Error: read ECONNRESET` or `Error: connect ETIMEDOUT`

This error means that something went wrong with your internet connection, and the installer
was not able to download the PhantomJS binary for your platform. Please try again.

##### I tried again, but I get `ECONNRESET` or `ETIMEDOUT` consistently.

Do you live in China, or a country with an authoritarian government? We've seen problems where
the GFW or local ISP blocks github, preventing the installer from downloading the binary.

Try visiting [the download page](https://bitbucket.org/ariya/phantomjs/downloads) manually.
If that page is blocked, you can try using a different CDN with the `PHANTOMJS_CDNURL`
env variable described above.

##### I am behind a corporate proxy that uses self-signed SSL certificates to intercept encrypted traffic.

You can tell NPM and the PhantomJS installer to skip validation of ssl keys with NPM's
[strict-ssl](https://www.npmjs.org/doc/misc/npm-config.html#strict-ssl) setting:

```
npm set strict-ssl false
```

WARNING: Turning off `strict-ssl` leaves you vulnerable to attackers reading
your encrypted traffic, so run this at your own risk!

##### I tried everything, but my network is b0rked. What do I do?

If you install PhantomJS manually, and put it on PATH, the installer will try to
use the manually-installed binaries.

##### I'm on Debian or Ubuntu, and the installer failed because it couldn't find `node`

Some Linux distros tried to rename `node` to `nodejs` due to a package
conflict. This is a non-portable change, and we do not try to support this. The
[official documentation](https://github.com/joyent/node/wiki/Installing-Node.js-via-package-manager#ubuntu-mint-elementary-os)
recommends that you run `apt-get install nodejs-legacy` to symlink `node` to `nodejs`
on those platforms, or many NodeJS programs won't work properly.

License
-------

Copyright 2012 [A Medium Corporation](http://medium.com/).

Copyright 2015 Julian Gruber.

Licensed under the Apache License, Version 2.0.
See the top-level file `LICENSE.txt` and
(http://www.apache.org/licenses/LICENSE-2.0).
