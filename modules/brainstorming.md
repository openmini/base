keyboard
usb
oscilloscope
FPU accelerator
NPU accelerator
mobile network card

ok, these devices need a protocol
what's widely supported and can be made multidrop through some complicated wizardry?
UART!
one line for data, one for power, one for ground, one for bus error
+3.3V, GND, XFER, ERR
xfer goes thru diode to TX and directly to RX

host openmini is always ID 000...
broadcast node is always ID 100...
address allocation occurs through a message sent by client devices to request an address
addresses are allocated using a balanced binary tree, like this:

```
 +
a b

   +
 +   b
a c

   +
 +   +
a c b d

       +
   +       +
 +   c   b   d
a e

       +
   +       +
 +   +   b   d
a e c f

       +
   +       +
 +   +   +   d
a e c f b g
```

all nodes know their id
furthermore, all nodes recieve allocation requests and responses, thus all nodes know when their address has changed
allocation requests are responded to by the host with the node count before adding the new node
if a node does not respond to a request within a set period of time it is removed from the tree:

```
       +
   +       +
 +   +   +   +
a e c f b g d h

       +
   +       +
 +   f   +   +
a e     b g d h

       +
   +       +
 +   f   b   +
a e         d h
```

the first of the highest nodes is split when adding nodes:

```
       +
   +       +
 +   +   b   +
a e f i     d h
```

a node with one child collapses through a "collapse" message from the host:

```
       +
   +       +
 +   +       +
a e f i     d h

"collapse 1;" = (110 -> 10, 111 -> 11)

       +
   +       +
 +   +   d   h
a e f i
```

"collapse A" means that all ids of the forms A0B or A1B should become AB (remove the bit after A for all IDs matching A?\*)

