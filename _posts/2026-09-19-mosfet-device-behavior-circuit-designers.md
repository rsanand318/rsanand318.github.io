{% comment %} filename: 2026-09-19-mosfet-device-behavior-circuit-designers.md {% endcomment %}

***
layout: post
title: "MOSFET Device Behavior for Circuit Designers"
date: 2026-09-19 18:00:00 -0400
description: "A first-principles walkthrough of MOSFET channel formation, the drain current equation, and the short-channel effects that matter for analog and RF design."
tags: mosfet device-physics analog fundamentals
categories: analog-design device-physics
related_posts: false
toc:
  sidebar: right
thumbnail: assets/img/posts/mosfet-device-behavior-circuit-designers/mosfet-cross-section.png
citation: true
<!-- gisqus_comments: true -->
***

Metal-Oxide-Semiconductor Field-Effect Transistors, or MOSFETs, are used extensively in modern chip designs, because they allow high device density and low power dissipation. As this technology lends itself to digital integrated circuits well, it has become a necessity for analog designers to design high-precision, low-noise, and wideband circuits with MOSFETs rather than bipolar transistors (such as BJTs) and other technologies which have better performance characteristics for analog circuits than MOSFETs — such as higher transconductance for the same bias current, lower noise, etc. Therefore, it is necessary to study the fundamental behavior of MOSFET devices to gain the intuition for using them appropriately in larger circuits.

## MOSFET Channel Formation

{% include figure.liquid path="assets/img/posts/mosfet-device-behavior-circuit-designers/mosfet-cross-section.png" title="Cross section of a typical NMOS device showing source, drain, gate, and body" class="img-fluid rounded z-depth-1" zoomable=true %}

The cross section of a typical NMOS device is shown above. Two heavily n-doped regions, called the Source (S) and the Drain (D), sit within a p-type substrate which is also called the Body (B). Between the source and the drain is the region which has the potential to form a channel. The Gate (G) is a conductor that sits above the channel region, and the gate-source voltage can be used to control the conductance of the channel underneath.

Now suppose you connect the source terminal to GND, and the gate is connected to a source $$V_G$$. As long as $$V_{GS}=V_{G}-V_{S}=0$$, there is no channel formation under the gate, since there are no mobile charges that can move from the source to the drain.

Notice that the source and drain both contain p-n junctions, with the substrate being the p-type and the n-wells being the n-type. Recall that:

- the p-type bulk has *holes* as majority mobile carriers, plus fixed −ve ionized acceptors.
- the n-wells have *electrons* as majority mobile carriers, plus fixed +ve ionized donors.

When a small positive voltage $$V_{GS}$$ is applied, the positive charge on the gate pushes holes away from the channel, leaving behind a region of negatively charged ionized acceptors — the technical term for this is the *depletion region*. This depletion region has fewer mobile holes than before.

As we continue to increase the gate voltage $$V_{GS}$$ further, something more interesting happens. The electric field from the gate becomes strong enough to attract the *minority* electrons in the substrate towards the surface beneath the gate. Eventually a point is reached where the electron concentration is larger than the hole concentration at the surface, and the depletion region completely "inverts" and becomes n-type — creatively, this is called *inversion*.

Thus, when the surface electron concentration becomes comparable to or greater than the hole concentration, we have an inversion region. With sufficiently large $$V_{GS}$$, a continuous sheet of electrons forms between the source and the drain. This is the MOSFET channel.

Notice that the gate voltage can be used as a knob to directly control or tune the electron concentration in the channel. The electrons in the channel come primarily from the heavily doped n-type source/drain regions. The role of the gate is simply to establish an electric field at the surface underneath that makes an electron-rich channel energetically favorable.

Thus the MOSFET goes from OFF → depletion → inversion with increasing $$V_{GS}$$.

## The Threshold Voltage

The gate-source voltage $$V_{GS}$$ required to produce an inversion region is called the *threshold voltage*, $$V_{th}$$. The exact value of this can be calculated by solving for the work functions of the materials involved, but for all practical purposes, it is a circuit-level marker for the onset of strong inversion and is usually provided by the foundry and found through simulation.

