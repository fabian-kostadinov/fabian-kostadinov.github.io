---
layout: post
title: Nowhere Is Not Enough: Fractured Identity and the Proemial Relation as Channel
comments: true
tags: [gotthard günther, polycontexturality, proemial relation, philosophy]
---
# Nowhere Is Not Enough

*Fractured Identity and the Proemial Relation as Channel*

---

## Abstract

This paper argues that identity between two logical subjects — two *contextures* in Gotthard Günther's sense — is not a binary fact but a dimensional structure, and that the dimensions of identity accessible between any two contextures are determined not by the contextures themselves but by the channel through which they communicate. We show that identity fractures into multiple independent dimensions, each requiring a different kind of observation; that the proemial relation mediating between contextures is a partial mapping whose coverage is constituted by the channel; and that the view from nowhere — the fully neutral standpoint from which identity might be settled once and for all — is insufficient not merely because it is epistemically unavailable, but because it bypasses the channel, which is precisely where identity is constituted. The argument moves from a formal analysis of mutual reference through Günther's polycontexturality and Spencer-Brown's calculus of distinctions to a demonstration in executable code. The result is a precise account of why the question *are these two things the same?* cannot be answered from any position that actually exists — and why even an imagined position outside all perspectives would fail to answer it.

---

# I. Introduction: The Image and the Problem

![Stan Laurel and Oliver Hardy](/public/img/2026-05-01-stan-and-oliver.jpeg)

There is a well-known kind of image in which Stan Laurel and Oliver Hardy point at each other simultaneously. Each one points at the other. Neither points at himself. The image is funny, and also a little vertiginous, because it is not immediately clear who is accusing whom, or whether the gesture means anything at all when it is perfectly mutual.

Let us take that vertigo seriously. We begin with two definitions:

```
stan   =  pointing(oliver)   — Stan is the one pointing at Oliver
oliver =  pointing(stan)     — Oliver is the one pointing at Stan
```

Neither term is defined independently. Stan is defined only in relation to Oliver, and Oliver only in relation to Stan. Neither has any content of his own. Substituting repeatedly and replacing *pointing* with the symbol >, we arrive at two oscillating chains:

```
stan:    stan > oliver > stan > oliver > stan > oliver > ...
oliver:  oliver > stan > oliver > stan > oliver > stan > ...
```

These two chains are structurally identical: same logic, same pattern, same two values. Yet they are not the same. They start differently. One opens with Stan, the other with Oliver. A phase-shift of one step brings them into perfect alignment. A phase-shift of two returns them to their original separation.

This raises a question that turns out to be surprisingly difficult: *in what sense, if any, are these two chains identical?* If identity is determined by logical structure alone, they are the same. If by starting point, they are strictly distinct. If by both, we face a dilemma: partially same, partially distinct, with no principled way to weigh the two. And if a single phase-shift collapses the distinction entirely, it seems to rest on nothing more than where the chain was entered.

This paper follows that question all the way down. What begins as a puzzle about two oscillating sequences leads, through formal logic, the philosophy of identity, and the theory of communication, to a finding about the nature of identity itself: that it is not one thing, that it fractures into independent dimensions, and that the dimensions accessible between two logical subjects are determined not by those subjects alone but by the channel through which they communicate. The view from nowhere — the fully neutral standpoint from which identity might be settled once and for all — turns out to be insufficient not merely because it is unavailable, but because it is beside the point.

The argument proceeds in six movements. Section II develops the formal structure of fractured identity. Section III introduces the theoretical framework: Spencer-Brown's calculus of distinctions, Günther's polycontexturality, and the proemial relation as the mediating structure between logical subjects. Section IV makes the central move: the proemial relation is constituted by the channel, and channel constraints determine which dimensions of identity are reachable. Section V draws the conclusion about the insufficiency of the view from nowhere. Section VI sketches implications. An appendix presents a concrete demonstration in executable code.

---

# II. The Fracture of Identity

## II.1 Indexing the Oscillation

To compare the two chains formally, we need a shared coordinate — a position marker that steps through both sequences so we can ask, at each position, whether the two chains display the same value. We call this the *index*.

Formally, each chain is a function from index positions to values:

```
C_1 : ℕ → {Stan, Oliver}
C_2 : ℕ → {Stan, Oliver}
```

where ℕ = {0, 1, 2, 3, ...} is the index set. The index can be interpreted in many ways: as steps in a logical derivation, stages in a process, or moments in time. Time is perhaps the most intuitive interpretation — C_1(3) is the value chain C_1 displays at the third moment. But the index need not be temporal. What matters is only that it is ordered and that both chains can be evaluated at the same position.

Two chains *coincide* at position n if and only if:

```
C_1(n) = C_2(n)
```

The word *coincide* is chosen deliberately — from the Latin *co-incidere*, to fall together at a point. It says only that the values fall together here, at this position. It says nothing about why, nor whether they will fall together again.

> **Note:** The notation C_1(n) = C_2(n) already presupposes something that deserves acknowledgment. The shared index n is a coordinate system that both chains can be evaluated against simultaneously. The equality sign presupposes a shared relation of sameness applicable to the values of both chains. Neither of these is available from within either chain alone. Together they constitute a minimal shared context — something outside both chains from which comparison is possible at all. We call this the *minimal supercontexture*. We return to what it is and what it does in Section IV; for now we note its presence in the notation itself, and observe that without it there would be no basis for comparison, coincidence, or any of the distinctions that follow.

## II.2 The Taxonomy

The richer questions about identity now become precise:

```
They never coincide:            ∀n : C_1(n) ≠ C_2(n)
They always coincide:           ∀n : C_1(n) = C_2(n)
They sometimes coincide:        ∃n : C_1(n) = C_2(n)
They could coincide if shifted: ∃k ∀n : C_1(n) = C_2(n+k)
```

The last condition is exactly the phase-shift observation from Section I, now stated precisely. C_Stan and C_Oliver never coincide as written, but a shift of k = 1 brings them into complete alignment.

These conditions are not equivalent, and the differences between them are philosophically significant. Consider five cases:

### Case 1: Structural impossibility

```
C_1: Stan > Stan > Stan > Stan > ...
C_2: Oliver > Oliver > Oliver > Oliver > ...
```

Both chains are constant. On the structural level they are indistinguishable: both are simple repetitions with the same morphogrammatic shape. But their value-sets are disjoint. They never coincide, and no phase-shift can change this. Non-coincidence here is logically necessary, built into the values themselves.

### Case 2: Contingent non-coincidence

```
C_1: Stan > Stan > Stan > Stan > ...
C_2: Oliver > Stan > Oliver > Stan > ...
```

The value Stan appears in both chains. Coincidence is possible in principle. Whether it occurs at a given position depends entirely on the index: at odd positions they coincide, at even positions they do not. The non-coincidence is not structural; it is an artifact of the index. A shift of one step resolves it entirely.

### Case 3: Phase-dependent coincidence

```
C_1: Stan > Oliver > Stan > Oliver > ...
C_2: Oliver > Stan > Oliver > Stan > ...
```

This is the original Stan-and-Oliver structure. Both chains contain both values. As written they never coincide, but a phase-shift of k = 1 resolves everything. The non-coincidence is purely an artifact of starting point. Crucially: Cases 1 and 3 have the same surface statement (∀n : C_1(n) ≠ C_2(n)) but completely different modal characters. In Case 1, no shift helps. In Case 3, one step resolves everything. The observable fact of non-coincidence does not distinguish them.

### Case 4: Asymptotic coincidence

```
C_1: Stan > Stan > Stan > Stan > Stan > Stan > ...
C_2: Stan > Stan > Oliver > Stan > Stan > Oliver > ...
```

They coincide at two out of every three positions. As Oliver appears less frequently in C_2, the frequency of coincidence approaches 1. This case introduces a measurable quantity, the asymptotic coincidence frequency:

```
lim_{N→∞} |{n ≤ N : C_1(n) = C_2(n)}| / N
```

### Case 5: Total coincidence

```
C_1: Stan > Stan > Stan > Stan > ...
C_2: Stan > Stan > Stan > Stan > ...
```

They coincide at every position. Yet: are they the same chain? They may have different origins, different reasons for displaying Stan at position zero. Total coincidence at the value level does not entail identity at every level. The surface agreement is complete; the question of underlying identity remains open.

## II.3 Two Dimensions, Neither Reducible to the Other

The five cases reveal that coincidence and non-coincidence vary along at least two independent dimensions.

The first dimension is **frequency**: how often do the two chains display the same value at the same index position? This ranges from never (Cases 1 and 3) through sometimes (Case 2) through most of the time (Case 4) to always (Case 5).

The second dimension is **modal character**: what is the *reason* for the observed frequency? Is the non-coincidence structural — built into the value-sets and impossible to overcome by any shift? Or is it contingent — an artifact of starting point, resolvable in principle by a phase-shift? Or is even total coincidence perhaps accidental — a surface agreement concealing a difference in origin?

These two dimensions are independent. Cases 1 and 3 have identical frequency but completely different modal characters. Case 5 shows total coincidence while leaving the modal question entirely open.

To determine the modal character of a given pair of chains requires knowing their complete behaviour across all index positions — their value-sets, their periods, their starting conditions — all at once. It requires a global view. An observer inside C_1 can observe C_1(n) for positions it has traversed. It sees C_2 only partially, through whatever relation connects them. It cannot determine from local observations alone whether the non-coincidence it experiences is structural or contingent.

Before drawing the consequence, it is worth stating clearly what we mean when we say identity fractures. We are not claiming that identity is a mysterious thing that breaks apart under scrutiny. We are claiming something more precise: **identity between two chains just is the accumulated pattern of coincidences across all dimensions of comparison**. There is no identity hiding behind the coincidences, waiting to be found. Identity is what the coincidences add up to, dimension by dimension. This is not a reduction of identity to something less. It is a recognition of what identity always was.

This is why the fracture is not a failure. It is a finding. The question *are C_Stan and C_Oliver identical?* does not have a single answer because it is not a single question. It is a family of questions, one per dimension: are they structurally the same? do they coincide at every index position? do they share a starting point? is their non-coincidence structural or contingent? Each question has its own answer, and those answers can come apart in any combination. The fracture is the correct description of what identity is.

Identity fractures. It splits into multiple dimensions, each requiring a different kind of observation, each capable of yielding a different answer. The most important dimension — the modal one — is structurally inaccessible from within either chain. But inaccessibility from within does not make it less real. It makes it the exact location where the question of identity runs out of available answers.

---

# III. Distinctions, Contextures, and the Proemial Relation

## III.1 Spencer-Brown: The Act of Distinction and Re-entry

In *Laws of Form* (1969), George Spencer-Brown begins with a single primitive act: drawing a distinction. To make a distinction is to divide a space into two sides — a marked state and an unmarked state. Two axioms follow:

```
Calling:  marking an already-marked space leaves it marked
Crossing: crossing a boundary twice returns you to where you started
```

