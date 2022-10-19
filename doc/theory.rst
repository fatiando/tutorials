.. _theory:

Theorerical background
======================




About coordinate systems
------------------------

The figure bellow shows a tesseroid,
the global coordinate system (X, Y, Z),
and the local coordinate system (:math:`x,\ y,\ z`) of a point P.

.. _tess-coords:

.. figure:: _static/tesseroid-coordinates.png
    :align: center
    :width: 300px
    :figwidth: 60%

    View of a tesseroid, the integration point Q,
    the global coordinate system (X, Y, Z),
    the computation P
    and it's local coordinate system (:math:`x,\ y,\ z`).
    :math:`r,\ \phi,\ \lambda` are
    the radius, latitude, and longitude, respectively,
    of point P.
    Original image (licensed CC-BY) at doi:`10.6084/m9.figshare.1495525
    <https://doi.org/10.6084/m9.figshare.1495525>`__.

The global system has origin on the center of the Earth
and Z axis aligned with the Earth's mean rotation axis.
The X and Y axis are contained on the equatorial parallel
with X intercepting the mean Greenwich meridian
and Y completing a right-handed system.

The local system has origin on the computation point P.
It's :math:`z` axis is oriented along the radial direction
and points away from the center of the Earth.
The :math:`x` and :math:`y` axis
are contained on a plane normal to the :math:`z` axis.
:math:`x` points North and :math:`y` East.

The gravitational attraction
and gravity gradient tensor
of a tesseroid
are calculated with respect to
the local coordinate system of the computation point P.

.. warning:: The :math:`g_z` component is an exception to this.
    In order to conform with the regular convention
    of z-axis pointing toward the center of the Earth,
    this component **ONLY** is calculated with an inverted z axis.
    This way, gravity anomalies of
    tesseroids with positive density
    are positive, not negative.

References
----------

﻿Asgharzadeh, M. F., R. R. B. von Frese, H. R. Kim, T. E. Leftwich,
and J. W. Kim (2007),
Spherical prism gravity effects by Gauss-Legendre quadrature integration,
Geophysical Journal International, 169(1), 1-11,
doi:10.1111/j.1365-246X.2007.03214.x.

Grombein, T.; Seitz, K.; Heck, B. (2013), Optimized formulas for the
gravitational field of a tesseroid, Journal of Geodesy,
doi: 10.1007/s00190-013-0636-1

Li, Z., T. Hao, Y. Xu, and Y. Xu (2011), An efficient and adaptive approach for
modeling gravity effects in spherical coordinates, Journal of Applied
Geophysics, 73(3), 221–231, doi:10.1016/j.jappgeo.2011.01.004.

Nagy, D., G. Papp, and J. Benedek (2000),
The gravitational potential and its derivatives for the prism,
Journal of Geodesy, 74(7-8), 552-560, doi:10.1007/s001900000116.

Nagy, D., G. Papp, and J. Benedek (2002),
Corrections to "The gravitational potential and its derivatives for the prism,"
Journal of Geodesy, 76(8), 475-475, doi:10.1007/s00190-002-0264-7.

Smith, D. A., D. S. Robertson, and D. G. Milbert (2001),
Gravitational attraction of local crustal masses in spherical coordinates,
Journal of Geodesy, 74(11-12), 783-795, doi:10.1007/s001900000142.

Wild-Pfeiffer, F. (2008),
A comparison of different mass elements for use in gravity gradiometry,
Journal of Geodesy, 82(10), 637-653, doi:10.1007/s00190-008-0219-8.
﻿

