---
title: "Docker Debugging"
date: 2020-01-28T15:49:39-07:00
categories: ["Tips"]
tags: ["Cloud Native", "Containers"]

---
When I first started using Docker in 2015, it was like magic. It was when I first learned about containers and I remember staring at the `Dockerfile` blankly. Recently A coworker was asking me a bunch of questions about why their docker-compose setup wasn't working. I don't use docker-compose much at all, but the basic docker debugging I learned helped a lot. I am by no means an expect with containers or Docker, but I thought I would share some tips I learned. If you are looking for a container guru, check out [Jessie Frazelle](https://twitter.com/jessfraz).

For the examples here, I am working with this repo: https://github.com/guygrigsby/docker-debugging

## What is happening in there ?!

My first biggest frustration was not having _any_ idea what was happening inside the container I was building. It would fail and I would be none the wiser. The first thing I learned was to override the entrypoint to open a shell. I use Alpine for most of my builds so I have to use `sh`. The following example builds an image and then runs it, but overrides the entrypoint so I can poke around and see what is happening. 

```
 docker build -t someimage:0.0.2 .
.
.
.
Successfully built 9bd9431e2106
Successfully tagged someimage:0.0.2

 docker run -it --entrypoint=/bin/sh someimage:0.0.2
/ # ls
COMMIT_EDITMSG  Makefile        config          entrypoint.sh   home            info            main.go         objects         refs            sbin            tmp
Dockerfile      app             description     etc             hooks           lib             media           opt             root            srv             usr
HEAD            bin             dev             go.mod          index           logs            mnt             proc            run             sys             var
```


As I get more used to working with containers, I don't need it as much, but sometimes I just can't sort out what is going on and have to break in.

## Entrypoint or CMD ?!

When I started I didn't know that there _was_ a difference between entrypoint and cmd. I just picked one and it worked most of the time. As I did more reading, I came to learn that the entrypoint is the program and cmd is the command for that program. This can manifest in a number of ways and it's not required in most cases TBH. My favorite way to use them though, is with an entrypoint script so I can run various things in the container if I need to. This may not be a great idea for an image that has sensitive information in it, but for a lot of my use cases it's great. In the repo https://github.com/guygrigsby/docker-debugging, you'll notice that the `Dockerfile` entrypoint calls a script that is moved into the container. 
### Dockerfile

```
FROM guygrigsby/go-protoc AS build

RUN apk update && apk --no-cache add ca-certificates dep git && update-ca-certificates

RUN mkdir /go/src/app 
ADD . /go/src/app/
WORKDIR /go/src/app

RUN go build -o app main.go

FROM alpine AS runtime

RUN adduser -S -D -H -h /app appuser
USER appuser
FROM alpine
COPY --from=build /go/src/app/* /
COPY entrypoint.sh /
ENTRYPOINT ["/entrypoint.sh"]
CMD ["/app"]
```

### entrypoint.sh

```
#!/bin/sh
set -euo pipefail
if [ $1 = "--help" ]; then

        printf 'Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.'

        exit 1
fi

exec "$@"
```

You might notice that The script has a help flag defined, that does nothing, but the interesting part is at the end. We call `exec "$@"` which just passes your command to the container to run. Like I said, not sure secure, but it makes for a great way to build tools, rather than production services. The end result is that we can do something like this:
```
 docker build -t someimage:0.0.3 .
.
.
.
Successfully built 0228dfb01039
Successfully tagged someimage:0.0.3

docker run -it someimage:0.0.3 /bin/sh
/ # ls
COMMIT_EDITMSG  Makefile        config          entrypoint.sh   home            info            main.go         objects         refs            sbin            tmp
Dockerfile      app             description     etc             hooks           lib             media           opt             root            srv             usr
HEAD            bin             dev             go.mod          index           logs            mnt             proc            run             sys             var
```

This was you can pass any command into the container to have the image run it.  I use it most frequently for listing dirs and dumping environment variables, but you could use it to check anything that doesn't require root. I don't use root in my containers because people smarter than I have advised against it. It's weird the security practices that stick with a person. :)

In [the repo](https://github.com/guygrigsby/docker-debugging) that I posted for this article, you'll find these files and a Makefile that I use to start a new tool. It's my basic Go template because I am a Gopher.
