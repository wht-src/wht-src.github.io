---
title: "Backend Made Easy"
date: 2025-12-26
tags: 
  - posts
layout: post.njk
---

Over the years, I've seen a lot of new developers telling me that they 
don't want to do server backend. The reason? It's complex. It's only 
natural. Look at popular frameworks like ASP.NET, the starting point 
involves tons of files that are frankly intimidating. Even beloved 
frameworks like Phoenix generates tons of files. Of course, I'm not saying 
that it's necessarily a bad thing to have tons of files pre-generated for 
a project. All of these files mean something, but to a beginner? Maximized
intimidation.

I'm sure these frameworks work nicely for large companies with thousands of 
users every second, but have you ever considered that a beginner doesn't really 
need all that extra addons, and they only serve to intimidate? We can do 
better than that.

This is where Clojure comes into play. Clojure likes to be simple, and it shows that
in how the community does things. Instead of making a raw framework and throwing 
everything at the programmer at once, we just stack libaries on demand. This way,
you know exactly what each part of your codebase does, allowing you to make the 
website of your dreams without setting your hair on fire.

This guide serves to get your feet wet in making web backends. I assume you already 
know a bit of programming. You should also know the basics of Clojure. If not, look at 
[this](https://clojure.org/guides/learn/clojure). Don't worry, there are only 6 chapters 
and each chapter is very short. After all, Clojure is a very simple language.

You will also need to install Clojure, specifics [here](https://clojure.org/guides/install_clojure).

# The overview

For the unaware, writing a server backend seems complex. But its actually quite simple.
Unless you are writing the entire thing in a low level language like C, the modern 
abstractions makes your backend looks like the following:

> Receive request -> Process request -> Generate Response -> Send it back to the client

It's quite simple, isn't it. 

In this guide, we will be making a really really simple todo list backend.

# Making the project

Let's first get the skeleton of the project made.

Create an empty directory. Let's name it backend. This name does 
not matter. Cd into it. We will call this directory the root 
directory.

```bash
mkdir backend
cd backend
```

Inside the directory, make another directory named `src`, after that
make a file named `deps.edn` and paste in the following:
```clojure
{:paths ["src"]
 :deps {org.clojure/clojure {:mvn/version "1.12.4"}}}
```

This tells Clojure that it uses the `1.12.4` version of the language, 
and Clojure should scan the path `src` for source code.

In `src` directory, make another directory named `backend`. This will 
be the name of the project. In said directory, make a file named 
`core.clj`. It will be the entry point to our program. Paste in the following:
```clojure
(ns backend.core)

(defn -main []
  (println "hello world"))
```

Now run your source code. `cd` back to the root directory. Run the following 
command to execute your program. Next time when I ask you to run the program,
do the following:
```bash
clj -M -m backend.core

# should print hello world
```

The preparation is done. 

## Receiving and sending requests

Recall the overview. To deal make a backend, we first need to receive requests.
How do we do that? 

Edit `deps.edn`, change the file so that it looks like the following:
```clojure
{:paths ["src"]
   :deps {org.clojure/clojure {:mvn/version "1.12.4"}
          http-kit/http-kit {:mvn/version "2.8.1"}}}

```

This will tell Clojure we want to install the `http-kit` library of version `2.8.1`.
This library contains code that allows us to recieve request and send responses 
back to the client.

Now in `core.clj`, change it to the following:
```clojure
(ns backend.core
  (:require [org.httpkit.server :as hk-server])) ; tell clojure we want to use httpkit server

(defn app [req] 
  {:status  200
   :body    "hello world"})

; the status field of a http request is quite simple, different numbers corresponds to 
; different statuses, 200 means OK and famously 404 means not found

(defn -main []
  (hk-server/run-server app {:port 8000})) ; we pass the function app to the run-server command 
  ; this input of the function will be the user's request formatted into a map,
  ; the return of the function will be send back to the user
```

Run the program and try it out! In another terminal, run:
```bash
curl localhost:8000
```

It should output `hello world`. 

As you can see, in just 9 lines of code, you have already made something that 
can send stuff to the client. Quite simple, isn't it.

## Mapping out the requests


