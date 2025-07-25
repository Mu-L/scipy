Linear Algebra (`scipy.linalg`)
===============================

.. sectionauthor:: Travis E. Oliphant

.. currentmodule:: scipy

When SciPy is built using the optimized ATLAS LAPACK and BLAS
libraries, it has very fast linear algebra capabilities. If you dig
deep enough, all of the raw LAPACK and BLAS libraries are available
for your use for even more speed. In this section, some easier-to-use
interfaces to these routines are described.

All of these linear algebra routines expect an object that can be
converted into a 2-D array. The output of these routines is
also a 2-D array.

scipy.linalg vs numpy.linalg
----------------------------

.. TODO: replace numpy.linalg HTML link with `numpy.linalg` once NumPy updates doc

:mod:`scipy.linalg` contains all the functions in
`numpy.linalg <https://www.numpy.org/devdocs/reference/routines.linalg.html>`__.
plus some other more advanced ones not contained in ``numpy.linalg``.

Another advantage of using ``scipy.linalg`` over ``numpy.linalg`` is that
it is always compiled with BLAS/LAPACK support, while for NumPy this is
optional. Therefore, the SciPy version might be faster depending on how
NumPy was installed.

Therefore, unless you don't want to add ``scipy`` as a dependency to
your ``numpy`` program, use ``scipy.linalg`` instead of ``numpy.linalg``.


numpy.matrix vs 2-D numpy.ndarray
---------------------------------

The classes that represent matrices, and basic operations, such as
matrix multiplications and transpose are a part of ``numpy``.
For convenience, we summarize the differences between :class:`numpy.matrix`
and :class:`numpy.ndarray` here.

``numpy.matrix`` is matrix class that has a more convenient interface
than ``numpy.ndarray`` for matrix operations. This class supports, for
example, MATLAB-like creation syntax via the semicolon, has matrix
multiplication as default for the ``*`` operator, and contains ``I``
and ``T`` members that serve as shortcuts for inverse and transpose:

    >>> import numpy as np
    >>> A = np.asmatrix('[1 2;3 4]')
    >>> A
    matrix([[1, 2],
            [3, 4]])
    >>> A.I
    matrix([[-2. ,  1. ],
            [ 1.5, -0.5]])
    >>> b = np.asmatrix('[5 6]')
    >>> b
    matrix([[5, 6]])
    >>> b.T
    matrix([[5],
            [6]])
    >>> A*b.T
    matrix([[17],
            [39]])

Despite its convenience, the use of the ``numpy.matrix`` class is
discouraged, since it adds nothing that cannot be accomplished
with 2-D ``numpy.ndarray`` objects, and may lead to a confusion of which class
is being used. For example, the above code can be rewritten as:

    >>> import numpy as np
    >>> from scipy import linalg
    >>> A = np.array([[1,2],[3,4]])
    >>> A
    array([[1, 2],
          [3, 4]])
    >>> linalg.inv(A)
    array([[-2. ,  1. ],
          [ 1.5, -0.5]])
    >>> b = np.array([[5,6]]) #2D array
    >>> b
    array([[5, 6]])
    >>> b.T
    array([[5],
          [6]])
    >>> A*b #not matrix multiplication!
    array([[ 5, 12],
          [15, 24]])
    >>> A.dot(b.T) #matrix multiplication
    array([[17],
          [39]])
    >>> b = np.array([5,6]) #1D array
    >>> b
    array([5, 6])
    >>> b.T  #not matrix transpose!
    array([5, 6])
    >>> A.dot(b)  #does not matter for multiplication
    array([17, 39])

``scipy.linalg`` operations can be applied equally to
``numpy.matrix`` or to 2D ``numpy.ndarray`` objects.


Basic routines
--------------

Finding the inverse
^^^^^^^^^^^^^^^^^^^

The inverse of a matrix :math:`\mathbf{A}` is the matrix
:math:`\mathbf{B}`, such that :math:`\mathbf{AB}=\mathbf{I}`, where
:math:`\mathbf{I}` is the identity matrix consisting of ones down the
main diagonal.  Usually, :math:`\mathbf{B}` is denoted
:math:`\mathbf{B}=\mathbf{A}^{-1}` . In SciPy, the matrix inverse of
the NumPy array, A, is obtained using :obj:`linalg.inv` ``(A)``, or
using ``A.I`` if ``A`` is a Matrix. For example, let

.. math::

    \mathbf{A} = \left[\begin{array}{ccc} 1 & 3 & 5\\ 2 & 5 & 1\\ 2 & 3 & 8\end{array}\right],

then

.. math::

    \mathbf{A^{-1}} = \frac{1}{25}
        \left[\begin{array}{ccc} -37 & 9 & 22 \\
                                  14 & 2 & -9 \\
                                  4 & -3 & 1
              \end{array}\right] = %
         \left[\begin{array}{ccc} -1.48 & 0.36 & 0.88  \\
                                   0.56 & 0.08 & -0.36 \\
                                   0.16 & -0.12 & 0.04
               \end{array}\right].

The following example demonstrates this computation in SciPy

    >>> import numpy as np
    >>> from scipy import linalg
    >>> A = np.array([[1,3,5],[2,5,1],[2,3,8]])
    >>> A
    array([[1, 3, 5],
          [2, 5, 1],
          [2, 3, 8]])
    >>> linalg.inv(A)
    array([[-1.48,  0.36,  0.88],
          [ 0.56,  0.08, -0.36],
          [ 0.16, -0.12,  0.04]])
    >>> A.dot(linalg.inv(A)) #double check
    array([[  1.00000000e+00,  -1.11022302e-16,  -5.55111512e-17],
          [  3.05311332e-16,   1.00000000e+00,   1.87350135e-16],
          [  2.22044605e-16,  -1.11022302e-16,   1.00000000e+00]])

