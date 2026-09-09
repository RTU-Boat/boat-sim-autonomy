# RTU Boat Autonomy Simulator

Unity-based development simulator for the RTU autonomous surface vessel being
prepared for the 2027 Njord Challenge. The project provides the vessel model,
water and buoy physics, competition marker assets, camera viewpoints, and a
starting point for ROS 2 integration.

The autonomy and perception nodes live in the separate
[`RTU-Boat/boat-autonomy`](https://github.com/RTU-Boat/boat-autonomy)
repository.

> **Current status:** early integration prototype. Boat and water physics are
> present, but the project-specific ROS 2 bridge is not implemented yet. This
> simulator must not be treated as a validated model of the physical vessel.

## Requirements

Use the versions recorded by the project whenever possible:

- Windows 10 or Windows 11, x86-64
- Unity Hub
- **Unity 6000.0.38f1** (`82314a941f2d`)
- DirectX 11/12-capable GPU
- 16 GB RAM recommended
- Git

The project uses Unity's High Definition Render Pipeline (HDRP) 17.0.3. A
dedicated GPU is recommended. Lower the render resolution and HDRP quality when
using older GPUs such as the GTX 1050 Ti.

The bundled Ros2ForUnity libraries are:

- Ros2ForUnity 1.3.0
- ROS 2 Humble
- standalone Windows x86-64 build

Ubuntu and macOS native ROS libraries are not included in this repository.

## Getting started

Clone the repository:

```powershell
git clone https://github.com/RTU-Boat/boat-sim-autonomy.git
cd boat-sim-autonomy
```

In Unity Hub:

1. Install Unity `6000.0.38f1` from the Unity archive if it is not already
   available.
2. Select **Add project from disk**.
3. Choose the `Autonomous Boat` directory inside this repository.
4. Allow Unity to restore the packages and import the assets.
5. Open `Assets/OutdoorsScene.unity`.
6. Check the Console for compilation or missing-reference errors before entering
   Play mode.

Do not open the repository root as the Unity project. The Unity project root is:

```text
Boat-Sim-Unity/Autonomous Boat/
```

## Scenes

| Scene | Purpose |
| --- | --- |
| `Assets/OutdoorsScene.unity` | Primary outdoor boat and buoyancy scene |
| `Assets/OutdoorsScene-Burzujs.unity` | Alternate development copy of the outdoor scene |

`OutdoorsScene.unity` should become the canonical test scene. Changes from the
alternate scene should be reviewed and deliberately merged rather than allowing
both scenes to diverge.

## Current controls

Depending on which movement component is attached to the vessel in the active
scene:

- Up/Down arrows: forward and reverse
- Left/Right arrows: steering or in-place rotation
- Left mouse drag: orbit the follow camera
- Mouse wheel: zoom
- `C`: cycle configured cameras

The current movement scripts are intended for simulator development only. Their
forces and torque values have not been identified against the physical boat.

## Project structure

```text
Autonomous Boat/
├── Assets/
│   ├── OutdoorsScene.unity
│   ├── OutdoorsScene-Burzujs.unity
│   ├── Props/                  Boat and competition marker models
│   ├── Ros2ForUnity/           Bundled ROS 2 Humble Unity integration
│   ├── Scripts/                Boat, buoyancy, camera, and bridge scripts
│   └── Settings/               HDRP and rendering configuration
├── Packages/                   Unity package manifest and lock file
└── ProjectSettings/            Unity 6000.0.38f1 project configuration
```

Important project scripts:

| Script | Responsibility |
| --- | --- |
| `Floaters.cs` | Distributed vessel buoyancy and water damping |
| `buoyFloaters.cs` | Buoy flotation and damping |
| `Movement.cs` | Differential motor-position force prototype |
| `BoatSimpleDifferential.cs` | Simplified force/torque movement prototype |
| `BoatCameraFollow.cs` | Orbiting boat camera |
| `CameraCycle.cs` | Switches between configured cameras |
| `ROS2.cs` | Reserved for the project ROS bridge; currently empty |

## ROS 2 integration target

The simulator and the physical vessel should expose the same autonomy-facing
contract. The planned first simulator interface uses only standard ROS messages.

Published by Unity:

| Topic | Type | Purpose |
| --- | --- | --- |
| `/boat/state/odometry` | `nav_msgs/msg/Odometry` | Boat pose and velocity in a local metric frame |
| `/camera/rgb/image_raw` | `sensor_msgs/msg/Image` | Simulated RGB camera image |
| `/camera/depth/image_raw` | `sensor_msgs/msg/Image` | Simulated depth image in metric units |
| `/clock` | `rosgraph_msgs/msg/Clock` | Simulation time when ROS simulated time is enabled |

Subscribed by Unity:

| Topic | Type | Purpose |
| --- | --- | --- |
| `/boat/cmd_vel_autonomy` | `geometry_msgs/msg/Twist` | Requested surge speed and yaw rate |

Coordinate frames, timestamps, image encodings, units, and command timeout
behavior must be explicitly tested. Unity uses a left-handed Y-up coordinate
system while ROS conventionally uses a right-handed coordinate system, so pose
and rotation values must not be copied without conversion.

## Known blockers

The repository is not currently ready for an end-to-end autonomy test:

1. `Assets/Scripts/ROS2.cs` contains no publishers or subscribers.
2. `BoatCameraFollow.cs` and `BoatCameraFollow-Burzujs.cs` both declare a public
   `BoatOrbitCamera` class, which can cause a duplicate-type compilation error.
3. Two separate movement implementations exist and no single command interface
   owns vessel actuation.
4. RGB, depth, odometry, and simulation clock topics are not implemented.
5. There is no command watchdog that returns thrust to neutral when commands
   become stale.
6. Physics and actuator parameters have not been calibrated against water-test
   data from the physical vessel.

The first integration milestone is:

```text
Unity RGB/depth + odometry
            ↓
ROS 2 perception and waypoint guidance
            ↓
/boat/cmd_vel_autonomy
            ↓
Unity differential-thrust boat model
```

## Development rules

- Keep `OutdoorsScene.unity` usable after every merged change.
- Do not commit `Library`, `Temp`, `Logs`, build outputs, IDE caches, or local
  Unity user settings.
- Commit matching `.meta` files for every Unity asset.
- Use SI units at the ROS boundary: metres, seconds, metres per second, and
  radians per second.
- Do not put competition logic inside Unity. Unity simulates the environment and
  vehicle; the same ROS autonomy nodes should control both simulation and
  hardware.
- Add deterministic tests for coordinate conversion, command timeout, and sensor
  encoding before relying on the simulator for controller validation.

## Safety

Simulator success does not prove that a controller is safe for the physical
boat. Commands must pass through the autonomy safety supervisor and the
ArduPilot adapter before reaching real actuators. Physical testing additionally
requires working RC takeover, physical and remote kill switches, a communications
watchdog, restrained initial tests, and an operator ready to intervene.

## License

A repository-wide license has not yet been selected. Do not redistribute project
assets or third-party content until their licenses and attribution requirements
have been reviewed.
