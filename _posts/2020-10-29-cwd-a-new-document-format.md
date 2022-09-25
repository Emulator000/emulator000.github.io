---
authors: ["emulator000"]
date: "2020-10-29"
share: true
tags: [cwd, pdf, compact, web, document, rust, c, c++, javascript, html, css, ffi, forms, blink, google]
title: "CWD: A new document format"
layout: post
languageCode: "en-EN"
toc: true
---

Recently, I'm working on a very ambitious project: a new document format, essentially is a PDF replacer. I created an under construction home page [here](https://www.compactwebdocument.com/){:target="_blank"}.

CWD stands for Compact Web Document and is a new powerful and multimedia document file that allows rich contents such images, videos, audios, live maps and of course, text.

![Screenshot](https://www.compactwebdocument.com/images/app.png)

With CWD you can easily build your own document with HTML, CSS and JavaScript and the resulting file is a high standard and compressed portable document, readable with a CWD Reader.

Why do this? Adobe PDF is still here, and we don't need another document format! Well, I think that is not true, PDF have a lot of limitations, I suggest the reading of [this](https://wiki.c2.com/?PdfSucks){:target="_blank"} article, it's very complete and exhaustive.

Manipulating a PDF is the absolute pain ever, you have to read carefully the PDF reference in order to do the things in the right way and this is not always true, every PDF Reader will render in a very different and specific way and the standard is now obsolete.

So here became the idea, we have HTML, CSS, JS and so many technologies that are standard and more extensible, also, a web page can hold ANY type of content today, even a PDF, you can even put a PDF inside a CWD file if you prefer!

## The format

You can open a CWD file as a ZIP file and read and extract the raw content, the result will be a set of file that a CWD Reader will merge and use for the document rendering.

You can think as a web page embedded on a portable file with all the benefits of a web page but the portability of a straightforward document.

Essentially, a CWD file, is a ZIP file with all HTML directives and resources included, usually a CWD Reader will read it in memory (RAM) and will render like a browser does, in fact, the official CWD Reader renders the page with the Blink engine.

## Under the hood

For this project I used many technologies, main are:

- Rust: for the CWD main core library
- C++: for the Reader
- C: for the libraries FFI bindings
- JavaScript: for the dynamic UI
- HTML: for pages content
- CSS: for pages styling

Initially I started to write a Rust library that is used for the main CWD core, it is used by any implemented Reader and holds the system close things, then I started writing the CWD format reader and I used WebKit but, for MANY limitations I switched to [CEF](https://en.wikipedia.org/wiki/Chromium_Embedded_Framework){:target="_blank"} that uses [Blink](https://en.wikipedia.org/wiki/Blink_(browser_engine)){:target="_blank"}, the render engine used by Google Chrome.

## How we can create and edit documents?

At moment there isn't a tool to create and edit a CWD file, but a CWD Editor is in development.

In order to create a CWD file from scratch, you can simply create a ZIP file and rename it as ".cwd", the archive should include these base files:

- `cwd.html` that is the main content of the CWD file, the HTML shouldn't contain any <html> tag
    - The file should be structured like:

```html
<div class="page" size="A4" layout="portrait">
    <h1>Sample title</h1>
    <p>Sample text.</p>
</div>
```

- `cwd.toml` that contains some document meta-information:
    - The file should be structured like:

```toml
version = "1.0"
title = "A test document"
```

- `resources/cwd.css` that is the main document stylesheet with any custom rule
- You can embed many resources as needed, just put files inside the `resources` directory and use in the `cwd.html` or `cwd.css` with the `cwd://resources/file.ext` url format.

## Sources

All the source codes will be available for the public, currently are hosted on GitHub. Once all the projects like the mobile app, and the site will be ready, all will be open source.

## Considerations

This a very new and experimental format so keep in mind that is not a final product.