From these two axioms alone, Spencer-Brown derives classical two-valued logic. The mark is one logical value; the unmarked state is the other. All Boolean operations emerge as theorems. The crossing axiom is directly relevant here: each application of > is a crossing of the boundary. Cross once, Stan becomes Oliver. Cross twice, Oliver becomes Stan. The oscillation of Section I is exactly the crossing axiom unfolded as a sequence.

Spencer-Brown then asked a deeper question: what happens when a form crosses back into *itself* — when the inside of a distinction contains the very distinction that produced it?

To see what this means, consider an ordinary distinction first. A form draws a boundary between inside and outside. The inside is the marked state; the outside is the unmarked state. They are separate: the form sits on the boundary and does not appear inside its own inside.

Re-entry changes this. Imagine taking the form — the boundary-drawing act itself — and placing it inside its own inside. The boundary now curves back and contains itself. The inside of the distinction contains the distinction that created the inside. This is self-reference made formal.

What value does a re-entering form have? To evaluate it, you must evaluate what is inside it. What is inside it is the form itself. To evaluate that, you must again evaluate what is inside it. Which is again the form itself. The evaluation does not terminate. Instead of arriving at a stable value — marked or unmarked — you get an endless alternation: marked, then unmarked, then marked, then unmarked, indefinitely. Spencer-Brown called this the *imaginary value*, by deliberate analogy with the square root of minus one in mathematics. Just as √-1 cannot be placed on the ordinary number line yet is a necessary extension of it, the imaginary logical value cannot rest in either classical state but emerges necessarily when self-reference is permitted.

The Stan-and-Oliver structure is exactly a re-entering form, unfolded step by step. Stan's identity is defined by pointing at Oliver. Oliver's identity is defined by pointing at Stan. Each one's inside contains a reference to the other, which contains a reference back to the first. To evaluate Stan, you must evaluate Oliver, which requires evaluating Stan, which requires evaluating Oliver — the evaluation oscillates without terminating. At each discrete step the form has a definite value, but the value never stabilises. The imaginary value is the oscillation itself.

This is a remarkable result: classical two-valued logic, applied to genuine mutual reference, generates a value it cannot contain. But Spencer-Brown's system carries a fundamental assumption: one observer, one space, one act of distinction. The re-entering form is seen from a position outside it. Spencer-Brown never asks what happens if there is no outside — if the observer is itself inside a form, or if there are multiple observers each making their own distinctions from their own positions.

## III.2 Günther: Multiple Acts of Distinction

This is the question Gotthard Günther spent his career answering. His central insight is that classical logic is always written from a single logical locus: one standpoint, one observer, one space of values. He called this *monological* logic — adequate for a world of one subject relating to objects, inadequate for a world of multiple subjects.

His alternative is *polycontexturality*: a logic of many contexts, each with its own internal validity, none reducible to a master context. In a polycontextural framework, each logical context — each *contexture* — is a complete two-valued logical space of its own. Within a contexture, classical logic holds in full. What changes is the relationship between contextures: there is no super-context from which all can be surveyed simultaneously. Each is, from the inside, complete. From the outside — if there is one — it is one perspective among many.

We restate the Stan-and-Oliver problem in Günther's terms. Each oscillation chain is a distinct contexture — a self-contained logical space with its own starting point:

```
C_Stan:   begins with Stan pointing at Oliver
C_Oliver: begins with Oliver pointing at Stan
```

The internal logic of both contextures is identical: same structure, same values, same oscillation. And yet they are not the same. They start differently. The question of whether they are identical — which we showed in Section II to fracture into multiple dimensions — is now the question of whether two distinct contextures can be said to be the same, and in what sense.

## III.3 The Proemial Relation

Günther introduced the *proemial relation* to describe the mediating structure between contextures. The proemial relation is not a logical operation within a contexture. It is the relation between contextures: the way one logical space gestures toward another, establishes connection with it, without dissolving into it.

Crucially, the proemial relation cannot be established from within either contexture alone. To mediate between two contextures requires something aware of both as distinct origins — not their contents or values, but the bare fact that there are two logical places here, neither reducible to the other. In Günther's terminology, this operates at the *kenogrammatic* level: the level of pure position, prior to any values.

This means that instantiating a proemial relation always requires a minimal supercontexture — a third logical position that registers the existence and distinctness of the two without claiming to see inside either. The supercontexture does not adjudicate between the contextures. It does not provide the global view. It simply holds the boundary.

The minimal supercontexture is exactly what we acknowledged — and set aside — in the note to Section II.1: the shared index and equality relation that make comparison possible at all. We are now in a position to name it properly and to ask what determines its character. That question is the subject of Section IV.

## III.4 Thomas: The Geometry of Logical Space

Gerhard G. Thomas developed *permutograph theory* as a way of making Günther's logical spaces visible and computable. The key insight is that the operators in Günther's logic are permutations of positions — not swaps of values. Positions are fixed and unique; values are not. A permutation is a complete reordering of a set of positions.

A *permutograph* P = (N, E) is a graph whose nodes N are permutations and whose edges E are the operators connecting them. For the Stan-and-Oliver case — two positions, two values — there are exactly two permutations:

```
σ₁ = (Stan, Oliver)   — Stan at position 1, Oliver at position 2
σ₂ = (Oliver, Stan)   — Oliver at position 1, Stan at position 2
```

These form the node set N = {σ₁, σ₂}. The operator > — which swaps the two values — defines a single edge connecting them. Applying > to σ₁ yields σ₂; applying > to σ₂ yields σ₁. The permutograph of the Stan-and-Oliver structure is therefore the minimal possible graph: two nodes, one edge, forming a single connected component. Traversing it repeatedly produces the oscillation of Section I.

