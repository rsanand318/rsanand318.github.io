---
layout: post
title: "LDO Design Part I: Fundamentals of Linear Regulators"
date: 2026-08-24 12:30:00 -0400
description: "A first-principles motivation for the LDO"
tags: ldo-design analog power-management circuits
categories: circuit-design analog
related_posts: true
toc:
  sidebar: right
citation: true
disqus_comments: false
---

## 1. Why do we need an LDO?

A typical power management system looks something like this:

{% include figure.liquid path="assets/img/posts/ldo-design-part-1-fundamentals/power-management-block-diagram.png" title="Power management chain: battery, DC-DC converter, multiple LDOs, and individual circuit blocks" class="img-fluid rounded z-depth-1" zoomable=true %}

The system takes the battery voltage and generates power for different circuits within the IC. These could be digital, analog, RF, and other blocks, and their power requirements are usually different. For example:

- Digital block → 1.0 V
- Analog block → 1.2 V
- RF block → 1.8 V

The problem is that the battery itself does not provide a precise, constant voltage. Take an iPhone as an example: a lithium-ion battery can supply around 4.4 V when fully charged, but its voltage can fall toward 3.6 V as it discharges. So we have a variable input voltage, while our circuits often need a precise and relatively quiet supply voltage.

This becomes even more important in modern processes such as BiCMOS, where different device types can coexist on the same die — each with its own threshold voltage, breakdown voltage, and current-handling capability. Different device types typically need different, precisely-set supply rails to operate correctly and reliably, so a single unregulated battery rail cannot directly serve all of them.

Obviously, we do not want a separate battery for every circuit block. We need a circuit that can take the battery voltage and generate stable supply voltages for the different blocks.

## 2. First attempt: the resistor divider

Let's first simplify the problem. Assume the battery provides a constant

$$
V_{BAT} = 2.4\text{ V}
$$

and we have a circuit block that needs

$$
V_{OUT} = 1.2\text{ V}.
$$

The simplest solution is a resistor divider:

{% include figure.liquid path="assets/img/posts/ldo-design-part-1-fundamentals/resistor-divider-circuit.png" title="Resistor divider with V_IN, V_OUT, R_IN, and R_L labeled" class="img-fluid rounded z-depth-1" zoomable=true %}

Assuming an unloaded (or negligibly-loaded) divider with two resistors $$R_1$$ and $$R_2$$:

$$
V_{OUT} = V_{BAT}\,\frac{R_2}{R_1+R_2}.
$$

If we pick $$R_1 = R_2 = R$$:

$$
V_{OUT} = 2.4\,\frac{R}{R+R} = 1.2\text{ V}.
$$

This is a very simple solution, and in many situations simplicity is exactly what we want. The problem is that this equation only holds as written when nothing else loads the output node — once we connect a real circuit block, the divider stops behaving this way.

### Problem 1: Load regulation

The divider equation above assumes the load draws negligible current, i.e., that it does not change the effective resistance at the output node. In reality, the load resistance $$R_L$$ sits in parallel with $$R_2$$ (or, depending on the topology in the note above, _is_ the bottom leg), so the actual divide ratio becomes a function of $$R_2 \parallel R_L$$ rather than a fixed value.

If the circuit's current demand increases (i.e., $$R_L$$ decreases), $$R_2 \parallel R_L$$ decreases, and the output voltage sags below the intended 1.2 V. The fixed resistor ratio has no way to compensate for this, because it has no feedback path that senses $$V_{OUT}$$ and adjusts anything in response — it's a purely passive, open-loop network. So we do not get good load regulation.

### Problem 2: Power efficiency

There is also a fundamental efficiency problem. A resistor divider continuously carries current from the battery, so power is dissipated in the resistors:

$$
P = I^2 R.
$$

Some of the power coming from the battery is therefore turned into heat instead of reaching the load. Ideally we want most of the input power to make it to the circuit — this is what motivates looking at elements that ideally do not dissipate power, such as capacitors and inductors.

## 3. Using capacitors and inductors: the DC-DC converter

Capacitors and inductors are useful as filters. A key observation: if we take a periodic waveform and pass it through a low-pass filter, the filter removes the high-frequency components and leaves us with the average (DC) component.

So instead of directly dividing 2.4 V with resistors, what if we convert the 2.4 V into a periodic waveform whose _average_ is 1.2 V, and then filter that waveform? This is the basic idea behind a switched DC-DC converter.

Imagine switching the battery node between $$V_{BAT}$$ and ground at the switch node, then low-pass filtering with an LC network. If the switching waveform has duty cycle $$D$$, the average (DC) value at the switch node is

$$
V_{OUT} = D\,V_{BAT},
$$

which the LC filter then passes through while removing the switching-frequency content. For

$$
D = 0.5, \qquad V_{BAT} = 2.4\text{ V},
$$

we get

$$
V_{OUT} = 0.5 \times 2.4 = 1.2\text{ V}.
$$

