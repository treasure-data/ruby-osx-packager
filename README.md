Building ruby pkg for Mac

## prepare

clone ruby-build submodule to install ruby

```sh
$ git submodule init
$ git submodule update
```

## Task

### default

Installing ruby into /usr/local/td/ruby directory and build ruby pkg

```sh
$ rake
```

### clean

```sh
$ rake clean
```

## NOTE

This repository includes patch applied readline and openssl to avoid dependent library version issue.

If you update these libraries, then see homebrew formula to apply the patch.

https://github.com/mxcl/homebrew/blob/master/Library/Formula/readline.rb
https://github.com/mxcl/homebrew/blob/master/Library/Formula/openssl.rb
