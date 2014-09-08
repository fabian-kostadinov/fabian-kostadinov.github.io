---
layout: post
title: A Short Introduction to Rhizomes
comments: true
---

Be <code>r<sub>k</sub> <= (r<sub>i</sub>, r<sub>j</sub>)</code> a _directed relation_ _r<sub>k</sub>_ between the ordered pair of relata _r<sub>i</sub>_ and _r<sub>j</sub>_, with <code>r<sub>k</sub> &#8800; r<sub>i</sub></code> and <code>r<sub>k</sub> &#8800; r<sub>j</sub></code>. We will call a relation a _terminal_ iff <code>r<sub>i</sub> = r<sub>j</sub></code>.

*__Definition__*:
<div class="message">A _rhizome_ is recursively defined as a relation between two relata where both relata are relations themselves.</div>
To be really of use a rhizome must be complemented by a (bidirectional) dictionary (or similar data structure) that maps between terminals and data items. The rhizome itself does not store any data in a traditional sense, but only relations.  
Here is a picture of a sample rhizome:

![A simple rhizome](/public/img/simplerhizome.jpg "A simple rhizome")

This rhizome consists of four terminal and several non-terminal relations:

<table>
  <tr>
    <th>
      Terminals
    </th>
    <th>
      Non-Terminals
    </th>
  </tr>
  <tr>
    <td>
      <ul>
        <li>r5 <= (r1, r1)</li>
        <li>r6 <= (r2, r2)</li>
        <li>r7 <= (r3, r3)</li>
        <li>r8 <= (r4, r4)</li>
      </ul>
    </td>
    <td>
      <ul>
        <li>r9 <= (r8, r5)</li>
        <li>r10 <= (r6, r7)</li>
        <li>r11 <= (r5, r8)</li>
        <li>r12 <= (r7, r5)</li>
        <li>r13 <= (r12, r10)</li>
        <li>r14 <= (r15, r12)</li>
        <li>r15 <= (r8, r10)</li>
        <li>r16 <= (r13, r15)</li>
       </ul>
    </td>
  </tr>
</table>

r1 to r4 are relata but not relators. Furthermore, r9, r11, r14 and r16 are only relators but not relata.  
On the right-hand-side of the rhizome, we added a sample dictionary. The four characters <code>A</code>, <code>B</code>, <code>C</code> and <code>D</code> are stored in the dictionary and they were assigned a terminal relation each. The rhizome tree can be traversed from top to bottom replacing each relator through its pair of relata until all relations are resolved to terminals. Finally, the terminals are replaced with values stored in the dictionary. This is a list of relators and the character strings they represent:

* r5 <= (r1, r1) = A
* r6 <= (r2, r2) = B
* r7 <= (r3, r3) = C
* r8 <= (r4, r4) = D
* r9 <= (r8, r5) = ((r4, r4), (r1, r1)) = DA
* r10 <= (r6, r7) = ((r2, r2), (r3, r3)) = BC
* r11 <= (r5, r8) = ((r1, r1), (r4, r4)) = AD
* r12 <= (r7, r5) = ((r3, r3), (r1, r1)) = CA
* r13 <= (r12, r10) = (((r3, r3), (r1, r1)), ((r2, r2), (r3, r3))) = CABC
* r14 <= (r15, r12) = ... = DBCCA
* r15 <= (r8, r10) = ... = DBC
* r16 <= (r13, r15) = ... = CABCDBC

It should be obvious from this simple example that relators higher up in the rhizome's hierarchy encode longer character strings. A rhizome is not a data storage container in a traditional sense. Whereas in (unpacked) data containers a character string like <code>DADADA</code> contains the same characters <code>D</code>, <code>A</code> and the combination <code>DA</code> thereof multiple times, a rhizome is always without redundancy. At the same time, data is never simply read and retrieved from the storage system but must be regenerated for every single access.

Working with rhizomes requires the user to define a vocabulary of atomic symbols (such as the letters <code>A</code> to <code>D</code> in the example). All more complex data types (e.g. <code>DDAC</code>) will be combinations of these atomic symbols. As the vocabulary's atomic data symbols are de facto immutable and because there exists always exactly one relator per ordered pair of relata, updating or deleting existing relations is - at least theoretically - not necessary.

## A comparison with graphs

In computer science, a vast number of data structures are commonly represented as graphs. According to most definitions, a _graph_ is a set of nodes (vertices) and edges (links, archs, spikes) between the nodes. Edges can be directed or undirected. Accepting such a definition for a graph, it should be noted that a rhizome is _not_ a graph, as the _only_ elements it consists of are relations (or directed edges). In graphs it is impossible to draw a link from one edge to another because edges are not "adressable" or "linkable". The only way to link one edge to another is to make it a node itself. A rhizome is therefore more general or fundamental than a graph.

A further important distinction is the distinct need for storage. The storage need for a graph with a high degree of connectivity grows exponentially with an increasing number of nodes. In contrast, the required storage for a rhizome grows logarithmically with an increasing number of relations. This is because the expressive power of higher order relations in a rhizome grows exponentially, whereas it remains flat for added nodes in a graph.