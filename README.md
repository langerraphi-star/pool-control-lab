# Pool Water-Level Control

An interactive simulation of automatic makeup-water control for a swimming pool. A PI controller adjusts the fill valve as water is lost through evaporation, bather splash, leakage, and backwashing. Rain adds water to the pool, and a Twiddle search can tune the controller for the selected conditions.

**[Try the live simulation](https://pool-control-lab.onrender.com/)**

## Background

I started this project in CODESYS, using Ladder Logic and Structured Text to build the controller, pool model, interlocks, and automatic tuning routine. My work in pool design gave me a familiar system to use for exploring PLC programming and feedback control.

The browser version makes the project easier to try without installing CODESYS. It was built with AI assistance around the original PLC logic, including the JavaScript translation, interface, and additional valve-response modeling. This repository contains the browser demo; the original PLC implementation was developed separately in CODESYS.

## Using the demo

The simulation starts at real-time speed. Higher speeds are available for watching changes that would otherwise take several minutes or hours.

1. Click **Change Pool Parameters** to pause the pool.
2. Adjust the environment, evaporation, maximum fill rate, occupancy, splash loss, rain, or leakage.
3. Click **Recalibrate** to run the tuning search.
4. When calibration finishes, click **Run simulation** to apply the new gains and resume.

Backwash can be started or stopped while the simulation is running. Its flow rate can be changed while backwash is off. A full cycle lasts five simulated minutes—five seconds on screen at 60× speed.

You can also disable automatic filling or simulate a level sensor stuck at a low reading. The charts and readouts show how the pool and controller respond.

## How it works

The pool has a target volume of **100,000 gallons** and a surface area of **2,500 square feet**. The default maximum fill rate is **50 GPM**. Each calculation advances the model by **0.1 simulated seconds**.

The model adds fill water and rainfall, subtracts evaporation, splash, leakage, and backwash, and converts the resulting volume into a level deviation from the target waterline. At this surface area, one inch of water is about 1,558 gallons, so small changes in level represent a substantial amount of water.

The PI controller uses the measured level error to calculate a valve demand between 0% and 100%. Conditional anti-windup prevents the integral from accumulating further when the output is saturated in the direction of the error. Actual valve position moves toward that demand, with a full stroke taking 20 simulated seconds. The level sensor has a fixed five-second first-order response.

An independent high-level interlock trips at **one inch above target** and commands the fill valve closed. The trip stays latched until it is manually reset with the level below that threshold. Applying new gains preserves the valve position and adjusts the integral to avoid a sudden change in controller output.

### Automatic tuning

Twiddle changes Kp and Ki one at a time, testing positive and negative adjustments as needed. Each candidate is evaluated in a separate 100-second simulation using a snapshot of the current pool conditions.

The score is the accumulated absolute water-level error over the trial. Improvements are accepted, successful search increments grow, and unsuccessful increments shrink. The search stops when both increments reach their tolerances or after 100 complete passes. Candidate trials that reach the high-level limit are rejected.

Backwash and sensor faults are excluded from tuning. The winning gains are the best found for that trial scenario, rather than a guarantee of optimal performance under every condition.

## A few modeling choices

- Outdoor evaporation is twice the base evaporation setting. Splash loss is proportional to the number of bathers and the per-bather splash setting.
- The fill valve can add water but cannot drain the pool. Excess rain can raise the level even with the valve closed; losses above the maximum fill capacity will lower it even with the valve fully open.
- Waterline movement is exaggerated in the illustration so that fractions of an inch are visible. The numeric readouts and chart use the actual simulated values.
- Valve motion and the associated flow illustration are slowed for visibility, independently of simulation speed. The actual position and flow remain available in the numeric readouts.

This is a controls demonstration, not a design for operating a physical pool.

## Running locally

Open `index.html` in a browser. There are no packages to install or build commands to run; the interface, controller, tuning routine, and pool model all run in the browser.

Built with CODESYS, Ladder Logic, Structured Text, JavaScript, HTML, CSS, and Canvas. The live browser demo is hosted on Render.
