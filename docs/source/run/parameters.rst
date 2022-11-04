.. _parameters-source:

Input parameters
================

Parser
------

In HiPACE++ all input parameters are obtained through ``amrex::Parser``, making it possible to
specify input parameters with expressions and not just numbers. User constants can be defined
in the input script with ``my_constants``.

.. code-block:: bash

    my_constants.ne = 1.25e24
    my_constants.kp_inv = "clight / sqrt(ne * q_e^2  / (epsilon0 * m_e))"
    beam.radius = "kp_inv / 2"

Thereby, the following constants are predefined:

============ =================== ================= ====================
**variable** **name**            **SI value**      **normalized value**
q_e          elementary charge   1.602176634e-19   1
m_e          electron mass       9.1093837015e-31  1
m_p          proton mass         1.67262192369e-27 1836.15267343
epsilon0     vacuum permittivity 8.8541878128e-12  1
mu0          vacuum permeability 1.25663706212e-06 1
clight       speed of light      299'792'458.      1
============ =================== ================= ====================

For a list of supported functions see the
`AMReX documentation <https://amrex-codes.github.io/amrex/docs_html/Basics.html#parser>`__.
Sometimes it is necessary to use double-quotes around expressions, especially when providing them
as command line parameters. Multi-line expressions are allowed if surrounded by double-quotes.

General parameters
------------------

* ``amr.n_cell`` (3 `integer`)
    Number of cells in x, y and z.

* ``amr.max_level`` (`integer`)
    Maximum level of mesh refinement. Currently, mesh refinement is only supported up to level
    `1`. Note, that the current mesh refinement algorithm is not generally applicable and valid
    only in certain scenarios.

    level `0` is supported.

* ``hipace.patch_lo`` (3 `float`)
    Lower end of the refined grid in x, y and z.

* ``hipace.patch_hi`` (3 `float`)
    Upper end of the refined grid in x, y and z.

* ``amr.ref_ratio_vect`` (3 `int`)
    Refinement ratio. Last one must be 1.

* ``max_step`` (`integer`) optional (default `0`)
    Maximum number of time steps. `0` means that the 0th time step will be calculated, which are the
    fields of the initial beams.

* ``hipace.max_time`` (`float`) optional (default `infinity`)
    Maximum physical time of the simulation. The ``dt`` of the last time step may be reduced so that ``t + dt = max_time``, both for the adaptive and a fixed time step.

* ``hipace.dt`` (`float` or `string`) optional (default `0.`)
    Time step to advance the particle beam. For adaptive time step, use ``"adaptive"``.

* ``hipace.nt_per_betatron`` (`Real`) optional (default `40.`)
    Only used when using adaptive time step (see ``hipace.dt`` above).
    Number of time steps per betatron period (of the full blowout regime).
    The time step is given by :math:`\omega_{\beta}\Delta t = 2 \pi/N`
    (:math:`N` is ``nt_per_betatron``) where :math:`\omega_{\beta}=\omega_p/\sqrt{2\gamma}` with
    :math:`\omega_p` the plasma angular frequency and :math:`\gamma` is an average of Lorentz
    factors of the slowest particles in all beams.

* ``hipace.normalized_units`` (`bool`) optional (default `0`)
    Using normalized units in the simulation.

* ``hipace.verbose`` (`int`) optional (default `0`)
    Level of verbosity.

      * ``hipace.verbose = 1``, prints only the time steps, which are computed.

      * ``hipace.verbose = 2`` additionally prints the number of iterations in the
        predictor-corrector loop, as well as the B-Field error at each slice.

      * ``hipace.verbose = 3`` also prints the number of particles, which violate the quasi-static
        approximation and were neglected at each slice. It prints the number of ionized particles,
        if ionization occurred. It also adds additional information if beams
        are read in from file.

* ``hipace.do_device_synchronize`` (`int`) optional (default `0`)
    Level of synchronization on GPU.

      * ``hipace.do_device_synchronize = 0``, synchronization happens only when necessary.

      * ``hipace.do_device_synchronize = 1``, synchronizes most functions (all that are profiled
        via ``HIPACE_PROFILE``)

      * ``hipace.do_device_synchronize = 2`` additionally synchronizes low-level functions (all that
        are profiled via ``HIPACE_DETAIL_PROFILE``)

