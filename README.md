# Real-time control of a Universal Robots arm from OptiTrack motion capture

**Upravljanje zglobom Universal Robots robota pomoću OptiTrack sustava u realnom vremenu**

> Seminar paper – Human-Robot Interaction (Interakcija čovjeka i robota), FSB, University of Zagreb, 2026
>
> Author: **Krešimir Hartl**

![Python](https://img.shields.io/badge/Python-3.10-3776AB)
![Robot](https://img.shields.io/badge/robot-UR3e%20%C2%B7%20ur__rtde-1E5BA8)
![Motion capture](https://img.shields.io/badge/mocap-OptiTrack%20Motive%20%C2%B7%20NatNet-555555)
![LaTeX](https://img.shields.io/badge/paper-LaTeX-008080)

A teleoperation pipeline that follows a person's arm with an OptiTrack motion capture system and
drives the joints of a Universal Robots UR3e collaborative robot in real time, at 125 Hz. The
seminar task was to map one human joint onto one robot joint with clearly defined limits of safe
behaviour. The work does that for the elbow (robot joint J3) and extends it to the whole arm, with
three rigid bodies on the upper arm, forearm and hand driving all six robot joints directly,
without inverse kinematics. The system was evaluated in 21 runs on the physical robot in the lab
and in VMware URSim.

**Paper:** [`dist/v1/HRI_seminar_Kresimir_Hartl.pdf`](dist/v1/HRI_seminar_Kresimir_Hartl.pdf)
(12 pages, Croatian) · **Presentation:**
[`docs/presentation/hri_presentation.pdf`](docs/presentation/hri_presentation.pdf)

---

## How it works

```
OptiTrack Motive 3.0.1  (120 Hz, 3 rigid bodies / 11 markers)
   │  NatNet over UDP
   ▼
Control pipeline (Python 3.10, 125 Hz)
   ├─ receiver (UDP buffer)
   ├─ kinematics: 3D poses -> joint angles
   ├─ One-Euro adaptive filter (noise and tremor)
   └─ safety chain:
        • joint velocity saturation
        • working range clamp
        • hold last state on marker occlusion (up to 150 ms)
        • emergency stop
   │  RTDE servoJ commands
   ▼
Universal Robots UR3e  (or URSim)
```

- **1-DOF mode:** the operator's elbow flexion drives robot joint J3.
- **6-DOF mode:** upper arm, forearm and hand drive joints J1 to J6.
- **Filters compared:** One-Euro, second-order Butterworth, exponential moving average, and no filter.

## What's inside

| Folder | Contents |
|---|---|
| [`dist/`](dist/) | the submitted paper as PDF |
| [`docs/`](docs/) | LaTeX source of the paper (`main.tex`, `chapters/`), figures, tables generated from telemetry, the presentation |
| [`src/`](src/) | the pipeline: `optitrack/` (NatNet client and sources), `kinematics/`, `filtering/`, `safety/`, `robot/` (RTDE backend), `pipeline/` (125 Hz loop), `tools/` (analysis scripts), `tests/` |
| [`testing/`](testing/) | lab protocols, session scripts and diagnostics (`lab/`), URSim scripts (`virtual/`) |
| [`data/`](data/) | raw lab recordings and telemetry, processed clips, benchmarks and sweeps, the literature list with links in `SOURCES_LOG.md` |
| [`scripts/`](scripts/) | `build-docs.ps1` and `build-docs.sh` to compile the paper |
| [`TASK/`](TASK/) | the work plan for the course task, the literature overview and the lab checklist |

[`src/README.md`](src/README.md) and [`testing/lab/README.md`](testing/lab/README.md) describe how
to run the system with the real robot and a live OptiTrack stream.

## Run it

```powershell
py -3.10 -m venv .venv              # Python 3.10, ur_rtde ships a cp310 wheel
.\.venv\Scripts\Activate.ps1        # Linux/macOS: source .venv/bin/activate
pip install -r requirements.txt

python -m pytest src/tests -q       # unit and integration tests

# the pipeline without a robot (dry run), then analyse the recorded track
python -m src.pipeline.main --sink dry_run --seconds 20 --track-csv run.track.csv --log-actual
python -m src.tools.track_analysis run.track.csv
```

Compile the paper with `.\scripts\build-docs.ps1` (Windows) or `./scripts/build-docs.sh`
(Linux/macOS). The output goes to `dist/v1/`.

The raw CSV files and the `.pptx` are stored with Git LFS, so run `git lfs install` before
cloning. The working copy is about 1 GB, most of it lab video and raw recordings. The papers cited
in the seminar are not in the repository for copyright reasons. [`data/SOURCES_LOG.md`](data/SOURCES_LOG.md)
links each one, and `data/sources/` is where to put the PDFs.

## Results

As reported in the paper, from the 21 runs on the UR3e:

- Computing the control signal takes 3 to 4 ms per cycle at 125 Hz.
- After removing the phase shift, the tracking error is 0.4 to 3.0° at a natural pace and up to
  6.2° when single joints move in isolation, with a correlation of 0.68 to 0.95.
- The total delay from arm motion to robot response at the elbow is 385 ms. The paper splits it
  into filter phase shift (200 ms), the safety velocity limit (73 ms) and the robot's servo
  dynamics (112 ms), so the response is set by noise filtering rather than by the network or the
  robot.
- Further runs test added network delay, packet loss, marker occlusion and each safety layer.

The per-run measurements are in
[`run_log.csv`](data/raw/lab_session_01092026_020000/telemetry/run_log.csv), and
[`docs/ANALIZA_REZULTATA_I_METRIKA.md`](docs/ANALIZA_REZULTATA_I_METRIKA.md) (Croatian) explains
how each number in the paper was derived.

## License

Apache License 2.0, see [`LICENSE`](LICENSE).
