---
theme: seriph
background: https://raw.githubusercontent.com/engineer-man/piston/master/var/docs/images/piston.svg
class: text-center
highlighter: shiki
lineNumbers: false
info: |
  ## Slidev Starter Template
  Presentation slides for developers.

  Learn more at [Sli.dev](https://sli.dev)
drawings:
  persist: false
transition: slide-left
title: Code execution engine
---

# The power of Piston

Code execution engine

---
transition: slide-left
---

<!--
Today i will be talk about code execution engine and the package Piston
-->

# What is Code execution engine

<br>

### Code execution engine (CEE) is a service that interprets and executes programming code written in various programming languages.

<br>

Some key aspects :

- 🛡️ **Isolation**
- 🎛️🔢 **Version Control**
- 📚🔗 **APIs and Integrations**
- 🛠️ **Language Support**
- 🔒 **Security**
- 💾 **Resource Management**
- 🏞️ **Execution Environment**

<!--
To describe Piston we will start by describing what is a code execution engine. So a code execution engine is a service that interprets and executes code written in any programming language.

The main task is to provide an environment in which code can be executed safely and efficiently.
The primary purpose of a Code Execution Engine is to run user code securely and with some kind of reliability.

As you can see we have here some key aspects that a code execution engine should have.

- Isolation : providing isolation will ensure that the executed code cannot harm the system or other running processes
- Version Control: Some code execution engine might support multiple programming language versions.
- APIs and Integrations: Allow the use of this feature in other software like online code editors.
- Language Support: allow multiple programming languages (compiled, functional and so on)
- Security: This one is imported, if not the most imported one. The engine needs to be secure while running code, because sometimes the code can be extraordinarily hostile and harmful.
- Resource Management: Also an imported aspect, because running code can take a lot of resources, and if it is bad code it can be even worse. So we need a good way to manage the system resources
- Execution Environment: We need a runtime environment for each code execution, which includes loading necessary libraries, providing standard input/output stream also handling errors on runtime.
-->

---
transition: slide-left
layout: image-right
image: https://raw.githubusercontent.com/engineer-man/piston/master/var/docs/images/piston.svg
---


# What is Piston?

code execution engine

<br>

## Piston is a high performance code execution engine that can be used to run source code on the web.

<br>
<br>

You can see more about Piston [here](https://github.com/engineer-man/piston)

<!--
This is where Piston comes in

So Piston is a high performance code execution engine that can be used to run any source code on the web.
-->

---
transition: slide-left
layout: image-right
image: https://external-content.duckduckgo.com/iu/?u=https%3A%2F%2Fi.pinimg.com%2Foriginals%2Fb7%2F9f%2F15%2Fb79f155970b3605295da4ba0ec2267bb.jpg&f=1&nofb=1&ipt=f02eeb88066434cec3d4c0baef28a2245178a22424f7fd6f0552a0e46565c0a2&ipo=images
---

# Piston

Code execution engine, design goals & some bumps

## Secure & Fast

<br>
<br>

### The problem with the code execution engine is that it needs to take in any code and execute it fast, near native speeds, and securely. Sometimes that code can be extraordinary hostile.

<!--
The main goal of a code execution engine is to run the code fast and securely.

So the design should be maid in a way that even if someone gave it hostile code it is able to resist any of the negative effects that the code might have.
-->

---
transition: slide-left
---

# Examples of hostile code

<br>

## Delete the entire system 😱

<br>

```sh
rm -rf /
```

<br>
<br>

## Fork bomb 💥

<br>

```sh
:(){ :|: &}:;
```

```python
while True: os.fork()
```

<!--
For instance, if someone wanted to run bash and gave it code like this one. (which removes everything from the system and the other flood the system with processes)

The code engine needed to be configured in such a way that won't delete the entire system or take all the resources of the system.
-->

---
transition: slide-left
layout: image-right
image: https://miro.medium.com/v2/resize:fit:400/1*KWeXamv1oqIvzKLlPhn-rA.png
---

# Make it secure

Docker

### Docker is a fantastic fit for the security design

<br>

### But... Docker is very slow for this use case.

<br>

- Start containers takes some time
- docker daemon is synchronous

<br>

### User experience will not be good 👎
<!--
The first instinct to solve those design problems is to use Docker which is a fantastic fit for the security design.

Since docker containers ceases to exist once stopped running, meaning that even if someone were to run the malicious code, as soon as the container stops it wouldn't affect the host machine.

It is also possible to set resource limits to containers, this would allow containers to not overuse resources.

BUT docker is slow for building a code execution engine!

Why is it slow ?

- Docker is not able to start containers fast enough
- docker daemon is synchronous which means if you want for example to startup 10 containers they happen one after another. So docker is not able to parallel startup containers (for instance if we want to run multiple codes for different users it would mean that each container would be the isolated code and starting up all those containers at once would not work properly, it will take a lot of time)
-->

---
layout: two-cols
---

# Alternative

<br>

## Docker in Docker

<br>
<br>

## Having a single docker container that would have multiple docker engines.

<br>
<br>
<br>

### Docker daemons is a code background service responsible for managing Docker containers.

<!--
One alternative to consider is to do what its called docker in docker

Which consists in a single docker container that would have a bunch of docker daemons (Docker Engine, is the core background service responsible for managing Docker containers and related resources on the host system), making it possible the execution of code in parallel (the sensation that it is in parallel), solving the problem of executing more that once code at once.

But this would not solve the problem of the starting delay of the containers.
-->

<style>
img {
  width: 90%;
  position: absolute;
  left: 10%;
  bottom: 25%;
}
</style>

::right::

<div>
  <img src="/docker-in-docker.png">
</div>

---
---

# Piston alternative

LXC

## Piston implemented their code execution engine in a environment of linux containers (LXC).

<br>
<br>

### Linux containers (LXC) are muiltiple isolated linux systems on a single host.

<br>
<br>
<br>

## On of the trade-offs would be complexity.

<!--
So to solve all the problems Piston implemented their code execution engine in a linux container environment.

So having multiple isolated linux systems on a single host.

One of the trade-offs is the complexity of installation and configuration

But regarding security and speed it solved most of the problems. Of curse they actually implemented some additional security checks.
-->

---
---

# How does Piston do it?

<br>

### When a request comes in with all the information it will create the the necessary files, then a custom script gets invoked to run the code.

<br>
<br>

### The script contains code that would look something like this:

<br>

```sh
cd /tmp/$2
runuser runner$1 -c "cd /tmp/$2 ; cat args.args | xargs -d '\n' timeout -s KILL 3 python3 code.code
```

<!--
But how does it work?

So once the incoming request comes in with all the necessary information it will create 2 files:

- one that contains the source code
- and the other that contains list of command line arguments separated by new lines

Once those 2 files are in place a custom script () gets invoked to run the code.

Each programming language contains a script

and it looks something like the example here

It will run the code with the given arguments, you can see that it also contains a timeout of 3s, so if the code takes more then 3s it would kill the process


-->

---
---

# Different accesses

<br>

### Discord bot

<img src="/discord-run-code.png">
<br>

## Public API

<br>

```
GET  https://emkc.org/api/v2/piston/runtimes # <- get all programming languages
POST https://emkc.org/api/v2/piston/execute # <- execute code
```


<!--
So how can you use Piston

They have several access, which can be a discord bot. Here in madeira we actually installed a bot on our server
-->

---
---

# Different accesses

<style>
img {
  width: 60%;
}
</style>


<img src="/api-run-code.png">