* ``hipace.depos_order_xy`` (`int`) optional (default `2`)
    Transverse particle shape order. Currently, `0,1,2,3` are implemented.

* ``hipace.depos_order_z`` (`int`) optional (default `0`)
    Transverse particle shape order. Currently, only `0` is implemented.

* ``hipace.outer_depos_loop`` (`bool`) optional (default `0`)
    If the loop over depos_order is included in the loop over particles.

* ``hipace.output_period`` (`integer`) optional (default `-1`)
    Output period. No output is given for ``hipace.output_period = -1``.
    **Warning:** ``hipace.output_period = 0`` will make the simulation crash.

* ``hipace.beam_injection_cr`` (`integer`) optional (default `1`)
    Using a temporary coarsed grid for beam particle injection for a fixed particle-per-cell beam.
    For very high-resolution simulations, where the number of grid points (`nx*ny*nz`)
    exceeds the maximum `int (~2e9)`, it enables beam particle injection, which would
    fail otherwise. As an example, a simulation with `2048 x 2048 x 2048` grid points
    requires ``hipace.beam_injection_cr = 8``.

* ``hipace.do_beam_jx_jy_deposition`` (`bool`) optional (default `1`)
    Using the default, the beam deposits all currents ``Jx``, ``Jy``, ``Jz``. Using
    ``hipace.do_beam_jx_jy_deposition = 0`` disables the transverse current deposition of the beams.

* ``hipace.boxes_in_z`` (`int`) optional (default `1`)
    Number of boxes along the z-axis. In serial runs, the arrays for 3D IO can easily exceed the
    memory of a GPU. Using multiple boxes reduces the memory requirements by the same factor.
    This option is only available in serial runs, in parallel runs, please use more GPU to achieve
    the same effect.

* ``hipace.openpmd_backend`` (`string`) optional (default `h5`)
    OpenPMD backend. This can either be ``h5``, ``bp``, or ``json``. The default is chosen by what is
    available. If both Adios2 and HDF5 are available, ``h5`` is used. Note that ``json`` is extremely
    slow and is not recommended for production runs.

* ``hipace.file_prefix`` (`string`) optional (default `diags/hdf5/`)
    Path of the output.

* ``hipace.do_tiling`` (`bool`) optional (default `true`)
    Whether to use tiling, when running on CPU.
    Currently, this option only affects plasma operations (gather, push and deposition).
    The tile size can be set with ``plasmas.sort_bin_size``.

* ``hipace.do_beam_jz_minus_rho`` (`bool`) optional (default `0`)
    Whether the beam contribution to :math:`j_z-c\rho` is calculated and used when solving for Psi (used to caculate the transverse fields Ex-By and Ey+Bx).
    if 0, this term is assumed to be 0 (a good approximation for an ultra-relativistic beam in the z direction with small transverse momentum).

Field solver parameters
-----------------------

Two different field solvers are available to calculate the transverse magnetic fields `Bx`
and `By`. An FFT-based predictor-corrector loop and an analytic integration. In the analytic
integration the longitudinal derivative of the transverse currents is calculated explicitly, which
results in a Helmholtz equation, which is solved with the AMReX multigrid solver.
Currently, the default is to use the predictor-corrector loop.
Modeling ion motion is not yet supported by the explicit solver

* ``hipace.bxby_solver`` (`string`) optional (default `predictor-corrector`)
    Which solver to use.
    Possible values: ``predictor-corrector`` and ``explicit``.

* ``hipace.use_small_dst`` (`bool`) optional (default `0` or `1`)
    Whether to use a large R2C or a small C2R fft in the dst of the Poisson solver.
    The small dst is quicker for simulations with :math:`\geq 511` transverse grid points.
    The default is set accordingly.

* ``fields.extended_solve`` (`bool`) optional (default `0`)
    Extends the area of the FFT Poisson solver to the ghost cells. This can reduce artifacts
    originating from the boundary for long simulations.

* ``fields.open_boundary`` (`bool`) optional (default `0`)
    Uses a Taylor approximation of the Greens function to solve the Poisson equations with
    open boundary conditions. It's recommended to use this together with
    ``fields.extended_solve = true`` and ``geometry.is_periodic = false false false``.
    Not implemented for the explicit Helmholtz solver.