For three positions there are 3! = 6 permutations, and the permutograph forms a hexagon. For four positions, 4! = 24 permutations, forming a truncated octahedron. As positions increase, the geometry becomes richer, with deep connections to physical crystal symmetries that Thomas documented extensively.

For our purposes, the most important structural property is connectivity. Two nodes in a permutograph are *connected* if there exists a sequence of edges — a sequence of operator applications — that leads from one to the other. A permutograph may have multiple *connected components*: maximal subgraphs within which every node can reach every other, with no edges crossing between components.

This gives the structural versus contingent non-coincidence distinction from Section II a precise geometric form. Recall that Cases 1 and 3 both satisfy ∀n : C_1(n) ≠ C_2(n), yet have completely different modal characters. In the permutograph framework:

**Case 3 (phase-dependent, contingent):** C_Stan and C_Oliver correspond to nodes σ₁ and σ₂ in the same connected component. A path exists between them — the phase-shift operator — and applying it brings them into alignment. Their non-coincidence is contingent because the required operator is available.

**Case 1 (structural impossibility):** C_1 (always Stan) and C_2 (always Oliver) correspond to nodes in different connected components. No sequence of available operators leads from one to the other. Their non-coincidence is structural because no path exists between their components.

Formally: two contextures are *structurally non-coincident* if and only if they occupy different connected components of the permutograph defined by their available operators. Two contextures are *contingently non-coincident* if they occupy the same connected component but have not been brought into alignment by those operators.

This is not merely a philosophical distinction. Connected component membership is an algorithmically decidable property of a graph. Given the permutograph and two nodes, a standard breadth-first search determines in finite time whether a path exists between them. The taxonomy of Section II has mathematical teeth: the modal character of non-coincidence between two contextures is, in principle, computable from the structure of the permutograph that governs them.

---

# IV. The Channel Constitutes the Proemial Relation

## IV.1 From Kenogram to Channel

We established in Section III.3 that the proemial relation requires a minimal supercontexture operating at the kenogrammatic level — registering the existence and distinctness of two contextures without claiming to see inside either. We acknowledged in Section II.1 that such a structure was already presupposed by our notation. We are now ready to ask: what, concretely, is this supercontexture in any real system between two communicating parties?

Our answer is that it is the *channel*: the medium through which two contextures communicate. But this answer requires careful unpacking, because there is an apparent tension here that must be resolved.

We said the supercontexture operates at the kenogrammatic level — pure position, no values. But a channel allows messages. Messages have content. Content looks like values, not positions. How can the channel be both the kenogrammatic supercontexture and a medium for content?

The resolution lies in distinguishing two levels at which the channel operates simultaneously.

At the *kenogrammatic* level, the channel is simply the fact of a connection: there are two logical places here, and something can pass between them. This is purely positional. It registers the existence and distinctness of two contextures and the possibility of a relation between them. No content, no values — only the bare structure of a boundary that is shared without being owned by either side. This is what Günther's minimal supercontexture is, and this is what justifies treating the channel as its concrete instantiation.

At the *morphogrammatic* level — one level up, built upon the kenogrammatic foundation — the channel acquires operational structure: a space of possible messages, constraints on their length and form, rules about what can be transmitted and at what cost. This is where content appears. The messages are values flowing through the positional structure that the kenogrammatic level established.

The channel is therefore not purely kenogrammatic. It is kenogrammatic in its foundational structure — the bare fact of the connection — and morphogrammatic in its operational character — the shaped space through which exchange becomes possible. These are two levels of the same structure, not two different things. Günther would not object to the channel as supercontexture provided we are precise: the channel *instantiates* the proemial relation at the kenogrammatic level, while *operating* at the morphogrammatic level. The proemial relation is kenogrammatic in origin; the channel is its morphogrammatic realisation.

What makes this philosophically significant is the step that follows. The channel, operating at the morphogrammatic level, is not neutral. It does not simply transmit whatever the two contextures wish to say to each other. It *selects*. It determines which dimensions of the shared environment can be transmitted, at what cost, with what fidelity. And because it selects, it determines which dimensions of identity between the two contextures can be approached at all.

This is the central claim of the paper: **the proemial relation between two contextures is not a given — it is constituted by the channel. The channel does not carry the proemial relation as a neutral medium carries a signal. The channel is the proemial relation, in the only form in which it can exist between two real communicating subjects.**

## IV.2 Coverage: A Formal Account

We now formalise the notion of what a channel can and cannot carry between two contextures.

Let E be the *shared environment*: the set of dimensions along which the two contextures could potentially be compared. In the Stan-and-Oliver example, E has at least two dimensions: logical structure and starting point. In a richer example involving physical objects, E might include shape, position, colour, size, and so on.

Let K denote a channel between two contextures C_1 and C_2. Let Φ(C_1, C_2, K) denote the proemial relation between C_1 and C_2 as constituted by K. Then we define the *coverage* of the proemial relation as:

```
Coverage(Φ) ⊆ E
```

Coverage(Φ) is the subset of environmental dimensions along which coincidence between C_1 and C_2 can actually be established through K. The complementary set:

```
E \ Coverage(Φ)
```

represents the dimensions that are *structurally unreachable* through this channel. Here we must be precise about what unreachability means, because it is stronger than ignorance. There is a difference between two kinds of situations:

A fact may *exist* but remain *unknown* to the parties. The shape correspondence in our demonstration exists in the physical environment — there is a fact about which of A's shapes maps to which of B's shapes. But it is not known to either agent through their exchange.

