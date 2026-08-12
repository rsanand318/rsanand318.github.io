---
layout: post
title: "Feedback I: What Negative Feedback Actually Does"
date: 2026-08-12 19:30:00 +0530
description: "An intuitive and mathematical derivation of the basic negative-feedback equation, loop gain, and why large loop gain forces the feedback signal to track the input."
tags: feedback-series analog-design fundamentals amplifiers
categories: analog-design feedback-series
related_posts: false
citation: true
disqus_comments: true
toc:
  sidebar: right
---

{% comment %}
Post: Feedback I — What Negative Feedback Actually Does
Source: author's personal study notes (Feedback-I_-Negative-Feedback-Amplifiers-2.md)
{% endcomment %}


## Why use negative feedback?

One of the most significant benefits of negative feedback in amplifiers is that it makes the closed-loop behavior less sensitive to changes in the underlying amplifier. For example, the open-loop gain can vary with supply voltage, temperature, process variation, device mismatch, and aging. Negative feedback can reduce the extent to which these variations appear in the overall closed-loop gain.

Feedback also gives the designer a way to control properties such as the input and output impedances of the overall circuit. In the idealized feedback framework, these closed-loop properties can be controlled through the feedback configuration while the underlying amplifier properties are treated separately.

Feedback can also be used deliberately to extend amplifier bandwidth, which will be explored later in the series.

The main tradeoff is that negative feedback reduces the closed-loop gain relative to the open-loop gain. In practice, this loss of gain can be compensated for by using additional amplifier stages when necessary.

## The basic negative-feedback configuration

Consider the basic feedback configuration below.

{% include figure.liquid path="assets/img/posts/feedback-i-negative-feedback-amplifiers/basic-feedback-loop.png" title="Block diagram of the basic negative-feedback configuration showing the summing node, amplifier gain a, and feedback network f" class="img-fluid rounded z-depth-1" zoomable=true %}

The system consists of four main elements:

- $$S_i$$: input signal.
- $$S_\epsilon$$: error signal applied to the basic amplifier.
- $$a$$: open-loop gain of the basic amplifier.
- $$S_o$$: output signal.
- $$f$$: feedback factor of the feedback network.
- $$S_{fb}$$: feedback signal returned to the input summing node.

The feedback is **negative** because the feedback signal is subtracted from the input signal at the summing node.

The key idea is that the amplifier does not directly amplify $$S_i$$. It amplifies the **error signal** $$S_\epsilon$$ created by the difference between the input and the feedback signal.

This gives the two fundamental relationships:

$$
S_o = a S_\epsilon
$$

and

$$
S_\epsilon = S_i - S_{fb}.
$$

The feedback network, in turn, produces a fraction of the output:

$$
S_{fb} = f S_o.
$$

Substituting the error equation into the amplifier equation gives

$$
S_o = a(S_i - S_{fb})
$$

or, using $$S_{fb} = f S_o$$,

$$
S_o = a S_i - a f S_o.
$$

Rearranging,

$$
S_o (1 + af) = a S_i.
$$

Therefore, the closed-loop gain is

$$
A \equiv \frac{S_o}{S_i} = \frac{a}{1+af}.
$$

This is the fundamental negative-feedback equation for the basic model.

## Open-loop gain, closed-loop gain, and loop gain

Here, $$a$$ is the **open-loop gain**: the gain of the basic amplifier when the feedback loop is not closed.

Once the feedback path is included, the overall gain becomes the **closed-loop gain**, which we denote by $$A$$:

$$
A = \frac{S_o}{S_i} = \frac{a}{1+af}.
$$

It is also useful to define

$$
T = af
$$

and call $$T$$ the **loop gain** for this basic feedback model. It represents the product of the forward-path gain and the feedback factor around the loop.

The closed-loop gain can then be written as

\begin{equation}
\label{eq:closed-loop-gain}
A = \frac{a}{1+T}.
\end{equation}

This equation is the starting point for understanding why feedback is so useful.

## What happens when the loop gain is large?

Suppose

$$
T = af \gg 1.
$$