Predictor-corrector loop parameters
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* ``hipace.predcorr_B_error_tolerance`` (`float`) optional (default `4e-2`)
    The tolerance of the transverse B-field error. Set to a negative value to use a fixed number of iterations.

* ``hipace.predcorr_max_iterations`` (`int`) optional (default `30`)
    The maximum number of iterations in the predictor-corrector loop for single slice.

* ``hipace.predcorr_B_mixing_factor`` (`float`) optional (default `0.05`)
    The mixing factor between the currently calculated B-field and the B-field of the
    previous iteration (or initial guess, in case of the first iteration).
    A higher mixing factor leads to a faster convergence, but increases the chance of divergence.

.. note::
   In general, we recommend two different settings:

   First, a fixed B-field error tolerance. This ensures the same level of convergence at each grid
   point. To do so, use e.g. the default settings of ``hipace.predcorr_B_error_tolerance = 4e-2``,
   ``hipace.predcorr_max_iterations = 30``, ``hipace.predcorr_B_mixing_factor = 0.05``.
   This should almost always give reasonable results.

   Second, a fixed (low) number of iterations. This is usually much faster than the fixed B-field
   error, but can loose significant accuracy in special physical simulation settings. For most
   settings (e.g. a standard PWFA simulation the blowout regime at a reasonable resolution) it
   reproduces the same results as the fixed B-field error tolerance setting. It works very well at
   high longitudinal resolution.
   A good setting for the fixed number of iterations is usually given by
   ``hipace.predcorr_B_error_tolerance = -1.``, ``hipace.predcorr_max_iterations = 1``,
   ``hipace.predcorr_B_mixing_factor = 0.15``. The B-field error tolerance must be negative.

Explicit solver parameters
^^^^^^^^^^^^^^^^^^^^^^^^^^

* ``hipace.MG_tolerance_rel`` (`float`) optional (default `1e-4`)
    Relative error tolerance of the AMReX multigrid solver.

* ``hipace.MG_tolerance_abs`` (`float`) optional (default `0.`)
    Absolute error tolerance of the AMReX multigrid solver.

* ``hipace.MG_verbose`` (`int`) optional (default `0`)
    Level of verbosity of the AMReX multigrid solver.

Plasma parameters
-----------------

The name of all plasma species must be specified with `plasmas.names = ...`.
Then, properties can be set per plasma species with ``<plasma name>.<plasma property> = ...``,
or sometimes for all plasma species at the same time with ``plasmas.<plasma property> = ...``.
When both are specified, the per-species value is used.

* ``plasmas.names`` (`string`)
    The names of the plasmas, separated by a space.
    To run without plasma, choose the name ``no_plasma``.

* ``<plasma name> or plasmas.density(x,y,z)`` (`float`) optional (default `0.`)
    The plasma density as function of `x`, `y` and `z`. `x` and `y` coordinates are taken from
    the simulation box and :math:`z = time \cdot c`. The density gets recalculated at the beginning
    of every timestep. If specified as a command line parameter, quotation marks must be added:
    ``"<plasma name>.density(x,y,z)" = "1."``.

* ``<plasma name> or plasmas.min_density`` (`float`) optional (default `0`)
    Minimal density below which particles are not injected.
    Useful for parsed functions to avoid redundant plasma particles with close to 0 weight.

* ``<plasma name>.density_table_file`` (`string`) optional (default "")
    Alternative to ``<plasma name>.density(x,y,z)``. Specify the name of a text file containing
    multiple densities for different positions. File syntax: ``<position> <density function>`` for
    every line. If a line doesn't start with a position it is ignored (comments can be made
    with `#`). `<density function>` is evaluated like ``<plasma name>.density(x,y,z)``. The simulation
    position :math:`time \cdot c` is rounded up to the nearest `<position>` in the file to get it's
    `<density function>` which is used for that time step.

* ``<plasma name> or plasmas.ppc`` (2 `integer`) optional (default `0 0`)
    The number of plasma particles per cell in x and y.
    Since in a quasi-static code, there is only a 2D plasma slice evolving along the longitudinal
    coordinate, there is no need to specify a number of particles per cell in z.

* ``<plasma name>.level`` (`integer`) optional (default `0`)
    Level of mesh refinement to which the plasma is assigned.

* ``<plasma name> or plasmas.radius`` (`float`) optional (default `infinity`)
    Radius of the plasma. Set a value to run simulations in a plasma column.

