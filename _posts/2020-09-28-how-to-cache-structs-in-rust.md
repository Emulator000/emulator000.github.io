---
authors: ["emulator000"]
date: "2020-09-28"
share: true
tags: [Cache, Item, Rust, Generics, Async, Await, Crate, Library, Tutorial]
title: "How-to cache structs in Rust"
layout: post
languageCode: "en-EN"
toc: true
---

## The cache

One of the best thing that you can do when you serve non in-memory data, obviously is the cache. In fact, if is used correctly, you can take the advantage of serving a resource fastly.

### Use cases

If you use an external database provider for data, such MySQL, SQLite, Redis or whatever else, and you have to query the service in order to get data, you can get the things worse in case of multiple queries per seconds or if you need to query for the same data each time repeatedly.

Supposing this piece of code using [SQLx](https://github.com/launchbadge/sqlx):
```rust
struct User {
    name: String,
    surname: String,
}

#[tokio::main]
async fn main() {
    let connection = SqlitePool::new("sqlite://dbfile.db")
        .await
        .unwrap_or_else(|_| panic!("Error connecting to Database"));

    let user_id = 1;
    let user = match sqlx::query_as::<_, User>("SELECT * FROM user WHERE id = {}")
        .bind(user_id)
        .fetch_one(&connection)
        .await
    {
        Ok(user) => Some(user),
        Err(_) => None,
    };
    
    if let Some(user) = user {
        println!("Username of User {} is: {}", user_id, user.name);
    }
}
```

### Getting help using a cache system

Imagine that you need to query the same user thousands of time, you get the same result each time (except if the user changed in the meantime) and probably you don't need this. So here is where the cache helps.

In Rust, you have many options to store those struct in-memory but can be very struggling because if you have many types of data, probably you have to store these data in different ad-hoc structs.

A basic approach to this problem is using an `HashMap` that in this case is specific for the `User`, something like `HashMap<i64, User>` and you have to implement the same approach for every data type.

So the problem is this, and here is where the [simple-cache](https://crates.io/crates/simple-cache) library could help. In fact, this library is an abstraction of this approach using `HashMap`s.

Here an example of the cache `User` struct using the `CacheItem` trait implemented for the struct:
```rust
use simple_cache::{Cache, CacheItem};

struct Data {
    cache: Cache<i64>,
}

struct User {
    name: String,
    surname: String,
}

impl CacheItem for User {}

#[tokio::main]
async fn main() {
    let data = Data {
        cache: Cache::new(),
    };

    let user_id = 1;
    let cached_user = data.cache.get::<User, _>(user_id).await;

    if let Some(wrapped_user) = cached_user { // User found in the cache
        if let Some(user) = wrapped_user { // User was also found in the DB, so we have the original struct
            println!("Username of User {} is: {}", user_id, user.name);
        } else {
            println!("User {} not found in the DB", user_id);
        }
    } else { // User not found in the cache, query the DB
        match sqlx::query_as::<_, User>("SELECT * FROM user WHERE id = {}")
            .bind(user_id)
            .fetch_one(&connection)
            .await
        {
            Ok(user) => {
                println!("Username of User {} is: {}", user_id, user.name);
            },
            Err(_) => {
                println!("User {} not found in the cache", user_id);
            },
        }
    }
}
```

Imagine this code in a web service or in a high demand service, the DB will say "thank you", and you save a lot of resources and of course, the data will be served very fastly as we have stored it in the memory.

#### Some library explanation

If you noticed, the `get()` functions returns two `Option` wrapped, something like `Option<Option<T>>` that because we want to store the `None` value of a `key` too in addition of the `None` value of a non-stored struct, if a user doesn't exist, why query the DB again and again?

If we store the `None` value, we avoid this too and if the user will be available, we can update the user later.

### Updating the cache

Don't forget to update the cache value if the original user will be modified or if you delete it or update it, probably you want to update the cache too.

### Conclusions

Cache is surely a good thing but must be used correctly in order to avoid outdated data serving, so, always update the values!