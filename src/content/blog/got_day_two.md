---
title: "Progress - Getting `git show` to work"
description: "Documenting my experience following along in James Coglan's book 'Building Git"
pubDate: "Sep 30, 2023"
heroImage: "/got.png" 
---
At this point of the book, I have gotten to a point where there is at least some resemblance of git in my program.

But this was not without struggle, I had joyfully been following along in the book believing that what I had written
was suitable. However, this was not the case. My first discovery was that DEFLATE was not the correct compression 
algorithm, when using `git show` I wound up with errors stating that the object files had the wrong header. This was
mostly due to my confusion as the Ruby function for zlib is `Zlib::Deflate.deflate(...)`.

Well, it is okay now I know. So I changed out DEFLATE for zlib
it should work... 

```fish
error: header for 089681426a6be45f85cdfe286386c7ed7c9c99b0 too long, exceeds 32 bytes
error: header for 089681426a6be45f85cdfe286386c7ed7c9c99b0 too long, exceeds 32 bytes
fatal: loose object 089681426a6be45f85cdfe286386c7ed7c9c99b0 (stored in .git/objects/08/9681426a6be45f85cdfe286386c7ed7c9c99b0) is corrupt
```

Huh? What is going on? If I decompress (inflate) the files they look right. If I use git and diff the inflated files from both programs they
are identical. I don't believe I could compare the actual bytes of the object files, cause as I discovered in the last post the bytes will
be different due to Go having its own zlib implementation. What gives? My implementation looks about right, I'll share what I had.
```go
func zlib_compress(str string) []byte {
	var buf bytes.Buffer
	w, err := zlib.NewWriterLevel(&buf, flate.BestSpeed)
	if err != nil {
		log.Fatal(err)
	}
	w.Write([]byte(str))
	w.Flush()
	defer w.Close()
	return buf.Bytes()
}
```

After hours of intense google-fu (really Kagi-fu), I stumbled across this [blog post](https://www.joeshaw.org/dont-defer-close-on-writable-files/).
Interesting, so it seems like something is going on with defer. Let's swap it out.
```go
func zlib_compress(str string) []byte {
	var buf bytes.Buffer
	w, err := zlib.NewWriterLevel(&buf, flate.BestSpeed)
	if err != nil {
		log.Fatal(err)
	}
	w.Write([]byte(str))
	w.Close()
	return buf.Bytes()
}
```

It works! I do not have the most solid understanding of why, but `git show` works! 

Other than that, things have been pretty smooth sailing. I have now implemented a parent line in the commits so that we can refer back to previous ones. 
As well as making sure two `got` programs on the same system don't overwrite eachother through a lock file.

[Source Code](https://github.com/Lukeisun/got) --- If you want to leave feedback, please feel free to email me!