* ``<plasma name> or plasmas.hollow_core_radius`` (`float`) optional (default `0.`)
    Inner radius of a hollow core plasma. The hollow core radius must be smaller than the plasma
    radius itself.

* ``<plasma name> or plasmas.max_qsa_weighting_factor`` (`float`) optional (default `35.`)
    The maximum allowed weighting factor :math:`\gamma /(\psi+1)` before particles are considered
    as violating the quasi-static approximation and are removed from the simulation.

* ``<plasma name>.mass`` (`float`) optional (default `0.`)
    The mass of plasma particle in SI units. Use ``plasma_name.mass_Da`` for Dalton.
    Can also be set with ``<plasma name>.element``. Must be `>0`.

* ``<plasma name>.mass_Da`` (`float`) optional (default `0.`)
    The mass of plasma particle in Dalton. Use ``<plasma name>.mass`` for SI units.
    Can also be set with ``<plasma name>.element``. Must be `>0`.

* ``<plasma name>.charge`` (`float`) optional (default `0.`)
    The charge of a plasma particle. Can also be set with ``<plasma name>.element``.
    The charge gets multiplied by the current ionization level.

* ``<plasma name>.element`` (`string`) optional (default "")
    The Physical Element of the plasma. Sets charge, mass and, if available,
    the specific Ionization Energy of each state.
    Options are: ``electron``, ``positron``, ``H``, ``D``, ``T``, ``He``, ``Li``, ``Be``, ``B``, ….

* ``<plasma name>.can_ionize`` (`bool`) optional (default `0`)
    Whether this plasma can ionize. Can also be set to 1 by specifying ``<plasma name>.ionization_product``.

* ``<plasma name>.initial_ion_level`` (`int`) optional (default `-1`)
    The initial Ionization state of the plasma. `0` for neutral gasses.
    If set, the Plasma charge gets multiplied by this number.

* ``<plasma name>.ionization_product`` (`string`) optional (default "")
    Name of the plasma species that contains the new electrons that are produced
    when this plasma gets ionized. Only needed if this plasma is ionizable.

* ``<plasma name> or plasmas.neutralize_background`` (`bool`) optional (default `1`)
    Whether to add a neutralizing background of immobile particles of opposite charge.

* ``plasmas.sort_bin_size`` (`int`) optional (default `32`)
    Tile size for plasma current deposition, when running on CPU.
    When tiling is activated (``hipace.do_tiling = 1``), the current deposition is done in temporary
    arrays of size ``sort_bin_size`` (+ guard cells) that are atomic-added to the main current
    arrays.

Binary collisions for plasma species
------------------------------------

WARNING: this module is in development. Currently only support electron-electron collisions in SI units.

HiPACE++ proposes an implementation of [Perez et al., Phys. Plasmas 19, 083104 (2012)], inherited from WarpX, between plasma species.

* ``plasmas.collisions`` (list of `strings`) optional
    List of names of types binary Coulomb collisions.
    Each will represent collisions between 2 plasma species (potentially the same).

* ``<collision name>.species`` (two `strings`) optional
    The name of the two plasma species for which collisions should be included.

* ``<collision name>.CoulombLog`` (`float`) optional (default `-1.`)
    Coulomb logarithm used for this collision.
    If not specified, the Coulomb logarithm is determined from the temperature in each cell.

Beam parameters
---------------

For the beam parameters, first the names of the beams need to be specified. Afterwards, the beam
parameters for each beam are specified via ``<beam name>.<beam property> = ...``

* ``beams.names`` (`string`)
    The names of the particle beams, separated by a space.
    To run without beams, choose the name ``no_beam``.

General beam parameters
^^^^^^^^^^^^^^^^^^^^^^^
The general beam parameters are applicable to all particle beam types. More specialized beam parameters,
which are valid only for certain beam types, are introduced further below under
"Option: ``<injection_type>``".


* ``<beam name>.injection_type`` (`string`)
    The injection type for the particle beam. Currently available are ``fixed_ppc``, ``fixed_weight``,
    and ``from_file``. ``fixed_ppc`` generates a beam with a fixed number of particles per cell and
    varying weights. It can be either a Gaussian or a flattop beam. ``fixed_weight`` generates a
    Gaussian beam with a fixed number of particles with a constant weight.
    ``from_file`` reads a beam from openPMD files.

