# tech-talk-code-execution-engine

# Code execution engine

Code execution engine, basically means that we can feed it a programming language and some source code and it will execute that in a sandbox environment and it will spit back the result.


## Different code execution engines

### Piston

Piston is a high performance code execution engine that can be used to run source code in the web.

> https://github.com/engineer-man/piston

> Public API

To use the public API you just need to specify a body with `language`, `source` and `args` if any

```json
{
  "language": "python3",
  "source": "print('hello')",
  "args": [
    "some arguments here"
  ]
}
```

output :

```json
{
  "ran": true,
  "language": "python3",
  "version": "3.6.9",
  "output": "hello"
}
```

> Discord bot

The discord bot allows you to run code in discord just by using the command `/run <language>` you will need to add the code in triple back ticks.

---

The design goals was to run the code fast and securely

**Secure**

The problem with the code execution engine is that it needs to take in any code and execute it as is and that code can be extraordinarily hostile.

So the secure design needed to execute the code, even if someone gave it hostile code it was able to resist any of the negative effects that the code might have.

For instance, if some wanted to run bash and gave it code like `rm -rf /` the code engine needed to be configured in such a way that won't delete the entire system.


So the first implementation for the secure design was to use `docker` which is a fantastic fit for the security design. Since docker containers ceases to exist once stopped running, meaning that even if somebody were to run the malicious code as soon as the container stops it wouldn't affect anything.

It's also possible to set resource limits so specific containers could not overuse resources.

BUT there is a problem !

Docker is very slow. (running code for the clients)

- docker is not able to start contains fast enough
- docker daemon is synchronous which means if you send 3 requests to start containers they happen one after another. Docker is not able to parallel start containers

So those 2 problems would not make the user experience any good.

The second implementation was to do what it's called `docker in docker` (commonly used for testing environments)

A single docker container that would have a bunch of docker daemons, this made it possible to artificially parallelize the execution of code. Solving the problem of executing more than one code at once. But this would not solve the problem of the starting delay of the container.

The final implementation was to use linux containers (LXC), (LXC allows running multiple isolated linux systems on single host)

One of the trade-offs is the complexity (installation for sure)

---

**Principle of operation**

Once the incoming request comes in with all the information it will write out a text file that contains the source code and a text file that contains new lines separated list of command line arguments.

Once those 2 files are in place a custom executioner(done per language) gets invoked. This executioner contains code that would looks something like this :

```sh
cd /tmp/$2
runuser runner$1 -c "cd /tmp/$2 ; cat args.args | xargs -d '\n' timeout -s KILL 3 python3 code.code
```

Other executioners can be more complex.

So once the code is executed and returned to the user it time for some clean ups.

---

## Other alternatives

### AWS Lambda

Which is a serverless computing service provided by amazon web services. It enables you to run code without managing servers. With Lambda, you can execute code in response to specific events, like HTTP requests (and others). This makes it a powerful choice for building a web code execution engine.

Maybe the best approached to use AWS Lambda to create a web code execution engine is to :

1. Choose a Runtime
2. Create an execution role
3. Write your function
4. Define an API Gateway
5. Deploy your lambda function
6. Implement Security
7. Monitoring and Scaling
8. Testing

### Google Cloud Functions

Very similar to AWS Lambda, it's a serverless computing service provided by Google Cloud. It allows you to run event-driven code in response to HTTP requests or other cloud events.

### Use WebAssembly (Wasm)

WebAssembly can indeed be a good option, especially for web-based applications. WebAssembly is a low-level binary instruction format that runs in modern web browsers. It allows you to execute code at near-native speeds, making it an excellent option for high performance and secure code execution on the web.

1. Performance
2. Security
3. Portability
4. Language Agnostic
5. Interoperability
6. Standardization

Some trade-offs would be

1. Limited direct access to web api
2. complexity and learning curve
3. code size
4. compilation overhead
5. browser support
6. debugging challenges
7. Garbage collection overhead
8. ecosystem maturity
