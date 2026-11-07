---
layout: post
title: "PVT-Tolerant CMOS Inverter Amplifiers Using Ring-Oscillator-Based Dynamic Voltage Scaling"
date: 2026-11-07 13:00:00 -0500
description: "How a replica ring oscillator, a frequency counter, and a programmable LDO form a feedback loop that keeps the bandwidth of self-biased CMOS inverter amplifiers stable across process, voltage, and temperature."
tags: analog cmos inverter-amplifier pvt ldo ring-oscillator dynamic-voltage-scaling
categories: analog-ic-design
related_posts: false
toc:
  sidebar: right
thumbnail: assets/img/posts/pvt-tolerant-inverter-amplifier-dvs/dvs-schematic-lakshmikumar-jssc2019.png
citation: true
disqus_comments: true
---

## The CMOS inverter as an amplifier

The CMOS inverter, biased at mid-supply ($$ V_{DD}/2 $$) for maximum gain linearity, offers several benefits as an amplifier building block [1]:

1. The input drives both devices, so the transconductances add: $$ g_m = g_{mn} + g_{mp} $$.
2. It operates in class AB, which is more power efficient than a class-A stage.
3. It has only one input node and one output node, so there are fewer parasitic poles to deal with than in, say, a folded-cascode stage.
4. It offers the highest output swing for a given supply.

The push-pull action is also what makes the stage linear. Writing the large-signal output current as the difference of the NMOS and PMOS currents,

$$
i_o = i_n - i_p = \left[k_n V_{OV,n} + k_p V_{OV,p}\right] v_i + \frac{1}{2}\left(k_n - k_p\right) v_i^2
$$

where $$ k = \mu C_{ox} W/L $$ is the conductance constant of each device and $$ V_{OV} $$ its overdrive. Notice that the square term is proportional to $$ k_n - k_p $$. If we size $$ W_n $$ and $$ W_p $$ such that $$ k_n = k_p $$, the square term cancels and the output current becomes a linear function of $$ v_i $$, which is a rather nice result for a two-transistor circuit. Even with some mismatch, the second-order term is only proportional to the difference in the $$ k $$ values [1].

## Noise first, linearity last

In practice, the amplifier is implemented as a cascade of gain stages. The noise requirement of the first stage matters the most. For $$ n $$ cascaded stages, the Friis equation gives the total noise factor as [4]

$$
F_{total} = F_1 + \frac{F_2 - 1}{G_1} + \frac{F_3 - 1}{G_1 G_2} + \cdots + \frac{F_n - 1}{G_1 G_2 \cdots G_{n-1}}
$$

where $$ F_i $$ is the (linear) noise factor of stage $$ i $$, with $$ NF_i = 10\log_{10} F_i $$, and $$ G_i $$ is its available power gain. Notice that the excess noise of each stage is divided by the total gain that precedes it, so after a reasonably high-gain first stage, the later stages contribute little.

Meanwhile, the signal swing grows from stage to stage, so linearity becomes more and more important further down the chain. Assuming third-order nonlinearity dominates and neglecting any phase cancellation between distortion components, the cascaded input-referred intercept is approximately [4]

$$
\frac{1}{A_{IIP3,total}^2} \approx \frac{1}{A_{IIP3,1}^2} + \frac{1}{A_{v1}^2 A_{IIP3,2}^2} + \frac{1}{A_{v1}^2 A_{v2}^2 A_{IIP3,3}^2} + \cdots
$$

where $$ A_{IIP3,i} $$ is the input intercept amplitude of stage $$ i $$ and $$ A_{vi} $$ its voltage gain. This is the mirror image of the Friis equation: each later stage's intercept is divided by the gain *before* it, so its nonlinearity is effectively magnified. The largest nonlinearity contributions therefore come from the later stages.

Therefore, the first stage is optimized for noise and input matching, and the subsequent stages are optimized for linear gain and bandwidth.

## The problem: an inverter has no bias current source

Skipping over the details of the amplifier implementation, we now focus on the process, voltage, and temperature (PVT) sensitivity of the inverter-based amplifier.

Traditional designs that use differential pairs have good common-mode and power-supply noise suppression. The inverter has no such immunity, and, more importantly, it is not biased by a current source; it is self-biased. Its bias current and transconductance are set entirely by the power supply and the devices themselves:

\begin{equation}
\label{eq:inverter-gm}
g_m \approx k V_{OV} = k\left(V_{IN} - V_T\right) = k\left(\frac{V_{DD}}{2} - V_T\right)
\end{equation}

