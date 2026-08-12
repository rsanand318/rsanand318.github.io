---
layout: post
title: "Feedback II: Distortion and Feedback in Analog Circuits"
date: 2026-08-12 17:07:00 +0530
description: "How negative feedback desensitizes closed-loop gain to the signal-dependent nonlinearity of the open-loop amplifier, and why this suppresses harmonic and intermodulation distortion."
tags: feedback distortion analog-design fundamentals
categories: analog-design feedback-series
related_posts: false
citation: true
disqus_comments: true
toc:
  sidebar: right
---

In the previous post of this series we derived the basic negative-feedback equation and looked at why feedback is used in the first place. That derivation quietly made an assumption worth revisiting: it treated the open-loop gain of the amplifier as a fixed number. In this post we drop that assumption and ask what happens when the open-loop gain is not constant — which, in any real transistor circuit, it never quite is. The answer turns out to be one of the cleanest illustrations of why negative feedback is such a powerful tool in analog design: it directly explains how feedback fights distortion.

## Where Nonlinearity Comes From

Recall the closed-loop gain we derived earlier, where $$S_o$$ and $$S_i$$ are the output and input signals, $$a$$ is the open-loop gain, and $$f$$ is the feedback factor:

\begin{equation}
\label{eq:closed-loop-gain}
\frac{S_o}{S_i} = A = \frac{a}{1+af}
\end{equation}

Equation \eqref{eq:closed-loop-gain} is exact, but it is only useful if we know what $$a$$ is — and in that derivation we quietly assumed $$a$$ was some fixed constant, independent of signal level.

That assumption breaks down as soon as you look inside the amplifier. A MOSFET’s $$I$$-$$V$$ characteristic is a square-law (or worse, in short-channel devices) relationship, and a BJT’s collector current depends exponentially on $$V_{BE}$$. Neither relationship is a straight line. So the small-signal gain you get by differentiating these characteristics — which is what $$a$$ really is — depends on where along the curve you are biased, and on how far the input signal swings around that bias point. In other words, $$a$$ is not a single number; it is a function of instantaneous signal level, of bias point, and of output swing.

{% include figure.liquid 
   path="assets/img/posts/feedback-distortion-in-analog-circuits/open-loop-transfer-characteristic.png" 
   title="Non-ideal open-loop amplifier transfer characteristic" 
   caption="Non-ideal open-loop amplifier transfer characteristic with regions of slope $a_1$, $a_2$, and $a_3=0$." 
   zoomable=true 
   class="img-fluid rounded z-depth-1" 
%}

The figure captures this directly. The open-loop transfer characteristic isn’t a single straight line through the origin; it has different slopes in different regions. Near one operating point the incremental gain is $$a_1$$; move to a different signal level and the slope becomes $$a_2$$; push the amplifier far enough and the slope eventually collapses to $$a_3 = 0$$ as the device leaves its useful region of operation entirely.

This is exactly what we mean by **distortion**: the output is no longer a faithfully scaled copy of the input, because the "scale factor" itself is changing with the input. A sinusoid fed through a slope that keeps changing does not come out as a clean sinusoid — it comes out with harmonics and, if there are two tones present, intermodulation products. Practically, this nonlinearity is what limits the useful dynamic range of an amplifier: push the signal too far, and the output stops tracking it faithfully. If we intend to tape out something usable, we need a way to make the gain far less sensitive to where the signal happens to sit on that curve.

## How Feedback Desensitizes the Gain

This is precisely the problem negative feedback is good at solving — not by removing the nonlinearity inside the device, but by making the *closed-loop* gain far less sensitive to whatever the open-loop gain happens to be doing.

Suppose the open-loop amplifier has two different operating regions, with gains $$a_1$$ and $$a_2$$ as before. Feedback maps each of these open-loop gains to its own closed-loop gain through the same relation as \eqref{eq:closed-loop-gain}:

$$
A_1 = \frac{a_1}{1+a_1 f}, \qquad A_2 = \frac{a_2}{1+a_2 f}
$$

Now define the loop gain $T = af$. If $T \gg 1$ in both regions, each expression collapses to the same limit:

$$
A_1 \approx \frac{1}{f}, \qquad A_2 \approx \frac{1}{f}
$$