Solving a linear system
^^^^^^^^^^^^^^^^^^^^^^^

Solving linear systems of equations is straightforward using the scipy
command :obj:`linalg.solve`. This command expects an input matrix and
a right-hand side vector. The solution vector is then computed. An
option for entering a symmetric matrix is offered, which can speed up
the processing when applicable. As an example, suppose it is desired
to solve the following simultaneous equations:

.. math::
   :nowrap:

    \begin{eqnarray*} x + 3y + 5z & = & 10 \\
                      2x + 5y + z & = & 8  \\
                      2x + 3y + 8z & = & 3
    \end{eqnarray*}

We could find the solution vector using a matrix inverse:

.. math::

    \left[\begin{array}{c} x\\ y\\ z\end{array}\right]=\left[\begin{array}{ccc} 1 & 3 & 5\\ 2 & 5 & 1\\ 2 & 3 & 8\end{array}\right]^{-1}\left[\begin{array}{c} 10\\ 8\\ 3\end{array}\right]=\frac{1}{25}\left[\begin{array}{c} -232\\ 129\\ 19\end{array}\right]=\left[\begin{array}{c} -9.28\\ 5.16\\ 0.76\end{array}\right].

However, it is better to use the linalg.solve command, which can be
faster and more numerically stable. In this case, it, however, gives the
same answer as shown in the following example:

    >>> import numpy as np
    >>> from scipy import linalg
    >>> A = np.array([[1, 2], [3, 4]])
    >>> A
    array([[1, 2],
          [3, 4]])
    >>> b = np.array([[5], [6]])
    >>> b
    array([[5],
          [6]])
    >>> linalg.inv(A).dot(b)  # slow
    array([[-4. ],
          [ 4.5]])
    >>> A.dot(linalg.inv(A).dot(b)) - b  # check
    array([[  8.88178420e-16],
          [  2.66453526e-15]])
    >>> np.linalg.solve(A, b)  # fast
    array([[-4. ],
          [ 4.5]])
    >>> A.dot(np.linalg.solve(A, b)) - b  # check
    array([[ 0.],
          [ 0.]])


Finding the determinant
^^^^^^^^^^^^^^^^^^^^^^^

The determinant of a square matrix :math:`\mathbf{A}` is often denoted
:math:`\left|\mathbf{A}\right|` and is a quantity often used in linear
algebra. Suppose :math:`a_{ij}` are the elements of the matrix
:math:`\mathbf{A}` and let :math:`M_{ij}=\left|\mathbf{A}_{ij}\right|`
be the determinant of the matrix left by removing the
:math:`i^{\textrm{th}}` row and :math:`j^{\textrm{th}}` column from
:math:`\mathbf{A}` . Then, for any row :math:`i,`

.. math::

    \left|\mathbf{A}\right|=\sum_{j}\left(-1\right)^{i+j}a_{ij}M_{ij}.

This is a recursive way to define the determinant, where the base case
is defined by accepting that the determinant of a :math:`1\times1` matrix is the only matrix element. In SciPy the determinant can be
calculated with :obj:`linalg.det`. For example, the determinant of

.. math::

    \mathbf{A=}\left[\begin{array}{ccc} 1 & 3 & 5\\ 2 & 5 & 1\\ 2 & 3 & 8\end{array}\right]

is

.. math::
   :nowrap:

    \begin{eqnarray*} \left|\mathbf{A}\right| & = & 1\left|\begin{array}{cc} 5 & 1\\ 3 & 8\end{array}\right|-3\left|\begin{array}{cc} 2 & 1\\ 2 & 8\end{array}\right|+5\left|\begin{array}{cc} 2 & 5\\ 2 & 3\end{array}\right|\\  & = & 1\left(5\cdot8-3\cdot1\right)-3\left(2\cdot8-2\cdot1\right)+5\left(2\cdot3-2\cdot5\right)=-25.\end{eqnarray*}.

In SciPy, this is computed as shown in this example:

    >>> import numpy as np
    >>> from scipy import linalg
    >>> A = np.array([[1,2],[3,4]])
    >>> A
    array([[1, 2],
          [3, 4]])
    >>> linalg.det(A)
    -2.0


Computing norms
^^^^^^^^^^^^^^^

Matrix and vector norms can also be computed with SciPy. A wide range
of norm definitions are available using different parameters to the
order argument of :obj:`linalg.norm`. This function takes a rank-1
(vectors) or a rank-2 (matrices) array and an optional order argument
(default is 2). Based on these inputs, a vector or matrix norm of the
requested order is computed.

For vector *x*, the order parameter can be any real number including
``inf`` or ``-inf``. The computed norm is