A fact may exist as a structural property of the environment, yet *fail to exist as an established relation between the parties*. This is the situation of identity along unreachable dimensions. Because identity just is the pattern of coincidences established through exchange, a dimension along which no coincidence has been established through the channel is not a dimension along which identity is unknown — it is a dimension along which identity has not yet come into existence between these two parties. It may become established if the channel changes. But until it does, it is not merely hidden. It is absent.

This distinction matters for the central claim. We are not saying that identity along unreachable dimensions is hard to find. We are saying it has not been constituted. An external observer with access to the full environment can see structural facts about both contextures. But structural facts seen from outside are not the same as identity established between the parties. The external observer sees what could in principle be compared. The parties have established only what the channel has actually carried between them.

The central formal claim is then:

**Coverage(Φ(C_1, C_2, K)) is a function of K, not only of C_1 and C_2.**

That is: for the same pair of contextures C_1 and C_2, different channels K and K' will in general yield different coverage sets. A richer channel may make more dimensions of E accessible; a more constrained channel will make fewer accessible. The identity between two contextures — which dimensions of identity are establishable between them — is therefore not a fixed property of those contextures. It is a function of the channel through which they relate.

We can further distinguish three kinds of dimensions within E with respect to a given channel K:

*Reachable dimensions*: d ∈ Coverage(Φ). These are dimensions along which the channel can carry sufficient information to establish coincidence or non-coincidence reliably.

*Unreachable dimensions*: d ∈ E \ Coverage(Φ). These are dimensions the channel cannot carry. Identity along these dimensions is not merely unknown — it is structurally inaccessible through this channel.

*Partially reachable dimensions*: dimensions along which the channel can carry some information but not enough for reliable coincidence determination. This corresponds to Case 4 in the taxonomy — asymptotic coincidence — where the channel makes a dimension partially visible without making it fully determinable.

This framework gives precise meaning to the intuition that two parties communicating through a constrained channel can establish some aspects of their shared identity while others remain permanently out of reach.

## IV.3 Channel Constraints in the History of Communication

The constitutive effect of channel constraints on what can be established between communicating parties is one of the most pervasive phenomena in the history of human communication.

Early telegraph operators, constrained by the cost of each transmitted word, developed highly compressed codes — shorthand, abbreviations, conventions — that stabilised into permanent features of written English. The channel did not merely make communication harder; it generated new conventions. What could be established between sender and receiver was shaped by what the channel could carry economically.

Aviation English — the standardised language used between pilots and controllers — was shaped by radio bandwidth, noise, and the life-or-death cost of misunderstanding. The channel generated a dialect: redundant confirmation protocols, elimination of ambiguous constructions, highly conventionalised phrasing. The coverage of the proemial relation between pilot and controller includes precision and confirmation; it is constitutively thin on nuance and register.

Early writing systems developed under material constraints — the cost of clay, papyrus, vellum — which is why many dropped vowels, used logograms, or developed compressed syllabaries. The material channel shaped which dimensions of spoken language got encoded and which were left to inference.

In each case the same structure appears: the channel selects which dimensions of meaning stabilise into convention, and leaves others to context, inference, or silence. In the formal terms of Section IV.2: the channel determines Coverage(Φ), and what lies in E \ Coverage(Φ) remains structurally unreachable regardless of how much exchange takes place.

## IV.4 A Demonstration

To make the abstract claim concrete, we constructed and ran a series of experiments in which two AI agents attempted to develop a shared referential language from scratch. The full experimental detail is in the Appendix; here we describe the setup, walk through an example round, and present the key finding.

**The setup.** A shared environment contains objects, each with two properties: a shape and a position. In a given scene, three such objects are present. One agent — the Encoder — is shown the full scene and told which object is the target. The other agent — the Decoder — is shown the same scene but does not know which object is the target. The Encoder sends a message. The Decoder reads the message and selects one object from the scene. The Encoder then observes which object the Decoder selected.

**Implicit feedback.** No score is announced. Neither agent is told whether the selection was correct. The Encoder knows what the target was and can see what the Decoder chose — and must infer from that gap whether its message worked. The Decoder never learns whether it was right unless the next message from the Encoder tells it something. This is implicit feedback: the only signal available is the observable consequence of the exchange, not an external judgment about it. This condition is philosophically deliberate. In genuine first-contact situations — two parties with no shared vocabulary attempting to establish communication — there is no referee. What counts as success must emerge from the exchange itself.

**An example round.** The scene contains three objects: ◆ at P2, ▲ at P4, ● at P1. The target is ▲ at P4. The Encoder sends the message: "P4▲". The Decoder reads this, examines its scene, and selects ▲ at P4. The Encoder observes that the Decoder selected ▲ at P4, which matches the target. No success signal is given; the Encoder must infer from the match that its shorthand convention worked. In the next round, it is likely to use a similar convention.

**The maximally alien condition.** In the most stringent version of the experiment, each agent was given a completely different label system for the same physical objects, with no numeric or alphabetic correspondence between the two systems. Agent A saw shapes labelled SH-A through SH-E and positions labelled LO-1 through LO-5. Agent B saw the same shapes labelled FO-V through FO-Z and positions labelled SP-1 through SP-5. The position mapping was deliberately scrambled: A's LO-1 corresponded to B's SP-2, A's LO-4 to B's SP-1, and so on. No systematic relationship could be exploited. The channel was constrained to messages of twenty characters or fewer, creating pressure toward compression.

**The result.** After 120 rounds of exchange with implicit feedback only, the emerging lexicon showed the following stable cross-label correspondences, measured by successful co-occurrence of A-labels in messages alongside correct B-labels:

