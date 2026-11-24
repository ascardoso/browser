FLUID CYMATICS LAB 01
====================

A standalone browser experiment for a simplified 2D fluid-surface wave model.

OPENING THE APP
1. Unzip fluid-cymatics-lab-01.zip.
2. Open the fluid-cymatics-lab folder.
3. Double-click index.html to run the simulation.

MICROPHONE INPUT
Microphone permissions require a secure browser context. On macOS, open Terminal,
change directory into this folder, and run:

    python3 -m http.server 8000

Then visit:

    http://localhost:8000

Click "Use microphone" and grant permission. Audio is analyzed locally in the
browser; it is not uploaded. The visual simulation maps the detected pitch and
loudness into a slower, visible wave-driving range rather than simulating each
individual audio-frequency cycle.

INTERACTION
- Click or drag on the surface to inject ripples.
- Choose a wave recipe to change the source arrangement.
- Adjust drive frequency, strength, phase and damping.
- Switch between Ripple Tank and Faraday Surface models.
- Play the built-in tone, or use a microphone / external instrument.

SCIENTIFIC NOTE
This is a pedagogical, visually enhanced finite-difference wave-field model.
It is inspired by ripple tanks and parametrically driven Faraday waves, but is
not a full Navier-Stokes / computational-fluid-dynamics simulation. The Faraday
mode is a simplified parametrically driven surface-wave approximation.