Once $$V_{GS}>V_{th}$$, there is a substantial amount of electrons available at the surface, and the amount of inversion charge can be approximated (using $$Q=CV$$) as:

$$
Q_{inv}=-C_{ox}(V_{GS}-V_{th})
$$

It is worth pointing out here that this equation suggests the fundamental operating principle of MOSFETs: the amount of charge (and therefore current) sitting in the channel is directly controlled by the gate voltage. The MOSFET acts like a voltage-controlled current source.

Now connect the drain D to a positive voltage and keep the source S at ground. The channel electrons experience a lateral electric field and drift toward the drain. Thus the current (conventional) flows from drain to source.

## Deriving the Current Equation

To derive the actual channel current equation, we start from the inversion charge after $$V_{DS}$$ has been applied, and remind ourselves that the channel voltage isn't exactly zero everywhere. The channel voltage, $$V(x)$$, starts at approximately 0 at the source and reaches $$V_{DS}$$ at the drain. Therefore, the effective local inversion charge becomes:

$$
Q_{inv}(x)=-C_{ox}(V_{GS}-V_{th}-V(x))
$$

Before we proceed, it is useful to define the overdrive voltage ($$V_{OV}$$) as

$$
V_{OV} = V_{GS}-V_{th}
$$

The drain current is equal to the inversion charge times the carrier velocity in the n-channel, times the width, since everything is expressed as line densities. We know from electromagnetics that

$$
v_n=\mu_n E_x
$$

and

$$
E_x=-\frac{dV}{dx}
$$

Thus we can combine the two equations to find:

$$
I_{DS}=W\, Q_{inv}\, v_n=W\left(-C_{ox}(V_{GS}-V_{th}-V(x))\right)\frac{dV}{dx}
$$

Rearranging this gives:

$$
I_{DS}\,dx=-\mu_{n}C_{ox} W (V_{OV} - V(x))\,dV
$$

We do this integral across the channel, so $$x$$ traverses from 0 to $$L$$ (channel length) and therefore $$V$$ changes from 0 to $$V_{DS}$$. Therefore:

$$
I_{DS}=-\mu_{n}C_{ox} \frac{W}{L}\left(V_{OV} V_{DS} - \dfrac{V_{DS}^2}{2}\right)
$$

And that is the fundamental MOSFET drain current equation.

## Triode/Linear Region

When $$V_{DS}$$ is much smaller than $$V_{OV}$$, the square term in the equation becomes irrelevant:

$$
I_{DS}=-\mu_{n}C_{ox} \frac{W}{L} V_{OV} V_{DS} = R_{DS}\,V_{DS}
$$

Thus the MOSFET looks like a resistor whose resistance is controlled by $$V_{OV}$$. Increasing $$V_{GS}$$ puts more electrons in the channel, thereby reducing the resistance.

## Saturation Region

Now, what happens if we continue increasing $$V_{DS}$$? The quadratic term can no longer be ignored, obviously, but more importantly, notice that the inversion charge at the drain keeps decreasing:

$$
Q_{inv}(x=L)=-C_{ox}(V_{GS}-V_{th}-V_{DS})
$$

Eventually, when the drain-source voltage becomes equal to the overdrive voltage, the local charge at the drain end becomes completely zero. That is, when $$V_{DS}=V_{OV}$$:

$$
Q_{inv}(x=L)=-C_{ox}(V_{GS}-V_{th}-V_{DS})=0
$$

This is the physical origin of *pinch-off*, or the saturation condition. The word "saturation" can be slightly misleading if you don't think about it physically. Pinch-off does *not* mean that the drain current becomes zero at the drain end, but that the inversion region reverts to being a depletion region. The electrons traveling through the channel reach this depleted region and are swept across it by the strong electric field into the drain.

Increasing $$V_{DS}$$ further mainly extends the pinch-off region into the channel rather than substantially increasing the channel current. The channel current therefore becomes less dependent on $$V_{DS}$$ and is now controlled primarily by $$V_{OV}$$.

