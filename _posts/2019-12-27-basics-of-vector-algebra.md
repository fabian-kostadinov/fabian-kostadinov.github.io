---
layout: post
title: Basics of vector algebra
comments: true
tags: [vector, algebra, euclidean distance, cosine distance]
---
Understanding vector algebra is a prerequisite to selecting meaningful distance metrics for text embeddings. For the fun of it, let's recall some of the basics.<span class="more"></span>

# Basics of vector algebra
Let __p__ and __q__ be each a n-dimensional vector in a n-dimensional Euclidean space.

## Addition and subtraction of vectors
Addition and subtraction of vectors of equal length is quite straight forward. Both operations produce a new vector with same number of components.
![Addition and Subtraction Formula](/public/img/2019-12-27-addition-subtraction.png "Addition and Subtraction Formula")

## Norms of a vector
The "length" or "magnitude" of a vector can be defined for a vector in various ways, we must at least distinguish between the level 1 and level 2 norms.

### Level 1 Norm
__Level 1 Norm (L1)__ is the sum of the absolute values of the vector's components.
![Level 1 Norm Formula](/public/img/2019-12-27-level1-norm.png "Level 1 Norm Formula")
The level 1 norm is always a scalar number, i.e. a singular value. For real valued vectors it cannot take a negative value, and it can only be 0 when all its componenrts are 0.

### Level 2 norm (Euclidean norm)
__Level 2 Norm (L2)__ or also called __Euclidean Norm__ is the square root of the sum of the squared (absolute) values of the vector's components.
![Level 2 Norm Formula](/public/img/2019-12-27-level2-norm.png "Level 2 Norm Formula")
The level 2 norm is always a scalar number, i.e. a singular value. For real valued vectors it cannot take a negative value, and it can only be 0 when all its components are 0.

Note that in some cases the L1 norm is also written as \|__p__\| and the L2 norm as \|\|__p__\|\| without the extra subscript. The L2 norm is so common that it is often referred to simply as "the norm" without indicating that the L2 norm is referred to rather than L1 norm.

### Generalized norm
Besides L1 and L2, in principle we can also calculate L3, L4 and even L&infin; norms. We can generalize the above formulas to the following one:
![Level k Norm Formula](/public/img/2019-12-27-levelk-norm.png "Level k Norm Formula")

## Multiplication of vectors
Vectors can be multiplied in different ways, the most basic ones are the scalar multiplication and the dot product. Another relatively common multiplication is the cross product (also called vector product).

### Scalar multiplication
The __scalar multiplication__ is a multiplication of a vector __p__ with a single number _c_. Scalar multiplication always returns a vector with the same number of components as the original. Geometrically speaking this can be interpreted as a re-scaling of the vector's length (it can even make it point in the opposite direction thus returning the _inverse vector_) but beyond that does not change its fundamental directionality. 
![Scalar Multiplication Formula](/public/img/2019-12-27-scalar-multiplication.png "Scalar Multiplication Formula")
Scalar multiplication with 0 returns the somewhat boring __0__ vector: 0 __p__ = __0__.

