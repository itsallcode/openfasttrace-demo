
# OFT introduction video script (3 min)

> Voice track set as normal text. Instructions as quoted text.

OpenFastTrace is a requirement tracing suit. Requirements define your product. They tell everyone what your product can do and what it should not do.

Let's take a look at a sinple example for a flashlight app on a phone and define the main feature in a simple Markdown text file.


spec.md

```
`feat~flashlight~1`
Tha flashlight app lets uses the phone's flash LED as a flashlight.

Needs:req
```

This is the minimal form of a requirement defined in OFT's Markdown notation. It starts with a unique ID, consisting of an artifact type prefix, a name or number and a revision number. This followed by a description. And it demands coverage in another requirement prefixed with `req`.

> Show text highlight parts while explaining.

If you run a trace on this, it must be incomplete, since the coverage is missing.

```sh
oft trace spec.md
```

You get an error telling you that you forgot to cover your feature.

Let's fix this now.

```
`req~light-switch~1`
The app displays a button to switch the flash LED on and off.

Covers:

* `feat~flashlight~1`
```


This is a function requirement covering your feature. You can have as many of these as you want, but one is the minimum to satisfy the required coverage.

```sh
oft trace spec.md
```

Now your first trace is green. In the next video you will learn how to trace into code.

## Summary

OFT lets you define requirements on different levels and run a trace to see if they are covered as specified.
