---
title: "Rust Proxy Part 2"
date: 2020-02-15T13:43:15-07:00
draft: false
---

## Rust is Different

So far I have created a super minimal proxy with a fixed upstream. No caching yet and I am pretty sure headers are just ignored. :). The final commit that corresponds to this post is [here](https://github.com/guygrigsby/roxyp/tree/6e74480ab345d9490abd70d889c57f0943d67621). It's been a while because learning Rust is harder than I thought. I would like to jump into that. 

Whew! I was unprepared for the Rust. It's a pretty steep learning curve. It could be that maybe I am slow. Most likely a little of column A and a little from column B. :) Using Rust has really required me to approach the problem from a different perspective. As I have noted, I write Go for my day job. Before that I wrote Java. I have very little xp in languages that aren't garbage collected. I worked a bit in C and C++, but never at length. I knew this coming in and assumed that I would miss some `free`s hered there or something like that, but this is a whole new world. To be memory "safe" and not garbage collected, Rust has been built with the concept of "ownership." 
This is actually pretty cool and seems super effective, but it requires some getting used to For example, I have not had to explicitly free any memory allocation, yet. I have, however, spend quite a lot of time struggling with [ownership](https://doc.rust-lang.org/book/ch04-00-understanding-ownership.html)

## Ownership

In Rust, every allocation on the heap has an owner and once that owner goes out of scope, the memory is freed. So for example a `String` is created to pass to a constructor. That struct now owns the `String` and when it goes out of scope, the the owned `String` is also freed. Great idea, right? right! But it get's complex. So what happens if you want to log that `String` later? Well you may have to "borrow" the string. Take a look at [this example from the Rust book](https://play.rust-lang.org/?version=nightly&mode=debug&edition=2018&gist=64248cea44bfcfa4c26bab0f76385575)(If you're like me, you'll spend a lot of time reading it):

```

fn main() {
    let s1 = String::from("hello");
    let struc = T { s: s1 };

    println!("{}, world!", s1);
}
struct T {
    s: String,
}
```
This code looks super simple, but it will not compile.
```
error[E0382]: borrow of moved value: `s1`
 --> src/main.rs:5:28
  |
2 |     let s1 = String::from("hello");
  |         -- move occurs because `s1` has type `std::string::String`, which does not implement the `Copy` trait
3 |     let _struc = T { s: s1 };
  |                         -- value moved here
4 | 
5 |     println!("{}, world!", s1);
  |                            ^^ value borrowed here after move
```

So what's happening here, I _think_, is that the struct is taking ownership of the String and so that String cannot be used any longer in the main scope. If we want to be able to use it in main, the struct needs to "borrow" it so we can maintainer ownership. Then we get into references and their lifetimes. It's actually pretty complex so I won't go into it, but please check out the [Rust book](https://doc.rust-lang.org/book/title-page.html). It's way smarter than I am.

## From Hello World to Proxy

Ok so let talk about the toy proxy I am writing. The [code](https://github.com/guygrigsby/roxyp/tree/6e74480ab345d9490abd70d889c57f0943d67621) is pretty short actually. It's just taken some time to learn a bit of Rust. Now that I look at it, it's a sad little amount of code. There is basically a [proxy struct](https://github.com/guygrigsby/roxyp/blob/6e74480ab345d9490abd70d889c57f0943d67621/src/proxy/mod.rs) and a [main](https://github.com/guygrigsby/roxyp/blob/6e74480ab345d9490abd70d889c57f0943d67621/src/main.rs). I have been testing it with a simple [go echo server](https://github.com/guygrigsby/echo) and it works for that single contrived use case :). I used hyper http library with an async library called tokio. In retrospect, I should have started syncronously and then added that later. I am so used to writing concurrent code that it just made sense to me to write it that way. Turns out that the concurrency primitives in go really make it easy to write concurrent code. I thought I knew that, but this exercise really pushed that idea home. Anyhow, Rust really likes to use closures and I thought that was super cool. From what I can tell, the closures make it easy to allocate memory within the owners scope so that you don't have to change ownership, called a `move`. This [line](https://github.com/guygrigsby/roxyp/blob/6e74480ab345d9490abd70d889c57f0943d67621/src/proxy/mod.rs#L52) kind of illustrates what I mean:
```
         parts.uri = Uri::builder()
            .scheme(parts.uri.scheme().unwrap_or_else(|| &Scheme::HTTP).clone())
            .authority(self.upstream.as_str())
            .path_and_query(
                parts
                    .uri
                    .path_and_query()
                    .expect("Cannot get path and query from original request")
                    .clone(),
            )
            .build()
            .expect("Failed to build upstream URI");
```
It looks weird to me. Builder patterns are not big in go, but I have used them in Java. What was strange to me was the seeminly never ending chain of method calls. You can see that there is a closure in there too. That's the `|| &Scheme::HTTP` part. All of the parts of this URI are arranged in various scopes inside the builder. What's more is that a few are being cloned so the new owner is the URI builder. What I didn't understand at first that was kind of an "aha" moment for me was that the `unwrap` method `moves` ownership`. So in the scheme, when we `unwrap` the result from `parts.uri.scheme()` the new owner becomes the Uri builder.

I feel like I have written a novella and barely scratched the surface of what I was able to learn. The next post, I will do a lot less work and write more about it. Rust is super cool. 

### Aside (Vim Frustrations)

I use [YouCompleteMe](https://github.com/ycm-core/YouCompleteMe) for completion, which has always been great for Go so I decided to use it for Rust too. What I didn't realize is [vim-go](https://github.com/fatih/vim-go), was doing _a lot_ of work too. Trying to get autocompletion, which I think is _super_ helpful for learning a new language, has fairly easy, but docs and `go-to-def` has been harder. Still working on it, will update. I finally installed [gutentags](https://github.com/ludovicchabant/vim-gutentags) and that seemed to sort out all my issues. I was using tags with go, but I forgot that I had go-tags installed.

From what I gathered, many folks use VSCode for Rust and like it a lot. Check out their [tools](https://www.rust-lang.org/tools) section.