To summarize, the transistor enters saturation (SAT) when

$$
V_{DS}\geq V_{OV},
$$

and substituting the pinch-off condition into the current equation gives:

$$
I_{DS}=-\frac{1}{2}\mu_{n}C_{ox} \frac{W}{L} V_{OV}^2
$$

This is the famous saturated current equation. The drain current depends on the square of $$V_{OV}$$, making it a "square-law" device. We can intuitively understand the equation above in the following way:

- $$V_{OV}$$ determines how much inversion charge you have.
- $$\frac{W}{L}$$ determines how effectively that charge can conduct between source and drain.

The parameters $$\mu_n$$ and $$C_{ox}$$ are set by the process and are out of the designer's control.

In essence, the large-signal model for the MOSFET device is a voltage-controlled current source, as shown below.

{% include figure.liquid path="assets/img/posts/mosfet-device-behavior-circuit-designers/mosfet-large-signal-model.png" title="Large-signal voltage-controlled current source model of the MOSFET" class="img-fluid rounded z-depth-1" zoomable=true %}

## Transconductance

How much drain current can a given change in $$V_{GS}$$ produce? We find this by differentiating with respect to $$V_{GS}$$ to find the transconductance of the device:

$$
g_{m}= \frac{dI_{DS}}{dV_{GS}} = \mu_{n}C_{ox} \frac{W}{L} V_{OV}.
$$

This is equivalent to

$$
g_{m}=  \frac{2I_D}{V_{OV}}.
$$

The transconductance connects device physics directly to circuit design, and is therefore one of the most useful parameters in analog design. Reducing $$V_{OV}$$ gives you higher $$g_m$$, but increasing $$V_{OV}$$ gives you more voltage headroom and a larger signal range/swing. Since $$V_{OV}$$ itself is a difficult parameter to control in real designs (the threshold voltage is not really well-defined), we design our circuits based on the parameter $$\frac{g_m}{I_D}$$, which is more controllable in simulation. Future posts will explore the practical procedure for the $$\frac{g_m}{I_D}$$ design methodology.

It is good to memorize the entire operational flow as below:

1. Gate electric field pushes holes away.
2. Depletion region forms at the surface.
3. Inversion occurs when $$V_{GS}>V_{th}$$.
4. Channel is formed.
5. $$V_{DS}$$ causes a lateral electric field.
6. Channel electrons drift from source to drain.
7. Channel pinches off when $$V_{DS} > V_{OV}$$.
8. Saturation.

Now, based on the two equations above, we can try to plot the I-V curves for the MOSFET model.

{% include figure.liquid path="assets/img/posts/mosfet-device-behavior-circuit-designers/mosfet-iv-curves.png" title="I-V characteristics of a MOSFET showing triode and saturation regions" class="img-fluid rounded z-depth-1" zoomable=true %}

For small $$V_{DS}$$ the MOSFET begins to act linearly, and eventually flattens out once $$V_{DS}\geq V_{OV}$$. Notice that while the ideal plots show the current saturating completely after $$V_{DS}\geq V_{OV}$$, the actual observed plot does not flatten out completely. This is primarily because of channel length modulation, which we study next.

## Channel Length Modulation

Earlier, in the ideal long-channel model, once the drain end pinches off, we pretended that $$V_{DS}$$ no longer plays a role in affecting the current.

Realistically, increasing $$V_{DS}$$ makes the depletion region around the drain extend farther toward the source (because the pinch-off condition is met before $$x=L$$). The junction becomes more reverse-biased, and the depletion region widens. This depletion region eats into the channel from the drain side, so the effective channel becomes shorter, and a shorter channel produces more current and is less resistive.

The effective length can be defined as

$$
L_{eff}= L - \delta L.
$$

Therefore,

$$
I_{DS}=-\frac{1}{2}\mu_{n}C_{ox} \frac{W}{L- \delta L} V_{OV}^2
$$

Thus, as $$\delta L$$ increases, the effective length decreases and $$I_D$$ increases. For small $$\frac{\delta L}{L}$$:

$$
\frac{1}{L- \delta L}=\frac{1}{L}\left( \frac{1}{1-\frac{\delta L}{L}} \right) \approx\frac{1}{L}\left(1+\frac{\delta L}{L}\right)
$$

The change in channel length $$\delta L$$ is approximately proportional to the additional drain voltage (once linearized), so we can write:

$$
I_D=I_{D,sat}(1+\lambda V_{DS})
$$

where $$\lambda$$ is the channel length modulation factor.

This means the transistor is still in saturation, but $$I_D$$ now has a finite dependence on $$V_{DS}$$, making the output characteristic no longer completely flat, as shown in the I-V curve earlier.

We can find this slope by computing:

$$
g_{ds}= \frac{\delta I_D}{\delta V_{DS}}= \lambda I_{D,sat} \implies r_o=\frac{1}{g_{ds}}=\frac{1}{\lambda I_D}
$$

In short-channel devices (nanometer nodes), the fraction $$\frac{\delta L}{L}$$ becomes larger for the same $$\delta L$$, since $$L$$ is smaller — this means CLM needs to be carefully considered when designing circuits. For instance, the equation above shows us that $$r_o$$ of the device will no longer be infinite (the ideal case), but will be limited by drain current and CLM.

In practice, whenever you care about voltage gain, current source accuracy, current mirrors, or output impedance, $$r_o$$ matters. Channel length modulation directly limits the intrinsic gain of your MOSFET device, and this is more pronounced in modern processes where the channel length is under 100 nm.

Channel length modulation is only the beginning of short-channel effects. The fundamental assumption of the long-channel MOSFET was that the gate alone controls the channel current and electric field. This breaks down when $$L$$ shrinks, because the depletion regions of the reverse-biased p-n junctions at the source and drain now occupy a larger portion of the channel length itself, meaning the electric fields from the drain and source also influence the channel's charge movement.

## Short-Channel Effects: Drain-Induced Barrier Lowering (DIBL)

{% include figure.liquid path="assets/img/posts/mosfet-device-behavior-circuit-designers/dibl-diagram.png" title="Drain-induced barrier lowering in a short-channel MOSFET" class="img-fluid rounded z-depth-1" zoomable=true %}

Drain-Induced Barrier Lowering (DIBL) is, at its core, a loss of gate control in a short-channel MOSFET.

Start from a long-channel NMOS. With $$V_{GS}<V_{th}$$, the gate creates a potential barrier near the source that prevents electrons in the source from entering the channel. The gate essentially controls this barrier.

As $$L$$ becomes small, the drain gets physically close to the source, so when $$V_{DS}$$ increases, the drain's electric field and depletion region extend farther into the channel than they used to. Some of the drain's electrostatic influence therefore reaches the source-side barrier.

So, for the same $$V_{GS}$$, more mobile carriers are able to pass through the barrier and flow into the channel, effectively reducing the threshold voltage. The drain voltage basically helps turn the transistor ON, and electrons can be injected from the source at a lower $$V_{GS}$$. Thus, $$V_{th}$$ decreases as $$V_{DS}$$ increases.

Recall that in subthreshold, current increases exponentially with $$V_{OV}$$. This is why DIBL is a challenge for low-power designs, because even when the transistor is OFF, a high drain voltage can substantially increase leakage current.

## Short-Channel Effects: Velocity Saturation

Our original derivation of the current equation assumed $$v_n=\mu_{n}E$$, which is approximately true when the field is small. At high electric fields, however, electrons scatter more strongly with the lattice in the substrate, and their velocity begins to saturate instead of continuing to increase linearly with $$E$$.

Thus we can rederive the current equation with:

$$
I_{D} = W \cdot Q_{inv}\cdot v_{sat} =W \cdot C_{ox} \cdot v_{sat} \cdot V_{OV}
$$

The current no longer follows a square law with $$V_{OV}$$, and looks more linear in overdrive. At sufficiently short channel lengths, carrier velocity saturation makes the $$I_D$$–$$V_{GS}$$ relationship linear instead of quadratic.

The transconductance also saturates:

