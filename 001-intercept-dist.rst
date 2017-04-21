***********************************************************************
NEP-001: Change default intercept distribution to CosineSimilarity(d+2)
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
multiplication is discussed in [2]_, but is not of direct relevance here.) Some
earlier experiments are documented in [3]_.

The choice of intercept distribution is of particular importance in the context
of the Semantic Pointer Architecture (SPA). Earlier methods of optimizing
Semantic Pointer representations [4]_ are superseded by a proper choice of the
intercept and evaluation point distributions [2]_. However, nengo_spa will
not be dependent on this enhancement proposal to make use of these
improvements 

The cosine similarity distribution is the distribution of the dot product (i.e.
cosine of the angle) between to random unit vectors. Note that it is already
provided by Nengo as ``nengo.dists.CosineSimilarity``. Derivations of this and
related distributions can be found in [4]_, [5]_, and [6]_. In particular, [6]_
shows that sampling from ``CosineSimilarity(d+2)`` is equivalent to sampling
coordinates from the uniform *d*-ball.

.. [1] `Gosmann, J. (2017) “The influence of the intercept distribution on representation in the NEF” <https://github.com/ctn-waterloo/internal_tech_reports/blob/master/The%20influence%20of%20the%20intercept%20distribution%20on%20representation%20in%20the%20NEF.ipynb>`_
.. [2] `Gosmann, J. (2016) “Precise multiplications with the NEF” <https://github.com/ctn-archive/technical-reports/blob/master/Precise-multiplications-with-the-NEF.ipynb>`_
.. [3] `nengo/nengo_extras#34: Improved accuracy in higher dimensions with alternate intercept distribution <https://github.com/nengo/nengo_extras/issues/34>`_
.. [4] `Gosmann, J. & Eliasmith, C. (2016) “Optimizing Semantic Pointer Representations for Symbol-Like Processing in Spiking Neural Networks” <http://journals.plos.org/plosone/article?id=10.1371/journal.pone.0149928>`_
.. [5] `Gosmann, J. (2015) “The Mathematics of High-Dimensional Representations in the NEF” <https://www.researchgate.net/publication/315829562_The_Mathematics_of_High-Dimensional_Representations_in_the_NEF>`_
.. [6] `Voelker, A. R., Gosmann, J. & Stewart, T. C. (2017) “Efficiently sampling vectors and coordinates from the n-sphere and n-ball” <https://www.researchgate.net/publication/312056739_Efficiently_sampling_vectors_and_coordinates_from_the_n-sphere_and_n-ball>`_

Proposal
========

Change the default intercept distribution from ``nengo.dists.Uniform(-1., 1.)``
to ``nengo.dists.CosineSimilarity(d+2)`` in the next major release.

Details
=======

The best intercept distribution depends on a number of factors like neuron
type, solver regularization, synapse, and more.
``CosineSimilarity(d+2)`` is not necessarily the best performing distribution in
all cases, but it is an improvement compared to the uniform distribution over
the broadest range of parameters. It leads
to an improvement with short and long synaptic time constants, as well as for
spiking and rate neurons given other parameters are at their default values.
The ``CosineSimilarity(d+2)`` intercept distribution will give worse performance
for rate neurons if the solver regularization is reduced, but as this already
requires to adjust a default parameter for optimal performance, it might be
tolerable to have two parameters that require adjustment.

Another reason that makes ``CosineSimilarity(d+2)`` a good candidate for the
default intercept distribution is that it reduces to a uniform distribution for
``d=1``.

Dependence on dimensionality
----------------------------

While changing the default of a parameter is simple implementation-wise, using
``CosineSimilarity(d+2)`` comes with a complication because the ensemble
dimensionality has to be known. Note that the dimensionality parameter of the
distribution may differ from the dimensionality that is being sampled (e.g.
intercepts are being sampled with dimensionality 1, but the ensemble might have
a different dimensionality *d*). Currently, this is not supported by Nengo. In
the following I will list a few possible approaches, but none is obviously the
right way to implement this. Other approaches might be possible.

Placeholder value
^^^^^^^^^^^^^^^^^

Instead of assigning the final distribution instance, use a placeholder value
that will be replaced during the build process with the actual distribution.

Pros:

* Easy to implement.
* No major changes necessary.

Cons:

* Makes Nengo behave slightly different in this one case.
* Requires special case code for just this one distribution in all backends.
* Does not generalize to other distributions that require the ensemble
  dimensionality.

Helper function
^^^^^^^^^^^^^^^

A helper function could be used to replace all intercept distributions in a
network.

Pros:

* No changes to the Nengo API necessary.
* Could be included in a minor release as it doesn't actually change the
  default and is thus backwards compatible.

Cons:

* If the ensemble dimensionality is changed after calling the helper function,
  the wrong dimensionality will be set for the distribution.
* The user needs to know that the function has to be called.
* Additional boilerplate when writing a model.
* Excluding certain ensemles (i.e. setting a uniform intercept distribution on
  specific ensembles) might be complicated.

Deferrables
^^^^^^^^^^^

Deferrables have been proposed and extensively discussed in
`nengo/nengo#995 <https://github.com/nengo/nengo/pull/995>`_. They provide a
mechanism to defer the final assignment of parameters in Nengo until build
time. At the time it was decided (for good reasons) to not implement
deferrables.
`This comment <https://github.com/nengo/nengo/pull/995#issuecomment-237948076>`_
tries to give a summary of the discussion, but note that several of the points
are in the context of earlier methods of optimizing Semantic Pointer
representations that are not applicable any longer. I only restate the major
applicable points here.

Pros:

* General solution

Cons:

* A lot of added complexity.
* Needs to be implemented by all backends.
* Order of final evaluation of the parameter values is undefined.

Assumptions about the distribution of represented values
--------------------------------------------------------

The improvement of accuracy with ``CosineSimilarity(d+2)`` is under the
assumption that the represented values are uniformly distributed in the
hyperball. This seems to be a reasonable default assumption, but can be
violated. For example, if the length of the input vector would be uniformly
distributed, values will cluster towards the center of the hyperball. This
can change how different distributions compare in terms of error.

It is also possible that ``CosineSimilarity(d+2)`` gives worse results for
decoding certain non-linear functions that amplify error in certain areas of
the input space. However, most “sane” non-linear functions will be more
accurately decoded if the base representation is improved.

Pros and cons
=============

Pros:

* Representations, especially high-dimensional represenations, will be more
  accurate.
* ``CosineSimilarity(d+2)`` gives a better noise scaling. The noise error is
  only proportional to O(d^(3/4)) instead of O(d).
* Virtually eliminates neurons that do not fire for any evaluation points in
  higher dimensions (that otherwise need to be simulated, but do not contribute
  anything).

Cons:

* Users are probably less familiar with the ``CosineSimilarity(d+2)``
  distribution than a uniform distribution.
* We currently have no theoretical (only an empirical) argument for
  ``CosineSimilarity(d+2)``.
* The ensemble dimensionality needs to be known for the distribution. See above
  for a discussion of potential implementations.
* Performs worse for rate neurons when also adjusting the solver regularization
  accordingly.
* Introduces a bias for certain areas into the input space distortion. Or in
  other words, it assumes inputs to be uniformly distributed. (Though, if this
  assumption is violated it does not follow that ``CosineSimilarity(d+2)``
  makes things worse, but it is possible.)