Then $$1 + af \approx af$$, so

$$
A = \frac{a}{1+af} \approx \frac{a}{af} = \frac{1}{f}.
$$

Thus, when the loop gain is sufficiently large, the closed-loop gain is approximately determined by the feedback factor:

$$
\boxed{A \approx \frac{1}{f}}.
$$

This is extremely useful because the open-loop gain $$a$$ is generally difficult to control precisely. It can depend strongly on process variations, device mismatch, supply conditions, temperature, and other changes in the underlying circuit.

The feedback factor $$f$$, on the other hand, can often be made much easier to control, particularly when it is implemented using relatively well-controlled passive components.

Therefore, negative feedback allows us to build a circuit whose overall gain is determined primarily by the feedback network rather than by the exact value of the underlying amplifier gain.

The important point is that the approximation $$A \approx 1/f$$ is not true merely because feedback is present. It requires sufficiently large loop gain:

$$
af \gg 1.
$$

## Why does the feedback signal approach the input?

The equations above also give a more intuitive picture of what the feedback loop is doing.

The amplifier amplifies the error:

$$
S_\epsilon = S_i - S_{fb}
$$

and therefore

$$
S_o = a S_\epsilon.
$$

The feedback network then generates

$$
S_{fb} = f S_o.
$$

Now consider what happens if the feedback signal is initially smaller than the input:

$$
S_{fb} < S_i.
$$

The error signal is positive and relatively large:

$$
S_\epsilon = S_i - S_{fb}.
$$

The amplifier therefore produces a larger output, which causes the feedback network to produce a larger $$S_{fb}$$. This reduces the difference between $$S_i$$ and $$S_{fb}$$.

Conversely, if the feedback signal becomes larger than the input,

$$
S_{fb} > S_i,
$$

then the error signal changes in the opposite direction. The amplifier output and consequently the feedback signal are driven back toward the input.

So the feedback loop continually acts to reduce the error between the input and feedback signals.

This is the central intuition behind negative feedback:

> **The amplifier amplifies the error, and the resulting output is fed back in such a way that the error is reduced.**

The feedback signal is therefore not made equal to the input by some independent mechanism. Rather, the loop gain causes even a small difference between them to produce a corrective output that pushes the feedback signal toward the input.

## Quantifying how closely the feedback tracks the input

We can derive the relationship between $$S_{fb}$$ and $$S_i$$ directly.

Starting with

$$
S_{fb} = f S_o
$$

and

$$
S_o = a S_\epsilon,
$$

we have

$$
S_{fb} = fa S_\epsilon.
$$

Since

$$
S_\epsilon = S_i - S_{fb},
$$

then

$$
S_{fb} = af(S_i - S_{fb}).
$$

Using $$T = af$$,

$$
S_{fb} = T(S_i - S_{fb}).
$$

Rearranging,

$$
S_{fb}(1+T) = T S_i,
$$

so

$$
\frac{S_{fb}}{S_i} = \frac{T}{1+T}.
$$


When $$T \gg 1$$,

$$
\frac{S_{fb}}{S_i} \approx 1.
$$

Therefore,

$$
S_{fb} \approx S_i.
$$

This makes the previous intuition mathematically precise: **large loop gain forces the feedback signal to become a close replica of the input signal.**

At the same time, because

$$
S_\epsilon = S_i - S_{fb},
$$

a feedback signal that closely tracks the input implies that the error signal must be small.

In fact, from the closed-loop relationships,

$$
\frac{S_\epsilon}{S_i} = \frac{1}{1+T}.
$$

Thus, for $$T \gg 1$$,

$$
S_\epsilon \ll S_i.
$$

This is the mechanism by which a large loop gain produces accurate closed-loop behavior: the amplifier does not need a large error signal; its large gain allows a very small error to generate the output required to make the feedback signal track the input.

## Connection to $$\Delta\Sigma$$ ADCs

The same basic feedback intuition appears in $$\Delta\Sigma$$ analog-to-digital converters.

