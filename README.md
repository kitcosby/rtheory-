# Projection Craft — Modular Book-16 Frontier Harness

This package is a continuation layer for the existing Projection Craft
reproducibility stack.  It is intentionally **not** a replacement for the
certified E8/Chevalley engine.

The earlier derivation bundle already separates exact E8 algebra from bounded
coefficient search, and requires candidate solutions to be reinserted into the
exact Hessian before BRST, positivity, and single-spin-2 promotion.  This
frontier package extends that discipline to the current quasi-linear/matched-
kernel state.

## Why this package exists

The present calculation has several logically independent unresolved gates:

1. dynamical selection/activation of the `(T_w,D)` Lorentz branch;
2. nonzero positive-degree/radial activation, including `g_PhiBB`;
3. full matched-kernel/BRST closure;
4. removal or lifting of the ten symmetric co-soldering components
   `f_(mu nu)`;
5. independently varied nondegenerate coframe rank `N4 != 0`;
6. the complete constrained healthy spectrum;
7. only afterward, mirror selection/cosmology.

A monolithic optimizer is a poor fit because a failure at one gate should be a
local no-go witness, not an invitation to tune unrelated coefficients.

## Central finite certificate: the ten symmetric f modes

Around a nondegenerate retained coframe,

    f^mu = f^mu{}_nu e^nu.

The already-derived algebraic wedge term sees only

    e_mu wedge f^mu ~ f_[mu nu].

There are six antisymmetric components and ten symmetric components.  The code
therefore treats

    Sym^2(R^4), dim = 10

as a dedicated diagnostic sector.

Export the full linearized D8/BRST constraint Jacobian with respect to all
sixteen `f_mu_nu` entries as

    cosoldering_constraint_jacobian_on_vec_f   # shape (m,16)

and the harness computes its restriction to `Sym^2(R^4)`.

The gate closes exactly when

    rank(J_sym) = 10.

If the rank is smaller, the JSON report returns an explicit basis for the
surviving symmetric directions.

This is the computational form of the current manuscript question.

## Quasi-linear regression module

`quasilinear.py` encodes only already-earned identities:

    P_cf(w)^2 = P_cf(w),

    Khat_PK(w) = P_cf(w) / sech(w),

    det Khat_PK(w) = 0,

    T_U(w) = T_PK(w) + I/4,

and the certified full spectrum

    {0^4, (1/4)^11, (3/4)^6, 1^172, (5/4)^55}.

These are regression tests.  They are not re-fit.

## D8 module

`d8.py` implements the reciprocal mixed source

    J_D = sum_i alpha_i wedge beta_i

for the ten mixed pairs.  This lets the exact E8/PCBF engine export a concrete
background and test whether the D-curvature source is nonzero and survives the
constraint system.

The package does not assume a nonzero mixed configuration.

## Matched-kernel module

The candidate first-order branch uses singular BF and BB kernels with the same
null bundle.  `matched_kernel.py` checks the kernel equality directly and never
uses the invalid finite-invertible-heavy argument on that branch.

A full-adjoint test should export

    k_bf_full
    m_bb_full

in the same basis and normalization.

## BRST module

`brst.py` represents a finite linearized complex as

    gauge parameters --G--> fields --C--> equations,

checks

    C G = 0,

and computes a gauge-fixed representative of

    ker(C) / im(G).

For the ten symmetric co-soldering fields, use

    field_dimension = 10
    target_physical_dimension = 0.

A nonzero ghost-number-zero remainder is reported, not hidden.

This is still a linearized finite certificate; nonlinear closure and
reducibility can be layered on once their operators are exported.

## Radial / positive-degree module

The exact live overlap is

    N_3875(D) = -(12/7) mu cosh(2w).

Given an actually derived `g_phi_bb`, a topological intersection number, and a
declared radial stabilizer `V0(mu)`, `radial.py` solves

    V0'(mu) = (12/7) g_phi_bb n cosh(2w)

and reports positive stationary points with positive second derivative.

Missing `g_phi_bb`, topology, or `V0` produces `OPEN`, not an invented value.

## Soldering-rank module

`soldering.py` accepts a candidate independently varied 4x4 coframe matrix and
tests its rank/determinant.  It does not manufacture a coframe from the
Projection scalar.

## Full constrained spectrum

`spectrum.py` reduces a supplied Hessian to

    ker(constraints) intersect im(gauge)^perp

and reports positive/zero/negative eigenvalue counts.

This does **not** by itself label a mode as spin-2.  The existing sector map
must supply that representation identification.

## Suggested additions to the existing NPZ export

Add these keys to the output of `projection_craft_full_pipeline.py` when they
become available:

    cosoldering_constraint_jacobian_on_vec_f
    cosoldering_gauge_map_sym
    cosoldering_constraint_map_sym
    candidate_coframe_4x4
    k_bf_full
    m_bb_full
    full_constrained_hessian
    full_constraint_map
    full_gauge_map

Do not reconstruct these with a second root ordering inside this package.  The
existing certified E8 engine remains the single source of algebraic truth.

## Run

From the directory containing this package:

    python -m projection_craft_frontier.run_frontier \
        --npz projection_craft_microscopic_reduction.npz \
        --config frontier_config.json \
        --out projection_craft_frontier_report.json

A minimal config can be

    {
      "w": 0.0,
      "epsilon_p": 1,
      "g_phi_bb": null,
      "intersection_number": null,
      "expected_physical_dimension": null
    }

Run tests with

    pytest -q projection_craft_frontier/tests

## Decision policy

Every module returns one of:

- `PASS`: the named finite certificate succeeded;
- `FAIL`: the tested candidate contradicts its own required identity;
- `OPEN`: required physical/action data are absent or unresolved;
- `CONDITIONAL`: the algebra is computed but a physical target has not been declared;
- `UNDERDETERMINED`: reserved for explicit families with insufficient data.

A `PASS` never auto-promotes a downstream gate.

## Immediate next implementation target

Do **not** add another coefficient optimizer first.

Extend the existing exact PCBF/E8 Hessian code so that, on the retained
nondegenerate X-coframe background, it exports

    d(EOM_constraints) / d(vec f)

including:

- Lorentz curvature terms from `[Y,Y]`;
- dilation terms from `[X,Y]`;
- D8 mixed-pair contributions through `J_D`;
- the moving-projector / rapidity-connection terms;
- all algebraic B equations and connection constraints that survive on the
  matched-kernel branch.

Then feed that matrix directly to

    audit_symmetric_constraint_jacobian(...).

That single rank calculation answers the current sharp question:

    are all ten f_(mu nu) components constrained/lifted,
    or does a symmetric strong-coupling sector survive?
