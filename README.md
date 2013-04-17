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