.. math::

    \left\Vert \mathbf{x}\right\Vert =\left\{ \begin{array}{cc} \max\left|x_{i}\right| & \textrm{ord}=\textrm{inf}\\ \min\left|x_{i}\right| & \textrm{ord}=-\textrm{inf}\\ \left(\sum_{i}\left|x_{i}\right|^{\textrm{ord}}\right)^{1/\textrm{ord}} & \left|\textrm{ord}\right|<\infty.\end{array}\right.



For matrix :math:`\mathbf{A}`, the only valid values for norm are :math:`\pm2,\pm1,` :math:`\pm` inf, and 'fro' (or 'f') Thus,

.. math::

    \left\Vert \mathbf{A}\right\Vert =\left\{ \begin{array}{cc} \max_{i}\sum_{j}\left|a_{ij}\right| & \textrm{ord}=\textrm{inf}\\ \min_{i}\sum_{j}\left|a_{ij}\right| & \textrm{ord}=-\textrm{inf}\\ \max_{j}\sum_{i}\left|a_{ij}\right| & \textrm{ord}=1\\ \min_{j}\sum_{i}\left|a_{ij}\right| & \textrm{ord}=-1\\ \max\sigma_{i} & \textrm{ord}=2\\ \min\sigma_{i} & \textrm{ord}=-2\\ \sqrt{\textrm{trace}\left(\mathbf{A}^{H}\mathbf{A}\right)} & \textrm{ord}=\textrm{'fro'}\end{array}\right.

where :math:`\sigma_{i}` are the singular values of :math:`\mathbf{A}`.

Examples:

    >>> import numpy as np
    >>> from scipy import linalg
    >>> A=np.array([[1, 2], [3, 4]])
    >>> A
    array([[1, 2],
          [3, 4]])
    >>> linalg.norm(A)
    5.4772255750516612
    >>> linalg.norm(A, 'fro') # frobenius norm is the default
    5.4772255750516612
    >>> linalg.norm(A, 1) # L1 norm (max column sum)
    6.0
    >>> linalg.norm(A, -1)
    4.0
    >>> linalg.norm(A, np.inf) # L inf norm (max row sum)
    7.0


Solving linear least-squares problems and pseudo-inverses
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Linear least-squares problems occur in many branches of applied
mathematics. In this problem, a set of linear scaling coefficients is
sought that allows a model to fit the data. In particular, it is assumed
that data :math:`y_{i}` is related to data :math:`\mathbf{x}_{i}`
through a set of coefficients :math:`c_{j}` and model functions
:math:`f_{j}\left(\mathbf{x}_{i}\right)` via the model

.. math::

    y_{i}=\sum_{j}c_{j}f_{j}\left(\mathbf{x}_{i}\right)+\epsilon_{i},

where :math:`\epsilon_{i}` represents uncertainty in the data. The
strategy of least squares is to pick the coefficients :math:`c_{j}` to
minimize

.. math::

    J\left(\mathbf{c}\right)=\sum_{i}\left|y_{i}-\sum_{j}c_{j}f_{j}\left(x_{i}\right)\right|^{2}.



Theoretically, a global minimum will occur when

.. math::

    \frac{\partial J}{\partial c_{n}^{*}}=0=\sum_{i}\left(y_{i}-\sum_{j}c_{j}f_{j}\left(x_{i}\right)\right)\left(-f_{n}^{*}\left(x_{i}\right)\right)

or

.. math::
   :nowrap:

    \begin{eqnarray*} \sum_{j}c_{j}\sum_{i}f_{j}\left(x_{i}\right)f_{n}^{*}\left(x_{i}\right) & = & \sum_{i}y_{i}f_{n}^{*}\left(x_{i}\right)\\ \mathbf{A}^{H}\mathbf{Ac} & = & \mathbf{A}^{H}\mathbf{y}\end{eqnarray*},

where

.. math::

    \left\{ \mathbf{A}\right\} _{ij}=f_{j}\left(x_{i}\right).

When :math:`\mathbf{A^{H}A}` is invertible, then

.. math::

    \mathbf{c}=\left(\mathbf{A}^{H}\mathbf{A}\right)^{-1}\mathbf{A}^{H}\mathbf{y}=\mathbf{A}^{\dagger}\mathbf{y},

where :math:`\mathbf{A}^{\dagger}` is called the pseudo-inverse of
:math:`\mathbf{A}.` Notice that using this definition of
:math:`\mathbf{A}` the model can be written

.. math::

    \mathbf{y}=\mathbf{Ac}+\boldsymbol{\epsilon}.

The command :obj:`linalg.lstsq` will solve the linear least-squares
problem for :math:`\mathbf{c}` given :math:`\mathbf{A}` and
:math:`\mathbf{y}` . In addition, :obj:`linalg.pinv` will find
:math:`\mathbf{A}^{\dagger}` given :math:`\mathbf{A}.`

The following example and figure demonstrate the use of
:obj:`linalg.lstsq` and :obj:`linalg.pinv` for solving a data-fitting
problem. The data shown below were generated using the model:

.. math::

    y_{i}=c_{1}e^{-x_{i}}+c_{2}x_{i},

where :math:`x_{i}=0.1i` for :math:`i=1\ldots10` , :math:`c_{1}=5`,
and :math:`c_{2}=4.` Noise is added to :math:`y_{i}` and the
coefficients :math:`c_{1}` and :math:`c_{2}` are estimated using
linear least squares.

.. plot::
   :alt: " "

   >>> import numpy as np
   >>> from scipy import linalg
   >>> import matplotlib.pyplot as plt
   >>> rng = np.random.default_rng()

   >>> c1, c2 = 5.0, 2.0
   >>> i = np.r_[1:11]
   >>> xi = 0.1*i
   >>> yi = c1*np.exp(-xi) + c2*xi
   >>> zi = yi + 0.05 * np.max(yi) * rng.standard_normal(len(yi))

   >>> A = np.c_[np.exp(-xi)[:, np.newaxis], xi[:, np.newaxis]]
   >>> c, resid, rank, sigma = linalg.lstsq(A, zi)

   >>> xi2 = np.r_[0.1:1.0:100j]
   >>> yi2 = c[0]*np.exp(-xi2) + c[1]*xi2

   >>> plt.plot(xi,zi,'x',xi2,yi2)
   >>> plt.axis([0,1.1,3.0,5.5])
   >>> plt.xlabel('$x_i$')
   >>> plt.title('Data fitting with linalg.lstsq')
   >>> plt.show()

..   :caption: Example of linear least-squares fit

Generalized inverse
^^^^^^^^^^^^^^^^^^^

The generalized inverse is calculated using the command
:obj:`linalg.pinv`. Let :math:`\mathbf{A}` be an
:math:`M\times N` matrix, then if :math:`M>N`, the generalized
inverse is

.. math::

    \mathbf{A}^{\dagger}=\left(\mathbf{A}^{H}\mathbf{A}\right)^{-1}\mathbf{A}^{H},

while if :math:`M<N` matrix, the generalized inverse is

.. math::

    \mathbf{A}^{\#}=\mathbf{A}^{H}\left(\mathbf{A}\mathbf{A}^{H}\right)^{-1}.

In the case that :math:`M=N`, then

.. math::

    \mathbf{A}^{\dagger}=\mathbf{A}^{\#}=\mathbf{A}^{-1},

as long as :math:`\mathbf{A}` is invertible.


Decompositions
--------------

In many applications, it is useful to decompose a matrix using other
representations. There are several decompositions supported by SciPy.


Eigenvalues and eigenvectors
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The eigenvalue-eigenvector problem is one of the most commonly
employed linear algebra operations. In one popular form, the
eigenvalue-eigenvector problem is to find for some square matrix
:math:`\mathbf{A}` scalars :math:`\lambda` and corresponding vectors
:math:`\mathbf{v}`, such that

.. math::

    \mathbf{Av}=\lambda\mathbf{v}.

For an :math:`N\times N` matrix, there are :math:`N` (not necessarily
distinct) eigenvalues --- roots of the (characteristic) polynomial

.. math::

    \left|\mathbf{A}-\lambda\mathbf{I}\right|=0.

The eigenvectors, :math:`\mathbf{v}`, are also sometimes called right
eigenvectors to distinguish them from another set of left eigenvectors
that satisfy

.. math::

    \mathbf{v}_{L}^{H}\mathbf{A}=\lambda\mathbf{v}_{L}^{H}

or

.. math::

    \mathbf{A}^{H}\mathbf{v}_{L}=\lambda^{*}\mathbf{v}_{L}.

With its default optional arguments, the command :obj:`linalg.eig`
returns :math:`\lambda` and :math:`\mathbf{v}.` However, it can also
return :math:`\mathbf{v}_{L}` and just :math:`\lambda` by itself (
:obj:`linalg.eigvals` returns just :math:`\lambda` as well).

In addition, :obj:`linalg.eig` can also solve the more general eigenvalue problem

.. math::
   :nowrap:

    \begin{eqnarray*} \mathbf{Av} & = & \lambda\mathbf{Bv}\\ \mathbf{A}^{H}\mathbf{v}_{L} & = & \lambda^{*}\mathbf{B}^{H}\mathbf{v}_{L}\end{eqnarray*}

for square matrices :math:`\mathbf{A}` and :math:`\mathbf{B}.` The
standard eigenvalue problem is an example of the general eigenvalue
problem for :math:`\mathbf{B}=\mathbf{I}.` When a generalized
eigenvalue problem can be solved, it provides a decomposition of
:math:`\mathbf{A}` as

.. math::

    \mathbf{A}=\mathbf{BV}\boldsymbol{\Lambda}\mathbf{V}^{-1},

where :math:`\mathbf{V}` is the collection of eigenvectors into
columns and :math:`\boldsymbol{\Lambda}` is a diagonal matrix of
eigenvalues.

By definition, eigenvectors are only defined up to a constant scale
factor. In SciPy, the scaling factor for the eigenvectors is chosen so
that :math:`\left\Vert \mathbf{v}\right\Vert
^{2}=\sum_{i}v_{i}^{2}=1.`

As an example, consider finding the eigenvalues and eigenvectors of
the matrix

.. math::

    \mathbf{A}=\left[\begin{array}{ccc} 1 & 5 & 2\\ 2 & 4 & 1\\ 3 & 6 & 2\end{array}\right].

The characteristic polynomial is

.. math::
   :nowrap:

    \begin{eqnarray*} \left|\mathbf{A}-\lambda\mathbf{I}\right| & = & \left(1-\lambda\right)\left[\left(4-\lambda\right)\left(2-\lambda\right)-6\right]-\\  &  & 5\left[2\left(2-\lambda\right)-3\right]+2\left[12-3\left(4-\lambda\right)\right]\\  & = & -\lambda^{3}+7\lambda^{2}+8\lambda-3.\end{eqnarray*}

The roots of this polynomial are the eigenvalues of :math:`\mathbf{A}`:

.. math::
   :nowrap:

    \begin{eqnarray*} \lambda_{1} & = & 7.9579\\ \lambda_{2} & = & -1.2577\\ \lambda_{3} & = & 0.2997.\end{eqnarray*}

The eigenvectors corresponding to each eigenvalue can be found using
the original equation. The eigenvectors associated with these
eigenvalues can then be found.

    >>> import numpy as np
    >>> from scipy import linalg
    >>> A = np.array([[1, 2], [3, 4]])
    >>> la, v = linalg.eig(A)
    >>> l1, l2 = la
    >>> print(l1, l2)   # eigenvalues
    (-0.3722813232690143+0j) (5.372281323269014+0j)
    >>> print(v[:, 0])   # first eigenvector
    [-0.82456484  0.56576746]
    >>> print(v[:, 1])   # second eigenvector
    [-0.41597356 -0.90937671]
    >>> print(np.sum(abs(v**2), axis=0))  # eigenvectors are unitary
    [1. 1.]
    >>> v1 = np.array(v[:, 0]).T
    >>> print(linalg.norm(A.dot(v1) - l1*v1))  # check the computation
    3.23682852457e-16


Singular value decomposition
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Singular value decomposition (SVD) can be thought of as an extension of
the eigenvalue problem to matrices that are not square. Let
:math:`\mathbf{A}` be an :math:`M\times N` matrix with :math:`M` and
:math:`N` arbitrary. The matrices :math:`\mathbf{A}^{H}\mathbf{A}` and
:math:`\mathbf{A}\mathbf{A}^{H}` are square hermitian matrices [#]_ of
size :math:`N\times N` and :math:`M\times M`, respectively. It is known
that the eigenvalues of square hermitian matrices are real and
non-negative. In addition, there are at most
:math:`\min\left(M,N\right)` identical non-zero eigenvalues of
:math:`\mathbf{A}^{H}\mathbf{A}` and :math:`\mathbf{A}\mathbf{A}^{H}.`
Define these positive eigenvalues as :math:`\sigma_{i}^{2}.` The
square-root of these are called singular values of :math:`\mathbf{A}.`
The eigenvectors of :math:`\mathbf{A}^{H}\mathbf{A}` are collected by
columns into an :math:`N\times N` unitary [#]_ matrix
:math:`\mathbf{V}`, while the eigenvectors of
:math:`\mathbf{A}\mathbf{A}^{H}` are collected by columns in the
unitary matrix :math:`\mathbf{U}`, the singular values are collected
in an :math:`M\times N` zero matrix
:math:`\mathbf{\boldsymbol{\Sigma}}` with main diagonal entries set to
the singular values. Then

.. math::

    \mathbf{A=U}\boldsymbol{\Sigma}\mathbf{V}^{H}

is the singular value decomposition of :math:`\mathbf{A}.` Every
matrix has a singular value decomposition. Sometimes, the singular
values are called the spectrum of :math:`\mathbf{A}.` The command
:obj:`linalg.svd` will return :math:`\mathbf{U}` ,
:math:`\mathbf{V}^{H}`, and :math:`\sigma_{i}` as an array of the
singular values. To obtain the matrix :math:`\boldsymbol{\Sigma}`, use
:obj:`linalg.diagsvd`. The following example illustrates the use of
:obj:`linalg.svd`:

    >>> import numpy as np
    >>> from scipy import linalg
    >>> A = np.array([[1,2,3],[4,5,6]])
    >>> A
    array([[1, 2, 3],
          [4, 5, 6]])
    >>> M,N = A.shape
    >>> U,s,Vh = linalg.svd(A)
    >>> Sig = linalg.diagsvd(s,M,N)
    >>> U, Vh = U, Vh
    >>> U
    array([[-0.3863177 , -0.92236578],
          [-0.92236578,  0.3863177 ]])
    >>> Sig
    array([[ 9.508032  ,  0.        ,  0.        ],
          [ 0.        ,  0.77286964,  0.        ]])
    >>> Vh
    array([[-0.42866713, -0.56630692, -0.7039467 ],
          [ 0.80596391,  0.11238241, -0.58119908],
          [ 0.40824829, -0.81649658,  0.40824829]])
    >>> U.dot(Sig.dot(Vh)) #check computation
    array([[ 1.,  2.,  3.],
          [ 4.,  5.,  6.]])

.. [#] A hermitian matrix :math:`\mathbf{D}` satisfies :math:`\mathbf{D}^{H}=\mathbf{D}.`

.. [#] A unitary matrix :math:`\mathbf{D}` satisfies :math:`\mathbf{D}^{H}\mathbf{D}=\mathbf{I}=\mathbf{D}\mathbf{D}^{H}` so that :math:`\mathbf{D}^{-1}=\mathbf{D}^{H}.`


LU decomposition
^^^^^^^^^^^^^^^^

The LU decomposition finds a representation for the :math:`M\times N`
matrix :math:`\mathbf{A}` as

.. math::

    \mathbf{A}=\mathbf{P}\,\mathbf{L}\,\mathbf{U},

where :math:`\mathbf{P}` is an :math:`M\times M` permutation matrix (a
permutation of the rows of the identity matrix), :math:`\mathbf{L}` is
in :math:`M\times K` lower triangular or trapezoidal matrix (
:math:`K=\min\left(M,N\right)`) with unit-diagonal, and
:math:`\mathbf{U}` is an upper triangular or trapezoidal matrix. The
SciPy command for this decomposition is :obj:`linalg.lu`.

Such a decomposition is often useful for solving many simultaneous
equations where the left-hand side does not change but the right-hand
side does. For example, suppose we are going to solve

.. math::

    \mathbf{A}\mathbf{x}_{i}=\mathbf{b}_{i}

for many different :math:`\mathbf{b}_{i}`. The LU decomposition allows this to be written as

.. math::

    \mathbf{PLUx}_{i}=\mathbf{b}_{i}.

Because :math:`\mathbf{L}` is lower-triangular, the equation can be
solved for :math:`\mathbf{U}\mathbf{x}_{i}` and, finally,
:math:`\mathbf{x}_{i}` very rapidly using forward- and
back-substitution. An initial time spent factoring :math:`\mathbf{A}`
allows for very rapid solution of similar systems of equations in the
future. If the intent for performing LU decomposition is for solving
linear systems, then the command :obj:`linalg.lu_factor` should be used
followed by repeated applications of the command
:obj:`linalg.lu_solve` to solve the system for each new
right-hand side.


Cholesky decomposition
^^^^^^^^^^^^^^^^^^^^^^

Cholesky decomposition is a special case of LU decomposition
applicable to Hermitian positive definite matrices. When
:math:`\mathbf{A}=\mathbf{A}^{H}` and
:math:`\mathbf{x}^{H}\mathbf{Ax}\geq0` for all :math:`\mathbf{x}`,
then decompositions of :math:`\mathbf{A}` can be found so that

.. math::
   :nowrap:

    \begin{eqnarray*} \mathbf{A} & = & \mathbf{U}^{H}\mathbf{U}\\ \mathbf{A} & = & \mathbf{L}\mathbf{L}^{H}\end{eqnarray*},

where :math:`\mathbf{L}` is lower triangular and :math:`\mathbf{U}` is
upper triangular. Notice that :math:`\mathbf{L}=\mathbf{U}^{H}.` The
command :obj:`linalg.cholesky` computes the Cholesky
factorization. For using the Cholesky factorization to solve systems of
equations, there are also :obj:`linalg.cho_factor` and
:obj:`linalg.cho_solve` routines that work similarly to their LU
decomposition counterparts.


QR decomposition
^^^^^^^^^^^^^^^^

The QR decomposition (sometimes called a polar decomposition) works
for any :math:`M\times N` array and finds an :math:`M\times M` unitary
matrix :math:`\mathbf{Q}` and an :math:`M\times N` upper-trapezoidal
matrix :math:`\mathbf{R}`, such that

.. math::

    \mathbf{A=QR}.

Notice that if the SVD of :math:`\mathbf{A}` is known, then the QR decomposition can be found.

.. math::

    \mathbf{A}=\mathbf{U}\boldsymbol{\Sigma}\mathbf{V}^{H}=\mathbf{QR}

implies that :math:`\mathbf{Q}=\mathbf{U}` and
:math:`\mathbf{R}=\boldsymbol{\Sigma}\mathbf{V}^{H}.` Note, however,
that in SciPy independent algorithms are used to find QR and SVD
decompositions. The command for QR decomposition is :obj:`linalg.qr`.


Schur decomposition
^^^^^^^^^^^^^^^^^^^

For a square :math:`N\times N` matrix, :math:`\mathbf{A}`, the Schur
decomposition finds (not necessarily unique) matrices
:math:`\mathbf{T}` and :math:`\mathbf{Z}`, such that

.. math::

    \mathbf{A}=\mathbf{ZT}\mathbf{Z}^{H},

where :math:`\mathbf{Z}` is a unitary matrix and :math:`\mathbf{T}` is
either upper triangular or quasi upper triangular, depending on whether
or not a real Schur form or complex Schur form is requested.  For a
real Schur form both :math:`\mathbf{T}` and :math:`\mathbf{Z}` are
real-valued when :math:`\mathbf{A}` is real-valued. When
:math:`\mathbf{A}` is a real-valued matrix, the real Schur form is only
quasi upper triangular because :math:`2\times2` blocks extrude from
the main diagonal corresponding to any complex-valued
eigenvalues. The command :obj:`linalg.schur` finds the Schur
decomposition, while the command :obj:`linalg.rsf2csf` converts
:math:`\mathbf{T}` and :math:`\mathbf{Z}` from a real Schur form to a
complex Schur form. The Schur form is especially useful in calculating
functions of matrices.

The following example illustrates the Schur decomposition:

    >>> from scipy import linalg
    >>> A = np.asmatrix('[1 3 2; 1 4 5; 2 3 6]')
    >>> T, Z = linalg.schur(A)
    >>> T1, Z1 = linalg.schur(A, 'complex')
    >>> T2, Z2 = linalg.rsf2csf(T, Z)
    >>> T
    array([[ 9.90012467,  1.78947961, -0.65498528],
           [ 0.        ,  0.54993766, -1.57754789],
           [ 0.        ,  0.51260928,  0.54993766]])
    >>> T2
    array([[ 9.90012467+0.00000000e+00j, -0.32436598+1.55463542e+00j,
            -0.88619748+5.69027615e-01j],
           [ 0.        +0.00000000e+00j,  0.54993766+8.99258408e-01j,
             1.06493862+3.05311332e-16j],
           [ 0.        +0.00000000e+00j,  0.        +0.00000000e+00j,
             0.54993766-8.99258408e-01j]])
    >>> abs(T1 - T2) # different
    array([[  1.06604538e-14,   2.06969555e+00,   1.69375747e+00],  # may vary
           [  0.00000000e+00,   1.33688556e-15,   4.74146496e-01],
           [  0.00000000e+00,   0.00000000e+00,   1.13220977e-15]])
    >>> abs(Z1 - Z2) # different
    array([[ 0.06833781,  0.88091091,  0.79568503],    # may vary
           [ 0.11857169,  0.44491892,  0.99594171],
           [ 0.12624999,  0.60264117,  0.77257633]])
    >>> T, Z, T1, Z1, T2, Z2 = map(np.asmatrix,(T,Z,T1,Z1,T2,Z2))
    >>> abs(A - Z*T*Z.H)  # same
    matrix([[  5.55111512e-16,   1.77635684e-15,   2.22044605e-15],
            [  0.00000000e+00,   3.99680289e-15,   8.88178420e-16],
            [  1.11022302e-15,   4.44089210e-16,   3.55271368e-15]])
    >>> abs(A - Z1*T1*Z1.H)  # same
    matrix([[  4.26993904e-15,   6.21793362e-15,   8.00007092e-15],
            [  5.77945386e-15,   6.21798014e-15,   1.06653681e-14],
            [  7.16681444e-15,   8.90271058e-15,   1.77635764e-14]])
    >>> abs(A - Z2*T2*Z2.H)  # same
    matrix([[  6.02594127e-16,   1.77648931e-15,   2.22506907e-15],
            [  2.46275555e-16,   3.99684548e-15,   8.91642616e-16],
            [  8.88225111e-16,   8.88312432e-16,   4.44104848e-15]])


Interpolative decomposition
^^^^^^^^^^^^^^^^^^^^^^^^^^^

:mod:`scipy.linalg.interpolative` contains routines for computing the
interpolative decomposition (ID) of a matrix. For a matrix :math:`A
\in \mathbb{C}^{m \times n}` of rank :math:`k \leq \min \{ m, n \}`
this is a factorization

.. math::

  A \Pi =
  \begin{bmatrix}
   A \Pi_{1} & A \Pi_{2}
  \end{bmatrix} =
  A \Pi_{1}
  \begin{bmatrix}
   I & T
  \end{bmatrix},

where :math:`\Pi = [\Pi_{1}, \Pi_{2}]` is a permutation matrix with
:math:`\Pi_{1} \in \{ 0, 1 \}^{n \times k}`, i.e., :math:`A \Pi_{2} =
A \Pi_{1} T`. This can equivalently be written as :math:`A = BP`,
where :math:`B = A \Pi_{1}` and :math:`P = [I, T] \Pi^{\mathsf{T}}`
are the *skeleton* and *interpolation matrices*, respectively.

.. seealso:: `scipy.linalg.interpolative` --- for more information.


Matrix functions
----------------

Consider the function :math:`f\left(x\right)` with Taylor series expansion

.. math::

    f\left(x\right)=\sum_{k=0}^{\infty}\frac{f^{\left(k\right)}\left(0\right)}{k!}x^{k}.

A matrix function can be defined using this Taylor series for the
square matrix :math:`\mathbf{A}` as

.. math::

    f\left(\mathbf{A}\right)=\sum_{k=0}^{\infty}\frac{f^{\left(k\right)}\left(0\right)}{k!}\mathbf{A}^{k}.

.. note::

    While this serves as a useful representation of a matrix function, it is
    rarely the best way to calculate a matrix function. In particular, if the
    matrix is not diagonalizable, results may be inaccurate.


Exponential and logarithm functions
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The matrix exponential is one of the more common matrix functions.
The preferred method for implementing the matrix exponential is to use
scaling and a Padé approximation for :math:`e^{x}`. This algorithm is
implemented as :obj:`linalg.expm`.

The inverse of the matrix exponential is the matrix logarithm defined
as the inverse of the matrix exponential:

.. math::

    \mathbf{A}\equiv\exp\left(\log\left(\mathbf{A}\right)\right).

The matrix logarithm can be obtained with :obj:`linalg.logm`.


Trigonometric functions
^^^^^^^^^^^^^^^^^^^^^^^

The trigonometric functions, :math:`\sin`, :math:`\cos`, and
:math:`\tan`, are implemented for matrices in :func:`linalg.sinm`,
:func:`linalg.cosm`, and :obj:`linalg.tanm`, respectively. The matrix
sine and cosine can be defined using Euler's identity as

.. math::
   :nowrap:

    \begin{eqnarray*} \sin\left(\mathbf{A}\right) & = & \frac{e^{j\mathbf{A}}-e^{-j\mathbf{A}}}{2j}\\ \cos\left(\mathbf{A}\right) & = & \frac{e^{j\mathbf{A}}+e^{-j\mathbf{A}}}{2}.\end{eqnarray*}

The tangent is

.. math::

    \tan\left(x\right)=\frac{\sin\left(x\right)}{\cos\left(x\right)}=\left[\cos\left(x\right)\right]^{-1}\sin\left(x\right)

and so the matrix tangent is defined as

.. math::

    \left[\cos\left(\mathbf{A}\right)\right]^{-1}\sin\left(\mathbf{A}\right).




Hyperbolic trigonometric functions
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The hyperbolic trigonometric functions, :math:`\sinh`, :math:`\cosh`,
and :math:`\tanh`, can also be defined for matrices using the familiar
definitions:

.. math::
   :nowrap:

    \begin{eqnarray*} \sinh\left(\mathbf{A}\right) & = & \frac{e^{\mathbf{A}}-e^{-\mathbf{A}}}{2}\\ \cosh\left(\mathbf{A}\right) & = & \frac{e^{\mathbf{A}}+e^{-\mathbf{A}}}{2}\\ \tanh\left(\mathbf{A}\right) & = & \left[\cosh\left(\mathbf{A}\right)\right]^{-1}\sinh\left(\mathbf{A}\right).\end{eqnarray*}

These matrix functions can be found using :obj:`linalg.sinhm`,
:obj:`linalg.coshm`, and :obj:`linalg.tanhm`.


Arbitrary function
^^^^^^^^^^^^^^^^^^

Finally, any arbitrary function that takes one complex number and
returns a complex number can be called as a matrix function using the
command :obj:`linalg.funm`. This command takes the matrix and an
arbitrary Python function. It then implements an algorithm from Golub
and Van Loan's book_ "Matrix Computations" to compute the function applied
to the matrix using a Schur decomposition.  Note that *the function
needs to accept complex numbers* as input in order to work with this
algorithm. For example, the following code computes the zeroth-order
Bessel function applied to a matrix.

    >>> from scipy import special, linalg
    >>> rng = np.random.default_rng()
    >>> A = rng.random((3, 3))
    >>> B = linalg.funm(A, lambda x: special.jv(0, x))
    >>> A
    array([[0.06369197, 0.90647174, 0.98024544],
           [0.68752227, 0.5604377 , 0.49142032],
           [0.86754578, 0.9746787 , 0.37932682]])
    >>> B
    array([[ 0.6929219 , -0.29728805, -0.15930896],
           [-0.16226043,  0.71967826, -0.22709386],
           [-0.19945564, -0.33379957,  0.70259022]])
    >>> linalg.eigvals(A)
    array([ 1.94835336+0.j, -0.72219681+0.j, -0.22270006+0.j])
    >>> special.jv(0, linalg.eigvals(A))
    array([0.25375345+0.j, 0.87379738+0.j, 0.98763955+0.j])
    >>> linalg.eigvals(B)
    array([0.25375345+0.j, 0.87379738+0.j, 0.98763955+0.j])

Note how, by virtue of how matrix analytic functions are defined,
the Bessel function has acted on the matrix eigenvalues.


Special matrices
----------------

SciPy and NumPy provide several functions for creating special matrices
that are frequently used in engineering and science.

====================  =================================  =========================================================
Type                  Function                           Description
====================  =================================  =========================================================
block diagonal        `scipy.linalg.block_diag`          Create a block diagonal matrix from the provided arrays.
--------------------  ---------------------------------  ---------------------------------------------------------
circulant             `scipy.linalg.circulant`           Create a circulant matrix.
--------------------  ---------------------------------  ---------------------------------------------------------
companion             `scipy.linalg.companion`           Create a companion matrix.
--------------------  ---------------------------------  ---------------------------------------------------------
convolution           `scipy.linalg.convolution_matrix`  Create a convolution matrix.
--------------------  ---------------------------------  ---------------------------------------------------------
Discrete Fourier      `scipy.linalg.dft`                 Create a discrete Fourier transform matrix.
--------------------  ---------------------------------  ---------------------------------------------------------
Fiedler               `scipy.linalg.fiedler`             Create a symmetric Fiedler matrix.
--------------------  ---------------------------------  ---------------------------------------------------------
Fiedler Companion     `scipy.linalg.fiedler_companion`   Create a Fiedler companion matrix.
--------------------  ---------------------------------  ---------------------------------------------------------
Hadamard              `scipy.linalg.hadamard`            Create an Hadamard matrix.
--------------------  ---------------------------------  ---------------------------------------------------------
Hankel                `scipy.linalg.hankel`              Create a Hankel matrix.
--------------------  ---------------------------------  ---------------------------------------------------------
Helmert               `scipy.linalg.helmert`             Create a Helmert matrix.
--------------------  ---------------------------------  ---------------------------------------------------------
Hilbert               `scipy.linalg.hilbert`             Create a Hilbert matrix.
--------------------  ---------------------------------  ---------------------------------------------------------
Inverse Hilbert       `scipy.linalg.invhilbert`          Create the inverse of a Hilbert matrix.
--------------------  ---------------------------------  ---------------------------------------------------------
Leslie                `scipy.linalg.leslie`              Create a Leslie matrix.
--------------------  ---------------------------------  ---------------------------------------------------------
Pascal                `scipy.linalg.pascal`              Create a Pascal matrix.
--------------------  ---------------------------------  ---------------------------------------------------------
Inverse Pascal        `scipy.linalg.invpascal`           Create the inverse of a Pascal matrix.
--------------------  ---------------------------------  ---------------------------------------------------------
Toeplitz              `scipy.linalg.toeplitz`            Create a Toeplitz matrix.
--------------------  ---------------------------------  ---------------------------------------------------------
Van der Monde         `numpy.vander`                     Create a Van der Monde matrix.
====================  =================================  =========================================================


For examples of the use of these functions, see their respective docstrings.

Advanced Features
-----------------

Batch Support
^^^^^^^^^^^^^
Some of SciPy's linear algebra functions can process batches of scalars, 1D-, or
2D-arrays given N-d array input.
For example, a linear algebra function that typically accepts a (2D) matrix may accept
an array of shape ``(4, 3, 2)``, which it would interpret as a batch of four 3-by-2
matrices. In this case, we say that the the *core shape* of the input is (3, 2) and the
*batch shape* is ``(4,)``. Likewise, a linear algebra function that typically accepts
a (1D) vector would treat a ``(4, 3, 2)`` array as a ``(4, 3)`` batch of vectors,
in which case the *core shape* of the input is ``(2,)`` and the *batch shape* is
``(4, 3)``. The length of the core shape is also referred to as the *core dimension*.
In these cases, the final shape of the output is the batch shape of the input
concatenated with the core shape of the output (i.e., the shape of the output when
the batch shape of the input is ``()``). For more information, see :doc:`linalg_batch`.

.. _book: https://www.press.jhu.edu/books/title/10678/matrix-computations
