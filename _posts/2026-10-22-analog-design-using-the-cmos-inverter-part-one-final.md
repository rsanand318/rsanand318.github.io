---

layout: post

title: "Analog Design Using The CMOS Inverter -- Part 1"

date: 2026-10-22 18:00:00 -0400

description: "Introduction, VTC, and Resistive Feedback Inverter For High-Speed"

tags: [mixed-signal, CMOS, analog, inverter, high-speed]

categories: [analog-design, mixed-signal]

related_posts: true

toc:

  sidebar: right

# thumbnail: assets/img/posts/mosfet-device-behavior-circuit-designers/mosfet-cross-section.png

citation: false

---

Technology scaling has been extraordinarily effective for digital circuits. Smaller devices and lower supply voltages have enabled higher density and switching speed, while process development has continued to improve the trade-off between on-state current and leakage. Analog design has benefited from the same technology, but not always in the same way.

A conventional analog amplifier often relies on a large transistor output resistance to obtain voltage gain. To first order, the intrinsic gain of a MOS transistor is

$$

A_{v,\mathrm{int}} = g_m r_o = \frac{g_m}{g_{ds}},

$$

where $g_m$ is the transconductance and $r_o=1/g_{ds}$ is the small-signal output resistance. Short-channel effects, including channel-length modulation and drain-induced barrier lowering (discussed in my previous [article](https://rsanand318.github.io/blog/2026/mosfet-device-behavior-circuit-designers/#short-channel-effects-drain-induced-barrier-lowering-dibl)), make it harder to preserve a large $r_o$ as channel lengths shrink. At the same time, the supply voltage has generally scaled faster than the threshold voltage. The available voltage headroom for stacking devices has therefore become more restrictive.

This is one reason that familiar gain-enhancement techniques, especially cascoding, have become difficult to use in low-voltage processes. A cascode can raise output resistance, but it also consumes voltage headroom and adds internal nodes and capacitances. Those extra nodes can reduce large-signal speed, even when the small-signal output resistance looks attractive. Whether a cascode is worthwhile depends on the process, bias conditions, load, and performance target; the point is that its benefits no longer look as attractive.

A CMOS inverter offers a different starting point. It contains a PMOS and an NMOS connected in a complementary push-pull arrangement, with their gates driven by the same input and their drains joined at the output. There is no cascode stack. More importantly, when the inverter is biased in its transition region, both devices conduct and both contribute transconductance. With the output capacitance held fixed, the effective transconductance is approximately

$$

g_{m,\mathrm{inv}} = g_{mn}+g_{mp}.

$$

The inverter can therefore provide substantial transconductance per unit of voltage headroom. This is especially useful in circuits where bandwidth and energy per transition matter more than obtaining a very large open-loop voltage gain.

The key question is how to bias the inverter in the *transition region*, where its transconductance is high. As a digital gate, the inverter is normally driven between the supply rails and spends most of its time near one of the two logic states. Digital designers generally avoid the center region to reduce the risk of metastability. As an analog circuit, however, the inverter is biased near the steep portion of its voltage-transfer curve, where a small change at the input produces a much larger change at the output. The same transistor pair supports both uses; the operating point determines which behavior we exploit.

A simple resistor connected from the output back to the input can establish this operating point without a separate precision bias circuit. That idea leads to several useful circuits, including inverter amplifiers, transimpedance amplifiers (TIAs), clock and data buffers, and source-series-terminated output drivers. The following sections build these applications from the inverter’s DC behavior and small-signal model.

## Section I: Voltage-transfer characteristics (VTC)

### What changes when the input is between the rails?

A CMOS inverter is usually introduced as a logic element: a low input produces a high output, and a high input produces a low output. The more interesting question for analog design is what happens while the input moves continuously between those two extremes.

Let the supply be $V_{DD}$, the NMOS threshold be $V_{TN}>0$, and the magnitude of the PMOS threshold be $\lvert V_{TP}\rvert$. We will initially use the long-channel square-law model. This is useful for understanding the operating regions and deriving a first estimate of the switching point, although it is not a precise device model for a modern nanoscale process.

The NMOS overdrive is

$$

V_{OV,n}=V_{in}-V_{TN},

$$

and the PMOS overdrive, using the magnitude convention for its threshold, is

$$

V_{OV,p}=V_{DD}-V_{in}-\lvert V_{TP}\rvert.

$$

The NMOS begins to conduct strongly when $V_{in}$ exceeds $V_{TN}$. The PMOS remains on while $V_{in}<V_{DD}-\lvert V_{TP}\rvert$. Between these limits, both transistors can conduct at the same time. That overlap is the operating region of interest for analog amplification.

### Five regions of the inverter

{% include figure.liquid path="assets/img/posts/Analog-Design-Using-The-CMOS-Inverter-Part-One/VTC.png" title="Voltage Transfer Characteristics of the Inverter" class="img-fluid rounded z-depth-1" zoomable=true %}

**Region A: NMOS nominally off, PMOS in triode.** When $V_{in}<V_{TN}$, the NMOS is off in the ideal cutoff model. The PMOS is on because its gate is low, and its drain-to-source voltage is small when $V_{out}$ is close to $V_{DD}$. It therefore operates in triode and pulls the output toward the supply. In an actual transistor, subthreshold current does not abruptly become zero below threshold, so the output can begin to move slightly before the ideal threshold is reached. The amount of movement depends on leakage, loading, temperature, and the rest of the circuit.

**Region B: NMOS in saturation, PMOS in triode.** As $V_{in}$ rises above $V_{TN}$, the NMOS begins to pull current from the output toward ground. While the output remains high, the NMOS drain-source voltage is large relative to its overdrive, so it is in saturation. The PMOS still has a small source-drain voltage and remains in triode. Increasing the input strengthens the NMOS and reduces the PMOS overdrive, so the output starts to fall more rapidly.

**Region C: both transistors in saturation.** As the output falls, the PMOS source-drain voltage grows. Once it is large enough, the PMOS also enters saturation. There is then a range in which both devices conduct in saturation. This region nominally contains the steepest part of the voltage-transfer curve. A rise in $V_{in}$ increases the NMOS pull-down current and decreases the PMOS pull-up current. Both effects move the output downward. In the small-signal model, their transconductances **add**, which is why this region is advantageous for voltage gain and bandwidth.

**Region D: NMOS in triode, PMOS in saturation.** As the output falls further, the NMOS drain-source voltage eventually becomes smaller than its overdrive. The NMOS enters triode, while the PMOS can remain in saturation. The output is now close to ground, and further increases in input mainly strengthen the pull-down path.

**Region E: PMOS nominally off, NMOS in triode.** When $V_{in}\geq V_{DD}-\lvert V_{TP}\rvert$, the PMOS is off in the ideal cutoff model. The NMOS pulls the output toward ground. As in Region A, practical leakage and short-channel effects soften the ideal transition.

### Deriving the input trip-point

A useful first estimate of the inverter switching point, often denoted $V_M$, is the point on the VTC where the input and output are equal:

$$

V_{in}=V_{out}=V_M.

$$

At this point, the output voltage is not being forced high or low. The NMOS pull-down current and PMOS pull-up current have equal magnitudes. Under the square-law saturation model,

$$

I_{D,n}=\frac{1}{2}\beta_n(V_M-V_{TN})^2,

$$

$$

\lvert I_{D,p}\rvert=\frac{1}{2}\beta_p(V_{DD}-V_M-\lvert V_{TP}\rvert)^2,

$$

where

$$

\beta_n=\mu_n C_{ox}\left(\frac{W}{L}\right)_n,

\qquad

\beta_p=\mu_p C_{ox}\left(\frac{W}{L}\right)_p.

$$

The parameters $\mu_n$ and $\mu_p$ are the carrier mobilities in the NMOS and PMOS, $C_{ox}$ is the oxide capacitance per unit area, and $W/L$ is the device aspect ratio. Current balance requires:

$$

\frac{1}{2}\beta_n(V_M-V_{TN})^2

=

\frac{1}{2}\beta_p(V_{DD}-V_M-\lvert V_{TP}\rvert)^2.

$$

Cancel the $1/2$ factor and take the square root, and we get:

$$

\sqrt{\beta_n}(V_M-V_{TN})

=

\sqrt{\beta_p}(V_{DD}-V_M-\lvert V_{TP}\rvert).

$$

Expanding the right-hand side and collecting the $V_M$ terms gives:

$$

V_M(\sqrt{\beta_n}+\sqrt{\beta_p})

=

\sqrt{\beta_n}V_{TN}

+\sqrt{\beta_p}(V_{DD}-\lvert V_{TP}\rvert).

$$

Therefore, the square-law estimate of the switching voltage is

$$

{

V_M=

\frac{\sqrt{\beta_n}V_{TN}+\sqrt{\beta_p}(V_{DD}-\lvert V_{TP}\rvert)}

{\sqrt{\beta_n}+\sqrt{\beta_p}}

}

$$

This expression is a weighted average of the NMOS turn-on voltage, $V_{TN}$, and the PMOS turn-off voltage, $V_{DD}-\lvert V_{TP}\rvert$. The weights are the square roots of the transistor strength parameters. A stronger NMOS pulls the switching point downward; a stronger PMOS pulls it upward.

It is worth being careful with the overdrive notation here. The NMOS and PMOS do not generally have the same overdrive. Equating $\tfrac{1}{2}\mu C_{ox}V_{OV}^2$ for both devices without distinguishing their overdrives would omit both device sizing and the different distances from the input voltage to each threshold. The current-balance derivation above retains those differences.

The limiting cases are intuitive:

- If $\beta_n\gg\beta_p$, the NMOS is much stronger, and $V_M\rightarrow V_{TN}$.

- If $\beta_n\ll\beta_p$, the PMOS is much stronger, and $V_M\rightarrow V_{DD}-\lvert V_{TP}\rvert$.

- If the effective strengths are comparable, the switching point lies between these limits. It is exactly at $V_{DD}/2$ only when the device strengths and threshold magnitudes satisfy the required symmetry.

Since $\beta$ includes mobility as well as geometry, equal $W/L$ does not automatically mean equal drive strength. The analog sizing optimum can also differ from the digital sizing optimum: in digital use, designers often size the PMOS wider to balance rise and fall delays; in an inverter biased for analog operation, the relevant target may instead be transconductance per input capacitance, noise, linearity, or output swing. There is no universal PMOS-to-NMOS width ratio. The mobility ratio, device architecture, layout parasitics, and PVT corners all matter.

For short-channel devices, the square-law expression should be read as intuition rather than sign-off mathematics. Velocity saturation, mobility degradation, DIBL, body effect, and non-ideal output conductance change the current equations. In practice, the DC bias point and maximum gain should be extracted from the PDK model and verified across corners and mismatch.

An important trade-off is that there is a balance between the transconductance of the inverter—or how steep the VTC is—and how much input swing it can tolerate while operating linearly. However, in most receiver applications, like the TIA in an optical link, the input swing is low enough and higher gain is prioritized over linearity.

### Boundaries of the saturated region (Region C)

The labels $V_{TOP}$ and $V_{BOT}$ are useful when describing the output-voltage interval over which both transistors remain saturated for a given input bias $V_G$.

For the NMOS, saturation requires

$$

V_{DS,n}\geq V_{GS,n}-V_{TN}.

$$

Using $V_{DS,n}=V_{out}$ and $V_{GS,n}=V_G$ gives

$$

V_{out}\geq V_G-V_{TN}.

$$

At equality, the NMOS is just at the boundary between saturation and triode. Define this lower boundary as

$$

{V_{BOT}=V_G-V_{TN}.}

$$

For the PMOS, saturation requires

$$

V_{SD,p}\geq V_{SG,p}-\lvert V_{TP}\rvert.

$$

Since $V_{SD,p}=V_{DD}-V_{out}$ and $V_{SG,p}=V_{DD}-V_G$, this becomes

$$

V_{DD}-V_{out}\geq V_{DD}-V_G-\lvert V_{TP}\rvert,

$$

or

$$

V_{out}\leq V_G+\lvert V_{TP}\rvert.

$$

At equality, the PMOS is just at the saturation-to-triode boundary. Define the upper boundary as

$$

{V_{TOP}=V_G+\lvert V_{TP}\rvert.}

$$

Thus, at a fixed input bias $V_G$, the output lies in the both-saturated interval when

$$

{V_G-V_{TN}\leq V_{out}\leq V_G+\lvert V_{TP}\rvert.}

$$

These are output-voltage boundaries evaluated for a fixed input bias. They should not be mistaken for universal input-voltage breakpoints on the entire VTC, because each region boundary depends on both $V_{in}$ and $V_{out}$. The width of this idealized output interval is $V_{TN}+\lvert V_{TP}\rvert$, provided the chosen bias actually turns both devices on and the supply permits such an interval.

### Transfer function in Region C

Around a DC operating point, a small input perturbation can be represented by $v_{in}$ and the corresponding output perturbation by $v_{out}$. The small-signal drain currents caused by the changing gate voltage are approximately $g_{mn}v_{in}$ for the NMOS and $g_{mp}v_{in}$ for the PMOS. Although the devices have complementary polarities, both incremental effects pull the output in the same direction when the input rises: the NMOS draws more current from the output, while the PMOS supplies less current to it.

Consequently, the inverter's effective transconductance is

$$

g_{m,\mathrm{inv}}=g_{mn}+g_{mp}.

$$

If the output resistance of the two devices is $r_{on}$ and $r_{op}$, the unloaded small-signal output resistance is approximately

$$

r_{out}=r_{on}\parallel r_{op}.

$$

Ignoring additional loads for the moment, its low-frequency voltage gain is

$$

{A_v=-(g_{mn}+g_{mp})(r_{on}\parallel r_{op}).}

$$

The minus sign reflects the inverter's inverting behavior. With a load conductance $G_L$ and a load capacitance $C_L$, the one-pole approximation is

$$

A_v(s)=-

\frac{g_{mn}+g_{mp}}

{g_{ds,n}+g_{ds,p}+G_L+sC_L}.

$$

This expression shows the central advantage and the central limitation at once. Adding the PMOS transconductance can raise the numerator, but the attainable gain is still limited by the output conductance and the loading. The same input also drives both transistor gates, so the input capacitance includes contributions from both devices and their gate-drain parasitics.

The inverter is not automatically the best amplifier for every application. Its gain is sensitive to the operating point, its transfer curve is nonlinear, and both devices conduct DC current while it is biased in the transition region. Its appeal is strongest when low-voltage operation, high transconductance, compactness, and speed outweigh the need for high open-loop gain or very low static power.

Noise deserves a careful treatment. For a simple MOS channel thermal-noise model, each device contributes a drain-current noise density proportional to $4kT\gamma g_m$. If the two uncorrelated channel-noise currents are referred to the input through the combined transconductance, the simplified input-referred voltage-noise density scales approximately as

$$

S_{v,in}=

\frac{4kT(\gamma_n g_{mn}+\gamma_p g_{mp})}

{(g_{mn}+g_{mp})^2}.

$$

Under the assumptions of this model, increasing combined transconductance can reduce input-referred thermal voltage noise. However, one must confirm this with simulations, as the ideal model does not include bias current, flicker noise, source impedance, load, device sizing, noise transfer from internal nodes, and the actual topology.

## Section II: Self-biasing with resistive feedback

### Why a bias circuit is needed

As discussed in the preceding sections, the inverter has its greatest small-signal gain near its switching point, but a bare inverter does not provide a convenient independent DC bias input. If we simply apply a DC input voltage, small changes in device threshold, mobility, or supply can shift the operating point. A precision external reference and bias network could be added, but that consumes area, power, and voltage headroom and can complicate a high-speed signal path. Adding a complicated precision bias circuit would reintroduce many of the problems that the inverter approach is intended to avoid.

A resistor $R_F$ from output to input offers a compact alternative. At DC, a MOS gate draws negligible current in the ideal model. If the input is otherwise not connected to a DC source, essentially no DC current flows through $R_F$, and therefore the voltage drop across it is zero:

$$

I_{R_F,DC}=0

\quad\Longrightarrow\quad

V_{in,DC}= V_{out,DC}.

$$

{% include figure.liquid path="assets/img/posts/Analog-Design-Using-The-CMOS-Inverter-Part-One/inverter.png" title="Self-biased inverter with resistive feedback." class="img-fluid rounded z-depth-1" zoomable=true %}

The DC operating point must therefore lie near the intersection of the inverter VTC and the line $V_{out}=V_{in}$. That is the switching point $V_M$, where both devices conduct and the magnitude of the local slope is usually high.

The feedback action is intuitive. Suppose the input node is perturbed upward. The inverter output moves downward. Through $R_F$, that lower output pulls the input node back down. The feedback opposes the original perturbation and stabilizes the DC operating point. In other words, the resistor turns the inverter's own transfer curve into its bias mechanism.

The precise bias is not guaranteed to equal the square-law $V_M$ derived earlier. Finite DC input currents, leakage, other resistors, an attached source, asymmetry, and non-ideal device models can shift it. The practical design target is a stable point near the high-gain portion of the measured or simulated VTC.

### Small-signal behavior with feedback

{% include figure.liquid path="assets/img/posts/Analog-Design-Using-The-CMOS-Inverter-Part-One/model.png" title="Small signal model with resistive feedback" class="img-fluid rounded z-depth-1" zoomable=true %}

The feedback resistor also changes the AC behavior. Use the inverter small-signal model with combined transconductance $g_m=g_{mn}+g_{mp}$, output resistance $r_o$, and total output capacitance $C_L$. Let the input-node small-signal voltage be $v_x$ and the output voltage be $v_o$. Applying KCL at the output node gives

$$

g_m v_x+\frac{v_o}{r_o}+sC_Lv_o

=\frac{v_x-v_o}{R_F}.

$$

Rearrange to collect the output terms:

$$

v_o\left(\frac{1}{r_o}+sC_L+\frac{1}{R_F}\right)

=v_x\left(\frac{1}{R_F}-g_m\right).

$$

Multiplying the numerator and denominator by $R_F$ yields

$$

{

\frac{v_o}{v_x}

=\frac{1-g_mR_F}

{1+R_F/r_o+sR_FC_L}

}.

$$

This model assumes the input node is driven by a sufficiently low-impedance small-signal source, so $v_x$ can be treated as the input variable. The expression contains one dominant output pole, set by the parallel combination $R_F \parallel r_o$ driving the load capacitance $C_L$. Its 3-dB bandwidth is

$$

\omega_{3\mathrm{dB}}

=\frac{1+R_F/r_o}{R_FC_L}

=\frac{1}{r_oC_L}+\frac{1}{R_FC_L}.

$$

Without feedback, the corresponding one-pole approximation is

$$

\omega_{3\mathrm{dB},0}=\frac{1}{r_oC_L}.

$$

The resistor adds a conductance of $1/R_F$ at the output. In other words, the feedback resistor is in shunt with the output impedance, so the effective output impedance driving the load capacitance is smaller, and the dominant pole moves upward! This is the small-signal explanation for how resistive feedback can extend bandwidth and can be used in high-speed designs. However, this is only an approximation, because input capacitance, finite source impedance, gate-drain capacitance, and additional internal poles must be included in the simulation for an exact value of the 3-dB bandwidth.

There is also a trade-off here. The same feedback path that reduces the effective output resistance also changes the closed-loop gain and loads the input/output nodes. A smaller $R_F$ gives stronger feedback and more bandwidth extension, but it can reduce signal gain, increase current through the feedback path, and alter linearity and output swing. A larger $R_F$ reduces AC loading but weakens the bandwidth and bias-restoration effects. The best value depends on the target gain, input/output impedances, capacitances, and data rate.

With feedback, the input resistance seen by an input current is reduced approximately by the loop gain. If $A$ is the magnitude of the inverter's voltage gain, then a useful first-order estimate is

$$

{R_{in}=\frac{R_F}{1+A}.}

$$

The intuition is that feedback makes the input node move much less than it would without the amplifier. A small input voltage is needed to accommodate a given input current, so the input appears low impedance. This is especially useful in a TIA, where the photodiode capacitance would otherwise create a low-frequency input pole.

The main advantage of the feedback resistor is that it provides three useful functions in one component: it establishes the DC operating point, implements negative feedback in the signal path, and provides a design knob for bandwidth and gain. Since the inverter uses one PMOS and one NMOS rather than a multi-device stack, it also preserves voltage headroom and can exploit both transconductances.

For applications that require only AC amplification, it is common to isolate the external signal's DC level with an input coupling capacitor. The feedback resistor then establishes the internal bias independently of the external signal common-mode voltage.

## References

1. Analog Circuits by Prof. Shanti Pavan on YouTube. [Watch the video](https://www.youtube.com/watch?v=WWWvibZdL70).

2. W. Bae, “CMOS Inverter as Analog Circuit: An Overview,” *Journal of Low Power Electronics and Applications*, vol. 9, no. 3, article 26, 2019. https://doi.org/10.3390/jlpea9030026

3. B. Razavi, “Fifty Applications of the CMOS Inverter—Parts 1–5 [The Analog Mind],” *IEEE Solid-State Circuits Magazine*.