This immediately tells us where the sensitivity comes from. $$ V_{DD} $$ and $$ V_T $$ appear side by side, and the overdrive $$ V_{DD}/2 - V_T $$ is typically small, so a modest shift in $$ V_T $$ or in mobility (through $$ k $$) produces a large relative change in $$ g_m $$. While the gain of the amplifier is somewhat stabilized by its feedback resistor, its bandwidth is primarily set by $$ g_m/C $$ and is therefore very sensitive to PVT. If the supply and the threshold vary, the bandwidth, and to a lesser extent the gain, of the inverter will vary significantly. Mitigating these variations is essential for a high-yield, robust design.

## Why not a constant-$$ g_m $$ bias?

A traditional constant-$$ g_m $$ bias circuit consumes static current, and its devices (and therefore its power) must be scaled up to suppress random mismatch. A ring oscillator, on the other hand, consumes only dynamic power, and that power is nearly independent of the number of stages. It is therefore possible to use a large number of stages to average out random variation in the oscillation frequency [2].

## Closing a loop around the supply

Since the supply sets the bias, the supply is also the natural control knob. What we want is a mechanism that detects which process corner and temperature the chip is operating at, and uses that information in a feedback loop to adjust $$ V_{DD} $$. For this purpose, an on-chip programmable linear regulator (LDO) is added, as shown in Figure 1 [1]. The LDO isolates the signal path from power-supply and substrate noise, and it can be calibrated so that bandwidth and gain variation across corners is minimized.

{% include figure.liquid path="assets/img/posts/pvt-tolerant-inverter-amplifier-dvs/dvs-schematic-lakshmikumar-jssc2019.png" title="Dynamic voltage scaling scheme: programmable linear regulator with replica load, process and temperature monitor (ring oscillator with MPT and MNT), and frequency comparator driven by an external reference clock" class="img-fluid rounded z-depth-1" zoomable=true %}

*Figure 1.* Schematic of the dynamic voltage scaling (DVS) scheme. The regulator output (0.75 V to 1.25 V) supplies both the 53 GBd linear TIA and the process and temperature monitor. Reproduced from Lakshmikumar et al. [1], Fig. 15.
How do we detect PVT variation on the chip? Briefly put, a ring oscillator (RO) made of the same unit-sized inverter acts as a PVT sensor [1]. Its frequency depends on the same mobility and threshold voltage as the amplifier's $$ g_m $$. One distinction is worth keeping in mind. The amplifier's inverters are biased at $$ V_{DD}/2 $$, so their overdrive is $$ V_{DD}/2 - V_T $$. The RO swings rail to rail, so its delay is governed by the full-supply overdrive $$ V_{DD} - V_T $$ (Eq. \eqref{eq:ro-delay} below). The next section shows how the two are nevertheless linked. The RO frequency is compared with an external reference clock in the frequency comparator. The digital output of the comparator then represents the process corner and temperature of the chip [1]. This digital output programs the LDO, which changes $$ V_{DD} $$ until the RO reaches a targeted frequency. That target represents the nominal response of the inverter (or TIA). The loop cannot change $$ V_T $$ or the mobility, but by adjusting the supply it compensates for their variation, so the inverter's transconductance stays tightly controlled even as PVT varies.

## The ring oscillator as a PVT sensor

When an odd number of inverters is connected in a ring, there is no stable DC operating point, and the loop satisfies the Barkhausen criterion at the frequency where the stage delays supply the remaining phase shift. The circuit therefore oscillates. Each stage's delay is set by how long its output current takes to charge the capacitance of the next stage, roughly $$ t_p \approx CV/(2I) $$. The oscillation frequency is

\begin{equation}
\label{eq:ro-frequency}
f_{osc} = \frac{1}{2N \cdot t_p}
\end{equation}

where $$ N $$ is the number of stages and $$ t_p $$ is the average inverter propagation delay [2]. The factor $$ 2N $$ arises because one oscillation period contains $$ 2N $$ stage transitions: each stage switches once high-to-low and once low-to-high. Similarly, the factor $$ 1/2 $$ in $$ t_p \approx CV/(2I) $$ reflects the fact that each output only has to swing to the next stage's switching threshold, $$ V/2 $$. The symmetric NMOS and PMOS drive strengths play a different role: they make the rising and falling delays nearly equal, which is why a single average $$ t_p $$ can be used [2].

Assuming that the gate delay is dominated by slewing and that the transistors obey the square law, the inverter delay in the RO is [2]

