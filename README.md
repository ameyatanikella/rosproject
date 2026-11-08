# Kibo-RPC Mission File — Standalone Snapshot

`YourService.java` — the 880-line mission class submitted to the JAXA/NASA **Kibo Robot Programming Challenge**, in which an uploaded APK autonomously flies the Astrobee free-flying robot around the ISS Kibo module, identifies items at each target area from the robot's NavCam, and reports them to the ground system.

This repository holds that one file on its own. **The full project — synthetic data pipeline, trained models, ArUco cropping harness and Android app — is at [ameyatanikella/NASA](https://github.com/ameyatanikella/NASA).** Start there; this copy is kept as a snapshot of the flight logic in isolation.

## What the file does

- **Mission sequence** (`runPlan1`) — starts the mission, then for each of the four target areas moves to a tuned pose and quaternion, settles the camera with repeated NavCam grabs, runs detection, and reports results through `api.setAreaInfo()` before returning to the astronaut to notify recognition and snapshot the target item.
- **On-device inference** — loads a YOLOv8n TensorFlow Lite model through the TFLite Task Vision `ObjectDetector`, runs it against the captured frame, and separates landmark items from treasure items with quantity counts per area.
- **ArUco-anchored cropping** — undistorts the frame using the camera intrinsics, locates the `DICT_5X5_250` marker on the item board, derives the in-plane rotation from its edge vector, and crops the item region as a multiple of the marker's known physical edge length. The marker doubles as a scale reference, which lets the robot classify items from its approach pose instead of flying closer and re-aiming at every area — mission time the competition scores directly.
- **Debug output** — intermediate crops and annotated frames are written with `api.saveMatImage()` for inspection after a simulator run.

## Dependencies

Java (Android, `minSdk 24`) · JAXA Kibo-RPC API (`jp.jaxa.iss.kibo.rpc.api`) · OpenCV 4.5.3 with contrib, for ArUco and `calib3d` · TensorFlow Lite 2.10 + Task Vision 0.4

This file will not compile on its own. It needs the competition-supplied Gradle modules (`guest_science_library`, `kibo_rpc_api`, `opencv-490`) and the `.tflite` weights in `app/src/main/assets/`, all of which are in the full repository linked above.
