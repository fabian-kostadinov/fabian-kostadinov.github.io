---
layout: post
title: Pointing, Oscillation, and the Fracture of Identity
comments: true
tags: [gotthard günther, polycontexturality, proemial relation, philosophy]
---
# Pointing, Oscillation, and the Fracture of Identity

There is a well-known photograph - or rather, a well-known kind of image - in which Stan Laurel and Oliver Hardy point at each other simultaneously. Each one points at the other. Neither points at himself. The image is funny, and also a little vertiginous, because it is not immediately clear who is accusing whom, or whether the gesture means anything at all when it is perfectly mutual.
It is worth taking that vertigo seriously. Let us try to say, from a philosophical perspective, exactly what is happening.

<span class="more"></span>

![Stan Laurel and Oliver Hardy](/public/img/2026-05-01-stan-and-oliver.jpeg)

## I. The Structure of Mutual Pointing
We begin with two simple definitions:
```{code}
stan = pointing(oliver)   - Stan is the one pointing at Oliver
oliver = pointing(stan)   - Oliver is the one pointing at Stan
```
Already, something is unusual. Neither term is defined independently. Stan is defined only in relation to Oliver, and Oliver only in relation to Stan. Neither has any content of his own - no properties, no attributes, nothing beyond the bare act of pointing at the other.
We can substitute. If stan = pointing(oliver), and oliver = pointing(stan), then:
```{code}
stan =   pointing(pointing(stan))     - Stan is the one pointing at the one pointing at Stan
oliver = pointing(pointing(oliver))   - Oliver is the one pointing at the one pointing at Oliver
```
And we can continue:
```{code}
stan = pointing(pointing(pointing(oliver)))
oliver = pointing(pointing(pointing(stan)))
```
Replacing pointing with the symbol > and removing the brackets, we get:
```{code}
1. stan = > oliver
2. stan = >> stan
3. stan = >>> oliver
4. stan = >>>> stan
5. stan = >>>>> oliver
6. ...
```
And correspondingly:
```{code}
1. oliver = > stan
2. oliver = >> oliver
3. oliver = >>> stan
4. oliver = >>>> oliver
5. oliver = >>>>> oliver
6. ...
```
The pattern is clear. Depending on whether the number of nested pointings is odd or even, we arrive at Stan or at Oliver. The two chains oscillate between the same two values. If we write them out as sequences:
```{code}
stan:   [stan  , oliver, stan  , oliver, stan  , oliver, ...]
oliver: [oliver, stan  , oliver, stan  , oliver, stan  , ...]
```
These two chains are the same sequence, offset by exactly one step. A phase-shift of one brings them into alignment. A phase-shift of two returns them to their original separation.

## II. Two Contextures
Following Gotthard Günther's theory of polycontexturality, we can now think of each chain as a distinct contexture - a self-contained logical space with its own starting point.
```{code}
C_Stan:   begins with Stan pointing at Oliver
C_Oliver: begins with Oliver pointing at Stan
```
The internal logic of both contextures is identical. The same alternating structure, the same two values, the same pattern of odd and even steps. And yet they are not the same. They start differently. C_Stan opens with Stan; C_Oliver opens with Oliver.
This immediately raises a question that turns out to be surprisingly difficult: in what sense, if any, are C_Stan and C_Oliver identical?

We might say: if identity is determined by logical structure alone, they are the same - they share the same morphogrammatic pattern, the same abstract form. If identity is determined by starting point, they are strictly distinct. If identity requires both, we face a dilemma: they are partially same and partially distinct, and it is not clear how to weigh the two. And if a single phase-shift of one step brings them into perfect alignment, then their distinctness seems to rest on something very thin indeed - not a difference in structure or content, but merely a difference in where the chain was entered.

The starting point, it turns out, is not a property. It is a position. And position only makes sense relative to something else.

## III. Indexed Sequences and the Question of Coincidence
To press the question of identity further, we need a way to compare two contextures systematically. We do this by introducing an index - a position marker that steps through the sequence. Each position in the sequence has a number, and at each numbered position, a contexture displays a value: Stan or Oliver.
Formally, each contexture is a function from positions to values:
```{code}
C_1 : ℕ → {Stan, Oliver}
C_2 : ℕ → {Stan, Oliver}
```
where ℕ = {0, 1, 2, 3, ...} is the index set. The index can be interpreted in many ways - as steps in a logical derivation, as stages in a process, as moments in time. Time is perhaps the most intuitive interpretation: C_1(3) is the value that contexture C_1 displays at the third moment. But the index need not be temporal. What matters is only that it is ordered, and that both contextures can be evaluated at the same position.
With this in hand, we can say precisely what it means for two contextures to coincide. They coincide at position n if and only if:
```{code}
C_1(n) = C_2(n)
```
That is: at the same index position, both contextures display the same value. The word coincide is chosen deliberately - from the Latin co-incidere, to fall together at a point. It says only that the values fall together here, at this position. Nothing more is assumed.
The richer questions about identity then become precise as well:
```{code}
They never coincide:            ∀n : C_1(n) ≠ C_2(n)
They always coincide:           ∀n : C_1(n) = C_2(n)
They sometimes coincide:        ∃n : C_1(n) = C_2(n)
They could coincide if shifted: ∃k ∀n : C_1(n) = C_2(n+k)
```
The last condition - coincidence under a phase-shift - is exactly the observation made at the end of Section I, now stated exactly. C_Stan and C_Oliver never coincide as written, but a shift of k=1 brings them into perfect alignment.
It turns out that non-coincidence is not a single condition. There are many structurally different ways in which two contextures can fail to coincide - or succeed in coinciding - and these ways are not philosophically equivalent. They deserve to be distinguished carefully.

