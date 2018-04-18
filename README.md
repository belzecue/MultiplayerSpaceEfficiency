
## Overview

This is a collection of techniques to improve space efficiency in multiplayer games.  Some of them are specific to sending data over the network, some are not.  Most of them build on each other and are dependent on the core strategy of representing data as integers where possible.



### 1.  Protocol Buffers
Most people familiar with protocol buffers know that it's a very compact format.  But a lot don't really know the real secret to protocol buffers.  Which is varint encoding.  Varint encoding is an efficient scheme for representing integers using the least number of bytes possible.

Combine that with the fact that pretty much all the data we are sending in a multiplayer game can be represented as integers, and you understand the basis of the larger strategy.

#### What about custom serialization?  Surely I can hand craft something better?
Most likely not.  First off you aren't going to come up with a better encoding then varints.  Which leaves you with can you beat the protocol buffer format itself.  If you make use of the packed/repeated options then it's nearly impossible to beat protocol buffers with a custom format.  Even if you can, it's going to be by such small margins as to not be worth it.


### 2.  Efficient use of integers
The first rule in space optimization for games is don't send what you don't need to send.  The most common application of this principle in the context of protocol buffers is to convert floats to integers when you send them over the wire.  And do so at just the precision that you need.  For most cases 2 points of precision is good enough.