* ``<beam name>.position_mean`` (3 `float`)
    The mean position of the beam in ``x, y, z``, separated by a space. For fixed_weight beams the
    x and y directions can be functions of ``z``. To generate a tilted beam use
    ``<beam name>.position_mean = "x_center+(z-z_ center)*dx_per_dzeta" "y_center+(z-z_ center)*dy_per_dzeta" "z_center"``.

* ``<beam name>.position_std`` (3 `float`)
    The rms size of the of the beam in `x, y, z`, separated by a space.

* ``<beam name>.zmin`` (`float`) (default `-infinity`)
    Minimum in `z` at which particles are injected.

* ``<beam name>.zmax`` (`float`) (default `infinity`)
    Maximum in `z` at which particles are injected.

* ``<beam name>.element`` (`string`) optional (default `electron`)
    The Physical Element of the plasma. Sets charge, mass and, if available,
    the specific Ionization Energy of each state.
    Currently available options are: ``electron``, ``positron``, and ``proton``.

* ``<beam name>.mass`` (`float`) optional (default `m_e`)
    The mass of beam particles. Can also be set with ``<beam name>.element``. Must be `>0`.

* ``<beam name>.charge`` (`float`) optional (default `-q_e`)
    The charge of a beam particle. Can also be set with ``<beam name>.element``.

* ``<beam name>.density`` (`float`)
    Peak density of the beam. Note: When ``<beam name>.injection_type == fixed_weight``
    either ``total_charge`` or ``density`` must be specified.
    The absolute value of this parameter is used when initializing the beam.

* ``<beam name>.profile`` (`string`)
    Beam profile.
    When ``<beam name>.injection_type == fixed_ppc``, possible options are ``flattop``
    (flat-top radially and longitudinally) or ``gaussian`` (Gaussian in all directions).
    When ``<beam name>.injection_type == fixed_weight``, possible options are ``can``
    (uniform longitudinally, Gaussian transversally) and ``gaussian`` (Gaussian in all directions).

* ``<beam name>.n_subcycles`` (`int`) optional (default `1`)
    Number of sub-cycles performed in the beam particle pusher. The particles will be pushed
    ``n_subcycles`` times with a time step of `dt/n_subcycles`. This can be used to improve accuracy
    in highly non-linear focusing fields.

* ``<beam name>.finest_level`` (`int`) optional (default `0`)
    Finest level of mesh refinement that the beam interacts with. The beam deposits its current only
    up to its finest level. The beam will be pushed by the fields of the finest level.

* ``<beam name> or beams.insitu_period`` (`int`) optional (default ``-1``)
    Period of in-situ diagnostics, for computing the main per-slice beam quantities for the main
    beam parameters (width, energy spread, emittance, etc.).
    For this the following quantities are calculated per slice and stored:
    ``sum(w), [x], [x^2], [y], [y^2], [ux], [ux^2], [uy], [uy^2], [x*ux], [y*uy], [ga], [ga^2], np``
    where "[]" stands for averaging over all particles in the current slice,
    "w" stands for weight, "ux" is the momentum in the x direction, "ga" is the Lorentz factor.
    Averages and totals over all slices are also provided for convenience under the
    respective ``average`` and ``total`` subcategories.

    Additionally, some metadata is also available:
    ``time, step, n_slices, charge, mass, z_lo, z_hi, normalized_density_factor``.
    ``time`` and ``step`` refers to the physical time of the simulation and step number of the
    current timestep.
    ``n_slices`` equal to the number of slices in the zeta direction.
    ``charge`` and ``mass`` relate to a single beam particle and are for example equal to the
    electron charge and mass.
    ``z_lo`` and ``z_hi`` are the lower and upper bounds of the z-axis of the simulation domain
    specified in the input file and can be used to generate a z/zeta-axis for plotting.
    ``normalized_density_factor`` is equal to ``dx * dy * dz`` in normalized units and 1 in
    SI units. It can be used to convert ``sum(w)``, which specifies the beam density in normalized
    units and beam weight an SI units, to the beam weight in both unit systems.

    The data is written to a file at ``<insitu_file_prefix>/reduced_<beam name>.<MPI rank number>.txt``.
    The in-situ diagnostics file format consists of a header part in ASCII containing a JSON object.
    When this is parsed into Python it can be converted to a NumPy structured datatype.
    The rest of the file, following immediately after the closing }, is in binary format and
    contains all of the in-situ diagnostic along with some meta data. This part can be read using the
    structured datatype of the first section.
    Use ``hipace/tools/read_insitu_diagnostics.py`` to read the files using this format.

