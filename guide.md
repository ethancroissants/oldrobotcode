# Team 1279 — Build, Deploy & Driver Station Guide

This guide is for getting the 2026 FRC robot code built, deployed to the roboRIO, and
working on the Driver Station — including the controller setup that trips people up
after a fresh Driver Station install.

Team number: **1279**
WPILib year: **2026**

---

## 1. One-time setup (new laptop / fresh install)

You need these installed **in this order**:

1. **WPILib 2026** — installs the correct JDK, VS Code, GradleRIO, and the WPILib VS
   Code extension. Download from
   [https://github.com/wpilibsuite/allwpilib/releases](https://github.com/wpilibsuite/allwpilib/releases).
   Pick the 2026 release and run the installer. Choose "Everything" when prompted so
   you get VS Code, the JDK, and the simulation tooling.
2. **FRC Game Tools 2026** (Windows only, from NI) — this installs the **Driver
   Station**, **roboRIO Imaging Tool**, and LabVIEW runtime. This is what the driver
   laptop uses to enable/disable the robot. Get it from
   [https://www.ni.com/en/support/downloads/drivers/download.frc-game-tools.html](https://www.ni.com/en/support/downloads/drivers/download.frc-game-tools.html).
3. **FRC Radio Configuration Utility** — for flashing the robot radio once per season.
4. **PathPlanner** (optional, but we use it) — install from the Microsoft Store or from
   [https://github.com/mjansen4857/pathplanner/releases](https://github.com/mjansen4857/pathplanner/releases).
   The autos live in `src/main/deploy/pathplanner/`.

Open this project with the **WPILib VS Code** shortcut, not a plain VS Code — the
WPILib version has the correct JDK on its PATH so `./gradlew` works.

---

## 2. Building the code

From VS Code:

- `Ctrl+Shift+P` → **WPILib: Build Robot Code**.

From a terminal in the project root:

```
./gradlew build
```

(On Windows: `gradlew.bat build`.)

The first build downloads GradleRIO and all vendor dependencies, so it will take a
few minutes and needs internet access. After the first build, subsequent builds are
cached and fast.

If you see "Plugin [id: 'edu.wpi.first.GradleRIO'] was not found", you are running a
system Gradle without internet access. Use the WPILib VS Code terminal (it provides
the right environment) and make sure you're online.

---

## 3. Deploying to the robot

Make sure you are **connected to the robot** first — either over the radio (2.4/5 GHz
Wi-Fi SSID `1279` or whatever the radio was flashed to), over USB-B to the roboRIO,
or over Ethernet to the radio's LAN port.

From VS Code:

- `Ctrl+Shift+P` → **WPILib: Deploy Robot Code**.

From a terminal:

```
./gradlew deploy
```

Deploy does: clean-build → upload jar + `src/main/deploy/` contents to the roboRIO →
restart the robot program. You'll see `*** SUCCESS ***` at the end on a good deploy.

If deploy hangs at "Looking for roboRIO", you are not actually on the robot's
network — check the Driver Station communications light first (see §5).

---

## 4. Running the simulator (no robot needed)

```
./gradlew simulateJava
```

Or in VS Code: `Ctrl+Shift+P` → **WPILib: Simulate Robot Code**.

The simulator opens a Sim GUI with a fake Driver Station. You can map keyboard keys
to joystick axes/buttons in there — the defaults for our sim are in `simgui-ds.json`.

`run_sim.bat` in the project root is a shortcut for Windows.

---

## 5. Driver Station setup (this is where the shooter controls issue comes from)

**Why the shooter controls "aren't working" after a fresh install:** on a fresh
Driver Station, joysticks are assigned to USB slots in whatever order Windows
enumerated them — which is almost never the order the code expects. Our code reads:

| Role     | USB slot | Defined in                                          |
|----------|----------|-----------------------------------------------------|
| Driver   | 0        | `Constants.GamePadIDs.DRIVER_GAMEPAD_ID   = 0`      |
| Operator | 1        | `Constants.GamePadIDs.OPERATOR_GAMEPAD_ID = 1`      |

If the operator controller ends up on slot 0 or slot 2 instead of slot 1, **every
operator binding — including the entire shooter — silently does nothing**. The robot
isn't broken; the Driver Station just hasn't been told which controller is which.

### Steps — every time the DS is freshly installed (or a USB cable moves)

1. Plug both Xbox controllers into the driver laptop.
2. Open the **FRC Driver Station**.
3. Click the **USB tab** (the gamepad/joystick icon on the left-hand column). You
   will see a list of four slots, 0 through 3.
4. Look at the greyed-out entries — each shows the controller name Windows sees
   (e.g. `Controller (Xbox One For Windows)`). They'll be in the wrong slots on a
   fresh install.
5. **Drag and drop** each controller onto the slot it should live in:
   - **Slot 0** → driver controller (drives the swerve base)
   - **Slot 1** → operator controller (everything shooter / elevator / hood / feeder)
6. Confirm the slot is "locked" — the entry turns green/active and the row is no
   longer greyed out.
7. **Wiggle the sticks** on each controller and watch the axis bars in the USB tab
   move. Press each face button and watch the button LEDs light up. If a button
   doesn't light up, the controller itself is dead — swap it.
8. Enable the robot in **Teleop** and re-test.

After you've done this once, the DS remembers the assignment **as long as you keep
plugging the same controller into the same physical USB port on the laptop**. If you
swap USB ports, you have to redo step 5.

### If the slots keep getting reshuffled

Open the Driver Station's **Setup tab** and make sure **Dashboard Type** is
`SmartDashboard` (we use SmartDashboard for the auto chooser — see §7). Then save
the DS config: top-right gear → **Save**. That writes a DS profile that survives
reboot.

---

## 6. Controller map

Both controllers are Xbox 360 / Xbox One style gamepads. The map below is what the
code actually binds right now — source of truth is
`src/main/java/frc/robot/RobotContainer.java`.

### Driver controller (USB slot 0)

| Input                          | Action                                              |
|--------------------------------|-----------------------------------------------------|
| Left stick Y                   | Drive forward / back (field-centric)                |
| Left stick X                   | Strafe left / right (field-centric)                 |
| Right stick X                  | Rotate                                              |
| **Left bumper**                | Reset field-centric heading (press once)            |
| **Right bumper (hold)**        | Slow drive mode                                     |
| **X button (hold)**            | Swerve X-brake (wheels form an X)                   |
| **B button (hold)**            | Point wheels toward left-stick direction            |
| Back + Y                       | SysId dynamic forward (tuning only)                 |
| Back + X                       | SysId dynamic reverse (tuning only)                 |
| Start + Y                      | SysId quasistatic forward (tuning only)             |
| Start + X                      | SysId quasistatic reverse (tuning only)             |

### Operator controller (USB slot 1)

| Input                          | Action                                              |
|--------------------------------|-----------------------------------------------------|
| **Left bumper (hold)**         | **FIRE** — spin up shooter, then run kicker + conveyor |
| **Right bumper (hold)**        | Clear out — run shooter/kicker/conveyor reversed    |
| **Left stick click (hold)**    | LAUNCH — far-shot shooter speed + kicker + conveyor |
| **Right stick click (hold)**   | Feeder in (pickup)                                  |
| **Y button (press)**           | Elevator up to top (PID)                            |
| **A button (hold)**            | Elevator down                                       |
| **X button (hold)**            | Hood down                                           |
| **B button (hold)**            | Hood up                                             |
| **Start button (hold)**        | Conveyor forward                                    |
| **Back button (hold)**         | Conveyor reverse                                    |

All operator buttons release to a "stop" command (ceaseFire / stopElevator /
stopHood / stopConveyor / stopFeeder), so letting go always kills the motor.

---

## 7. Autonomous

Autos are built with PathPlanner. Paths and autos are stored under:

```
src/main/deploy/pathplanner/paths/
src/main/deploy/pathplanner/autos/
```

The selected auto comes from the **SmartDashboard** `Auto Chooser` widget —
`RobotContainer.java` builds it with `AutoBuilder.buildAutoChooser()`. Before a
match:

1. Deploy the code.
2. Open **SmartDashboard** (comes with WPILib).
3. Pick the auto from the `Auto Chooser` dropdown.
4. Save the SmartDashboard layout so the chooser shows up next power-cycle.

Named commands wired to PathPlanner right now (see `RobotContainer.configureBindings`):

- `Shoot` → `AutoFire`
- `FeederIn` → `FeederIn`

If you add a new named command to a `.auto` file, you **must** also register it with
`NamedCommands.registerCommand(...)` in `RobotContainer.RobotContainer()` or the
auto will silently skip that step.

---

## 8. Troubleshooting

**"I press the operator buttons and nothing happens."**
→ 99% of the time this is §5 — operator controller isn't in USB slot 1. Open the DS
USB tab and check.

**"Driver controls work, operator does nothing."**
→ Same as above. Proves one controller is on slot 0; the other isn't on slot 1.

**"Shooter spins but ball doesn't launch."**
→ Hold longer. `AutoFire` waits 0.7 s for the shooter to spin up before the kicker +
conveyor engage (`AutoFire.java` line 35). Short taps never make it past spin-up.

**"Shooter runs backwards."**
→ Check the motor phase / inversion in `MotorControllers.java` and the sign of
`Constants.MotorSpeeds.SHOOTER`. The shooter uses closed-loop velocity
(`m_request.withVelocity(-39)` in `OperatorSubsystem.shooterOut()`) — the negative
sign is intentional; flipping it reverses the wheel.

**"Deploy succeeds but robot doesn't move."**
→ Check the Driver Station communications + robot code lights. If comms is green
and robot code is red, the robot program crashed on startup — open
`RioLog`/`Driver Station Log Viewer` and look for the stack trace.

**"CAN bus error / motor not responding."**
→ Verify CAN IDs against `Constants.ControllerIDs` using Phoenix Tuner X. Every
motor ID in the table there needs to match a physical controller on the bus.

**"The DS keeps forgetting my controller slots."**
→ Always plug the same controller into the same USB port. Save the DS config
(top-right gear → Save) after you set up the slots.
