# 06 Systems And Deployment Outline

## Goal

Explain VLA deployment as a closed-loop systems problem.

## Core Diagrams

- Synchronous vs asynchronous VLA control.
- Cloud, edge, and on-device deployment.
- Compression and adaptive computation in the serving path.

## Systems To Cover

- SmolVLA async inference.
- Gemini Robotics On-Device.
- QuantVLA.
- AC^2-VLA.

## Questions To Answer

- What latency budget does a robot control loop have?
- What happens when model inference is slower than action execution?
- When is quantization safe for continuous control?
- What should run on robot vs remote GPU?

