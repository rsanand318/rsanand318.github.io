---
layout: post
title: "Why Analog Designers Intentionally Throw Away Gain"
date: 2026-08-07
categories:
  - Analog IC Design
tags:
  - Feedback
  - Amplifiers
  - Analog Design
math: true
toc:
  sidebar: right
---

If you look at almost any op-amp datasheet, you'll notice something that seems mildly contradictory. Modern amplifiers are designed to have enormous open-loop gain — often well above 100 dB. Yet in nearly every practical circuit built around that amplifier, we immediately surround it with resistors whose entire job is to *reduce* that gain.

Why spend so much design effort building an amplifier with tremendous gain, only to throw most of it away deliberately?

The short answer:

> **Negative feedback lets us trade excess gain for predictability.**

And in most analog design contexts, predictability turns out to be worth far more than raw gain.

---

## The Real Problem

Suppose you've designed an amplifier with an open-loop gain of

$$
a = 10^5.
$$

That looks great on paper. Unfortunately, this gain isn't actually a fixed number — it drifts with

- temperature,
- supply voltage,
- process variation,
- transistor mismatch,
- aging,
- and a long list of second-order effects.

In other words, the gain you so carefully designed for is one of the *least* reliable quantities in the entire circuit. As designers, we don't want an amplifier whose gain shifts every time the die temperature changes or the process corner moves. We want control over the transfer characteristic, even if that means giving up some raw performance to get it.

---

## Enter Negative Feedback

The basic feedback configuration is conceptually simple: the amplifier produces an output, and a fraction of that output is fed back to the input.

![Basic feedback amplifier block diagram](assets/img/posts/feedback/basic_feedback.png)

Instead of amplifying the input signal directly, the amplifier now amplifies the **error** between the desired input and the fed-back version of the output. Mathematically,

$$
S_\varepsilon = S_i - fS_o
$$

where

- $S_i$ is the input signal,
- $S_o$ is the output signal,
- $f$ is the feedback factor,
- $S_\varepsilon$ is the error signal.

The amplifier itself simply does what amplifiers do:

$$
S_o = aS_\varepsilon.
$$

Combining these two relations gives

$$
S_o = a(S_i - fS_o).
$$

Rearranging,

$$
\boxed{
\frac{S_o}{S_i} = \frac{a}{1+af}
}
$$

This is arguably the single most important equation in feedback theory. The quantity

$$
A_{CL} = \frac{a}{1+af}
$$

is the **closed-loop gain** — the gain of the amplifier once the feedback loop is "closed" around it.

---

## What Happens When Loop Gain Is Large

Now consider the case where the *loop gain* $af$ is much greater than one:

$$
af \gg 1.
$$

Then the closed-loop gain simplifies to

$$
A_{CL} \approx \frac{1}{f}.
$$

Notice what disappeared from this expression: the amplifier's own gain, $a$. That's precisely the quantity we said was sensitive to process, temperature, and aging. Once $af \gg 1$, the closed-loop gain no longer depends on it — the gain is set almost entirely by the feedback network instead.

This is useful because the feedback network is usually built from passive components (resistors, capacitors, or carefully sized transistor ratios) that can be matched and controlled far more precisely than transistor transconductance or intrinsic gain. Instead of trusting the amplifier's imperfect, variation-prone gain, we let geometry and component ratios set the closed-loop behavior.

---

## So Why Give Up Gain?

Negative feedback intentionally reduces gain — from $a$ down to roughly $1/f$. On its face, that looks like a loss. In exchange, though, we get several properties that are usually worth much more than the gain we gave up:

- predictable, well-defined closed-loop gain,
- reduced sensitivity to process and temperature variation,
- control over input and output impedance,
- increased bandwidth,
- improved linearity,
- reduced sensitivity to device aging.

In most analog design situations, these benefits outweigh the value of maximizing raw gain. If more gain is needed after applying feedback, the usual fix is simply to add another amplifier stage — gain is relatively cheap to generate. Predictability is not.

---

## An Intuitive Way to Think About It

Think about steering a car. If you drift slightly toward one edge of the lane, you notice the error and make a small correction. You're not trying to maximize your steering angle — you're continuously trying to minimize the error between where you are and where you want to be.

Negative feedback works the same way. The amplifier isn't trying to maximize its output; it's trying to drive the error signal $S_\varepsilon$ toward zero. The larger the open-loop gain $a$, the more aggressively the loop suppresses this error — which is exactly why large open-loop gain is still valuable, even though it doesn't show up directly in $A_{CL}$.

---

## Key Takeaways

Negative feedback is one of the foundational ideas in analog circuit design. Rather than chasing the highest possible open-loop gain, feedback lets us convert an amplifier with unpredictable, variation-sensitive gain into a system whose closed-loop behavior is set by passive components we can control precisely.

The result

$$
A_{CL} \approx \frac{1}{f} \qquad (af \gg 1)
$$

shows that once sufficient loop gain is available, the amplifier's closed-loop behavior is governed far more by the feedback network than by the transistor-level gain itself.

Giving something up often feels like a compromise. Negative feedback is a case where deliberately sacrificing one performance metric — raw gain — is exactly what makes it possible to improve nearly every other metric that matters.

---

## Further Reading

- Gray & Meyer — *Analysis and Design of Analog Integrated Circuits*
- Razavi — *Design of Analog CMOS Integrated Circuits*
