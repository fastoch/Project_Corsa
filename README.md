# Project_Corsa

Microsoft effort to port the TypeScript compiler and toolset to native code (started in 2024).  

src = https://www.youtube.com/watch?v=pNlq-EVld70 - A 10x faster TypeScript

# Introduction

Since the inception of TypeScript (TS) back in 2012, TS has been written in itself, which brought some challenges in terms 
of performance and scalability.  

The JavaScript (JS) runtime platform is really optimized for UI and browser usage, but not so much for compute intensive workloads 
like compilers and system-level tools.  

One of the most commonly reported issues that TS users face is "out-of-memory" situations.  
And we likely reached the limit of what we can squeeze out of JS.  

JS is a **JIT**-compiled language. JIT stands for Just in Time, which means code is compiled into machine code at runtime, 
rather than ahead of time (AOT) like traditional compiled languages (e.g., C, C++).  

And because JS is a JIT-compiled language, there's always startup cost to JIT-compile your code.  
There are other limitations that explain why Microsoft felt the need to port the TS compiler to native code.  

## Illustrating the problem and showcasing the solution

If we try and run a full compile of the VS Code project, which is about one and half million lines of code, we'll see that it takes about a minute.  
![image](https://github.com/user-attachments/assets/347f4278-8ca6-45f3-a3c1-d1911932d4ae)  

Now, if we use the new native code compiler `tsgo`, this will take about 6 seconds:  
![image](https://github.com/user-attachments/assets/8e9bc8b1-7aa0-48f3-ba41-06e8a577d61a)  

# We wanted a port, not a rewrite

Microsoft wanted to port their existing compiler, lock, stock and barrel, and get all of the same semantics in the new code base.  
For that port, they chose the Go language.  

They have spent a lot of time prototyping in many languages, and they found that Go is the most suitable for that purpose.  
It's the lowest level language they can get to, and it gives them:
- full optimized native code support on all platforms
- great control over data layout (see side note below)
- the ability to have cyclic data structures
- automatic memory management with a garbage collector
- great access to concurrency (ability to efficiently handle multiple tasks or processes simultaneously)

### Side note

Data layout in programming refers to the way variables, data structures, or objects are organized and stored in memory.  
It describes how the individual components of data are arranged and accessed within a computer's memory system.  

# What about the language service?

Microsoft are not just building a CLI compiler. There's also the language service, which is arguably the most important thing.  
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




@9
