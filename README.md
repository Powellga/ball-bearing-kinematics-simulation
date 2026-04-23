# Radial Ball-Bearing Kinematic Sculpture Simulation

A single-file, browser-based physics design tool for a magnetic ball-bearing kinematic sculpture: a wooden plate with radial channels, a ball bearing in each channel, and a neodymium disc magnet orbiting beneath the plate. As the magnet rotates, it drags each ball to the perpendicular foot of its channel — the eight foot-points together trace a circle whose center itself orbits at half the magnet's radius (Thales' circle).  See it in action here: https://powellga.github.io/ball-bearing-kinematics-simulation/

![Preview](screengrab.jpg)

## The mechanism

- A hardwood plate with `N` full-diameter channels radiating through the center.
- One chrome-steel ball per channel, free to slide along its diameter line.
- A rotating arm beneath the plate carries one or more N-grade neodymium disc magnets at a fixed orbit radius.
- The magnet's in-plane pull projects onto each channel, so each ball's stable equilibrium is at `p = R · cos(θ − φ)` where `R` is the orbit radius, `θ` is the magnet angle, and `φ` is the channel angle.
- Geometry (Thales): all equilibrium points lie on a circle of diameter `R`, whose center rides at radius `R/2`. The eight balls therefore appear as a rotating circle on the plate surface.

## Features

- Full K&J-style pull-force fit with a gap-dependent decay exponent.
- Live analysis panel: peak pull force, required force at RPM, safety margin, max stable RPM, peak ball speed and acceleration, peak torque, mechanical/electrical power, and battery life.
- Battery life exposed both in the output panel and as a computed parameter in the Drive & Power section (switches between hours and days).
- Metric ↔ imperial unit toggle.
- Three built-in presets (8", 12", 16") plus a user preset library: name a design, save it with today's date, load or delete from a list. Stored in `localStorage`.
- `Reset` (balls to center) and `Restart (outer)` (balls to outer ring) controls that don't mutate parameters.
- Adjustable view speed, direction, internal-mechanism overlay, CSV export of ball-position log.

## Running

Open `ballbearing_Kinematic_simulation_v8.html` in any modern desktop browser. No server, no build step, no dependencies beyond two Google Fonts loaded from the CDN.

## The mathematics

### Setup

Place the origin at the plate center. Each channel `i` (for `i = 0 .. N-1`) is a full-diameter line at angle $\varphi_i = i\pi/N$, so with $N=8$ the channel angles are $0, \pi/8, \pi/4, \dots, 7\pi/8$. Each channel carries a single ball, and we use a signed scalar $p_i \in [-R_{max}, +R_{max}]$ to describe its position along its channel. The ball's Cartesian position is

$$
(x_i, y_i) = (p_i \cos\varphi_i,\; p_i \sin\varphi_i).
$$

The magnet orbits below the plate at radius $R$ and angle $\theta(t) = \omega t$, so its 3D position is $(R\cos\theta,\; R\sin\theta,\; -g)$ where $g$ is the fixed vertical air gap (plate thickness plus clearance).

### Force on a ball

The magnet pulls the ball with a force $\vec{F}_{mag}$ directed from ball to magnet. Because the ball is constrained to its channel, only the along-channel component drives motion. With channel unit vector $\hat{u}_i = (\cos\varphi_i, \sin\varphi_i)$, the scalar driving force is

$$
F_\parallel \;=\; \vec{F}_{mag} \cdot \hat{u}_i.
$$

The in-plane displacement from ball to magnet is $(R\cos\theta - p_i\cos\varphi_i,\; R\sin\theta - p_i\sin\varphi_i)$. Projecting onto $\hat{u}_i$ and using the addition formulas:

$$
F_\parallel \;\propto\; \frac{R\cos(\omega t - \varphi_i) - p_i}{d},
$$

where $d$ is the 3D distance from ball to magnet. This identity is the engine of the whole sculpture: the along-channel pull is proportional to how far the ball is from the perpendicular foot of the magnet on its channel axis, divided by distance.

### Equilibrium and the Thales circle

Setting $F_\parallel = 0$ gives the instantaneous equilibrium

$$
p_i^\star(t) \;=\; R\cos(\omega t - \varphi_i).
$$

If each ball sits at its equilibrium, the eight of them each oscillate sinusoidally, phase-shifted by $\varphi_i$. The ball's instantaneous Cartesian position is

$$
(x_i, y_i) \;=\; \bigl(R\cos(\omega t - \varphi_i)\cos\varphi_i,\; R\cos(\omega t - \varphi_i)\sin\varphi_i\bigr).
$$

Applying product-to-sum identities:

$$
x_i = \tfrac{R}{2}\bigl[\cos(\omega t) + \cos(\omega t - 2\varphi_i)\bigr], \qquad
y_i = \tfrac{R}{2}\bigl[\sin(\omega t) + \sin(2\varphi_i - \omega t)\bigr].
$$

So ball $i$ is the midpoint of two rotating vectors, both of length $R/2$ from the origin: one pointing at $\omega t$ (common to every ball), the other pointing at $2\varphi_i - \omega t$ (unique per channel). Equivalently, ball $i$ lies on a circle of radius $R/2$ centered at $\bigl(\tfrac{R}{2}\cos\omega t,\; \tfrac{R}{2}\sin\omega t\bigr)$.

That circle is the **Thales circle**: the unique circle passing through the origin and the magnet's projected position. By Thales' theorem (every inscribed angle subtending a diameter is a right angle), every perpendicular foot from the magnet onto a line through the origin must sit on this circle. The sculpture is a physical proof of Thales.

The collective visual is therefore **eight balls, all on a circle of diameter $R$, whose center orbits at radius $R/2$ with period $2\pi/\omega$**. That rotating ring on the plate is exactly what the design is built to produce.

### Dynamics

Reality does not place balls at equilibrium. Inertia and the finite bandwidth of the magnetic force drag them off. Newton's second law for ball $i$, with rolling inertia $m_{eff} = \tfrac{7}{5} m_{ball}$ (solid sphere, pure rolling), channel damping $c$, and the full nonlinear magnetic force $F_\parallel(p_i, t)$:

$$
m_{eff}\,\ddot{p}_i \;=\; F_\parallel(p_i, t) \;-\; c\,\dot{p}_i.
$$

Linearizing $F_\parallel$ around $p_i^\star(t)$ with a local stiffness $k_{eff}(t)$:

$$
F_\parallel \;\approx\; -k_{eff}(t)\bigl(p_i - p_i^\star(t)\bigr).
$$

Substituting gives the governing equation for every ball:

$$
\boxed{\;m_{eff}\,\ddot{p}_i \;+\; c\,\dot{p}_i \;+\; k_{eff}(t)\,p_i \;=\; k_{eff}(t)\,R\cos(\omega t - \varphi_i)\;}
$$

This is a **driven, damped, parametrically-modulated harmonic oscillator**: the standard ODE of mechanical vibrations, with one twist. The stiffness $k_{eff}$ is itself time-varying because the magnet's distance to the ball oscillates as the magnet orbits. When the design is well inside its tracking regime, the parametric modulation is weak and can be ignored.

### Tracking solution (steady state)

Approximate $k_{eff}(t) \approx \bar{k}$, its minimum value, which occurs when the magnet is $\tfrac{\pi}{2}$ out of phase with the channel (the worst geometry for that ball). The equation becomes linear time-invariant, and the steady-state response to the cosine forcing is

$$
p_i(t) \;=\; A\cos(\omega t - \varphi_i - \delta)
$$

with

$$
\frac{A}{R} = \frac{1}{\sqrt{\bigl(1 - (\omega/\omega_n)^2\bigr)^2 + \bigl(2\zeta\,\omega/\omega_n\bigr)^2}},
\qquad
\tan\delta = \frac{2\zeta\,\omega/\omega_n}{1 - (\omega/\omega_n)^2}
$$

and

$$
\omega_n = \sqrt{\bar{k}/m_{eff}}, \qquad
\zeta = \frac{c}{2\sqrt{m_{eff}\,\bar{k}}}.
$$

This is the canonical transfer function of a forced mass-spring-damper. It holds per ball, independently, with the phase offset $\varphi_i$ baked into the drive.

### When the circle is tight

For the Thales ring to be crisp, every ball must have $A \approx R$ and $\delta \approx 0$. Writing $\beta = \omega/\omega_n$:

$$
\frac{A}{R} \approx 1 - \beta^2, \qquad
\delta \approx 2\zeta\beta \quad (\beta \ll 1).
$$

Keeping tracking error under 3% requires $\beta < 0.17$, i.e.

$$
\omega_n > 5.8\,\omega, \qquad \bar{k} > 34\,m_{eff}\,\omega^2.
$$

Because $\bar{k} = F(R)/R$ at the worst geometry (stiffness equals pull force divided by lateral distance when the magnet is off to the side of the channel), this becomes a direct constraint on the magnet's pull force at distance $R$:

$$
\boxed{\;F(R) \;>\; 34\,m_{eff}\,\omega^2\,R.\;}
$$

For a 10 mm chrome ball ($m_{eff} = 5.71\times 10^{-3}\,\text{kg}$) at $R = 0.088$ m and 30 RPM ($\omega = \pi$), this requires $F(R) > 0.167$ N. The default $\varnothing 24 \times 5$ mm N52 disc delivers only about 4 mN at that distance, a shortfall of 40x. A $\varnothing 25 \times 25$ mm N52 disc delivers about 0.76 N, leaving a comfortable 4.5x safety margin and a clean rotating circle in the sim.

Magnet thickness is the dominant lever because the K&J pull-force fit is approximately

$$
F(d) \;\approx\; F_0 \left(\frac{t}{t + d}\right)^{n(d,t)}, \qquad n \approx 3\text{ to }4,
$$

so holding diameter fixed and going from 5 mm to 25 mm thick gains roughly $(5/30)^{-n} \times (25/45)^{n} \sim$ two orders of magnitude at $d = R$.

### Transients, damping, and the walls

Steady-state analysis is silent on cold-start behavior. Balls begin at $p_i = 0$ and must spiral in to the tracking orbit. The critical knob is the damping ratio $\zeta$. Heuristically, $\zeta \in [0.2, 0.8]$ converges cleanly in a few orbits without wall strikes. Too little damping and the balls ring forever and slam the outer rim at $R_{max}$; too much and the phase lag $\delta$ grows to smear the ring.

The simulation exposes $c$ as a direct input (`Channel damping`, N s/m). In the physical build, useful damping sources are:

- A thin aluminum or copper sheet laminated under the wood plate. The moving steel balls induce eddy currents in the non-ferrous sheet, producing clean viscous-like damping with no moving parts.
- A light smear of silicone grease in the grooves.
- Felt or silicone channel liners.

### What the simulation integrates

`stepPhysics` evaluates the full nonlinear magnetic force on each ball, using a K&J empirical curve fit with a gap-dependent decay exponent, and advances the ODE with sub-stepped semi-implicit Euler. The linear theory above is only used for intuition and for the up-front design checks (max stable RPM, safety margin). The simulation is what actually tells you whether the balls converge, oscillate, or slam the walls.

## Related systems

The device sits at the intersection of several well-studied areas.

**Forced mechanical vibrations.** The governing ODE is the canonical forced mass-spring-damper. Textbook resonance peak, amplitude and phase response, and quality factor apply directly. The design problem is just resonance-avoidance on a rotating drive.

**Phased oscillator arrays and polyphase AC machines.** The eight balls are independent oscillators driven by a common rotating source with spatial phase offsets $\varphi_i$. This is structurally identical to the stator slots of a three-phase or multi-phase AC motor, read in reverse: in a motor, slot currents synthesize a rotating field; here the rotating field is given and the slot "currents" (ball positions) read it out. The sculpture is effectively a mechanical polyphase sensor.

**Trammel of Archimedes.** The per-channel projection $p_i = R\cos(\omega t - \varphi_i)$ is exactly the motion constrained by a trammel. The sculpture is $N$ trammels sharing one crank, which is why the outputs coherently reconstruct a full circle.

**Thales' theorem (Euclidean geometry).** The collective Cartesian locus of the eight balls follows from a 2500-year-old result: every perpendicular foot from a point on a circle to a diameter lies on a Thales circle. The sculpture is a mechanical demonstration.

**Hypotrochoid / rolling-circle kinematics.** A point on a circle of radius $R/2$ whose center itself rotates on a circle of radius $R/2$ describes a degenerate Spirograph. More general epicycloid and hypotrochoid mechanisms (the Wankel rotor, planetary gear trains, classic spirograph toys) share the same two-rotation kinematic grammar.

**Discrete Fourier sampling.** Writing the magnet as a complex phasor $M(t) = R e^{i\omega t}$, the along-channel ball position is $\mathrm{Re}\bigl(M(t)\,e^{-i\varphi_i}\bigr)$: the real projection onto a rotated axis. The eight balls are an 8-point spatial sampling of a rotating phasor, close kin to a DFT bank.

**Kuramoto oscillators and synchronization.** A family of $N$ oscillators synchronized by a common drive. The sculpture is the trivial uncoupled case, with phase imposed externally. The same math governs systems where the coupling is internal and self-organizing: metronomes on a shared platform, firefly flash synchronization, neural oscillators, power-grid generators.

**Magnetic couplings and gears.** The ball-magnet interaction uses the same pull-force physics as contactless magnetic drives, magnetically coupled aquarium stirrers, canned pumps, and magnetic spur gears. The $F(d) \propto \bigl(t/(t+d)\bigr)^n$ decay and the need to stay well inside the breakaway margin are common to every such design.

**Wagon-wheel aliasing.** If a video camera samples the sculpture at a rate near a multiple of $\omega$, the ring appears to stand still or drift backward, the classic strobe effect. In practice this is a handy diagnostic: if your phone video shows the circle frozen or reversing, you have tracking, not drift near a resonance.

## Status

Experimental design tool. The simulation is good enough to rank designs against each other and to find parameter windows where the balls lock into clean tracking (magnet thickness and effective channel damping turn out to dominate). It is not a replacement for a prototype.

Thanks to my buddy Claude :)

