---
title: "Rust Proxy Part1"
date: 2020-02-09T11:49:52-07:00
---

## RUST

I am totally new to Rust, but after reading a recent article about [Discord is ditching Go for Rust](https://blog.discordapp.com/why-discord-is-switching-from-go-to-rust-a190bbca2b1f), I thought I would give it a shot. I love go. I am Gopher, but I am also a professional and need to be able to choose the right tool for the job. Discord is a solid platform with very smart engineers so I wanted to see what was up.

I decided to write a caching proxy server in Rust. I have been working recently on an open source caching proxy called [Trickster](https://github.com/Comcast/trickster) so that stuff is fresh in my mind. I have never written a line of Rust so this will be fun. 

I think this is going to be a long post so I will only be adding snippets here, but the whole repo can be accessed on [github](https://github.com/guygrigsby/roxyp). I can already tell that because Rust moves so fast, I will be comparing sources and figuring it out as I go. This post will have the overall idea, but for running code make sure to check the repo. I am using the nightly build and a `Dockerfile` based on Rust 1.40.0.

### Setup

Rust itself was a snap. So far, I am super impressed. The Rust toolchain is very well done and [easy to setup](https://www.rust-lang.org/learn/get-started). Basically, I just ran:
```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source $HOME/.cargo/env
cargo new roxyp
```
and added `source $HOME/.cargo/env` to my `.zshrc`. 

I use a Mac with vim as my editor for Go so I wanted to to the same for Rust. I got a couple vim plugins for highlighting and moving around, including support from [YCM](https://github.com/ycm-core/YouCompleteMe#rust-semantic-completion). I have not gotten it setup quite the way I like because jumping to definitions and documentation is not working, but I want to write code, not mess with my editor. I can look up the docs online.

So to build a proxy, I needed an http library. I went researching and at first found [Rocket](https://rocket.rs) and that looked pretty cool. [hyper](https://github.com/hyperium/hyper) also looked really good, but it's lower level.	Rocket looks like it's more about the framework, kind of the way I think about Rails or Spring. I think. The docs I was reading for hyper also made pretty heavy use of the Rust concurrency model, futures. That is _super_ interesting! Maybe I'll do it in both and write about my pros and cons

So at first go, it was a bit tough. They syntax is very different that what I am used to in Go or Java. I found a good [server example](https://rust-lang.github.io/async-book/01_getting_started/05_http_server_example.html) as my starting point. Between that and the [hyper guides](https://hyper.rs/guides) I was able to scrape together a functioning hello world server. The hardest part was reading in the value contained in the `PORT` envvar. It took me like two hours! Rust seems to use a lot of functional programming paradigm. I think it makes sense because it's side effect free and in Rust we have to manage our own memory. If we don't allocate a lot of objects, we don't have to free it. :) That's the initial thought. This is a bit lower than I am used to so I am going to learn a lot.

### Deploying

I wanted to use GCP Cloud Run to deploy the proxy because I like the idea of just running a container. It's more flexible that a Cloud function, but not as complicated as a pod in k8s. Sounds like a happy medium. We'll see how hard it is to connect the proxy to a Redis, once I get to that point. 

You have to create a project in GCP and use their docker build `gcloud` command to build your image and push it to the gcp container registry. There are a few special steps if you have never done that before. This [quickstart](https://cloud.google.com/run/docs/quickstarts/build-and-deploy) is straight forward. If you're new to Docker, it may take a little more time, but the commands worked as is for me and you can reference the [`Dockerfile`](https://github.com/guygrigsby/roxyp) in this project's repo. After I got a minimal Rust server, I had to ensure that the server took the envvar `PORT`, gcp provides that, and listens on it. Now I run
```bash
gcloud builds submit --tag gcr.io/PROJECT-ID/roxyp:0.0.1
gcloud run deploy --image gcr.io/PROJECT-ID/roxyp:0.0.1 --platform managed
```
and that's it. For those following along at home, this is where I created [tag v0.0.1](https://github.com/guygrigsby/roxyp/releases/tag/v0.0.1). 

It works!
```curl
curl https://roxyp.grigsby.dev
hello, world!%
```

You'll notice it's running at my domain. Let's go over how I did that. If you don't care about that, [skip this part](#Proxying Requests)

### Routing

Since I really have to plan and am just documenting a random experiment, my next step is to make this server available at https://roxyp.grigsby.dev. I use Google Domains for grigsby.dev, so all I needed to do was go to Cloud Run -> Damain Mappings and point it toward a subdomain. For other provides if you are using http you can create a CNAME DNS entry that points to the url for your cloud func. You could do that too if you have the cert for your domain and all subdomains. For TLS that's routing various places for various subdomains it's more complex and beyond the score of this example. 


Now that I am a little in, I think I will make this a multiparter. Otherwise it's going to take weeks. Stay tuned for part 2. Ship it!