## IV. A Taxonomy of Coincidence and Non-Coincidence
Consider the following cases. In each one, we ask: do these two contextures coincide? And if not - or if so - why?
Example 1: Structural impossibility of coincidence
```{code}
C_1: Stan > Stan > Stan > Stan > Stan > ...
C_2: Oliver > Oliver > Oliver > Oliver > Oliver > ...
```
Both contextures are constant sequences. C_1 always displays Stan; C_2 always displays Oliver. On the structural level - the level of form and pattern - they are indistinguishable. Both are simple repetitions; the morphogrammatic shape is identical. But at the level of values, they never coincide:
```{code}
∀n : C_1(n) ≠ C_2(n)
```
And no phase-shift can help. There is no k such that C_1(n) = C_2(n+k), because C_1 only ever produces Stan and C_2 only ever produces Oliver. The value-sets are disjoint. Non-coincidence here is not an accident of position or starting point. It is logically guaranteed by the values themselves. This is non-coincidence as structural necessity.
Example 2: Contingent non-coincidence due to index
```{code}
C_1: Stan > Stan > Stan > Stan > Stan > ...
C_2: Oliver > Stan > Oliver > Stan > Oliver > ...
```
Now C_2 oscillates between Oliver and Stan, while C_1 holds constant at Stan. The value Stan appears in both sequences, so coincidence is possible in principle. Whether it occurs at a given position depends entirely on the index:
```{code}
C_1(n) = Stan for all n
C_2(n) = Oliver if n is even, Stan if n is odd
```
So C_1(n) = C_2(n) if and only if n is odd. They coincide at every odd position and fail to coincide at every even position. The non-coincidence at even positions is not structural - it is contingent on the index. A different starting point for C_2, or a different choice of position, changes the result.
Example 3: Mutual oscillation, phase-dependent coincidence
```{code}
C_1: Stan > Oliver > Stan > Oliver > Stan > ...
C_2: Oliver > Stan > Oliver > Stan > Oliver > ...
```
This is the original Stan-and-Oliver structure, now treated as two contextures running in parallel. Both oscillate fully between Stan and Oliver. As written, they are always out of phase:
```{code}
∀n : C_1(n) ≠ C_2(n)
```
But this non-coincidence is not structural. The value-sets are identical - both contain Stan and Oliver. A phase-shift of k=1 would bring them into perfect alignment:
```{code}
∃k=1 such that ∀n : C_1(n) = C_2(n+1)
```
The non-coincidence is purely an artifact of starting point. Compare this to Example 1: the surface statement is the same (they never coincide), but the modal situation is entirely different. In Example 1, no shift can produce coincidence. Here, one step resolves everything. The observable fact of non-coincidence does not distinguish these two cases.
Example 4: Asymmetric frequency, high rate of coincidence
```{code}
C_1: Stan > Stan > Stan > Stan > Stan > Stan > ...
C_2: Stan > Stan > Oliver > Stan > Stan > Oliver > ...
```
C_1 is constant; C_2 oscillates with period three, displaying Oliver only every third step. They coincide at every position where C_2 displays Stan - which is two out of every three positions. The asymptotic frequency of coincidence is 2/3:
```{code}
lim_{N→∞} |{n ≤ N : C_1(n) = C_2(n)}| / N = 2/3
```
We can generalise: as Oliver appears less and less frequently in C_2 - as the period of oscillation lengthens - the frequency of coincidence approaches 1. In the limit, if Oliver disappears entirely from C_2, we arrive at the next case.
Example 5: Total coincidence
```{code}
C_1: Stan > Stan > Stan > Stan > Stan > ...
C_2: Stan > Stan > Stan > Stan > Stan > ...
```
Both contextures are constant, and both display the same value at every position:
```{code}
∀n : C_1(n) = C_2(n)
```
They coincide always, without exception. The question of identity seems to answer itself: these two sequences are, at the level of values, entirely indistinguishable.
And yet - are they the same contexture? They have the same structure, the same values, the same behaviour at every position. But they may still have different origins, different starting conditions, different reasons for displaying Stan at position zero. The surface agreement is total; the question of underlying identity remains open. Total coincidence at the value level does not entail identity at every level.