\begin{equation}
\label{eq:ro-delay}
t_p = \frac{V_{DD}}{2} \cdot \frac{C_{gg} + C_{dd}}{\frac{W}{2L}\mu C_{ox}\left(V_{DD} - V_T\right)^2}
\end{equation}

Here $$ C_{gg} + C_{dd} $$ is the capacitance each stage drives (the next stage's gate plus its own drain), and the denominator is the square-law saturation current of a device with its gate at the full supply. ([2] writes the threshold in this equation as $$ V_{TH} $$; it is the same $$ V_T $$ as below.) Observe that the delay depends on exactly the same $$ \mu $$ and $$ V_T $$ as the amplifier's $$ g_m $$, so it moves with PVT in the same direction.

For the inverters in the analog signal path, the gates are biased at $$ V_{DD}/2 $$, so their transconductance is [2]

\begin{equation}
\label{eq:signal-gm}
g_m = \frac{W}{L}\mu C_{ox}\left(\frac{V_{DD}}{2} - V_T\right)
\end{equation}

Notice that $$ V_{DD} $$ directly sets the analog inverter's $$ g_m $$, and thus also its unity-gain frequency

$$
\omega_u = \frac{g_m}{C_{gg} + C_{dd}}, \qquad f_u = \frac{\omega_u}{2\pi}
$$

We now want to express the ring frequency in terms of this same $$ g_m $$ [2]. The derivation takes three steps.

**Step 1: Invert the stage delay.** Rearranging Eq. \eqref{eq:ro-delay}, the factor of $$ 1/2 $$ in $$ V_{DD}/2 $$ cancels the factor of 2 in $$ W/2L $$:

$$
\frac{1}{t_p} = \frac{2}{V_{DD}} \cdot \frac{\frac{W}{2L}\mu C_{ox}\left(V_{DD} - V_T\right)^2}{C_{gg} + C_{dd}} = \frac{W}{L}\mu C_{ox} \cdot \frac{\left(V_{DD} - V_T\right)^2}{V_{DD}\left(C_{gg} + C_{dd}\right)}
$$

Substituting into Eq. \eqref{eq:ro-frequency} gives the oscillation frequency in terms of device parameters:

$$
f_{osc} = \frac{1}{2N t_p} = \frac{1}{2N} \cdot \frac{W}{L}\mu C_{ox} \cdot \frac{\left(V_{DD} - V_T\right)^2}{V_{DD}\left(C_{gg} + C_{dd}\right)}
$$

**Step 2: Eliminate the device constant using the amplifier's $$ g_m $$.** The product $$ (W/L)\mu C_{ox} $$ is common to the ring and the amplifier, since both use the same unit inverter. Solving Eq. \eqref{eq:signal-gm} for it,

$$
\frac{W}{L}\mu C_{ox} = \frac{g_m}{\frac{V_{DD}}{2} - V_T}
$$

and substituting into the previous expression gives the oscillation frequency in terms of $$ g_m $$ [2]:

\begin{equation}
\label{eq:fosc-gm}
f_{osc} = \frac{1}{2N} \cdot \frac{\left(V_{DD} - V_T\right)^2}{V_{DD}\left(\frac{V_{DD}}{2} - V_T\right)} \cdot \frac{g_m}{C_{gg} + C_{dd}}
\end{equation}

Notice that the mobility, the oxide capacitance and the device aspect ratio have all disappeared. Only the supply, the threshold and the amplifier's own $$ g_m/C $$ remain.

**Step 3: Write it in terms of $$ f_u $$.** Since $$ g_m/(C_{gg} + C_{dd}) = \omega_u = 2\pi f_u $$,

\begin{equation}
\label{eq:fosc-fu}
f_{osc} = \frac{1}{2N} \cdot \alpha \cdot 2\pi f_u = \frac{\pi}{N}\,\alpha f_u
\end{equation}

where all the voltage dependence is collected into a single dimensionless factor:

\begin{equation}
\label{eq:alpha}
\alpha = \frac{\left(V_{DD} - V_T\right)^2}{V_{DD}\left(\frac{V_{DD}}{2} - V_T\right)}
\end{equation}

Eq. \eqref{eq:fosc-fu} is the central result. The ring frequency is the inverter's unity-gain frequency multiplied by a factor that depends only weakly on the operating point. In general $$ \alpha $$ is a function of $$ V_{DD} $$. However, when $$ V_{DD} \gg V_T $$, the threshold terms in Eq. \eqref{eq:alpha} become negligible, and

$$
\alpha \;\rightarrow\; \frac{V_{DD}^2}{V_{DD} \cdot \frac{V_{DD}}{2}} = 2, \qquad f_{osc} \;\rightarrow\; \frac{2\pi}{N} f_u
$$

so $$ f_{osc} $$ becomes directly proportional to $$ f_u $$. In [2], $$ V_T $$ is about 150 mV and the nominal $$ V_{DD} $$ is 700 mV. Evaluating Eq. \eqref{eq:alpha} at that point gives $$ \alpha = (0.55)^2 / (0.7 \times 0.2) \approx 2.16 $$, already close to the limiting value. The simulated $$ \alpha $$ in [2] stays within ±5% over the relevant supply range.

Notice also that the stage capacitance $$ C_{gg} + C_{dd} $$ appears in both $$ t_p $$ and $$ \omega_u $$. The ring therefore automatically tracks $$ g_m $$ *per unit capacitance*, which is exactly what sets the amplifier's poles.

We can therefore tune $$ V_{DD} $$ with the programmable LDO to set the oscillation frequency, and holding $$ f_{osc} $$ constant across corners stabilizes $$ f_u $$. This translates into a relatively fixed pole frequency in the amplifier's transfer function across corners [2].

### MPT and MNT

One key implementation detail of the RO-based sensor in Figure 1 is the pair of triode-region devices, MPT and MNT, between the ring and its two supply rails [1]. The inverter-based amplifier sees a very small swing, so its devices are biased near mid-supply and never see the full regulated voltage across any two terminals. The inverters in the RO, on the other hand, swing rail to rail. Since the regulated voltage can reach 1.25 V across corners, an unprotected RO device would see the full supply across its terminals. MPT and MNT are added to prevent this over-voltage, keeping the voltage across any two terminals of the RO devices within the rated limit [1].

## Measuring the ring frequency

Take a stable reference clock, for example 100 MHz, and open a counting window lasting $$ M $$ reference periods. Count the number $$ N $$ of RO edges inside this window:

$$
N = \frac{f_{RO}}{f_{REF}/M} = \frac{M f_{RO}}{f_{REF}}
$$

For example, a 2 GHz ring counted over a 150 ns window (15 periods of a 100 MHz reference) gives $$ N = 300 $$. One count is then $$ 1/300 \approx 0.3\% $$ of frequency. The reference crystal is accurate to parts per million, so the count is an absolute measure of this die's speed.

## The calibration loop

Now compare the measured $$ N $$ with a target $$ N^* $$:

- If $$ N > N^* + \text{margin} $$, the die is too fast, so $$ D $$ goes down by 1. We want a smaller $$ f_{osc} $$, or equivalently a larger $$ t_p $$. Reducing $$ V_{DD} $$ increases $$ t_p $$ and lowers the oscillation frequency.
- If $$ N < N^* - \text{margin} $$, the die is too slow, so $$ D $$ goes up by 1. We want a larger $$ f_{osc} $$, or a smaller $$ t_p $$. Increasing $$ V_{DD} $$ reduces $$ t_p $$ and raises the oscillation frequency.
- Otherwise, the loop stops. The margin is set by the tolerance (discussed below).

Note that $$ t_p $$ is not strictly inversely proportional to $$ V_{DD} $$. From Eq. \eqref{eq:ro-delay}, $$ t_p \propto V_{DD}/(V_{DD} - V_T)^2 $$, which decreases monotonically as $$ V_{DD} $$ increases. Monotonicity is all the loop needs: it guarantees that every correction moves the frequency in the intended direction.

The sign of the correction is what makes this negative feedback. A higher code gives a higher $$ V_{REG} $$, a faster ring, and a larger count, which makes the error larger. The rule then pushes the code back down. The correction always opposes the error, so the system works much like a thermostat.

There is a practical limit, however: the loop can only move $$ V_{DD} $$ as far as the supply range allows. Set the nominal $$ V_{REG} $$ below the maximum so that slow and hot dies have room to go up.

## The programmable LDO

A digital code $$ D $$ sets the supply in small voltage steps:

$$
V_{REG} = V_{min} + D\,\Delta V
$$

The Cisco design used $$ \Delta V = 8 $$ mV steps [3]. A subtle point is that the LDO's absolute accuracy does not matter. Even if the reference is 20 mV off, the feedback loop simply settles at a different code $$ D $$, because it locks to a frequency, not to a voltage. Consequently, the regulator does not need a bandgap reference; a reference clock on the board is sufficient [3].

## Choosing the design numbers

Several questions remain: how many stages should the RO have, what should the target frequency be, how is the frequency comparator built, and how is its output turned into an LDO code? The following numbers must be chosen.

### Number of ring stages

As discussed above, RO power is nearly independent of the number of stages, while random variation in the oscillation frequency averages out over more stages [2]. A longer ring is therefore preferable for mismatch. It also lowers the oscillation frequency for a given stage delay (Eq. \eqref{eq:ro-frequency}), which brings the count into a convenient range. For reference, the 740 MHz nominal frequency and 5 ps stage delay quoted in [2] imply, through Eq. \eqref{eq:ro-frequency}, a ring of roughly 135 stages.

### The target count $$ N^* $$

There are two sensible choices:

1. **The nominal corner.** Measure $$ N $$ at TT, 27 °C and the nominal supply, and use that as the target. This is the corner where the design has already been verified and works optimally, so the loop reproduces the verified design on every die.
2. **A performance target.** Choose the ring frequency that gives the desired amplifier bandwidth. Zheng et al. set their frequency target externally for the best CTLE bandwidth, power and performance [2], and the target in the Cisco design is programmable [1], [3].

The first choice is the natural starting point; the second is a refinement once the relationship between ring frequency and amplifier bandwidth has been characterized.

### Step size and tolerance

One code changes the count by

$$
G = N^* \cdot S \cdot \Delta V, \qquad S = \frac{\partial \ln f}{\partial V}
$$

Here $$ S $$ is the fractional change of ring frequency per volt, so $$ S\,\Delta V $$ is the fractional frequency change per code, and multiplying by $$ N^* $$ converts it to counts. With $$ S \approx 1.7\ \mathrm{V^{-1}} $$ and $$ \Delta V = 10 $$ mV, a single code changes the count by about 5.

The tolerance window around the target contains $$ 2\,\text{tol} + 1 $$ integer counts. The important point is that this window must be wider than one step:

$$
2\,\text{tol} + 1 > G
$$

To see why, suppose the window were narrower than one step. Then the count would jump over the window on every update. One code lands just above the window, so the loop steps down. The next code lands just below it, so the loop steps up. The loop then toggles between those two codes forever, which is a limit cycle. With the window wider than one step, at least one code always lands inside it, and the loop stops there.

### Timing

During each iteration, the loop must:

1. wait for the LDO to settle (at least $$ 5\tau $$);
2. count ring edges for $$ M $$ reference periods;
3. compare $$ N $$ with $$ N^* $$;
4. update $$ D $$.

For simulation, a counting window of a few hundred nanoseconds keeps the runtime short.

### Code range

The code range must cover the slowest and hottest corners without exceeding the device voltage rating.

### Biasing anything outside the regulated domain

Anything biased from outside the regulated domain must follow $$ V_{REG}/2 $$. Either AC-couple the amplifier input, or derive its DC level from $$ V_{REG} $$.

The reason follows from Eq. \eqref{eq:inverter-gm}. The inverter's trip point sits at about $$ V_{REG}/2 $$, and the loop deliberately moves $$ V_{REG} $$ from corner to corner. A source with a fixed DC level is no longer at the trip point once $$ V_{REG} $$ moves. The high-gain inverter amplifies that DC offset, and the output saturates at some corners. AC coupling, or a DC level derived from $$ V_{REG} $$, keeps the input centred on the trip point wherever the loop settles. The same applies to any following stage that assumes a fixed common-mode level.

### Foreground or background calibration

Decide between running the loop once at start-up (foreground) and re-checking periodically (background). In background mode, add hysteresis so that small drifts do not make the supply dither. In practice, this means a wider error threshold for restarting a correction than for stopping one.

## Is the Zheng–Murmann FSM implementation required?

In the implementation of Zheng et al. [2], the ring oscillator's output itself clocks a finite state machine (FSM). The FSM controls the LDO, which in that design regulates the ground of the core and reference circuits rather than the supply. The FSM uses an available 100 MHz reference to tune the LDO until the ring reaches a programmable frequency target (nominally 740 MHz, corresponding to a 5 ps inverter delay), with hysteresis. The target is set externally for the best CTLE bandwidth, power and performance, and the whole replica-bias block (reference diode, ring oscillator, LDO and FSM) can be shared across several transceiver channels to amortize its area and power [2].

For an ideal or behavioral feedback loop, most of these details are optional rather than necessary:

- **Clocking the FSM from the ring** is a hardware convenience, not a requirement. Any controller that counts ring edges in a reference-timed window and updates the code implements the same loop. The Cisco design uses firmware instead [1].
- **Regulating ground instead of the supply** is an interface choice in [2]. The Cisco design regulates the supply [1]. The sensing principle is the same either way.
- **Hysteresis** is worth keeping, because it prevents dithering in background operation.
- **Sharing the bias block** across channels matters only for area and power in a multi-channel chip.

What is necessary is common to both designs: a ring built from the same unit inverter on the same regulated net, a stable reference to measure its frequency against, a programmable regulator, and a controller whose correction has the right sign.

## Summary

A self-biased CMOS inverter is an efficient, linear amplifier, but its $$ g_m $$, and with it its bandwidth, is set directly by $$ V_{DD} - 2V_T $$ and the device mobility, which makes it highly sensitive to PVT. A ring oscillator built from the same unit inverter oscillates at a frequency proportional to the inverter's unity-gain frequency, $$ f_{osc} = (\pi/N)\,\alpha f_u $$, with $$ \alpha \approx 2 $$. Counting the ring's edges against a stable reference clock gives an absolute measure of the die's speed. A programmable LDO, driven by a simple negative-feedback controller, then adjusts the supply until the count matches a target. The result is a nearly constant $$ f_u $$ across corners, without a bandgap and with only dynamic power spent in the sensor. Its limits are the available supply range and how closely the rail-to-rail ring tracks the amplifier biased at mid-supply.

Behavioral modeling of the DVS feedback loop in Cadence will be explored in a future post.

## References

[1] K. R. Lakshmikumar, A. Kurylak, M. Nagaraju, R. Booth, R. K. Nandwana, J. Pampanin, and V. Boccuzzi, "A Process and Temperature Insensitive CMOS Linear TIA for 100 Gb/s/λ PAM-4 Optical Links," *IEEE Journal of Solid-State Circuits*, vol. 54, no. 11, pp. 3180–3190, Nov. 2019. [doi:10.1109/JSSC.2019.2939652](https://doi.org/10.1109/JSSC.2019.2939652)

[2] K. Zheng, Y. Frans, K. Chang, and B. Murmann, "A 56 Gb/s 6 mW 300 µm² inverter-based CTLE for short-reach PAM2 applications in 16 nm CMOS," in *Proc. IEEE Custom Integrated Circuits Conference (CICC)*, 2018. [doi:10.1109/CICC.2018.8357076](https://doi.org/10.1109/CICC.2018.8357076)

[3] K. Lakshmikumar, A. C. Kurylak, M. Nagaraju, and R. V. H. Booth, "Process and temperature insensitive linear circuit," U.S. Patent 10,505,509 B2, Dec. 10, 2019. [Google Patents](https://patents.google.com/patent/US10505509)

[4] B. Razavi, *RF Microelectronics*, 2nd ed. Upper Saddle River, NJ, USA: Prentice Hall, 2012.

{% comment %}

IMAGE MANIFEST — copy these files into the repository before deploying

Post image directory:
assets/img/posts/pvt-tolerant-inverter-amplifier-dvs/

Required files:
1. Source/role: Figure 1, DVS schematic (Lakshmikumar et al., JSSC 2019, Fig. 15); also used as the thumbnail
   Exact filename: dvs-schematic-lakshmikumar-jssc2019.png
   Exact repository path: assets/img/posts/pvt-tolerant-inverter-amplifier-dvs/dvs-schematic-lakshmikumar-jssc2019.png
   Used in post as: {% include figure.liquid path="assets/img/posts/pvt-tolerant-inverter-amplifier-dvs/dvs-schematic-lakshmikumar-jssc2019.png" title="Dynamic voltage scaling scheme: programmable linear regulator with replica load, process and temperature monitor (ring oscillator with MPT and MNT), and frequency comparator driven by an external reference clock" class="img-fluid rounded z-depth-1" zoomable=true %}
   Status: supplied

Thumbnail, if used:
Exact repository path: assets/img/posts/pvt-tolerant-inverter-amplifier-dvs/dvs-schematic-lakshmikumar-jssc2019.png

Deployment checks:
- Confirm every listed file exists at its exact case-sensitive path.
- Confirm every image include path in the article matches the manifest exactly.
- Confirm the thumbnail path, if present, matches a real file.
- Commit both the Markdown post and all listed image files before deployment.

{% endcomment %}
