
## Overview

This is a collection of techniques to improve space efficiency in multiplayer games.  Some of them are specific to sending data over the network, some are not.  Most of them build on each other and are dependent on the core strategy of representing data as integers where possible.



### 1.  Protocol Buffers
Most people familiar with protocol buffers know that it's a very compact format.  But a lot don't really know the real secret to protocol buffers.  Which is varint encoding.  Varint encoding is an efficient scheme for representing integers using the least number of bytes possible.

Combine that with the fact that pretty much all the data we are sending in a multiplayer game can be represented as integers, and you understand the basis of the larger strategy.

#### What about custom serialization?  Surely I can hand craft something better?
Most likely not.  First off you aren't going to come up with a better encoding then varints.  Which leaves you with can you beat the protocol buffer format itself.  If you make use of the packed/repeated options then it's nearly impossible to beat protocol buffers with a custom format.  Even if you can, it's going to be by such small margins as to not be worth it.


### 2.  Efficient use of integers
The first rule in space optimization for games is don't send what you don't need to send.  The most common application of this principle in the context of protocol buffers is to convert floats to integers when you send them over the wire.  And do so at just the precision that you need.  For most cases 2 points of precision is good enough.

Here are a couple of extension methods that I use for converting back and forth.

```csharp
public static float ToFloat(this int i)
{
    return i / 100f;
}

public static int ToInt(this float i)
{
    return (int)(Math.Round(i * 100, 2));
}
```

Another trick is to think of different ways to represent the same idea.  For example in a lot of games we have characters that rotate only on a single axis.  So why not send that as a single float representing a heading of 0-360 instead of the naive Quaternion approach?

Along the same lines, when sending positions you generally only need to send data for 2 axis, because the height can generally be obtained on the client.  Things with positions are usually on some surface, you already know what the height will be so no need to send it over the wire.

### 3. Protocol buffer packed/repeated and zigzag encoding.

Protocol buffers sends a small header for each message that identifies the start of a message type in the stream.  You can optimize most of those out by using the packed format on lists/arrays of data.  Where it will only include the message info once for the array itself.

This generally requires formatting your data differently.  Basically where you would have a class with a set of integer fields and you send a list of that class over the wire.  Now the fields on that class are all List fields, and you pack multiple logical entities into a single instance of your class.  This requires adding the data uniformly so that all the data for a logical entity is represented by the same index into all of the List fields.
  
Another issue is that signed numbers need some extra encoding.  Generally try if you can to design that out of your design, but it's not always possible or necessarily worth it.  In that case make sure you tell protocol buffers to use zigzag encoding.  Otherwise if you do not then the straight varint encoding does very badly, and can end up using close to the full natural field length (32/64 bytes).

```csharp
[ProtoMember(4, IsPacked = true, DataFormat = DataFormat.ZigZag)]
public List<int> Vaxis = new List<int>();

[ProtoMember(5, IsPacked = true)]
public List<int> Heading = new List<int>();
```

### 4. Dealing with large integer id's
Now of course you are using integers as the id's in your database right?  But you still have the issue of those id's get pretty big.  So if you are sending location updates 60 times a second, you need to send the id of the entities involved.  If those ids are large that's a lot of data.

A way to handle this is to use a temporary id pool.  You map the real id to a temporary id that is much shorter, and send that temporary id over the wire for messages in your hot path.  You need to initially send the real id and it's associated temporary id so the client can match them up.

I usually code that logic using a stack, so that you hand out smaller temporary id's first.

### 5. Delta encoding
This technique works really well for position updates.  The basic idea is only send how much an entity has moved since the last update, versus sending the complete position.  Usually this equates to a 2 digit integer for each axis.  

Now the problem with just that part is you are likely using udp so messages can get lost which means you can get out of sync.  The solution is just send a full update periodically.  Say once per second.

Another not so obvious benefit of delta encoding is that the data stay's at a consistent (small) size regardless of how large the full coordinate is.  Now you will need to use zigzag encoding here, unless you want to take things to another level of complexity and have separate fields for signed and unsigned values. Which if you do that in the context of packed fields can get a little complicated.  The savings even with zigzag are substantial, IMO usually good enough.