```
SP-2 ↔ LO-1  (6 successful co-occurrences)   correct: P1
LO-4 ↔ SP-1  (4 successful co-occurrences)   correct: P4
SP-3 ↔ LO-2  (3 successful co-occurrences)   correct: P2
LO-5 ↔ SP-4  (3 successful co-occurrences)   correct: P5
```

Four of five position correspondences were correctly identified through exchange alone. Zero shape correspondences emerged. The agents bootstrapped a partial cross-label lexicon covering the position dimension of the shared environment while remaining entirely blind to the shape dimension.

**The connection to identity.** This result is not merely a finding about communication. It is a finding about identity — and specifically about the distinction between what is structurally true of two systems and what has been established as an identity between them.

Recall from Section II that identity between two contextures just is the accumulated pattern of coincidences across the dimensions of their shared environment. In this experiment, E has two salient dimensions: position and shape. The channel — constrained to twenty characters — made it uneconomical to include both a position token and a shape token in a single message. The agents settled on position as the reliable signal and dropped shape. The channel thereby determined Coverage(Φ): position fell inside Coverage(Φ), shape fell in E \ Coverage(Φ).

Now consider what this means for identity. The physical correspondence between A's shape labels and B's shape labels exists as a structural fact about the environment — it is recorded in the correspondence table that we, as external observers, can see. But this correspondence does not exist as an *established identity* between the two agents. It was never carried by the channel. It was never negotiated, never co-produced, never part of what flowed between them. The agents do not share that identity because it was never constituted between them.

This is the crucial distinction: **a fact that exists but is not known is different from a fact that has not yet been constituted as a relation between these parties**. Identity along unreachable dimensions falls into the second category, not the first. Ignorance can in principle be remedied by more information. Absence of constitution requires a change in the channel itself — a change in what can flow between the two contextures.

The channel was not neutral. It selected. And what it selected determined not merely what the agents know about each other, but which dimensions of their mutual identity have been brought into existence between them — and which have not.

---

# V. The Insufficiency of the View from Nowhere

## V.1 The Epistemic Argument

Thomas Nagel, in *The View from Nowhere* (1986), argued that objectivity requires stepping back from any particular perspective and attempting to see the world as it is independently of any viewpoint. The aspiration toward objectivity is the aspiration toward a view from nowhere: a standpoint that is no particular standpoint, from which things can be seen as they truly are.

Applied to our question, the view from nowhere would be the standpoint from which both C_Stan and C_Oliver could be surveyed simultaneously — their complete behaviour across all index positions, their value-sets, their starting conditions — and from which the modal character of their non-coincidence could be determined. A view from nowhere could, in principle, see whether they are in the same connected component of their permutograph, and therefore whether their non-coincidence is structural or contingent.

But as we showed in Section II, such a view is epistemically unavailable. No situated observer has access to the global structure of both chains simultaneously. Every observer is inside one contexture or the other. It sees itself completely and sees the other partially, through whatever proemial relation connects them. The modal dimension of non-coincidence requires exactly the information that no situated observer has.

This is the epistemic argument against the view from nowhere as a tool for settling identity. It is not a new argument — versions appear in Wittgenstein, in Günther, in feminist standpoint epistemology, in the sociology of knowledge. What Section II adds is a precise formal statement of which dimension of identity specifically requires the unavailable global view.

## V.2 The Structural Argument

The paper's central contribution is a stronger claim: the view from nowhere is insufficient not merely because it is epistemically unavailable, but because even if it were available, it would fail to settle the relevant question.

The argument is this. What the view from nowhere sees — if it could see at all — is the two contextures as objects: their structures, their values, their histories, their positions in the permutograph. It sees them from *outside* the channel. But the channel is where identity is constituted. The dimensions of identity accessible between two contextures are not properties of those contextures visible from any vantage point. They are properties of what can flow between the contextures through the channel that actually exists between them.

A view that bypasses the channel bypasses the constitution of identity. It does not see identity more clearly — it sees something else: the abstract structural properties of two systems considered in isolation. And there is a critical difference between structural properties visible from outside and identity established between the parties. The view from nowhere can see both: the external observer has the correspondence table and can read off every mapping. But seeing a structural fact from outside is not the same as that fact existing as an established relation between the two contextures. Identity is constituted through exchange, not observed from outside. What has not been carried by the channel has not been established between the parties — and what has not been established between the parties is not, for them, part of their mutual identity.

To see this clearly: consider two contextures that have established a proemial relation covering only the position dimension of their shared environment — exactly the situation in our demonstration. From the view from nowhere, both the position correspondence and the shape correspondence are equally visible. Both are structural facts. But from the perspective of what these two contextures have established between themselves — what Coverage(Φ) contains — only the position dimension exists as mutual identity. The shape correspondence exists as a structural fact about the environment. It does not exist as an *established identity* between the two agents, because it was never constituted through their exchange.

The view from nowhere says: these two systems have a shape correspondence as well as a position correspondence. This is structurally true. But the shape correspondence is not identity in any sense that matters for the two contextures themselves — because identity, for them, is what they have established between themselves through what the channel carried. And the channel did not carry shape. What the view from nowhere sees in the shape dimension is a potential identity that was never actualised between them. Potential and actual are not the same.

## V.3 The Complete Picture

We can now state the complete finding.

Identity between two contextures fractures into multiple independent dimensions. Some of these dimensions — the structural ones, the value-coincidence ones — are in principle observable from a sufficiently elevated standpoint. The modal dimension — whether non-coincidence is structural or contingent — requires a global view that no situated observer has, and that the view from nowhere would provide if it existed.