The two switches are driven by complementary clock signals, $$CLK$$ and $$\overline{CLK}$$, so that one connects the switch node to $$V_{BAT}$$ while the other connects it to ground, and vice versa. The switch-node waveform is therefore a square wave, not a clean DC voltage — the LC filter's job is to remove most of the switching-frequency components and pass through approximately the DC/average value. This is the basic operating principle of a buck converter.

## 4. Why is this much better than a resistor divider?

The important advantage is efficiency. With the resistor divider, we are deliberately dissipating power in resistors to drop the excess voltage. With an ideal switched converter, the switches, capacitors, and inductors can theoretically transfer energy with very little loss, because ideal switches and reactive elements do not dissipate power the way a resistor does.

In practice there are still losses:

- Switch $$R_{ON}$$
- Inductor resistance
- Capacitor ESR
- Switching losses
- Gate-driver power
- Other parasitic losses

But the efficiency can still be much higher than simply burning off the excess voltage in a resistor. Another useful property is that the output voltage can be controlled digitally, simply by changing the duty cycle $$D$$.

So now we have an efficient way to convert the battery voltage down to a lower voltage.

## 5. But there is a problem: ripple

The output of the switched converter is not perfectly DC. The switch-node waveform contains the fundamental switching frequency and its harmonics. The LC filter suppresses these high-frequency components, but — being a real, finite-order filter — it cannot remove all of them. So the output looks more like

$$
V_{OUT}(t) = V_{DC} + v_{ripple}(t),
$$

where $$V_{DC}$$ is the desired average voltage and $$v_{ripple}(t)$$ is the residual switching ripple that leaked through the filter. For example, we might want $$V_{OUT} = 1.2\text{ V}$$ but actually get something like $$1.2\text{ V} + \text{small periodic ripple}$$.

This may be acceptable for some loads, but it can be a real problem for sensitive analog, RF, and mixed-signal circuits, which are exactly the blocks that need a quiet supply in the first place.

## 6. Why can't we simply make the LC filter much better?

In principle, we could make the low-pass filter much more aggressive so that almost all of the switching harmonics are removed. The problem is the required component values and physical area.

We want the filter's cutoff frequency well below the switching frequency,

$$
f_{LPF} \ll f_{SW},
$$

so the filter strongly attenuates the switching components. But achieving a low $$f_{LPF}$$ relative to a given $$f_{SW}$$ generally means a larger inductor and/or capacitor. Large inductors are particularly painful on-chip — they typically require spiral inductors, which consume significant die area and also carry non-negligible series resistance (adding to the loss mechanisms in Section 4).

So there is a practical tradeoff:

- Lower $$f_{SW}$$ → easier switching, but the filter needs larger passives to keep $$f_{LPF} \ll f_{SW}$$.
- Higher $$f_{SW}$$ → the filter can use smaller passives for the same relative attenuation, but the switches themselves become harder to drive (see Section 7) and switching losses rise.

We therefore cannot simply make the filter arbitrarily sharp without cost.

## 7. Why not just increase the switching frequency?

At first this looks like the obvious fix: raise $$f_{SW}$$ (say, toward 1 GHz) so the switching harmonics sit far from DC, and a much smaller filter can suppress them. But now the switches themselves become difficult to operate well.

At high switching frequencies:

- The switches must turn on and off very quickly.
- Their gate capacitances must be charged and discharged every cycle.
- Larger switches (needed for low $$R_{ON}$$) have larger gate capacitance, which requires more gate-drive current to switch at the same speed.
- Switching losses increase with frequency, since energy $$\tfrac{1}{2}CV^2$$ is dissipated in the switch/driver path once per cycle.
- The gate driver itself consumes more power as a result.

There is also a direct tradeoff in switch sizing: a wider switch lowers $$R_{ON}$$ (good — less conduction loss and voltage drop), but also increases the switch's parasitic gate capacitance $$C_{gate}$$, which must be driven every cycle (bad — more dynamic/driver power, especially at high $$f_{SW}$$). So we end up trading:

$$
R_{ON} \quad \leftrightarrow \quad C_{gate} \quad \leftrightarrow \quad \text{driver power}.
$$

This is one of the basic practical tradeoffs in a high-frequency switched converter: pushing $$f_{SW}$$ up to shrink the passive filter components pushes switching and driver losses up at the same time. So in practice, both the switching frequency and the passive component sizes are chosen at a reasonable operating point rather than pushed to an extreme in either direction.

## 8. The practical DC-DC output

Because of these practical limitations, a real DC-DC converter produces something like

$$
V_{DC} + v_{ripple}(t),
$$