## V. The Spectrum
What the examples reveal is that coincidence and non-coincidence are not binary conditions but a spectrum, varying along at least two independent dimensions.
The first dimension is frequency: how often do the two contextures display the same value at the same index position? The cases range from never (Example 1), through sometimes (Examples 2 and 3), through most of the time (Example 4), to always (Example 5). This can be measured precisely as the asymptotic proportion of positions at which C_1(n) = C_2(n).

The second dimension is modal character: what is the reason for the frequency? Is the non-coincidence structural - built into the value-sets themselves and impossible to overcome by any shift? Or is it contingent - an artifact of starting point or index, removable in principle by a phase-shift? Or is the coincidence itself perhaps superficial - a surface agreement that conceals underlying difference in origin?
These two dimensions are independent. Examples 1 and 3 have identical frequency (zero coincidences) but completely different modal characters: in Example 1, no shift can produce coincidence; in Example 3, a shift of one step resolves everything. Examples 2 and 3 also have the same surface behaviour at even positions - non-coincidence - but for structurally different reasons. Example 5 shows total coincidence at the value level while leaving open the question of identity at the level of origin.

The observable fact - what value each contexture displays at each index position - does not by itself determine which case one is in. Two observers, each seeing only their own contexture, cannot determine from their local observations whether their non-coincidence is necessary or contingent, permanent or resolvable by a shift. To determine that requires knowing the global properties of both functions - their complete behaviour across all positions, their value-sets, their starting conditions - simultaneously.

## VI. The Problem of the Observer
This is where the problem becomes genuinely difficult. To place a given pair of contextures within the taxonomy - to determine whether their non-coincidence is structural or contingent, whether their coincidence is deep or superficial - requires a standpoint from outside both contextures. It requires an observer who can evaluate both functions across all index positions simultaneously, compare their value-sets, and assess the full modal situation.

But this is precisely the standpoint that Günther's polycontexturality denies. In a polycontextural framework, every logical operation takes place from within a contexture. There is no view from outside all contextures - no neutral position from which both can be surveyed at once. Each contexture has full access to its own values. It sees the other only through the proemial relation, which is a mediated and partial connection, not a transparent window onto the other's complete structure.

A situated observer inside C_1 knows C_1(n) for positions it has traversed. It may know something of C_2(n) through the proemial relation. But it cannot know the global properties of C_2 - its value-set, its period, its behaviour at all positions - from within C_1 alone. And the modal question - is our non-coincidence necessary or contingent? does a shift exist that would resolve it? - is precisely a question about global properties.

The consequence is sharp: the taxonomy exists, and the cases are genuinely distinct. But no observer situated inside either contexture can determine with certainty which case they are in. The taxonomy is real; its application is structurally inaccessible from any position that actually exists.

## VII. The Fracture of Identity
We began with a simple image: two men pointing at each other. We have arrived somewhere more uncomfortable.

The question 'are these two contextures identical?' turns out not to be one question but several. Are they structurally the same? Do they display the same values at every index position? Do they share the same starting point? Do they coincide always, sometimes, or never - and is the coincidence or non-coincidence necessary or contingent?

These questions are independent of each other. Any combination of answers is in principle possible. Two contextures can be structurally identical but value-distinct, as in Example 1. They can be value-compatible but contingently non-coincident, as in Examples 2 and 3. They can coincide at every observable position and still differ in origin, as in Example 5. Identity is not a single thing that either holds or does not hold. It fractures - into multiple dimensions, each requiring a different kind of observation, each potentially yielding a different answer.

And some of those observations are not available from within any position that actually exists. The dimension that requires global knowledge of both functions - the modal dimension, the question of whether coincidence is necessary or contingent, structural or resolvable - is precisely the dimension that a situated observer cannot access.
This is not a failure to find the answer. It is a finding about the structure of the question. The discomfort of not being able to say, definitively, whether C_Stan and C_Oliver are the same or different - whether the two men pointing at each other are one thing or two - is not a gap in our knowledge that further inquiry might close. It reflects something about what identity is, and about what it requires of the one who asks.

To answer the question fully, one would need a standpoint outside both contextures - a view from nowhere within the system. But such a standpoint is not available. It can be postulated, as philosophers have postulated it for centuries. Postulating it, however, does not create it. And without it, the question of identity does not resolve. It fractures, and stays fractured.

Stan and Oliver point at each other. Each is defined only by pointing at the other. The question of whether they are the same admits no answer from inside the pointing. And there is nowhere else to stand.
