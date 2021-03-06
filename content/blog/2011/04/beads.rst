title: Looking for Beads
slug: looking-for-beads
timestamp: Apr 12 2011 23:50
categories: examples
author: Luis Pedro Coelho <luis@luispedro.org>
---

Recently, in the `pythonvision mailing list
<http://groups.google.com/group/pythonvision>`_, Alex Liberzon asked how to
perform `normalised cross-correlation
<http://en.wikipedia.org/wiki/Cross-correlation>`__ in Python.

His data looks like the following:

.. image:: /media/files/images/blog/2011/beads_original.jpeg
   :width: 66%
   :align: center

and the problem is to detect the beads that are visible. The proposed solution
is to use normalised cross-correlation.

Here is the code to implement it. We start with a few imports::

    import numpy as np
    import mahotas
    from scipy import ndimage

Now, we load the image and take out a bead by hard-coding its coordinates::

    img = mahotas.imread('copy_cam1.jpg')
    img = img.mean(2)
    img = img.astype(float)
    bead = img[433:443,495:512]
    sy,sx = bead.shape

Here is now the code for NCC::

    def ncc(x,y):
        x = x.ravel()
        x_ = x.mean()
        sx = x.std()
        y = y.ravel()
        y_ = y.mean()
        sy = y.std()
        return np.dot(x-x_, y-y_)/sx/sy

We now only need to apply it to the whole image with ``bead`` as the second
argument. However, it is clear that it will be inefficient if ``y`` is always
the same argument to permanently compute its mean and standard deviation. So,
we normalise ``bead`` first::

    bead = bead.astype(float)
    bead = bead.ravel()
    bead -= bead.mean()
    bead /= bead.std()

    def ncc(x,y):
        x = x.ravel()
        x_ = x.mean()
        sx = x.std()
        return np.dot(x-x_, y)/sx

Finally, we apply it to the whole image::

    nc = np.zeros(img.shape, float)
    for y in xrange(img.shape[0]-sy):
        for x in xrange(img.shape[1]-sx):
            nc[y+sy//2,x+sx//2] = ncc(img[y:y+sy,x:x+sx],bead)
    nc /= sx*sy

This is a pretty bad algorithm, but it takes less than 30s on a laptop.

Here is how we display the result::

    import matplotlib.pyplot as plt
    import pymorph
    plt.imshow(
        pymorph.overlay(img.astype(np.uint8),
        pymorph.dilate(pymorph.dilate(nc > .7))))

By trial and error, we set a threshold at *0.7* and call ``pymorph.dilate``
twice to make the dots bigger.

The result looks like:

.. image:: /media/files/images/blog/2011/beads_result.jpeg
   :width: 66%
   :align: center

Not great, but we got most of the beads with very little effort.

The full code is available as a `gist <https://gist.github.com/916944>`__.

