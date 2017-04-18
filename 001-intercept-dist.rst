***********************************************************************
NEP-000: Change default intercept distribution to CosineSimilarity(d+2)
***********************************************************************

=================  ==================================
Stage              Idea
Proposed by        Jan Gosmann <jan@hyper-world.de>
Approved by
Implemented by
Completion date
Implemented in
=================  ==================================

Context
=======

The intercept distribution has an important influence on both the noise and
distortion error in the Neural Engineering Framework. An in depth investigation
with focus on high-dimensional representations can be found in [1]_ which forms
the basis of this enhancement proposal. (Another example with respect to
multiplication is discussed in [2]_, but is not of direct relevance here.)

The choice of intercept distribution is of particular importance in the context
of the Semantic Pointer Architecture (SPA). Earlier methods of optimizing
Semantic Pointer representations [3]_ are superseded by a proper choice of the
intercept and evaluation point distributions [2]_. However, nengo_spa will
not be dependent on this enhancement proposal to make use of these
improvements 

The cosine similarity distribution is the distribution of the dot product (i.e.
cosine of the angle) between to random unit vectors. Note that it is already
provided by Nengo as ``nengo.dists.CosineSimilarity``. Derivations of this and
related distributions can be found in [3]_, [4]_, and [5]_  In particular, [5]_
shows that sampling from ``CosineSimilarity(d+2)`` is equivalent to sampling
coordinates from the uniform *d*-ball.

.. [1] TOOD
.. [2] `Gosmann, J. (2016) “Precise multiplications with the NEF” <https://github.com/ctn-archive/technical-reports/blob/master/Precise-multiplications-with-the-NEF.ipynb>`_
.. [3] `Gosmann, J. & Eliasmith, C. (2016) “Optimizing Semantic Pointer Representations for Symbol-Like Processing in Spiking Neural Networks” <http://journals.plos.org/plosone/article?id=10.1371/journal.pone.0149928>`_
.. [4] `Gosmann, J. (2015) “The Mathematics of High-Dimensional Representations in the NEF” <https://www.researchgate.net/publication/315829562_The_Mathematics_of_High-Dimensional_Representations_in_the_NEF>`_
.. [5] `Voelker, A. R., Gosmann, J. & Stewart, T. C. (2017) “Efficiently sampling vectors and coordinates from the n-sphere and n-ball” <https://www.researchgate.net/publication/312056739_Efficiently_sampling_vectors_and_coordinates_from_the_n-sphere_and_n-ball>`_

Proposal
========

Change the default intercept distribution from ``nengo.dists.Uniform(-1., 1.)``
to ``nengo.dists.CosineSimilarity(d+2)``.

Details
=======

* dependence on dimensions

Provide additional details in this section.
While you should keep other sections short,
feel free to use as much space as you want here.

You can make additional sections
--------------------------------

Which can include source code::

  "like this!"

And references to `external resources <https://github.com/nengo/>`_.

Pros and cons
=============

No proposal is perfect.
In order to best review a proposal,
please provide:

* A list of pros

And:

* A list of cons

These lists help maintainers evaluate the proposal.
Maintainers may add to these lists during the review process.


* Choice of particular distribution (Uniform is special case)

* change eval points? -> no because assumption that dimensions are independent?
* better noise scaling

* Require access to ens dim
* Harder to explain
* No theoretical argument, only empirical
* Only fully applies to spiking neurons
* Affects product network assumptions (square input space)
