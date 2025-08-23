## Brief Summary
This project developed a model-based controller for a DC motor with unknown specifications. Using system identification and MATLAB tools, the controller achieved a settling time of ~0.2 s. Stability was confirmed with Bode plots showing a gain margin of 16.3 dB and a phase margin of 64.6°, and disc margin analysis demonstrated robustness to combined gain and phase variations. The controller successfully regulated the motor’s velocity as intended.

0. [Hardware](#hardware)
1. [System Identification](#system-identification)
2. [Controller Design](#controller-design)
3. [Stability Analysis](#stability-analysis)
4. [Results](#results)

$~~~~~~~~~~$

## Hardware
The parts which I used in this project are listed below; they are connected as shown in Figure 1. 
- STM32F103C8T6 Microcontroller
- ST-Link V2 Debugger
- 5V DC Motor
- MT6701 Hall Effect Encoder
- L293D Motor Driver
- HW-131 Breadboard Power Supply
- 9V Battery

<p align="center">
  <kbd>
    <img src="https://raw.githubusercontent.com/keatinl1/Motor_System_ID_and_PID_Control/refs/heads/main/figs/circuit.png">
  </kbd>
</p>
<p align="center">
Figure 1: Circuit Diagram.
</p>

$~~~~~~~~~~$

## System Identification
To identify the system dynamics, I applied a PRBS (Pseudo-Random Binary Sequence) voltage input to the motor, then measured the output angular velocity. There are two advantages to this method: 
  1) PRBS is simpler and faster to implement on embedded hardware than a frequency sweep
  2) The frequency spectrum of the random sequence is white noise, so all frequencies are excited with the same density.

After I collected the data, there were some clearly incorrect data points (outliers) which were removed by simply dropping the rows. The clean input-output data is shown in Figure 2.

<p align="center">
  <kbd>
    <img src="https://raw.githubusercontent.com/keatinl1/dc_motor_id_and_control/refs/heads/main/figs/prbs_input_output.png">
  </kbd>
</p>
<p align="center">
Figure 2: PRBS input-output.
</p>

An Autoregressive Moving Average with Exogenous Inputs (ARMAX) model was used in the MATLAB System Identification toolbox to make a discrete model of the system. A predictive accuracy of 74% was achieved.

The model was then transformed into the transfer function below. It has two poles and two zeros, which might seem like overkill for such a simple system, but simpler models had a steep drop in accuracy (down to 53%).

$$
  G(z^{-1}) = \frac{94.83 z^{-1} + 193.3 z^{-2}}{1 - 0.5899 z^{-1} - 0.03679 z^{-2}}
$$

$~~~~~~~~~~$

## Controller Design

The control requirements were a fast settling time, $\approx0.2s$, and as long as the PID controller was reasonably robust, that was acceptable.

Using the settling time equation below and setting the crossover frequency ($\omega_c$) to 20rad/s

$$
  T_s = \frac{4}{\omega_c}
$$

The MATLAB pidtune() app was used to design the controller, and the gains in the table below were found.
<p style="text-align: center;"><strong>Table 1: PID Gains</strong></p>

<div style="display: flex; justify-content: center;">
  <table>
    <tr><th>Gain</th><th>Value</th></tr>
    <tr><td>Kp</td><td>0.0038</td></tr>
    <tr><td>Ki</td><td>0.0254</td></tr>
    <tr><td>Kd</td><td>6.58e-05</td></tr>
  </table>
</div>

When the PID controller is applied to the model, the step response in Figure 3 is produced.
<p align="center">
  <kbd>
    <img src="https://raw.githubusercontent.com/keatinl1/dc_motor_id_and_control/refs/heads/main/figs/step_response.png">
  </kbd>
</p>
<p align="center">
Figure 3: Model Step Response.
</p>

$~~~~~~~~~~$

## Stability Analysis
For stability analysis, I will examine the classic gain and phase margins, which assume exclusive variations. Then I will also look at disc margin, which examines the combined gain & phase robustness.

The gain margin (GM) and phase margin (PM) were found using the MATLAB margin(L) function (where L is the controller C and the identified model G in series); the resulting Bode plot is shown in Figure 4. A gain margin of 16.3dB and a phase margin of 64.6 degrees is a very stable system. Usually, we aim for our GM to be greater than 2 and PM to be greater than 30. Since our system is already robust, there is no need to change the controller design.

Another way to confirm is modulus margin; this is the maximum gain of the sensitivity function (S = 1/(1+L)). We typically look for a gain of less than 2 (6dB) at the peak to imply good gain and phase margins. At the peak of the Bode magnitude plot of S, the maximum gain is 1.4 (2.93dB), which further confirms the stability findings.

<p align="center">
  <kbd>
    <img src="https://raw.githubusercontent.com/keatinl1/dc_motor_id_and_control/refs/heads/main/figs/margins.png">
  </kbd>
</p>
<p align="center">
Figure 4: Gain and Phase Margins.
</p>

Next, we can examine the disc margin. Using the diskmargin(L) function in MATLAB, the following margins can be found in Table 2.
<p style="text-align: center;"><strong>Table 2: Disc Margin Values</strong></p>

<div style="display: flex; justify-content: center;">
  <table>
    <tr><th>Metric</th><th>Value</th></tr>
    <tr><td>Gain Margin</td><td>[0.3255, 3.0722]</td></tr>
    <tr><td>Phase Margin</td><td>[-53.94°, 53.94°]</td></tr>
    <tr><td> Margin</td><td>1.0177</td></tr>
  </table>
</div>

These can also be visualised as shown in Figure 5. Where the unit circle is in black and the disc is in blue. Where the disc intersects the real axis is the disc margin values 0.3255 and 3.0722. Where the disc intersects the unit circle at -53.94° and 53.94° is the phase margin.

<p align="center">
  <kbd>
    <img src="https://raw.githubusercontent.com/keatinl1/dc_motor_id_and_control/refs/heads/main/figs/disk_margin_new.png">
  </kbd>
</p>
<p align="center">
Figure 5: Disc Margins.
</p>

How we interpret these values is that any combination of gain and phase inside that disc guarantees closed-loop stability.

$~~~~~~~~~~$

## Results
Finally, when the controller is applied to the system, the result in Figure 6 is achieved.
<p align="center">
  <kbd>
    <img src="https://raw.githubusercontent.com/keatinl1/dc_motor_id_and_control/refs/heads/main/figs/results1.png">
  </kbd>
</p>
<p align="center">
Figure 6: Velocity Control Achieved.
</p>
