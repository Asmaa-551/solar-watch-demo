# Solar Watch Demo

The film-ready demo for Team WALL-E's SatForge UAE 2026 mission. It is a separate website from the data dashboard.

It shows the CubeSat on its real SGP4 orbit over real satellite imagery (Esri World Imagery, NASA Black Marble at night), with farm outlines from OpenStreetMap.

## The sequence

| # | Scene | What happens |
|---|---|---|
| 1 | From space | Whole Earth, distance readout, fly-in to the satellite |
| 2 | Approach | Follow the satellite to the UAE, it rolls toward the target farm |
| 3 | Camera view | Onboard camera goes full screen: 75 mm lens, sensor blur, haze, noise |
| 4 | AI detection | YOLO runs live on the captured frame, then area, capacity and energy |
| 5 | Dust storm | Haze rises, sections turn orange, flagged list |
| 6 | Downlink | LoRa beam to the Abu Dhabi ground station, bytes vs raw photo |
| 7 | Ground crews | Crews drive to flagged sections only, recovered energy |
| 8 | Summary | Pull back to the whole Earth |

Controls: **Space** pause, **← →** chapters, **H** hide controls (for filming), **C** hide captions. **Explore** frees the camera, and **Settings** has the lens, detector and confidence options.

## Put it online

1. Create a public repo named `solar-watch-demo` and upload `index.html` (and this README).
2. Go to **Settings, Pages**, set the source to deploy from the `main` branch, and save.
3. The demo is then at `https://asmaa-551.github.io/solar-watch-demo/`. The dashboard's "Open the live demo" button already points there.

## Run your YOLO model live on the camera

The detector runs in the browser on the actual camera frames (after blur, haze and noise), using onnxruntime-web.

1. Export the trained Ultralytics model (YOLOv8 or YOLO11, detection or segmentation):
   ```bash
   yolo export model=best.pt format=onnx imgsz=640 opset=12 simplify=True
   ```
2. Either drag `best.onnx` onto the demo page, or put it in the repo as `models/best.onnx` and open:
   ```
   https://asmaa-551.github.io/solar-watch-demo/?model=models/best.onnx&classes=solar_panel
   ```
   `classes` lists the class names in training order, separated by commas. Use `imgsz=` if you exported at a size other than 640.
3. The chip at the top right says "Detector: best.onnx, live". Scene 4 then draws the model's boxes or masks and computes area from them.

**Resolution matters.** The flight camera gives about 9 m per pixel. A model trained on 0.1–0.3 m aerial tiles will not work at 9 m. Either train on imagery at the camera's resolution (Sentinel-2 at 10 m), or use **Settings, Camera lens, What-if** to render at 1 m or 0.3 m. The what-if lenses are labelled on screen as beyond what a 1U can do from 500 km.

## Drive the attitude from Gazebo

When the Gazebo simulation (see the VM brief, milestone M5) is running with rosbridge, the demo takes the satellite's roll from Gazebo instead of its built-in model, and sends the roll setpoint back.

- Subscribes `/cubesat/telemetry` (std_msgs/String, JSON with `roll`, `wheel_rpm`, ...)
- Publishes `/cubesat/roll_setpoint` (std_msgs/Float64, degrees)

Browsers block an `https` page from connecting to a plain `ws://` address, so for the Gazebo link serve the demo from the VM over `http`:

```bash
cd ~/solar-watch-demo            # folder with index.html
python3 -m http.server 8000 --bind 0.0.0.0
```

Then on the Mac open `http://<VM_IP>:8000/?ros=ws://<VM_IP>:9090`. The chip changes to "Attitude: Gazebo live".

## Filming

- Use a 1920×1080 browser window. Press **H** to hide the controls, and keep the captions or press **C** to hide them.
- Record with QuickTime (File, New Screen Recording) or OBS.
- Put the finished film in the dashboard repo as `demo.mp4`, with a still frame as `demo-poster.jpg`. The dashboard plays it at the top.

## Credits

Imagery © Esri, Maxar, Earthstar Geographics. Night lights NASA Black Marble (GIBS). Farm outlines © OpenStreetMap contributors (ODbL). Orbit propagation satellite.js (SGP4). Globe CesiumJS. Detector onnxruntime-web.
