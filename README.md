# Thermal-Design-of-a-Regenerative-Cooling-Duct---Georgios-Miltsios
Overview

Scramjet combustor walls see heat fluxes above 1 MW/m². One way to survive that is regenerative cooling: the fuel flows through channels in the wall and takes the heat away before it is burned. This project builds a MATLAB model of one such cooling channel. It asks three questions:

How hot does the wall get? It solves coupled conduction and convection, both steady-state and transient, with properties that change with temperature.
Which design parameters matter? It sweeps coolant mass flow, wall thickness and inlet temperature.
Will the wall last? It estimates thermal stress and Norton–Bailey creep life over repeated flight cycles.

It compares hydrogen and RP-1 (kerosene) as coolants.

Key results (baseline H₂ case)
Quantity	Value
Peak hot-wall temperature	303.1 K (29.9 °C)
Minimum coolant-side wall temperature	293.4 K
Coolant temperature rise (inlet → exit)	280.0 → 287.8 K
Peak Reynolds number	≈ 5.2 × 10⁵ (fully turbulent)
RP-1 peak wall temperature	494.2 K, so H₂ runs 191 K cooler
Parametric range of T_max (ṁ, t_wall sweep)	292.2 – 326.4 K
Peak thermal stress (H₂)	6.1 MPa
Creep (H₂)	Negligible: the wall stays far below the creep-active range

Main finding: at this heat flux, hydrogen keeps the copper-alloy wall close to room temperature. Coolant mass flow controls peak wall temperature far more than wall thickness does (a 30 K swing across the ṁ sweep against about 4 K across the t_wall sweep). RP-1 pushes the wall roughly 190 K hotter, up to the edge of the regime where CuCr1Zr starts to creep.

Validation
Check	Criterion	Result
V1 Numerical vs. closed-form two-resistance solution	< 2 %	0.11 % (T_max), 0.80 % (T_min)
V2 Mesh independence (N_z = 30 / 60 / 120)	< 2 %	0.000 % change in T_max
V3 Transient energy balance (heat in vs. coolant enthalpy rise)	< 2 %	−0.16 %
Variable vs. constant wall conductivity	—	0.02 K difference (k_w almost constant over the 293–303 K range)
Physical model
Geometry and boundary conditions
Layer / boundary	Position	Details
Hot face	x = 0	Prescribed heat flux q″ = 1.3 MW/m² (Mach 7), Neumann BC
Outer wall layer	0 – 0.5 mm	CuCr1Zr
Inner wall layer	0.5 – 1.0 mm	CuCr1Zr
Coolant face	x = 1 mm	Convection to coolant, Robin BC
Coolant channel	—	2 mm × 1 mm, ṁ = 0.0093 kg/s, T_in = 280 K
Duct length	z = 0 – 0.4 m	Uniform cross-section
Governing relations

Heat transfer to the coolant (Dittus–Boelter):

$$Nu = 0.023 \, Re^{0.8} \, Pr^{0.4}$$

Thermal stress (1-D proxy):

$$\sigma_{th} = \frac{E \, \alpha \, \Delta T_{wall}}{1 - \nu}$$

Creep (Norton–Bailey power law):

$$\dot{\varepsilon} = A \, \sigma^{n} \exp\left(-\frac{Q}{R T}\right)$$

Creep life is the number of 300 s flight cycles it takes to reach 1 % accumulated strain.

Modelling choices
Wall conduction: 1-D Fourier conduction through the thickness.
Coolant: 1-D axial energy balance along the duct.
Pressure drop: Blasius friction factor for turbulent flow, Hagen–Poiseuille for laminar.
Properties: temperature-dependent conductivity, density and specific heat for the wall, and temperature-dependent c_p, μ, k and Pr for supercritical H₂ (10 MPa) and RP-1 (5 MPa). All are evaluated at each node.
Numerical methods
Solver	Method	Notes
Steady state	Node-by-node iterative two-resistance model	Up to 10 fixed-point iterations per station, tolerance 0.05 K
Transient	Backward-Euler implicit through the thickness, Thomas algorithm (tridiagonal) at each axial station	Unconditionally stable. Uses Δt = 0.05 s, compared with ~10⁻⁴ s for an explicit scheme (Fourier number r ≈ 1670)
Coolant march	First-order upwind in z	h and Re updated every time step
Parametric study	8 × 7 grid over ṁ ∈ [0.004, 0.020] kg/s and t_wall ∈ [0.6, 1.6] mm, plus a T_in sweep	Maps of T_max, thermal stress and Δp

Grid: N_z = 60 axial nodes × N_x = 20 through-thickness nodes. The full script runs in about 6 s on a laptop.

Running the code
Open RegenCoolingDuct_final.mlx in MATLAB R2025b or later. No toolboxes are needed beyond base MATLAB.
Click Run (or press Ctrl+Enter section by section).
Results print to the command window and the figures open automatically.

To try a different design point, edit Section 0:

matlab
q_flux = 1.3e6;     % hot-gas heat flux        [W/m²]
mdot   = 0.0093;    % coolant mass flow        [kg/s]
T_cin  = 280;       % coolant inlet temperature [K]
t1 = 0.5e-3; t2 = 0.5e-3;   % wall layer thicknesses [m]
Script structure
Section	Content
0	Geometry, boundary conditions, grid
1	Temperature-dependent material & fluid properties
2	Analytical validation baseline (V1)
3 / 3b	Steady-state solver, H₂ vs RP-1, property-sensitivity figures
4	Mesh independence (V2)
5	Parametric study
6	Transient implicit solver + energy balance (V3)
7	Norton–Bailey creep and life estimate (H₂ baseline and RP-1)
8	Export of results for FEA
9–11	Figures (Figs 1–22, A1–A3, 3-D surfaces S1–S6, creep figures)
Local function	ss_solver, the steady-state conduction–convection solver reused by the validation, coolant-comparison and parametric sections
Assumptions & limitations
Single, uniform channel with heat flux on one face. No tapering, and no channel-to-channel interaction.
Dittus–Boelter is used outside its strict validity range for supercritical H₂. Real-gas effects near the pseudo-critical point are not captured.
No radiation heat transfer.
Pressure drop uses the bulk mean velocity and assumes incompressible flow.
The thermal-stress expression is a 1-D proxy, not a full FEA stress field.
The Norton–Bailey constants for CuCr1Zr come from open literature and are calibrated only above about 500 K. The H₂ wall (≈ 303 K) is well below that, so the H₂ creep figures are shown for contrast only and are not physically meaningful.
