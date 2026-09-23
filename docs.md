# FRC Robot Code - Build & Run Guide

This is a WPILib FRC (First Robotics Competition) Java project using Gradle.

## Prerequisites

- Java 17 (required)
- WPILib tools (for real robot deployment)

## Gradle Commands

### Build & Test
```bash
./gradlew build        # Build the project
./gradlew test         # Run unit tests
./gradlew jar          # Create deployable JAR
```

### Simulation
```bash
./gradlew simulateJava     # Run robot simulation with GUI
./gradlew simulateJavaDebug # Run simulation with debug
```

### Deploy to Robot
```bash
./gradlew deploy           # Deploy to RoboRIO (requires driver station)
```

### Code Tasks
```bash
./gradlew check            # Run all checks (tests, static analysis)
./gradlew clean            # Clean build artifacts
```

### PathPlanner Simulation
```bash
./gradlew runPPSim         # Run offline PathPlanner controller sim
./gradlew plotPPSim        # Plot simulation results to PNG
```

## Project Structure

- `src/` - Robot source code
- `vendordeps/` - Vendor dependencies (Phoenix, PathPlanner, etc.)
- `build/` - Build output
- `gradle/` - Gradle wrapper

## Notes

- Desktop support is disabled (`includeDesktopSupport = false`) - simulation is for testing logic only
- Team number is loaded from `.wpilib/wpilib_preferences.json` or passed via command line
- To deploy: connect to RoboRIO via USB/ethernet and run `./gradlew deploy`