rather than a perfectly clean DC voltage. The average value can be very well controlled (it's set by $$D$$), but some switching ripple remains no matter how the frequency/filter tradeoff is chosen.

For a power-management system, this is often acceptable at the first stage, because the DC-DC converter's main job is power efficiency, not precision. However, sensitive circuit blocks need a much cleaner supply than this. This is where the LDO comes in.

## 9. What is the LDO actually doing?

The basic idea is:

**DC-DC converter → LDO → sensitive circuit**

The DC-DC converter gives us an efficient voltage, but with some ripple. The LDO takes that voltage and generates a cleaner, more tightly regulated output. For example:

$$
V_{DC\text{-}DC} \approx 1.3\text{ V with ripple}
$$

and the LDO could generate

$$
V_{LDO} = 1.2\text{ V}
$$

with much smaller ripple. So the LDO provides two important things:

### Regulation

The output should stay close to the desired voltage even when:

- The input voltage changes.
- The load current changes.
- There are process and temperature variations.

### Ripple/noise suppression

The LDO should suppress the ripple coming from the DC-DC converter. This is particularly important for sensitive analog, RF, and mixed-signal circuits.

## 10. Why not use another DC-DC converter?

At first it seems like we could simply cascade another switched DC-DC converter after the first one, for another filtering/regulation stage. The problem is area and complexity: on-chip DC-DC converters require relatively large inductors and capacitors compared with normal CMOS circuitry (Section 6), so we do not want to keep adding switched converters everywhere just to clean up one more stage of ripple.

Instead, we want a much simpler circuit for the final regulation and filtering stage. This brings us back toward the spirit of the resistor divider — a relatively simple circuit that sets the output voltage — but now we need something better than a _passive_ resistor divider, because we saw in Section 2 that a passive divider has no way to correct for load or input variation. We need a circuit that can **actively regulate** the output. That is essentially what an LDO does.

## 11. The overall picture

The power-management chain can be understood as two stages with different jobs:

$$
V_{BAT} \rightarrow \text{DC-DC Converter} \rightarrow \text{LDO} \rightarrow \text{Circuit}
$$

**DC-DC converter** — main goal: **power efficiency**. It efficiently converts the battery voltage to something closer to the required supply voltage, using switching and inductors/capacitors, so some switching ripple remains (Sections 3–8).

**LDO** — main goals: **regulation + ripple/noise suppression**. It takes the somewhat noisy DC-DC output and produces a much cleaner supply for the circuit (Section 9).

So the basic philosophy is:

> Let the DC-DC converter handle the big voltage conversion efficiently, and let the LDO clean up and precisely regulate the voltage before it reaches the sensitive circuit.

This is why the original figure shows one DC-DC converter feeding several LDOs. The DC-DC converter does the bulk power conversion, while each LDO generates a supply appropriate for its particular circuit block:

$$
V_{DC\text{-}DC} \rightarrow
\begin{cases}
LDO_1 \rightarrow V_{LDO1} \rightarrow \text{Circuit}_1 \\
LDO_2 \rightarrow V_{LDO2} \rightarrow \text{Circuit}_2 \\
\vdots \\
LDO_N \rightarrow V_{LDON} \rightarrow \text{Circuit}_N
\end{cases}
$$

The key question for the rest of LDO design is therefore:

**How do we build this active voltage regulator so that it gives us good regulation and ripple rejection while consuming as little area and power as possible?**

In this part, we motivated the use of the LDO in the context of providing a reliable power supply to different circuit blocks in the IC. In the next parts of this series, we will get into the details of exactly what is required from the LDO, and how we can build up a simple LDO from the design specifications.

{% comment %}

IMAGE MANIFEST — copy these files into the repository before deploying

Post image directory:
assets/img/posts/ldo-design-part-1-fundamentals/

Required files:

1. Source/role: Section 1 opening diagram showing the overall power-management chain (Battery → DC-DC Converter → LDOs → Individual Circuit Blocks)
   Exact filename: power-management-block-diagram.png
   Exact repository path: assets/img/posts/ldo-design-part-1-fundamentals/power-management-block-diagram.png
   Used in post as: {% include figure.liquid path="assets/img/posts/ldo-design-part-1-fundamentals/power-management-block-diagram.png" title="Power management chain: battery, DC-DC converter, multiple LDOs, and individual circuit blocks" class="img-fluid rounded z-depth-1" zoomable=true %}
   Status: missing — author must provide

2. Source/role: Section 2 resistor divider schematic with V_IN, V_OUT, R_IN, and R_L labeled
   Exact filename: resistor-divider-circuit.png
   Exact repository path: assets/img/posts/ldo-design-part-1-fundamentals/resistor-divider-circuit.png
   Used in post as: {% include figure.liquid path="assets/img/posts/ldo-design-part-1-fundamentals/resistor-divider-circuit.png" title="Resistor divider with V_IN, V_OUT, R_IN, and R_L labeled" class="img-fluid rounded z-depth-1" zoomable=true %}
   Status: missing — author must provide

Thumbnail, if used:
None — no suitable thumbnail image was supplied in the source notes, so the thumbnail field has been omitted from the front matter.

Deployment checks:

- Confirm every listed file exists at its exact case-sensitive path.
- Confirm every image include path in the article matches the manifest exactly.
- Confirm the thumbnail path, if present, matches a real file.
- Commit both the Markdown post and all listed image files before deployment.

{% endcomment %}