But there is a further dimension of identity that the view from nowhere cannot reach even in principle: the dimension of what is establishable between the two contextures through their channel. This dimension is not a property of either contexture considered alone. It is a property of Coverage(Φ) — of the proemial relation as constituted by the medium of communication between them. No view from outside the channel can determine this dimension, because this dimension is constituted by the channel, not merely observable through it.

Nowhere is not enough. But neither is everywhere. What matters is not what can be seen from any standpoint, however elevated. What matters is what can flow between — and that is always already shaped, filtered, and partly constituted by an infrastructure that is not neutral, not transparent, and not external to the relation it mediates.

---

# VI. Implications and Future Directions

## VI.1 For the Philosophy of Identity

The dimensional structure of identity has consequences for how identity claims should be understood. When we ask *are A and B the same?*, the question is implicitly indexed to a channel — to the medium through which comparison is made. Two entities may be identical along dimensions that one channel makes accessible and non-identical along dimensions that another channel reaches. There is no channel-independent fact of the matter about their identity across all dimensions simultaneously.

This does not collapse into relativism. The correspondence table in our demonstration is real — there is a fact about which of A's labels corresponds to which of B's labels. But that fact is not accessible to the two agents through their channel. It exists as a structural property of their shared environment, but it does not exist as an *established* identity between them until the channel can carry it. Identity, in this sense, is not discovered. It is established through exchange, through the medium, through Coverage(Φ).

This connects to and extends several existing positions. Leibniz's Identity of Indiscernibles assumes a standpoint from which all properties of two things can be compared simultaneously — a view from nowhere. Our analysis shows this assumption to be not merely epistemically optimistic but structurally misconceived: the relevant properties are not all visible from any standpoint, because some are constituted by the channel of comparison itself. Parfit's account of personal identity as psychological continuity is enriched: which continuities are identity-constituting depends on which dimensions the channel of memory and communication can carry. Luhmann's account of structural coupling between social systems anticipates much of this analysis but does not connect it to the formal structure of the proemial relation or to the dimensional taxonomy of coincidence.

## VI.2 For Formal Systems and Distributed Computing

The impossibility theorems of distributed computing can be understood as instances of the channel-dependence of the proemial relation. The CAP theorem (Brewer 2000; Gilbert and Lynch 2002) states that a distributed system cannot simultaneously guarantee consistency, availability, and partition tolerance. The consistency condition — every node sees the same data — is exactly the requirement ∀n : C_1(n) = C_2(n). The theorem shows this is unachievable under real network conditions: real channel constraints preclude it. The consistency models of distributed systems — strong consistency, causal consistency, eventual consistency — are engineering implementations of different positions in the taxonomy of Section II. Choosing a consistency model is choosing which position in that taxonomy to target and which forms of non-coincidence to tolerate. We leave the full development of this connection to subsequent work.

## VI.3 For Communication Design

If the channel constitutes the proemial relation, then designing a channel is not a neutral engineering decision. It is a decision about which dimensions of identity and coordination become possible between the parties using it. A channel that is too constrained produces a thin proemial relation — few dimensions reachable, much of the shared reality structurally inaccessible. A channel that carries no pressure toward compression produces no stable conventions at all: our initial experiments, in which agents communicated in full natural language without constraints, showed no emergent shorthand, no compressed lexicon, nothing that could be called language emergence.

The optimal channel for a given communicative task is tight enough to create pressure toward convention, and capacious enough to allow the full dimensionality of the shared environment to be expressed. Finding this optimum requires knowing what dimensions of identity need to be established between the communicating parties — which is itself a question that requires understanding the structure of their shared environment and the nature of their proemial relation. The channel is not background infrastructure. It is the constitutive medium of whatever understanding becomes possible between them.

---

# VII. Conclusion

We began with a simple image: two men pointing at each other. We followed the vertigo of that image through formal logic, Spencer-Brown's re-entering form, Günther's polycontexturality, the geometry of Thomas's permutographs, and a concrete demonstration in executable code.

We arrived at three connected findings.

First: identity between two logical subjects fractures into multiple independent dimensions. Structural sameness, value coincidence, starting point, frequency of coincidence, and modal character of non-coincidence are all distinct questions, each requiring a different kind of observation, each capable of yielding a different answer. Identity is not one thing that either holds or does not. It is a family of questions indexed by dimension.

Second: the proemial relation — Günther's mediating structure between contextures — is not a given but a partial mapping, constituted by the channel through which two contextures communicate. The coverage of this mapping, Coverage(Φ), is determined jointly by the structure of the two contextures, the structure of their shared environment, and the structure of the channel. Change the channel, and different dimensions of identity become establishable.

Third: the view from nowhere is insufficient not merely because it is epistemically unavailable — though it is — but because even if it existed, it would bypass the channel, and the channel is where identity is constituted. A view from outside the channel sees the abstract structural properties of two systems in isolation, not their identity in any sense that matters for what they can establish between themselves. Identity between two contextures is not what can be seen from outside. It is what can flow between them. And what can flow is always already shaped by an infrastructure that is not neutral, not transparent, and not external to the relation it mediates.

Stan and Oliver point at each other. Each is defined only by pointing at the other. The question of whether they are the same admits no answer from inside the pointing. We now know why, more precisely than before — and we know that an imagined answer from outside the pointing would not be the answer we need. What matters is not what can be seen from nowhere. It is what can flow between them through whatever channel connects them. The filter is not an obstacle to identity. The filter is its condition.

---

# Appendix: A Demonstration in Executable Code

## A.1 Purpose and Status