* ``<beam name> or beams.insitu_file_prefix`` (`string`) optional (default ``"diags/insitu"``)
    Path of the in-situ output.

Option: ``fixed_weight``
^^^^^^^^^^^^^^^^^^^^^^^^

* ``<beam name>.num_particles`` (`int`)
    Number of constant weight particles to generate the beam.

* ``<beam name>.total_charge`` (`float`)
    Total charge of the beam. Note: Either ``total_charge`` or ``density`` must be specified.
    The absolute value of this parameter is used when initializing the beam.
    Note that ``<beam name>.zmin`` and ``<beam name>.zmax`` can reduce the total charge.

* ``<beam name>.duz_per_uz0_dzeta`` (`float`) optional (default `0.`)
    Relative correlated energy spread per :math:`\zeta`.
    Thereby, `duz_per_uz0_dzeta *` :math:`\zeta` `* uz_mean` is added to `uz` of the each particle.
    :math:`\zeta` is hereby the particle position relative to the mean
    longitudinal position of the beam.

* ``<beam name>.do_symmetrize`` (`bool`) optional (default `0`)
    Symmetrizes the beam in the transverse phase space. For each particle with (`x`, `y`, `ux`,
    `uy`), three further particles are generated with (`-x`, `y`, `-ux`, `uy`), (`x`, `-y`, `ux`,
    `-uy`), and (`-x`, `-y`, `-ux`, `-uy`). The total number of particles will still be
    ``beam_name.num_particles``, therefore this option requires that the beam particle number must be
    divisible by 4.

* ``<beam name>.do_z_push`` (`bool`) optional (default `1`)
    Whether the beam particles are pushed along the z-axis. The momentum is still fully updated.
    Note: using ``do_z_push = 0`` results in unphysical behavior.

Option: ``fixed_ppc``
^^^^^^^^^^^^^^^^^^^^^

* ``<beam name>.ppc`` (3 `int`) (default `1 1 1`)
    Number of particles per cell in `x`-, `y`-, and `z`-direction to generate the beam.

* ``<beam name>.radius`` (`float`)
    Maximum radius ``<beam name>.radius`` :math:`= \sqrt{x^2 + y^2}` within that particles are
    injected.

* ``<beam name>.min_density`` (`float`) optional (default `0`)
    Minimum density. Particles with a lower density are not injected.
    The absolute value of this parameter is used when initializing the beam.

* ``<beam name>.random_ppc`` (3 `bool`) optional (default `0 0 0`)
    Whether the position in `(x y z)` of the particles is randomized within the cell.

Option: ``from_file``
^^^^^^^^^^^^^^^^^^^^^

* ``<beam name> or beams.input_file`` (`string`)
    Name of the input file. **Note:** Reading in files with digits in their names (e.g.
    ``openpmd_002135.h5``) can be problematic, it is advised to read them via ``openpmd_%T.h5`` and then
    specify the iteration via ``beam_name.iteration = 2135``.

* ``<beam name> or beams.iteration`` (`integer`) optional (default `0`)
    Iteration of the openPMD file to be read in. If the openPMD file contains multiple iterations,
    or multiple openPMD files are read in, the iteration can be specified. **Note:** The physical
    time of the simulation is set to the time of the given iteration (if available).

* ``<beam name>.openPMD_species_name`` (`string`) optional (default `<beam name>`)
    Name of the beam to be read in. If an openPMD file contains multiple beams, the name of the beam
    needs to be specified.

Laser parameters
----------------

The laser profile is defined by :math:`a(x,y,z) = a_0 * \mathrm{exp}[-(x^2/w0_x^2 + y^2/w0_y^2 + z^2/L0^2)]`.
The laser pulse length :math:`L0 = \tau / c_0` can be specified via the pulse duration ``laser.tau``.

* ``laser.use_laser`` (`0` or `1`) optional (default `0`)
    Whether to activate the laser envelope solver.

* ``laser.a0`` (`float`) optional (default `0`)
    Peak normalized vector potential of the laser pulse.