$$
g_{m}= \frac{dI_{DS}}{dV_{GS}}=W\,C_{ox}\, v_{sat}
$$

A typical transfer characteristic shows three regions:

- **OFF:** no current.
- **Subthreshold:** current is dominated by diffusion leakage and increases exponentially with $$V_{OV}$$.
- **Moderate inversion:** the device obeys the square law approximately.
- **Strong inversion:** $$I_D$$ starts to increase more linearly with $$V_{OV}$$ due to velocity saturation.

## Summary of Operating Regions

It is useful to think of the MOSFET both from the perspective of I-V transitions (OFF/linear/saturation) and from the standpoint of device physics (strong/weak inversion, etc.). The following table (copy-pasted from ChatGPT) summarizes the previous discussion quite well.

| Operating region   | Definition / condition                                  | Current / operation characteristic                                                                       | Pros                                                                       | Cons                                                                                                              | Typical applications / avoid                                                                                                               |
| ------------------ | ------------------------------------------------------- | -------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| Deep subthreshold  | VGS well below VTH; inversion charge is extremely small | Diffusion-dominated; ID changes exponentially with VOV; very small current                               | Extremely high gm/ID; very low static power                                | Very low speed; leakage/variation become important; poor drive capability                                         | Use: ultra-low-power sensors, always-on circuits, biomedical, energy harvesting. Avoid: high-speed signal paths                            |
| Weak inversion     | VGS≲VTH; weak inversion layer exists                    | Mainly diffusion; ID remains approximately exponential in VGS; gm/ID≈1/(nUT)                              | Excellent current efficiency; useful gm at very low current                | Limited bandwidth/current drive; sensitive to VTH, temperature, mismatch                                          | Use: low-power analog, sensor interfaces, bias circuits. Avoid: demanding high-speed/RF circuits                                           |
| Moderate inversion | Transition between weak and strong inversion            | Drift and diffusion both contribute; square-law and pure exponential models are both inaccurate          | Good compromise between gm/ID, speed, power, noise, and linearity          | Requires more accurate models; operating point can be less intuitive                                              | Use: low-power OTAs, amplifiers, ADCs, precision analog. Often a useful design region                                                      |
| Strong inversion   | VGS>VTH with substantial inversion charge               | Drift-dominated; high channel charge and current capability                                              | High gm, high current drive, better speed; familiar circuit behavior       | Lower gm/ID; higher power; short-channel effects become important                                                 | Use: high-speed analog, RF, ADC front ends, digital logic. Avoid: when extreme energy efficiency is the primary constraint                 |
| OFF / cutoff       | Ideally VGS<VTH, with negligible ID                     | In practice, subthreshold and other leakage currents remain                                              | Very low static current; useful for switching                              | Leakage increases in nanometer CMOS; DIBL can worsen OFF current                                                  | Use: digital logic and power gating. Important: "OFF" does not mean ID=0                                                                   |
| Triode / linear    | Strong inversion with approximately VDS<VGS−VTH         | Continuous channel from source to drain; approximately voltage-controlled resistor: Ron≈1/[μCox(W/L)VOV] | Useful switching behavior; low Ron; can implement variable resistance      | Ron depends on signal and VTH, causing distortion                                                                 | Use: transmission gates, analog switches, sample-and-hold, switched-capacitor circuits. Avoid: precision current-source operation           |
| Saturation         | Classical condition VDS≥VGS−VTH in strong inversion      | Drain-end channel charge approaches zero; current becomes primarily controlled by VGS                    | Enables voltage-to-current conversion; useful for gain and current sources | ro is finite; in nanoscale CMOS, velocity saturation, CLM, DIBL and mobility degradation strongly modify behavior | Use: amplifiers, current sources, differential pairs, active loads. Important: nanometer "saturation" is not the ideal textbook saturation |

## References

 Notes from ECE 483 Analog IC Design (Prof. Pavan Kumar Hanumolu) @ UIUC.

 Gray, P. R., Hurst, P. J., Lewis, S. H., & Meyer, R. G. (2009). *Analysis and Design of Analog Integrated Circuits* (5th ed.). John Wiley & Sons.