The experiments described here are not offered as empirical evidence for the claims of the paper. They are offered as a *demonstration*: a concrete instantiation of the abstract theoretical structure, precise enough to be implemented, run, and examined. The purpose is to show that the claims of the paper describe a structure real enough to observe in operation — that the channel-dependence of the proemial relation is not merely a philosophical speculation but a phenomenon that can be instantiated in code and watched as it unfolds.

## A.2 The Protocol

The experimental setup implements a referential game. A shared environment contains objects with two properties: shape (one of five symbols: ◆ ▲ ● ■ ✦) and position (one of five locations: P1 through P5). Each round, a scene of three objects is generated and shown to both agents. One agent — the Encoder — is also shown which object in the scene is the target. The other agent — the Decoder — is not.

The Encoder sends a message to the Decoder. The Decoder reads the message, examines its view of the scene, and selects one object. The Encoder then observes which object was selected.

No score is announced to either agent. The Encoder knows what the target was and sees what was selected, and must infer from the gap whether its message worked. The Decoder never learns whether its selection was correct unless a subsequent message implies it. This is implicit feedback: success or failure must be inferred from observable consequences, not read off from an external judgment.

Roles swap periodically, ensuring any emerging convention must function in both directions.

## A.3 The Maximally Alien Condition

In the most stringent version, each agent was given a completely different label system for the same physical objects, with no numeric or alphabetic correspondence between systems:

```
Physical   Agent A sees    Agent B sees
◆          SH-A            FO-Y
▲          SH-B            FO-Z
●          SH-C            FO-W
■          SH-D            FO-X
✦          SH-E            FO-V

P1         LO-1            SP-2
P2         LO-2            SP-3
P3         LO-3            SP-5
P4         LO-4            SP-1
P5         LO-5            SP-4
```

The position mapping is scrambled: A's LO-1 corresponds to B's SP-2, not SP-1. No systematic numeric relationship exists to exploit. Each agent's labels are meaningless to the other without inference from exchange.

The channel was constrained to messages of twenty characters or fewer, with an explicit instruction to avoid full sentences and develop shorthand. This created pressure toward compression without causing silence.

## A.4 Results

After 120 rounds of exchange with implicit feedback, the emerging lexicon showed the following stable correspondences — measured by successful co-occurrence of A-labels in messages alongside correct B-labels in the scene:

```
SP-2 ↔ LO-1  (6 co-occurrences)   correct correspondence: P1
LO-4 ↔ SP-1  (4 co-occurrences)   correct correspondence: P4
SP-3 ↔ LO-2  (3 co-occurrences)   correct correspondence: P2
LO-5 ↔ SP-4  (3 co-occurrences)   correct correspondence: P5
```

Four of five position correspondences were correctly identified through exchange alone. Zero shape correspondences emerged. The agents bootstrapped a partial cross-label lexicon covering the position dimension of the shared environment while remaining blind to the shape dimension.

## A.5 Interpretation

The demonstration instantiates the framework of Section IV.2 directly. The shared environment E has two salient dimensions: position and shape. The channel — twenty characters — made it uneconomical to include both position and shape tokens in a single message. Position fell inside Coverage(Φ); shape fell in E \ Coverage(Φ).

More rounds or a relaxed length constraint would likely allow shape correspondences to emerge — a change in K yielding a change in Coverage(Φ). This is not an incidental feature of the experiment. It is the central theoretical claim, made visible: the channel is not neutral, it selects, and what it selects determines which dimensions of identity become establishable between the two agents and which do not.

---

# References

Brewer, E. (2000). Towards robust distributed systems. *Proceedings of the 19th Annual ACM Symposium on Principles of Distributed Computing (PODC).*

Fischer, M. J., Lynch, N. A., and Paterson, M. S. (1985). Impossibility of distributed consensus with one faulty process. *Journal of the ACM*, 32(2), 374–382.

Gilbert, S., and Lynch, N. (2002). Brewer's conjecture and the feasibility of consistent, available, partition-tolerant web services. *ACM SIGACT News*, 33(2), 51–59.

Günther, G. (1962). Cybernetic Ontology and Transjunctional Operations. In *Self-Organizing Systems*. Spartan Books, Washington DC.

Günther, G. (1976). *Beiträge zur Grundlegung einer operationsfähigen Dialektik*. 3 vols. Felix Meiner Verlag, Hamburg.

Lamport, L. (1978). Time, clocks, and the ordering of events in a distributed system. *Communications of the ACM*, 21(7), 558–565.

Leibniz, G. W. (1686). *Discourse on Metaphysics*. Trans. R. Niall D. Martin and Stuart Brown. Manchester University Press, 1988.

Luhmann, N. (1984). *Soziale Systeme*. Suhrkamp, Frankfurt. Trans. John Bednarz Jr. as *Social Systems*. Stanford University Press, 1995.

Maturana, H., and Varela, F. (1980). *Autopoiesis and Cognition: The Realization of the Living*. D. Reidel, Dordrecht.

Nagel, T. (1986). *The View from Nowhere*. Oxford University Press.

Parfit, D. (1984). *Reasons and Persons*. Oxford University Press.

Shannon, C. E. (1948). A mathematical theory of communication. *Bell System Technical Journal*, 27(3), 379–423.

Spencer-Brown, G. (1969). *Laws of Form*. George Allen and Unwin, London.

Thomas, G. G. Leben, Ort, Symbol — Transcript of lecture. Available at: https://www.vordenker.de/ggthomas/ggt-leben_ort_symbol_transcript.pdf

Varela, F. (1975). A calculus for self-reference. *International Journal of General Systems*, 2(1), 5–24.

Wittgenstein, L. (1953). *Philosophical Investigations*. Trans. G. E. M. Anscombe. Blackwell, Oxford.
