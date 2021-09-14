# RPC vs REST

My notes from Google's blog post on [REST vs RPC](https://cloud.google.com/blog/products/application-development/rest-vs-rpc-what-problems-are-you-trying-to-solve-with-your-apis).

## RPC

Procedure
:  a.k.a functions. Is the dominant construct for organizing computer code. 

Remote Procedure Call (RPC)
:  Is when a computer program causes a **procedure** (function, subroutine) to execute in a different address space. (This is usually on a different computer on a shared network). This procedure is coded as if it were a normal (local) procedure call, without the programmer explicitly coding the details for the remote interaction.

Here is an example of a RPC through HTTP:
