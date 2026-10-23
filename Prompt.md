Build an impressive, interactive, browser-based **3D Sailboat Aerodynamics Lab**.

The experience should let me explore how wind direction, sail trim, keel forces, and rudder angle affect a sailing boat. It must combine beautiful 3D graphics with a coherent numerical physics model.

I want a working application, not a design proposal. Deliver the complete implementation.

## 1. Deliverable and technical constraints

Create one file named `sailboat-lab.html`, containing the HTML, CSS, and JavaScript.

- It must work when opened directly in a modern desktop browser.
- You may load Three.js and its controls from a public CDN, with pinned, compatible versions.
- No build tools, backend, API keys, local server, or externally loaded 3D models.
- Generate all geometry procedurally.
- Avoid assets or loading mechanisms that fail under `file://`.
- Show a useful error message if WebGL or the CDN dependency fails.
- Use actual 3D rendering, with orbit, pan, and zoom controls.
- Aim for smooth performance on a normal modern laptop.
- Make the interface usable at 1440 × 900 and adapt gracefully to smaller screens.

Prioritize a complete, stable, attractive simulation over excessive features.

## 2. Concept and visual direction

Title: **AERONAUT / Sailing Physics Lab**

Create the feeling of an interactive engineering exhibit in a premium science museum.

The main scene is a detailed sailing boat floating in a rectangular water test basin, surrounded by a subtly outlined, transparent wind-tunnel volume.

Visual treatment:

- Dark navy environment, restrained cyan and amber accents.
- Soft lighting, believable shadows, and subtle atmospheric depth.
- Water with restrained animated ripples and reflections or a convincing lightweight approximation.
- Thin measurement markings along the basin.
- Elegant typography, compact instrument panels, and clear units.
- A coherent colour system shared by vectors, charts, and readouts.
- A polished initial camera angle showing the sails, hull, underwater keel, and surrounding airflow.

Construct a recognizable sailing boat with:

- Shaped hull, deck, cockpit, mast, boom, and rudder.
- One curved mainsail with visible seams or panel lines.
- An underwater keel, visible through translucent water or an underwater-view toggle.
- A contrasting sail-edge treatment that makes trim angle easy to read.

Use a single mainsail to keep the aerodynamic model understandable. Model the sail as a curved surface, not a flat triangle. Its boom and geometry must respond to trim.

Decorative water motion must not inject unexplained energy into the boat.

## 3. Two operating modes

### A. Fixed test mode — default

Hold the boat at a fixed location and heading, as if mounted on a test rig.

Allow wind direction, wind speed, and sail trim to change. Calculate and display aerodynamic forces continuously.

Be explicit that the rig supplies the reaction forces that prevent translation and rotation. A nonzero sail force must not appear to be an unexplained equilibrium.

This mode is for inspecting airflow, angle of attack, and force decomposition.

### B. Free sailing mode

Release the boat and integrate its movement from forces and moments.

Support:

- Horizontal translation in two dimensions.
- Heading and yaw rate.
- Heel angle and roll rate.

The boat must accelerate, turn, develop leeway, and settle toward a speed determined by the balance of forces.

Use a follow camera and a sufficiently large water area. If the boat reaches the simulation boundary, pause with a clear reset option rather than teleporting it or applying an invisible force.

Switching modes must reset or transfer state predictably.

## 4. Physics model

Use a simplified but physically coherent sailing model. Do not claim this is full CFD.

Keep all constants and tunable coefficients in a clearly organized configuration object. Use SI units internally, and document your coordinate and sign conventions.

### Apparent wind

Compute local apparent wind from:

`apparent air velocity = true air velocity − velocity of the sail’s centre of effort`

Include the centre-of-effort velocity caused by boat rotation where practical.

Distinguish between the direction air travels and the nautical direction wind comes from. Label the interface so these cannot be confused.

Show true and apparent wind separately.

### Sail forces

Derive sail angle of attack from apparent wind and the actual sail orientation.

Use:

`q = 0.5 × air density × apparent wind speed²`

`lift magnitude = q × sail area × CL`

`drag magnitude = q × sail area × CD`

Implement continuous, bounded lift and drag coefficient functions with:

- Sensible small-angle lift behaviour.
- A clear stall region and progressive lift reduction.
- Increased drag after stall.
- Plausible broadside drag for downwind sailing.
- Correct behaviour on both port and starboard tacks.
- Stable behaviour near zero wind and angle wrapping.

Explain the coefficient model in a collapsible information panel.

Drag must act in the direction of the air velocity relative to the sail. Lift must be perpendicular to that velocity, with its sign determined consistently by sail orientation and angle of attack.

Resolve sail force into:

- Forward driving force.
- Lateral force.
- Heeling moment.
- Yaw moment from the force’s application point.

Do not simply assign forward thrust from wind speed or sail angle.

### Water forces

Model:

- Hull resistance opposing motion relative to the water.
- Keel lift and drag based on local water-relative velocity and leeway.
- Rudder lift and drag based on local inflow and rudder angle.
- Moments from the actual positions of these forces.
- Appropriate yaw and roll damping.

Use a simple bounded hydrodynamic coefficient model. Avoid singularities near zero speed.

The rudder should have little effect when there is essentially no water flow across it.

Keel lift must be perpendicular to its local water-relative flow; drag must oppose motion relative to water. Do not add arbitrary forward propulsion from the keel.

### Heel and stability

Integrate heel dynamically using:

