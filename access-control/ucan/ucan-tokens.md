# UCAN Tokens

Fission‘s protocol for commands and mutations is called a User Controlled Authorization Network, or ”UCAN” \(pronounced ”you can”\). These are a way of doing authorization where users are fully in control. Whee OAuth is designed for a centralized world, UCAN is the distributed, user-controlled equivalent.

![UCAN Sam](https://s3.fission.codes/2020/05/UCAN_SAM-1.png)

### User Controlled

Users control the data \(and other resources\) that they create. They also have the ability to delegate control to others.

### Authorization Network

“Network” here is meant in the sense of a graph.

The UCAN format is designed as an _authenticated_ digraph in some larger _authorization_ space. The other way to view this is as a function from a set of authorizations \(“UCAN proofs“\) to a subset output \(“UCAN capabilities”\).

