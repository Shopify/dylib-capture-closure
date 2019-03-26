# dylib-capture-closure

`dylib-capture-closure` is a utility to analyze a directory recursively for any macOS shared library
references pointing outside of that directory, and copy those libraries into the directory, changing
the references to point at those new copies.

This allows you to build a tarball or other package from a directory without it depending on shared
libraries from (e.g.) `/usr/local`.

## Usage

```bash
dylib-capture-closure <STAGE> <PREFIX>
```

`dylib-capture-closure` rewrites references under `$STAGE`, copying in to stage any libraries that
are not already within the `$STAGE` or `/usr/lib` paths, and rewriting those references. The
references are rewritten to the equivalent path under `$PREFIX`, to support staging changes in one
directory (`$STAGE`) and later installing them to the final (`$PREFIX`) directory via a package
installer, etc.

For example:

### Simple case (`$STAGE == $PREFIX`)

`dylib-capture-closure /opt/mything /opt/thing` will capture all of the shared library references
under `$STAGE` (`/opt/thing`), copy them into `$STAGE/lib.closure` (`/opt/thing/lib.closure`), and
change all the references under `$STAGE` (`/opt/thing`) to the corresponding paths under `$PREFIX`
(`/opt/thing/lib.closure`, in this case, the same paths).

### Staging case (`$STAGE != $PREFIX`)

`dylib-capture-closure ~/src/thing/build/stage /opt/thing` will capture all of the shared library
references under `$STAGE` (`~/src/thing/build/stage`), copy them into `$STAGE/lib.closure`
(`/opt/thing/lib.closure`), and change all the references under `$STAGE` (`~/src/thing/build/stage`)
to not the copies under `$STAGE`, but their counterparts under `$PREFIX` (`/opt/thing/lib.closure`).

### Visually...

Let's say we want to package `xdelta3` from homebrew:

```
$ otool -L /usr/local/bin/xdelta3
/usr/local/bin/xdelta3:
        /usr/lib/libSystem.B.dylib (compatibility version 1.0.0, current version 1238.0.0)
        /usr/local/opt/xz/lib/liblzma.5.dylib (compatibility version 8.0.0, current version 8.2.0)
```

We can copy this into a new directory...

```
$ mkdir /tmp/package
$ cp /usr/local/bin/xdelta3 /tmp/package
$ tree /tmp/package
/tmp/package
└── xdelta3

0 directories, 1 file
```

...then run `dylib-capture-closure`:

```
$ dylib-capture-closure /tmp/package /tmp/package
# <...output...>
$ tree /tmp/package
/tmp/package
├── lib.closure
│   └── liblzma.5.dylib
└── xdelta3

1 directory, 2 files
```

This copied in the library that `xdelta3` referenced, and we can see that the binary was changed to
point at it:

```
$ otool -L xdelta3
xdelta3:
        /usr/lib/libSystem.B.dylib (compatibility version 1.0.0, current version 1238.0.0)
        /tmp/package/lib.closure/liblzma.5.dylib (compatibility version 8.0.0, current version 8.2.0)
```

## Caveats

* This is not guaranteed to work perfectly (in fact, it's almost guaranteed to not work perfectly);
* It doesn't copy things from `/usr/lib`, since these should be the same on all systems;
* It needs XCode and probably some random other things installed.
* Multiple different referred libraries with the same basename will simply clobber each other when
  imported.