- Aerodynamic heeling moment.
- A restoring moment based on displacement, gravity, and an assumed metacentric height.
- Roll inertia and damping.

A simplified restoring law such as `mass × g × GM × sin(heel)` is acceptable within a stated operating range.

The boat should lean progressively under lateral loading and recover when the wind drops.

Clearly indicate when heel exceeds the model’s valid range; pause at an extreme heel threshold rather than pretending to simulate a realistic capsize.

### Numerical integration

- Use a fixed physics timestep with an accumulator.
- Use a stable integration approach such as semi-implicit Euler with suitable substeps.
- Limit catch-up work after a suspended browser tab.
- Keep physics independent of rendering frame rate.
- Avoid NaNs, unbounded forces, and artificial minimum boat speeds.
- Pausing must freeze simulation time, including gusts and particle advection.
- Reset must restore a deterministic initial state.

## 5. Airflow visualization

Make airflow one of the most visually compelling elements.

Render animated particles or short trails entering the tunnel, passing around the sail, and forming a downstream wake.

The visualization should communicate:

- Wind speed and direction.
- Deflection around the sail.
- A relatively orderly wake at moderate angle of attack.
- A more disturbed wake when stalled.
- Reduced aerodynamic effectiveness during luffing.

Use a lightweight illustrative velocity field driven by the current wind, sail orientation, and aerodynamic state.

Important: label this **“Illustrative airflow — not a CFD solution.”**

The particles are a visualization of the aerodynamic state. Do not pretend particle collisions calculate the forces.

Keep the coordinate frame consistent: free-mode particles must reflect world-space air motion, or clearly identify a boat-relative visualization mode.

Do not add random turbulence everywhere. Wake disturbance should develop downstream and scale with stall severity.

Provide particle-density and airflow-visibility controls.

## 6. Controls

Provide organized, compact controls for:

**Environment**
- True wind speed: 0–20 m/s.
- Wind-from direction: 0–360°.
- Gust strength.
- Gust toggle, using smooth deterministic time variation.

**Boat**
- Sail trim: −85° to +85°, clearly defined relative to the boat.
- Rudder angle: −30° to +30°.
- Initial heading, editable in fixed mode or applied on reset.
- Boat mass through a sensible bounded range.

**Simulation**
- Fixed / free mode.
- Play / pause.
- Reset.
- Simulation speed: 0.25×, 0.5×, 1×, 2×.

**View**
- Airflow.
- Force vectors.
- Wake / track.
- Underwater view.
- Camera presets: perspective, overhead, stern, sail close-up.

Every control must affect the actual simulation or visualization. No decorative sliders.

## 7. Instrumentation

Show live values for:

- True wind speed.
- Apparent wind speed and angle.
- Boat speed in knots.
- Heading.
- Leeway angle.
- Sail angle of attack.
- Lift and drag.
- Forward driving force.
- Heel angle.
- Current aerodynamic state: attached flow, approaching stall, stalled, or luffing.

Use “—” for undefined quantities such as leeway at essentially zero speed.

Add colour-coded 3D arrows for:

- True wind.
- Apparent wind.
- Sail lift.
- Sail drag.
- Total aerodynamic force.
- Keel force.

Place force arrows at their application points. Show a legend and use a consistent, clearly disclosed display scale.

Include compact rolling charts for boat speed and heel over the last 30 seconds of simulation time.

## 8. Guided experiments

Include buttons that apply coherent presets:

1. **Beam reach** — a clear demonstration of lift producing forward drive.
2. **Close-hauled** — sailing obliquely into the wind with noticeable lateral loading.
3. **Dead downwind** — drag-dominated sailing.
4. **Stall demonstration** — fixed rig, excessive angle of attack, visibly disturbed wake.
5. **Gust response** — free sailing with dynamic heel and recovery.

Each preset should set wind, heading, sail trim, mode, and initial state consistently, and display one short explanation of what to observe.

Choose a visually interesting, physically sensible beam-reach configuration as the default.

## 9. Validation

Include a collapsible **“Physics checks”** panel with actual pass/fail results from small numerical checks.

Verify that:

- Zero apparent wind produces zero aerodynamic force.
- Doubling apparent wind speed at fixed geometry produces approximately four times the aerodynamic force.
- Aerodynamic drag follows relative airflow.
- Hydrodynamic drag opposes water-relative motion.
- Mirroring a symmetric configuration reverses lateral force while preserving approximately the same forward force.
- A stationary boat in still air and water does not spontaneously accelerate.
- A rudder produces negligible force without local water flow.
- Equivalent runs at different render frame rates produce closely matching physics states.

Do not hard-code successful results. Report failures honestly.

## 10. Quality bar and implementation priorities

Avoid:

- Scripted boat paths.
- Predetermined swaying masquerading as heel dynamics.
- Particle effects disconnected from the wind controls.
- Impossible propulsion directly into the wind.
- Arbitrary force signs that work on only one tack.
- Large overlays obscuring the boat.
- Unstable physics hidden by excessive clamping.
- Placeholder features or TODO sections.

Implement in this order:

1. Coherent force model and stable integration.
2. Working 3D boat and camera.
3. Interactive controls and instrumentation.
4. Airflow and wake visualization.
5. Visual polish and numerical checks.

Organize the code into understandable sections and comment the non-obvious physics decisions.

If you can execute browser tests, test startup, controls, mode switching, reset, and numerical stability. If you cannot, state that honestly.

Deliver the complete runnable HTML file, followed by a brief explanation of how to open it, its controls, and the main physical approximations. Do not stop at a plan.