# UAV Ground Control Station Lab (ArduPilot SITL + QGroundControl)

A small lab where I set up a simulated drone and a Ground Control Station (GCS) on my own laptop, connected through MAVLink, and documented the problems I ran into along the way.

I built it to understand how a GCS talks to a vehicle: the link, the ports, the telemetry and the commands. It is a learning project, not production work.

## What it does

- Runs an **ArduPilot Copter** simulator (SITL) on Ubuntu inside WSL2.
- Connects **QGroundControl** (the GCS) to it over **MAVLink / UDP**.
- From the GCS I can arm, take off, send the drone to a point on the map, and return it to launch.

![QGroundControl showing the simulated drone flying to a "Go here" point](docs/qgc-flight.png)

![MAVProxy console and map showing accepted commands, 100% link and 0 lost packets](docs/qgc-console.png)

## Setup

Environment: Windows 11 + WSL2, Ubuntu 24.04 (TODO: confirm with `lsb_release -a`).

**1. ArduPilot SITL** (inside WSL, Python virtual environment active):

```bash
cd ~/ardupilot/ArduCopter
sim_vehicle.py -v ArduCopter --console --map
```

**2. QGroundControl** (inside WSL, using the Linux AppImage and WSLg for the window):

```bash
sudo apt install -y libfuse2 libxcb-xinerama0 libxkbcommon-x11-0 libxcb-cursor0
wget https://d176tv9ibo4jno.cloudfront.net/latest/QGroundControl-x86_64.AppImage
chmod +x QGroundControl-x86_64.AppImage
./QGroundControl-x86_64.AppImage
```

**3. Point the telemetry at the GCS.** In the MAVProxy prompt:

```
output add 127.0.0.1:14550
```

QGroundControl connects automatically and shows the vehicle on the map.

## Problems I ran into (and how I fixed them)

| Problem | Cause | Fix |
|---|---|---|
| QGroundControl for Windows crashed on start with *Bad Image* (`gstreamer-1.0-0.dll`, error `0xc0e90002`) | Windows blocked a binary that is not recognised/signed, most likely Windows 11 Smart App Control | Did not disable the protection. Ran the Linux AppImage inside WSL instead |
| QGroundControl showed *Disconnected* although the simulator was running | `sim_vehicle.py` detected WSL and sent telemetry to the Windows host (`172.17.192.1:14550`), not to localhost where the GCS was running | Added a second output with `output add 127.0.0.1:14550` |
| `takeoff` failed in MAVProxy | The vehicle was not in `GUIDED` mode | Set the mode to `GUIDED` before arming and taking off |

## What I learned

- A GCS is a client on a link: the interesting work is the link itself (ports, routing, firewalls, link quality), not the buttons.
- How to read what MAVProxy prints to work out where the telemetry is actually going.
- How to document a problem so someone else can reproduce the fix.

## Next steps (planned, not done yet)

- Put the vehicle side on a Raspberry Pi and the GCS on a separate machine, connected over Wi-Fi.
- Add a WireGuard VPN between them and a firewall rule that only allows MAVLink (UDP 14550).
- Measure latency and packet loss, simulate a link failure, and write up how I diagnosed it.

## Tech

ArduPilot SITL · MAVProxy · MAVLink · QGroundControl · WSL2 · Ubuntu · UDP