### Element-wise multiplication
The __Hadamard product__ or also called __element-wise multiplication__ is a component by component multiplication of two vectors __p__ and __q__ with _n_ dimensions. Element-wise multiplication returns a vector with the same dimensionality as the two originals. Giving a [geometric explanation](https://math.stackexchange.com/questions/2754366/geometric-interpretation-of-element-wise-vector-multiplication) is tricky, so I will skip this here. Its definition on the other hand is quite simple.
![Element-Wise Multiplication Formula](/public/img/2019-12-27-element-wise-multiplication.png "Element-Wise Multiplication Formula")

### Dot product
The __dot product__ (also called __scalar product__) is a multiplication of two vectors __p__ and __q__ of same dimensionality. Algebraic dot product as expressed by me:

> The [algebraic] dot product of two vectors can be defined as the sum of all multiplied components of two vectors.

Geometric dot product as expressed by [Wikipedia](https://en.wikipedia.org/wiki/Multiplication_of_vectors):

> The [geometric] dot product of two vectors can be defined as the product of the magnitudes of the two vectors and the cosine of the angle [&theta;] between the two vectors.
![Dot Product Formula](/public/img/2019-12-27-dot-product.png "Dot Product Formula")

As you can see there exists both an algebraic and a geometric version of the dot product. You can find a mathematical proof online in various places that the two indeed lead to same results, i.e. algebraic dot product = geometric dot product. The dot product is very important. It contains information about the angle (&theta;) between two vectors. If they are orthogonal to each other then the dot product is 0. If they are pointing into the same direction, i.e. when they are parallel) the dot product will be as positive as it gets. If they point into the opposite direction (also parallel to each other) the dot product will be as negative as it gets. (By the way, many math libraries require the angle &theta; to be converted into a radians number before entering it as a parameter to the cosine function.)

Did you also notice what I just claimed in above formula? I wrote: __p__&#8729;__q__ = &lt;__p__,__q__&gt;. This means: The dot product of two vectors __p__&#8729;__q__ is equal to their inner product &lt;__p__,__q__&gt;. This rule generally works for the text embedding algorithms I'm familar with, but if we want to be very precise then the inner product might be defined differently than the dot product, especially if you are leaving the world of Euclidean spaces. See [here](https://math.stackexchange.com/questions/476738/difference-between-dot-product-and-inner-product) and [here](https://math.stackexchange.com/questions/3188850/alternate-inner-products-on-euclidean-space). (Don't worry, we won't dive into such territory here.)

We can also rewrite the geometric interpretation of the dot product formula to calculate the angle &theta; by its vectors:
![Angle between vectors](/public/img/2019-12-27-angle-between-vectors-formula.png "Angle between vectors")

### Cross product
The __cross product__ (also called __vector product__) is also a multiplication of two vectors of same dimensionality, but a different one than the dot product. It is only defined for vectors with at least 3 components, i.e. it cannot be applied in 1- or 2-dimensional Euclidean spaces. (The following formula gives the algebraic definition for 3 dimensions only.)
![Cross Product](/public/img/2019-12-27-cross-product-formula.png "Cross Product")
The new vector __z__ (also often denoted as __n__) is the __unit vector__ that is orthogonal to the planar area spanned by vectors __p__ and __q__. A unit vector has a magnitude (L2 norm) of 1, i.e. \|\|__z__\|\|<sub>2</sub> = 1. Multiplication with __z__ ensures that the cross product vector points into the "right" direction. The geometric version of the cross product is usually used to determine the angle &theta; between two vectors, that is not in the form written above. As it is of less importance to my concerns (text embeddings!) I don't want to go into details. You can find a [short introduction for 3D spaces here](http://tutorial.math.lamar.edu/Classes/CalcII/CrossProduct.aspx), a [nice video illustration on Wikipedia](https://en.wikipedia.org/wiki/Cross_product), and a [short discussion on how to generalize to more dimensions on Stackexchange](https://math.stackexchange.com/questions/185991/is-the-vector-cross-product-only-defined-for-3d).

We could also say that the dot product focuses interactions of vector components in the same dimension, whereas the cross product focuses on vector components in distinct dimensions. ([More explanation here](https://betterexplained.com/articles/cross-product/).)

## Division of vectors
Logically speaking, division of vectors should be the inverse to multiplication, no? Immediately questions arise. As we have seen, there are at least three different types of multiplication.

### Division by a scalar
__Division of a vector by a scalar__ is rather simple - as long as the scalar is not zero! In fact, we could redefine division by a scalar as multiplication with 1/scalar (where scalar &ne; 0). Thus:
![Division by Scalar Formula](/public/img/2019-12-27-division-by-scalar.png "Division by Scalar Formula")

### Element-wise division?
__Element-wise division__ is already more complicated. In theory, this could be achieved as long as the divisor vector does not contain any elements that are zero. In reality however this is much harder to control than when dividing by a scalar. So, we won't pursue this any further.

### Division for inverse of dot and cross product?
And, what about division as the inverse of the dot and cross product? Setting the potential problem of division by zero aside for a moment, it turns out such a thing does not exist (see e.g. [here](https://math.stackexchange.com/questions/246594/what-is-vector-division) or [here](https://www.quora.com/Can-we-divide-a-vector-by-a-vector-and-why) or [here](http://mathworld.wolfram.com/VectorDivision.html)).

## Vector unit-normalization
A (non-zero) vector's length can be unit-normalized. The result is a (rescaled) vector of same dimensionality as the original with magnitude (L2 norm) of 1. The new vector is called a "unit vector" or "normalized vector". Note that we divide each component of __p__ with the scalar __p__'s magnitude. Hence, \|\|__p&#770;__\|\|<sub>2</sub> = 1.
![Unit Vector Formula](/public/img/2019-12-27-unit-vector-formula.png "Unit Vector Formula")

# Similarity and distance measures for two vectors
When working with text embeddings a common need is to measure the (dis-) similarity or distance between two vectors. In text embeddings a very common problem is to find the nearest neighbours of a given vector: Which out of all my other vectors are the most similar to a given vector? To answer this question we first have to establish a distance or similarity measure. We will continue to operate in an Euclidean space only, many other distance measures exist in non-Euclidean spaces that are of no concern here. So, let's look into two extremely popular distance/similarity measures, the __Euclidean distance__ and the __cosine similarity__.

## Euclidean distance
Remember that in an Euclidean space we can understand a given pair of Cartesian coordinates as either coordinates, or alternatively, as vectors from the origin to the coordinates? For example, in a 2d-space where we have the coordinates _p_ = (1, 5) and _q_ = (4, 3) we could interpret the coordinates _p_ and _q_ also as vectors __p__ = ((0,0), (1,5)) and __q__ = ((0,0), (4,3)). Therefore, in Euclidean space, we can interpret the distance between p and q in two different ways - either as a distance between Cartesian coordinates _p_ and _q_ or as a distance between two vectors __p__ and __q__. Calculating the distance between two coordinates in a 2d-space is something we learned in high-school using Pythagorean maths where _a_<sup>2</sup> +_b_<sup>2</sup> =  _c_<sup>2</sup> for a triangle with a right angle betwee _a_ and _b_.

<code>distance(p, q) = sqrt( (4-(-1))^2 + (3-5)^2 ) = sqrt(29)</code>

The Euclidean distance is nothing but a generalization of this formula to _n_ dimensions. Letl __q__ - __p__ be the so called _displacement vector_ between __p__ and __q__, that is a vector pointing from coordinate _p_ to _q_. Thus:

__q__ - __p__ = (q<sub>1</sub> - p<sub>1</sub>, q<sub>2</sub> - p<sub>2</sub>, ..., q<sub>n</sub> - p<sub>n</sub>)

We have already seen how we can measure the length of a vector by it's Euclidean or L2 norm. Thus, the length or magnitude of the vector between coordinate _p_ and _q_ is nothing but the L2 norm of the displacement vector __q__ - __p__, that is \| \|__q__ - __p__ \| \|. Furthermore, as it is a length measure it does not really matter whether we calculate the distance between _p_ and _q_ or _q_ and _p_, and therefore \| \|__q__ - __p__ \| \| = \| \|__p__ - __q__ \| \|.

![Euclidean Distance Formula](/public/img/2019-12-27-euclidean-distance-formula.png "Euclidean Distance Formula")

As you can see from the formula the Euclidean distance is the square root of the inner product of __p__ - __q__ (and also of __q__ - __p__). Since we are using the dot product as the inner product, it turns out that the Euclidean distance is same as the L2 norm (Euclidean norm) \| \|__p__ - __q__ \| \|. The last two lines give a different style of writing again.

The Euclidean distance is always a non-negative number. If two vectors are identical, it is 0. The higher the Euclidean distance, the further two coordinates and thus their vectors are apart.

## Cosine similarity and cosine distance
A different distance way to measure similarity (and thus distance) between two vectors is to look solely at the angle &theta; occurring between two vectors. The definition of the cosine similarity of the angle can be derived very simply from the above mentioned definition of the dot product. The relation between cosine similarity and cosine distance is simply <code>cosine distance = 1 - cosine similarity</code>. Thus, the cosine distance is defined in a range [0, 2], whereas the cosine similarity is defined in a range [-1, 1].

![Cosine Distance Formula](/public/img/2019-12-27-cosine-distance-formula.png "Cosine Distance Formula")

| Angle | Cosine Similarity | Cosine Distance | Geometric Interpretation                                 |
|--------|---------------------|--------------------|----------------------------------------------------|
| 0°       | 1                          | 0                         | Overlapping vectors with same directionality |
| 90°     | 0                          | 1                        | Orthogonal vectors |
| 180°   | -1                         | 2                        | Vectors pointing into opposite directions |
| 270°   | 0                          | 1                        | Orthogonal vectors again |
| 360°   | 1                          | 0                        | Overlapping vectors with same directionality |

## Comparison of Euclidean and cosine distance
Whereas the Euclidean distance depends on the vectors' magnitude and also the angle between the vectors, the cosine distance only depends on the angle but not on the magnitudes. This is an important difference between the two distance metrics.

However, when we unit normalize all our vectors then the Euclidean and the cosine distance fall together! Geometrically speaking unit normalized vectors have the same (L2) magnitude of 1. Thus, the distance between their coordinates directly correlates with the angle between the vectors, and therefore the Euclidean distance and the cosine distance are correlated with each other.


# Further resources
_General introduction_:
* [http://wiki.fast.ai/index.php/Linear_Algebra_for_Deep_Learning](http://wiki.fast.ai/index.php/Linear_Algebra_for_Deep_Learning)

_Dot and cross product_:
* [Understanding the Dot Product and the Cross Product](https://www.math.ucla.edu/~josephbreen/Understanding_the_Dot_Product_and_the_Cross_Product.pdf) by Joseph Breen
* [https://betterexplained.com/articles/cross-product/](https://betterexplained.com/articles/cross-product/)
* [https://en.wikipedia.org/wiki/Cross_product](https://en.wikipedia.org/wiki/Cross_product)
* [https://www.mathsisfun.com/algebra/vectors.html](https://www.mathsisfun.com/algebra/vectors-cross-product.html)
* [https://www.mathsisfun.com/algebra/vectors-cross-product.html](https://www.mathsisfun.com/algebra/vectors-cross-product.html)

_Difference between dot product and inner product_:
* [https://math.stackexchange.com/questions/476738/difference-between-dot-product-and-inner-product](https://math.stackexchange.com/questions/476738/difference-between-dot-product-and-inner-product)

_Unit vector_:
* [Stackexchange discussion on unit vector in cross product](https://math.stackexchange.com/questions/1717682/cross-product-equation-with-sine-i-dont-understand-the-unit-vector-n)
* [Another Stackexchange discussion on the role of the unit vector in cross product](https://math.stackexchange.com/questions/2169077/why-does-the-cross-product-have-n-in-its-formula-a-times-b-vert-a-vert-v)
* [https://www.khanacademy.org/computing/computer-programming/programming-natural-simulations/programming-vectors/a/vector-magnitude-normalization](https://www.khanacademy.org/computing/computer-programming/programming-natural-simulations/programming-vectors/a/vector-magnitude-normalization)

_Eudlicean versus cosine distance_:
* [https://cmry.github.io/notes/euclidean-v-cosine](https://cmry.github.io/notes/euclidean-v-cosine)
* [Euclidean and cosine distance for unit-normalized vectors](http://mlwiki.org/index.php/Cosine_Similarity)
