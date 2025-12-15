INTERACTIVE CUBIC BEZIER ROPE SIMULATION

------------------------------------

OVERVIEW
--------
This project is a simple interactive simulation of a cubic Bézier curve.
The goal was to understand how Bézier curves work mathematically and how
they can be animated using basic physics.

The curve reacts to user input in real time and behaves like a rope with
springy motion. All logic is written manually without using any built-in
Bézier or animation libraries.


PLATFORM
--------
Web (HTML Canvas + JavaScript)


BEZIER CURVE
------------
The curve used in this project is a cubic Bézier curve defined using four
control points.

P0 and P3 are fixed endpoints.
P1 and P2 are control points that move dynamically.

The curve position is calculated using the standard cubic Bézier formula:

B(t) = (1 - t)^3 * P0
     + 3(1 - t)^2 * t * P1
     + 3(1 - t) * t^2 * P2
     + t^3 * P3

The curve is sampled for values of t between 0 and 1 using small steps and
drawn as line segments on the canvas.


TANGENTS
--------
To show the direction of the curve, tangent vectors are computed using the
derivative of the cubic Bézier equation.

The derivative is calculated manually and normalized before drawing short
line segments along the curve. These tangents update in real time as the
curve changes.


PHYSICS AND MOTION
------------------
The middle control points (P1 and P2) use a simple spring-damper model to
create smooth motion.

The acceleration is computed using:

acceleration = -k * (position - target) - damping * velocity

Velocity and position are updated every frame. Delta time is clamped to
avoid unstable motion and jitter.


INTERACTION
-----------
Mouse movement controls the target positions of the control points.
Because of the spring system, the curve does not move instantly and instead
lags slightly, which gives a rope-like effect.

The endpoints remain fixed throughout the simulation.


RENDERING
---------
The simulation runs using requestAnimationFrame to maintain smooth updates.
Each frame draws:
- the Bézier curve
- the control points
- tangent vectors


RESTRICTIONS
------------
No external libraries were used.
No built-in Bézier or physics utilities were used.
All math and physics logic was implemented manually.


FILES
-----
index.html     : Source code
README.txt     : Project explanation
Screen recording showing interaction
