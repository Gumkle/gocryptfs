GoCryptFS [![Build Status](https://travis-ci.org/rfjakob/gocryptfs.svg?branch=master)](https://travis-ci.org/rfjakob/gocryptfs) ![Release Status](https://img.shields.io/badge/status-beta-yellow.svg?style=flat)
==============
An encrypted overlay filesystem written in Go.

gocryptfs is built on top the excellent
[go-fuse](https://github.com/hanwen/go-fuse) FUSE library and its
LoopbackFileSystem API.

This project was inspired by [EncFS](https://github.com/vgough/encfs)
and strives to fix its security issues (see EncFS tickets 9, 13, 14, 16).
For details on the security of gocryptfs see the
[SECURITY.md](SECURITY.md) document.

Current Status
--------------

Beta. You are advised to keep a backup of your data outside of gocryptfs, in
addition to storing the *master key* in a safe place (the master key is printed
when mounting).

That said, I am dogfooding on gocryptfs for some time now. In fact, all gocryptfs
development happens inside a mounted gocryptfs filesystem, with no issues so far.

Only Linux is supported at the moment. Help wanted for a Mac OS X port.

Testing
-------

gocryptfs comes with is own test suite, run it using `./test.bash`.

In addition, i have ported `xfstests` to FUSE, the result is the
[fuse-xfstests](https://github.com/rfjakob/fuse-xfstests) project. gocryptfs
passes the "generic" tests with one exception, results:  [XFSTESTS.md](XFSTESTS.md)

A lot of work has gone into this. The testing has found bugs in gocryptfs
as well as in go-fuse.

The one exception is generic/035, see [go-fuse issue 55](https://github.com/hanwen/go-fuse/issues/55)
for details. While this is a POSIX violation, I do not see any real-world impact.

Install
-------

	$ go get github.com/rfjakob/gocryptfs

Use
---

Quickstart:

	$ mkdir cipher plain
	$ $GOPATH/bin/gocryptfs --init cipher
	  [...]
	$ $GOPATH/bin/gocryptfs cipher plain
	  [...]
	$ echo test > plain/test.txt
	$ ls -l cipher
	  total 8
	  -rw-rw-r--. 1 user  user   33  7. Okt 23:23 0ao8Hyyf1A-A88sfNvkUxA==
	  -rw-rw-r--. 1 user  user  233  7. Okt 23:23 gocryptfs.conf
	$ fusermount -u plain

See [MANPAGE.md](MANPAGE.md) for a description of available options. If you already
have gocryptfs installed, run `./MANPAGE-render.bash` to bring up the rendered manpage in
the pager (requires pandoc).

Storage Overhead
----------------

* Empty files take 0 bytes on disk
* 18 byte file header for non-empty files (2 bytes version, 16 bytes random file id)
* 28 bytes of storage overhead per 4kB block (12 byte nonce, 16 bytes auth tag)

Performance
-----------

gocryptfs uses openssl through
[spacemonkeygo/openssl](https://github.com/spacemonkeygo/openssl)
for a 3x speedup compared to Go's builtin AES-GCM implementation (see
[go-vs-openssl.md](openssl_benchmark/go-vs-openssl.md) for details).

Run `./benchmark.bash` to run the benchmarks.

The output should look like this:

	./benchmark.bash
	gocryptfs v0.3.1-30-gd69e0df-dirty; on-disk format 2
	PASS
	BenchmarkStreamWrite-2	     100	  12246070 ns/op	  85.63 MB/s
	BenchmarkStreamRead-2 	     200	   9125990 ns/op	 114.90 MB/s
	BenchmarkCreate0B-2   	   10000	    101284 ns/op
	BenchmarkCreate1B-2   	   10000	    178356 ns/op	   0.01 MB/s
	BenchmarkCreate100B-2 	    5000	    361014 ns/op	   0.28 MB/s
	BenchmarkCreate4kB-2  	    5000	    375035 ns/op	  10.92 MB/s
	BenchmarkCreate10kB-2 	    3000	    491071 ns/op	  20.85 MB/s
	ok  	github.com/rfjakob/gocryptfs/integration_tests	17.216s

Changelog
---------

v0.4 (in progress)
* Add `-plaintextnames` command line option
 * Can only be used in conjunction with `-init` and disables filename encryption
   (added on user request)
* Add `FeatureFlags` config file paramter
 * This is a config format change, hence the on-disk format is incremented
 * Used for ext4-style filesystem feature flags. This should help avoid future
   format changes.
* On-disk format 2

v0.3
* Add file header that contains a random id to authenticate blocks
 * This is an on-disk-format change
* On-disk format 1

v0.2
* Replace bash daemonization wrapper with native Go implementation
* Better user feedback on mount failures

v0.1
* First release
* On-disk format 0

See https://github.com/rfjakob/gocryptfs/releases for the release dates
and associated tags.
