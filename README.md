# Project_Corsa

Microsoft effort to port the TypeScript compiler and toolset to native code (started in 2024).  

src = https://www.youtube.com/watch?v=pNlq-EVld70 - A 10x faster TypeScript  

GitHub Repo of this project = https://github.com/microsoft/typescript-go

# Introduction

Since its inception in 2012, TypeScript has been written in itself, which brought some challenges in terms of performance and scalability.  

The JavaScript (JS) runtime platform is really optimized for UI and browser usage, but not so much for compute intensive workloads 
like compilers and system-level tools.  

One of the most commonly reported issues that TS users face is "out-of-memory" situations.  
And we likely reached the limit of what we can squeeze out of JS.  

JS is a **JIT**-compiled language. JIT stands for Just in Time, which means code is compiled into machine code at runtime, 
rather than ahead of time (AOT) like traditional compiled languages (e.g., C, C++).  

And because JS is a JIT-compiled language, there's always startup cost to JIT-compile your code.  
There are other limitations that explain why Microsoft felt the need to port the TS compiler to native code.  

![image](https://github.com/user-attachments/assets/b35437f6-4773-43c8-a8dc-43a0f9a4c8ca)

## Illustrating the problem and showcasing the solution

If we try and run a full compile of the VS Code project, which is about one and half million lines of code, we'll see that it takes about a minute.  
![image](https://github.com/user-attachments/assets/347f4278-8ca6-45f3-a3c1-d1911932d4ae)  

Now, if we use the new native code compiler `tsgo`, this will take about 6 seconds:  
![image](https://github.com/user-attachments/assets/8e9bc8b1-7aa0-48f3-ba41-06e8a577d61a)  

# We wanted a port, not a rewrite

Microsoft wanted to port their existing compiler, lock, stock and barrel, and get all of the same semantics and behavior in the new code base.  
For that port, they chose the Go language.  

They have spent a lot of time prototyping in many languages, and they found that Go is the most suitable for that purpose.  
It's the lowest level language they can get to, and it gives them:
- full optimized native code support on all platforms
- great control over data layout (see side note below)
- the ability to have cyclic data structures
- automatic memory management via garbage collection
- great access to concurrency (ability to efficiently handle multiple tasks or processes simultaneously)

### Side note

Data layout in programming refers to the way variables, data structures, or objects are organized and stored in memory.  
It describes how the individual components of data are arranged and accessed within a computer's memory system.  

# What about the language service?

Microsoft are not just building a command-line compiler. There's also the language service, which is arguably the most important thing.  
If we launch the implementation of VS Code that runs the Go language service, and restart the language server, it will take like 3 seconds to complete.  

That's at least 5 times faster than project loads with the old language service (`tsserver`).  
Which means we're going to see some really nice benefits in our coding experience.  

For this new language service, Microsoft is moving to the LSP (Language Server Protocol) architecture.  
The LSP was introduced by Microsoft in **June 2016**, but tsserver (TypeScript's language server) was developed before the LSP was established.  

This new Go language service isn't necessarily going to be a feature-for-feature port of the old language service.  
It's obviously going to have all the basics, but in today's world, there are many cases where AI-assisted capabilities are more appropriate 
than what is in our existing language service, especially when it comes to things like refactorings...  

# How did Microsoft get all of this extra performance?

It turns out that about half of the performance gain comes from moving to native code.  
The other half comes from the ability to use concurrency (concurrent programming).  

To illustrate that, let's run a full compile of the old TS compiler using this very same old compiler:  
![image](https://github.com/user-attachments/assets/52df2585-bd6b-4821-b136-f2a7772cea74)  

Now, let's compile the old compiler using the new compiler but forcing a single-threaded compilation:  
![image](https://github.com/user-attachments/assets/39006027-8feb-4a54-a1e1-8ef29166bc2c)  

The new compiler is more performant, even when running single-threaded. And if we use concurrency:  
![image](https://github.com/user-attachments/assets/74e07928-31c5-400e-acb1-202265378109)  

# Anders Hejlsberg explains leveraging concurrency

## Parsing

To parse a source file basically means:
- load it into memory
- build a data structure that represents the source text in an Abstract Syntax Tree (AST)
- leave it in memory

You can do that for each file, and if you have 8 cores available, you can go 8 times faster because it's completely parallelizable.  

## Type checking

Type checking isn't quite parallelizable in the same manner as parsing, because types kind of reach across multiple files.  
So instead of running one type checker on a program, we create 4 checkers and we gave each of them the same program but we tell them 
to check a quarter of the files.   

By doing so, we can get 2 to 3 times faster in the check phase, at the cost of maybe consuming 20 to 25% more memory.  
But overall, we still consume less memory than we used to.  

# Final thoughts

Microsoft do expect within 2025 to have a fully functional replacement command-line TS compiler that supports:
- JS
- JSDoc
- JSX
- and possibly incremental compile as well

This should also feature a new inter-process API such that from other languages you can talk to the compiler.  

Microsoft are looking at using this performance gain to think about new kinds of AI features that become possible, such as:
- immediate type checking of an LLM's output
- provision of semantic information in the LLM prompts
- and so forth...
