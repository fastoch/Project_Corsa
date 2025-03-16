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

Microsoft wanted to port their existing compiler, lock, stock and barrel, and get the all of the same semantics in the new code base.  
For that port, Microsoft chose the Go language.  

They have spent a lot of time prototyping in all the various languages, and they found that Go is the most suitable for that purpose.  
It's the lowest level language the can get to, that gives them:
- full optimized native code support on all platforms
- great control over data layout (see side note below)
- the ability to have cyclic data structures
- automatic memory management with a garbage collector
- great access to concurrency (ability to efficiently handle multiple tasks or processes simultaneously)

### Side note

Data layout in programming refers to the way variables, data structures, or objects are organized and stored in memory.  
It describes how the individual components of data are arranged and accessed within a computer's memory system.  



@3