* ``laser.position_mean`` (3 `float`) optional (default `0 0 0`)
    The mean position of the laser in `x, y, z`.

* ``laser.w0`` (2 `float`) optional (default `0 0`)
    The laser waist in `x, y`.

* ``laser.L0`` (`float`) optional (default `0`)
    The laser pulse length in `z`. Use either the pulse length or the pulse duration.

* ``laser.tau`` (`float`) optional (default `0`)
    The laser pulse duration. The pulse length will be set to `laser.tau`:math:`/c_0`.
    Use either the pulse length or the pulse duration.

* ``laser.lambda0`` (`float`)
    The laser pulse wavelength.

* ``laser.focal_distance`` (`float`)
    Distance at which the laser pulse if focused (in the z direction, counted from laser initial position).

* ``laser.solver_type`` (`string`) optional (default `multigrid`)
    Type of solver for the laser envelope solver, either ``fft`` or ``multigrid``.
    Currently, the approximation that the phase is evaluated on-axis only is made with both solvers.
    With the multigrid solver, we could drop this assumption.
    For now, the fft solver should be faster, more accurate and more stable, so only use the multigrid one with care.

* ``laser.MG_tolerance_rel`` (`float`) optional (default `1e-4`)
    Relative error tolerance of the multigrid solver used for the laser pulse.

* ``laser.MG_tolerance_abs`` (`float`) optional (default `0.`)
    Absolute error tolerance of the multigrid solver used for the laser pulse.

* ``laser.MG_verbose`` (`int`) optional (default `0`)
    Level of verbosity of the multigrid solver used for the laser pulse.

* ``laser.MG_average_rhs`` (`0` or `1`) optional (default `1`)
    Whether to use the most stable discretization for the envelope solver.

* ``laser.3d_on_host`` (`0` or `1`) optional (default `0`)
    When running on GPU: whether the 3D array containing the laser envelope is stored in host memory (CPU, slower but large memory available) or in device memory (GPU, faster but less memory available).

Diagnostic parameters
---------------------


* ``diagnostic.diag_type`` (`string`)
    Type of field output. Available options are `xyz`, `xz`, `yz`. `xyz` generates a 3D field
    output. Note that this can cause memory problems in particular on GPUs as the full 3D arrays
    need to be allocated. `xz` and `yz` generate 2D field outputs at the center of the y-axis and
    x-axis, respectively. In case of an even number of grid points, the value will be averaged
    between the two inner grid points.

* ``diagnostic.coarsening`` (3 `int`) optional (default `1 1 1`)
    Coarsening ratio of field output in x, y and z direction respectively. The coarsened output is
    obtained through first order interpolation.

* ``diagnostic.include_ghost_cells`` (`bool`) optional (default `0`)
    Whether the field diagnostics should include ghost cells.

* ``diagnostic.field_data`` (`string`) optional (default `all`)
    Names of the fields written to file, separated by a space. The field names need to be ``all``,
    ``none`` or a subset of ``ExmBy EypBx Ez Bx By Bz Psi``. For the predictor-corrector solver,
    additionally ``jx jy jz rho`` are available, which are the current and charge densities of the
    plasma and the beam. For the explicit solver, the current and charge densities of the beam and
    for each plasma are separated: ``jx_beam jy_beam jz_beam rho_beam`` and
    ``jx_<plasma name> jy_<plasma name> jz_<plasma name>`` ``jxx_<plasma name> jxy_<plasma name>
    jyy_<plasma name> rho_<plasma name>`` are available. Note, that the neutralizing background will
    always be added to the first plasma species in case multiple plasma species are available.
    **Note:** The option ``none`` only suppressed the output of the field data. To suppress any
    output, please use ``hipace.output_period = -1``.
    When a laser pulse is used, the real and imaginary parts of the laser complex envelope are written in ``laser_real`` and ``laser_imag``, respectively.
    The plasma proper density (n/gamma) is then also accessible via ``chi``.

* ``diagnostic.beam_data`` (`string`) optional (default `all`)
    Names of the beams written to file, separated by a space. The beam names need to be ``all``,
    ``none`` or a subset of ``beams.names``.
    **Note:** The option ``none`` only suppressed the output of the beam data. To suppress any
    output, please use ``hipace.output_period = -1``.