{% comment %}

IMAGE MANIFEST — copy these files into the repository before deploying

Post image directory:
assets/img/posts/mosfet-device-behavior-circuit-designers/

Required files:
1. Source/role: NMOS cross-section diagram used in "MOSFET Channel Formation" (original note filename: Screenshot 2026-09-19 at 12.54.17 PM.png)
   Exact filename: mosfet-cross-section.png
   Exact repository path: assets/img/posts/mosfet-device-behavior-circuit-designers/mosfet-cross-section.png
   Used in post as: {% include figure.liquid path="assets/img/posts/mosfet-device-behavior-circuit-designers/mosfet-cross-section.png" title="Cross section of a typical NMOS device showing source, drain, gate, and body" class="img-fluid rounded z-depth-1" zoomable=true %}
   Status: missing — author must provide (referenced via Obsidian wiki-link in source notes; image file not supplied in this session)

2. Source/role: Large-signal VCCS model diagram used after the saturation-current derivation (original note filename: Pasted image 20260919154529.png)
   Exact filename: mosfet-large-signal-model.png
   Exact repository path: assets/img/posts/mosfet-device-behavior-circuit-designers/mosfet-large-signal-model.png
   Used in post as: {% include figure.liquid path="assets/img/posts/mosfet-device-behavior-circuit-designers/mosfet-large-signal-model.png" title="Large-signal voltage-controlled current source model of the MOSFET" class="img-fluid rounded z-depth-1" zoomable=true %}
   Status: missing — author must provide (referenced via Obsidian wiki-link in source notes; image file not supplied in this session)

3. Source/role: I-V characteristic curves (triode/saturation) used in "Transconductance" section (original note filename: Pasted image 20260919154104.png)
   Exact filename: mosfet-iv-curves.png
   Exact repository path: assets/img/posts/mosfet-device-behavior-circuit-designers/mosfet-iv-curves.png
   Used in post as: {% include figure.liquid path="assets/img/posts/mosfet-device-behavior-circuit-designers/mosfet-iv-curves.png" title="I-V characteristics of a MOSFET showing triode and saturation regions" class="img-fluid rounded z-depth-1" zoomable=true %}
   Status: missing — author must provide (referenced via Obsidian wiki-link in source notes; image file not supplied in this session)

4. Source/role: DIBL band/depletion diagram used in "Short-Channel Effects: DIBL" section (original note filename: Pasted image 20260919160844.png)
   Exact filename: dibl-diagram.png
   Exact repository path: assets/img/posts/mosfet-device-behavior-circuit-designers/dibl-diagram.png
   Used in post as: {% include figure.liquid path="assets/img/posts/mosfet-device-behavior-circuit-designers/dibl-diagram.png" title="Drain-induced barrier lowering in a short-channel MOSFET" class="img-fluid rounded z-depth-1" zoomable=true %}
   Status: missing — author must provide (referenced via Obsidian wiki-link in source notes; image file not supplied in this session)

5. Source/role: Transfer-characteristic plot (OFF/subthreshold/moderate/strong inversion) referenced in "Short-Channel Effects: Velocity Saturation" — not included as an include; flagged instead with an Author Note placeholder in the article text.
   Exact filename: transfer-characteristic-regions.png
   Exact repository path: assets/img/posts/mosfet-device-behavior-circuit-designers/transfer-characteristic-regions.png
   Used in post as: not currently included (see Author Note placeholder in "Short-Channel Effects: Velocity Saturation")
   Status: missing — author must provide (source notes linked to a third-party image hosted externally; cannot be copied into the repository as-is)

Thumbnail, if used:
Exact repository path: assets/img/posts/mosfet-device-behavior-circuit-designers/mosfet-cross-section.png

Deployment checks:
- Confirm every listed file exists at its exact case-sensitive path.
- Confirm every image include path in the article matches the manifest exactly.
- Confirm the thumbnail path, if present, matches a real file.
- Commit both the Markdown post and all listed image files before deployment.

{% endcomment %}