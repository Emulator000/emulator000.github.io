---
authors: ["emulator000"]
date: "2020-09-29"
share: true
tags: [rust, pdf, manipulation, library, tutorial]
title: "PDF manipulation with Rust and considerations"
layout: post
languageCode: "en-EN"
toc: true
---

The **Portable Document Format (PDF)** is a file format developed by Adobe in 1993 to present documents, including text formatting and images, in a manner independent of application software, hardware, and operating systems.

**Rust** is a multi-paradigm programming language focused on performance and safety, especially safe concurrency. Rust is syntactically similar to C++, and provides memory safety without using garbage collection.

## How can I manipulate PDF programmatically in Rust?
Let's introduce [lopdf](https://crates.io/crates/lopdf){:target="_blank"} crate, a Rust library that let you to manipulate PDF file following the [reference](https://www.adobe.com/content/dam/acom/en/devnet/pdf/pdfs/PDF32000_2008.pdf) by Adobe.

I recently contributed to this project introducing features like:
- Possibility to merge PDF documents in one single PDF file
- Bugfixes, refactoring and better image manipulation

With this library you can basically do anything with the document elements, altering the `Root`, the `Catalog` and all other kinds of `Contents`, it's a low level library, and it's perfect for building some wrapper based on it.

An example of a library built on top of **lopdf** is the [pdf_form](https://crates.io/crates/pdf_form){:target="_blank"} crate, this library in fact, let you to manipulate the `AcroForm` (the PDF form fields) easily and without struggling about discovering elements inside the page tree.

## Load and modify a PDF
The **lopdf** library loads the PDF in memory, quoting what the author says about that:

> Normally a PDF document won't be very large, ranging form tens of KB to hundreds of MB. Memory size is not a bottle neck for today's computer. By keep the whole document in memory, stream length can be pre-calculated, no need to use a reference object for the Length entry, the resulting PDF file is smaller for distribution and faster for PDF consumers to process.

Here an example of code for text replacing:
```rust
use lopdf::Document;

fn main() {
    let mut document = Document::load("example.pdf")?;
    document.version = "1.4".to_string();
    
    document.replace_text(1, "Hello World!", "Modified text!");
    
    document.save("modified.pdf")?;
}
```

With this piece of code, the library will replace all object elements that have that string and will save it as a new file.

## Merging more PDF documents

This is a particular example code that I personally written for the repository and shows step by step how the file merging done:
```rust
use std::collections::BTreeMap;

use lopdf::{Document, Object, ObjectId};

fn main() {
    // Generate a stack of Documents to merge
    let documents = vec![
        Document::load("example.pdf").unwrap(),
        Document::load("example2.pdf").unwrap(),
        Document::load("example3.pdf").unwrap(),
        Document::load("example4.pdf").unwrap(),
    ];

    // Define a starting max_id (will be used as start index for object_ids)
    let mut max_id = 1;

    // Collect all Documents Objects grouped by a map
    let mut documents_pages = BTreeMap::new();
    let mut documents_objects = BTreeMap::new();

    for mut document in documents {
        document.renumber_objects_with(max_id);

        max_id = document.max_id + 1;

        documents_pages.extend(
            document
                    .get_pages()
                    .into_iter()
                    .map(|(_, object_id)| {
                        (
                            object_id,
                            document.get_object(object_id).unwrap().to_owned(),
                        )
                    })
                    .collect::<BTreeMap<ObjectId, Object>>(),
        );
        documents_objects.extend(document.objects);
    }

    // Initialize a new empty document
    let mut document = Document::with_version("1.5");

    // Catalog and Pages are mandatory
    let mut catalog_object: Option<(ObjectId, Object)> = None;
    let mut pages_object: Option<(ObjectId, Object)> = None;

    // Process all objects except "Page" type
    for (object_id, object) in documents_objects.iter() {
        // We have to ignore "Page" (as are processed later), "Outlines" and "Outline" objects
        // All other objects should be collected and inserted into the main Document
        match object.type_name().unwrap_or("") {
            "Catalog" => {
                // Collect a first "Catalog" object and use it for the future "Pages"
                catalog_object = Some((
                    if let Some((id, _)) = catalog_object {
                        id
                    } else {
                        *object_id
                    },
                    object.clone(),
                ));
            }
            "Pages" => {
                // Collect and update a first "Pages" object and use it for the future "Catalog"
                // We have also to merge all dictionaries of the old and the new "Pages" object
                if let Ok(dictionary) = object.as_dict() {
                    let mut dictionary = dictionary.clone();
                    if let Some((_, ref object)) = pages_object {
                        if let Ok(old_dictionary) = object.as_dict() {
                            dictionary.extend(old_dictionary);
                        }
                    }

                    pages_object = Some((
                        if let Some((id, _)) = pages_object {
                            id
                        } else {
                            *object_id
                        },
                        Object::Dictionary(dictionary),
                    ));
                }
            }
            "Page" => {}     // Ignored, processed later and separately
            "Outlines" => {} // Ignored, not supported yet
            "Outline" => {}  // Ignored, not supported yet
            _ => {
                document.objects.insert(*object_id, object.clone());
            }
        }
    }

    // If no "Pages" found abort
    if pages_object.is_none() {
        println!("Pages root not found.");

        return;
    }

    // Iter over all "Page" and collect with the parent "Pages" created before
    for (object_id, object) in documents_pages.iter() {
        if let Ok(dictionary) = object.as_dict() {
            let mut dictionary = dictionary.clone();
            dictionary.set("Parent", pages_object.as_ref().unwrap().0);

            document
                    .objects
                    .insert(*object_id, Object::Dictionary(dictionary));
        }
    }

    // If no "Catalog" found abort
    if catalog_object.is_none() {
        println!("Catalog root not found.");

        return;
    }

    let catalog_object = catalog_object.unwrap();
    let pages_object = pages_object.unwrap();

    // Build a new "Pages" with updated fields
    if let Ok(dictionary) = pages_object.1.as_dict() {
        let mut dictionary = dictionary.clone();

        // Set new pages count
        dictionary.set("Count", documents_pages.len() as u32);

        // Set new "Kids" list (collected from documents pages) for "Pages"
        dictionary.set(
            "Kids",
            documents_pages
                    .into_iter()
                    .map(|(object_id, _)| Object::Reference(object_id))
                    .collect::<Vec<_>>(),
        );

        document
                .objects
                .insert(pages_object.0, Object::Dictionary(dictionary));
    }

    // Build a new "Catalog" with updated fields
    if let Ok(dictionary) = catalog_object.1.as_dict() {
        let mut dictionary = dictionary.clone();
        dictionary.set("Pages", pages_object.0);
        dictionary.remove(b"Outlines"); // Outlines not supported in merged PDFs

        document
                .objects
                .insert(catalog_object.0, Object::Dictionary(dictionary));
    }

    document.trailer.set("Root", catalog_object.0);

    // Update the max internal ID as wasn't updated before due to direct objects insertion
    document.max_id = document.objects.len() as u32;

    // Reorder all new Document objects
    document.renumber_objects();
    document.compress();

    // Save the merged PDF
    document.save("merged.pdf").unwrap();
}
```

The important thing here are the single object ids. In fact, every PDF's object should have a unique id in order to work, it's something like a key of a hashmap. The render will fail if more than object shares the same id.

## Filling form fields

As said earlier, another crate that I contributed the **pdf_form** library, helps for PDF's fields filling. Let's see an example of code here:
```rust
use pdf_form::Form;

fn main() {
    // Load the PDF into a Form from a path
    let mut form = Form::load("path/to/pdf").unwrap();
    
    // Set the first field (you can use the "form.get_all_types()" function in order to iter all fields) text
    form.set_text(0, String::from("filling the field"));
    
    // Save the new document
    form.save("new.pdf");
}
```

At moment, all form fields are supported, such: `Text`, `Button`, `Radio`, `CheckBox`, `ListBox` and `ComboBox`.

## Insert an image
You can add an image using the **lopdf** feature library called `embed_image`, something like:
```rust
use lopdf::Document;
use lopdf::xobject;

fn main() {
    let mut document = Document::load("example.pdf")?;
    document.version = "1.4".to_string();
    
    // If the stream is loaded correctly
    if let Ok(stream) = xobject::image("image.png") {
        // we need a "page_id", the position coordinates and the size of the image
        document.insert_image(page_id, stream, (x, y), (width, height));
    }
    
    document.save("image.pdf")?;
}
```

## Considerations

Of course, the library is not 100% complete, it's a low level PDF library and more crates like **pdf_form** are needed, I can imagine a crate of a certain feature of PDF manipulation needed.

I will continue to contribute to this library and try to port in the Rust world, a comfortable PDF manipulation, like other languages already does.