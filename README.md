
## Overview

This is a collection of techniques to improve space efficiency in multiplayer games.  Some of them are specific to sending data over the network, some are not.  Most of them build on each other and are dependent on the core strategy of representing data as integers where possible.


### 1.  Leveraging varint encoding, or why everything should be integers