{% include figure.liquid path="assets/img/posts/feedback-i-negative-feedback-amplifiers/delta-sigma-loop.png" title="Delta-sigma ADC feedback loop showing integrator, comparator, and 1-bit DAC feedback path" class="img-fluid rounded z-depth-1" zoomable=true %}

In the shown modulator, the quantized output is converted back into an analog feedback signal through the 1-bit DAC and fed to the input summing node. The integrator and comparator form the forward path, while the DAC provides the feedback path.

The negative-feedback loop acts to reduce the difference between the input and the feedback signal. As a result, the feedback signal approximates the input over time, with the quantized output switching between the available DAC levels.

This is the same fundamental feedback idea seen in the basic amplifier: the loop uses the forward-path gain to act on the difference between the desired input and the feedback signal.

## Takeaway

The basic negative-feedback relationship is

$$
\boxed{A = \frac{a}{1+af}}
$$

with loop gain

$$
\boxed{T = af}.
$$

When the loop gain is large,

$$
T \gg 1,
$$

the closed-loop gain becomes approximately

$$
\boxed{A \approx \frac{1}{f}},
$$

while the feedback signal approaches the input:

$$
\frac{S_{fb}}{S_i} = \frac{T}{1+T} \approx 1.
$$

The underlying mechanism is simple: **the amplifier amplifies the error, and the amplified error drives the output in a direction that reduces the error through the feedback path.**

This basic mechanism is what allows feedback to make the overall circuit less dependent on the exact characteristics of the underlying amplifier. The later parts of the series will examine how this same principle can be used to desensitize gain, improve linearity, and extend bandwidth.

{% comment %}

IMAGE MANIFEST — copy these files into the repository before deploying

Post image directory:
assets/img/posts/feedback-i-negative-feedback-amplifiers/

Required files:
1. Source/role: Basic feedback loop block diagram referenced in "The basic negative-feedback configuration" section (originally embedded as ![][image1] in the source notes)
   Exact filename: basic-feedback-loop.png
   Exact repository path: assets/img/posts/feedback-i-negative-feedback-amplifiers/basic-feedback-loop.png
   Used in post as: {% include figure.liquid path="assets/img/posts/feedback-i-negative-feedback-amplifiers/basic-feedback-loop.png" title="Block diagram of the basic negative-feedback configuration showing the summing node, amplifier gain a, and feedback network f" class="img-fluid rounded z-depth-1" zoomable=true %}
   Status: missing — author must provide

2. Source/role: Delta-sigma modulator loop diagram referenced in "Connection to Delta-Sigma ADCs" section (originally embedded as ![][image2] in the source notes)
   Exact filename: delta-sigma-loop.png
   Exact repository path: assets/img/posts/feedback-i-negative-feedback-amplifiers/delta-sigma-loop.png
   Used in post as: {% include figure.liquid path="assets/img/posts/feedback-i-negative-feedback-amplifiers/delta-sigma-loop.png" title="Delta-sigma ADC feedback loop showing integrator, comparator, and 1-bit DAC feedback path" class="img-fluid rounded z-depth-1" zoomable=true %}
   Status: missing — author must provide

Thumbnail, if used:
No suitable thumbnail image was supplied in the source notes; the thumbnail field has been omitted from the front matter. If desired, add one of the above figures (or a dedicated hero image) and re-add the thumbnail field, e.g.:
thumbnail: assets/img/posts/feedback-i-negative-feedback-amplifiers/basic-feedback-loop.png

Deployment checks:
- Confirm every listed file exists at its exact case-sensitive path.
- Confirm every image include path in the article matches the manifest exactly.
- Confirm the thumbnail path, if present, matches a real file.
- Commit both the Markdown post and all listed image files before deployment.

{% endcomment %}

## References

1. Notes from ECE 483 Analog IC Design (Prof. Pavan Kumar Hanumolu) @ UIUC
2. Gray, P. R. (2009). *Analysis and Design of Analog Integrated Circuits*, 5th ed.
3. Razavi, Behzad. *Design of Analog CMOS Integrated Circuits*, 2nd ed.