Notice what just happened: $$a_1$$ and $$a_2$$ can be wildly different numbers, but $$A_1$$ and $$A_2$$ converge to nearly the same value, $$1/f$$. The closed-loop gain has become almost independent of which open-loop region the amplifier is operating in. This is the entire mechanism in one line — feedback has not touched the device physics that made $$a$$ nonlinear in the first place. It has simply arranged things so that, as long as loop gain is high, the *overall* gain is set by the feedback network rather than by the amplifier’s own nonlinear characteristic. The feedback network — typically a resistive divider or similarly well-controlled passive structure — is doing the job the transistor used to do, and it does that job far more linearly.

{% include figure.liquid 
   path="assets/img/posts/feedback-distortion-in-analog-circuits/closed-loop-transfer-characteristic.png" 
   title="Closed-loop transfer characteristic" 
   caption="Closed-loop transfer characteristic after feedback is applied, compared to the open-loop case." 
   zoomable=true 
   class="img-fluid rounded z-depth-1" 
%}

The transfer-characteristic picture makes the same point visually. In the open-loop curve, the slope visibly changes from $$a_1$$ to $$a_2$$ as the signal moves. Once feedback is added and the loop gain is large enough, those same swings in internal slope get compressed into much smaller changes in the closed-loop slope. The closed-loop transfer characteristic looks far closer to a straight line than the open-loop one did — even though the underlying amplifier is exactly as nonlinear as before.

One qualification is worth stating explicitly, because it’s easy to over-read the result above. The approximation $$A_i \approx 1/f$$ is only as good as the condition $$T_i = a_i f \gg 1$$ actually being satisfied in every region the signal visits. If the amplifier’s gain drops sharply enough in some region — near clipping, for instance, where $$a_3 \to 0$$ — loop gain in that region can fall to where $$T \not\gg 1$$ any longer, and the closed-loop gain will noticeably depart from $$1/f$$. So the correct statement isn’t "feedback linearizes the amplifier" — it’s "feedback suppresses the effect of nonlinearity exactly to the extent that sufficient loop gain is maintained across the signal range in question." Once loop gain runs out — near the edges of the swing — so does the benefit.

## The Distortion Connection

This is exactly why the technique shows up so often in audio power amplifiers. The core amplifier inside one of these is frequently quite nonlinear on its own — often deliberately simple, for efficiency or headroom reasons — but wrapping it in a negative-feedback loop with sufficient loop gain $T$ makes the *overall* transfer characteristic close to linear.

More precisely, the effect of the amplifier’s internal distortion on the output is attenuated by roughly the same factor that suppresses any other open-loop error term in a feedback system:

$$
\frac{1}{1+T}
$$

This is a direct extension of the desensitization result above: whatever nonlinear distortion component the open-loop amplifier would have produced on its own gets divided down by $$1+T$$ once the loop is closed. A sinusoidal input therefore comes out closer to a pure sinusoid, and two-tone intermodulation products shrink by the same factor. The chain of reasoning that gets us here is worth stating explicitly, since each step depends on the one before it:

1. Device nonlinearities (square-law MOSFET behavior, exponential BJT behavior) make the open-loop gain $$a$$ depend on signal level.
2. A signal-dependent gain is, by definition, a nonlinear transfer characteristic — and a nonlinear transfer characteristic is what produces distortion.
3. Negative feedback makes the closed-loop gain $$A$$ far less sensitive to variations in $$a$$.
4. When loop gain $$T = af$$ is large, $$A$$ is set almost entirely by $$1/f$$, regardless of which value $$a$$ happens to take.
5. Consequently, the harmonic and intermodulation distortion that the open-loop amplifier’s nonlinearity would have caused is suppressed by roughly $$1/(1+T)$$.

## Summary

The picture to keep in mind is this: distortion, at its root, is signal-dependent gain in a nonlinear amplifier. Negative feedback doesn’t repair that nonlinearity inside the device — it makes the closed-loop transfer characteristic largely indifferent to it, provided the loop gain stays large across the operating range of interest.

Put differently, the feedback loop stabilizes the overall voltage or current transfer function around the value set by the feedback network itself. Done properly, and with enough loop gain to spare, this gives you an output that tracks the input far more faithfully than the bare nonlinear amplifier ever could on its own.

The next post in this series looks at the four canonical feedback configurations and how each one shapes gain, impedance, and — building on what we’ve covered here — distortion.

## References

1. Notes from ECE 483 Analog IC Design (Prof. Pavan Kumar Hanumolu) @ UIUC
2. Gray, P. R. (2009). *Analysis and Design of Analog Integrated Circuits*, 5th ed.
3. Razavi, Behzad. *Design of Analog CMOS Integrated Circuits*, 2nd ed.
