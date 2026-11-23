# ABC Lab — Interactive Artificial Bee Colony Teaching Tool

> **Live Demo:** [https://abc-lab.vercel.app](https://softcomputing-steel.vercel.app/) *(deploy placeholder)*

**ABC Lab** is a browser-based, zero-dependency teaching and simulation tool for the **Artificial Bee Colony (ABC)** optimization algorithm, developed by Dervis Karaboga (2005). It runs entirely on client-side vanilla HTML5, CSS3, and modern JavaScript—no libraries, no frameworks, and no build step required.

---

## Table of Contents
1. [What is Artificial Bee Colony (ABC)?](#what-is-artificial-bee-colony-abc)
2. [Key Features](#key-features)
3. [Controls Guide](#controls-guide)
4. [How the Hand-Check Works](#how-the-hand-check-works)
5. [Textbook Preset & Invariants Self-Test](#textbook-preset--invariants-self-test)
6. [Local Development & How to Run](#local-development--how-to-run)
7. [Deployment](#deployment)

---

## What is Artificial Bee Colony (ABC)?

The Artificial Bee Colony algorithm models the intelligent foraging behavior of honey bee swarms to solve numerical optimization problems:

$$\min_{\vec{x}} f(\vec{x}), \quad \vec{x} = (x_1, x_2) \in [-B, B]^2$$

In the colony, the number of employed bees equals the number of food sources ($SN$), and the number of onlooker bees also equals $SN$.

### The Three Bee Roles:
1. **Employed Bees ($i = 1 \dots SN$):**
   - Each employed bee is assigned to a specific food source $\vec{x}_i$.
   - It searches for an alternative candidate food source $\vec{v}_i$ in its neighborhood by modifying a random dimension $j \in \{1, 2\}$ using a randomly chosen partner source $\vec{x}_k$ ($k \neq i$):
     $$v_{i,j} = x_{i,j} + \phi_{i,j} \cdot (x_{i,j} - x_{k,j}), \quad \phi_{i,j} \in [-1, 1]$$
   - **Greedy Selection:** If the candidate's fitness is superior ($f(\vec{v}_i) < f(\vec{x}_i)$), the bee moves to $\vec{v}_i$ and resets its trial counter to $0$. Otherwise, the bee stays at $\vec{x}_i$ and increments its trial counter ($\text{trials}_i \leftarrow \text{trials}_i + 1$).

2. **Onlooker Bees ($m = 1 \dots SN$):**
   - Wait in the hive dance area and observe waggle dances displaying nectar quality.
   - Select food sources probabilistically via **roulette wheel selection** proportional to fitness:
     $$\text{fitness}_i = \begin{cases} \frac{1}{1 + f(\vec{x}_i)}, & f(\vec{x}_i) \ge 0 \\ 1 + |f(\vec{x}_i)|, & f(\vec{x}_i) < 0 \end{cases}$$
     $$p_i = \frac{\text{fitness}_i}{\sum_{k=1}^{SN} \text{fitness}_k}$$
   - Fitter sources attract more onlookers, concentrating exploitation on promising regions. Once chosen, an onlooker evaluates a neighbor perturbation using the same candidate formula and greedy update rule.

3. **Scout Bees (Abandonment Mechanism):**
   - Prevent the swarm from trapping in local optima.
   - If a food source cannot be improved after a specified number of consecutive attempts ($\text{trials}_i > \text{limit}$), it is deemed exhausted and **abandoned**.
   - The employed bee converts into a scout and discovers a fresh, uniformly random solution in the domain:
     $$x_{i,j} = -B + \text{rand}(0, 1) \cdot (2B), \quad \text{trials}_i = 0$$

---

## Key Features

- **Interactive 2D Heatmap Canvas (560×560):**
  - High-precision rendering of mathematical objective functions on a logarithmic intensity scale.
  - Coordinate axes crossing through $(0, 0)$ with clear tick labels and domain boundary indicators.
  - Numbered amber circles for food sources with drop shadows and high-contrast labels.
  - Solid green arrows for accepted candidate moves; dashed red arrows for rejected exploratory attempts.
  - Pink target rings ($\times$) marking locations where exhausted food sources were abandoned.
  - Golden five-pointed star crowning the global best solution discovered so far.

- **Reproducible PRNG:**
  - Seeded 32-bit PRNG (`mulberry32`) ensures runs and experiments are bit-identical and reproducible across machines.

- **Real-Time Step-by-Step Hand-Check Panel:**
  - Full arithmetic substitution showing exact real numbers:
    $$v_j = x_{ij} + \phi \cdot (x_{ij} - x_{kj})$$
  - Domain bounds clipping detection (`clipped to [-B, B]`).
  - Expanded objective function evaluations (e.g. $(x_1)^2 + (x_2)^2 = a + b = \text{val}$) for complete hand verification.
  - Color-coded verdict badges: green **Keep new position** vs red **Discard, trials + 1**.

- **Convergence Line Chart (Log10 Scale):**
  - Plots $\log_{10}(\text{best } f)$ over elapsed cycles with integer powers-of-10 grid ticks ($10^2, 10^1, 10^0, 10^{-1}, 10^{-2}, \dots$).
  - Updates at the end of every Scout phase.

- **Fair Random Search Benchmark Toggle:**
  - "Compare with random search" plots a pure uniform random baseline on the convergence chart.
  - Evaluates the exact same count of objective evaluations per cycle ($2 \cdot SN + \text{abandonedCount}$) to benchmark ABC's search efficiency.

- **Stepping Granularity Selector:**
  - **Phase mode:** Step through an entire algorithmic phase (Employed $\rightarrow$ Onlooker $\rightarrow$ Scout) with colony-wide vectors.
  - **Single bee mode:** Step one individual bee at a time, highlighting the active bee (blue ring) and its chosen neighbor partner (cyan dashed ring), accompanied by a plain-language explanation of acceptance/rejection.

- **One-Click Algorithmic Self-Test:**
  - Automatically loads the textbook preset with seed 1, executes a complete cycle, and validates three core invariants with PASS/FAIL badges:
    1. Greedy selection ($f(\text{new}) < f(\text{old})$ for all accepted moves).
    2. Probability distribution ($\sum p_i = 1.0 \pm 10^{-4}$).
    3. Monotonic convergence ($f_{\text{best}}(t+1) \le f_{\text{best}}(t)$).

- **Clean Accessible UI:**
  - Responsive single-column layout on phones and two-column on desktop.
  - High-contrast typography, visible `:focus-visible` keyboard focus indicators, and comprehensive tooltips.
  - Collapsible tutorial banner ("What is ABC?") explaining swarm foraging concepts in plain words.

---

## Controls Guide

| Control | Description |
|---|---|
| **Problem Dropdown** | Select benchmark surface: **Textbook example (hand-check)**, **Sphere**, **Rastrigin**, **Rosenbrock**, or **Ackley**. Displays known global minimums. |
| **Food sources (SN)** | Number of food sources / employed bees ($SN \ge 2$). In Textbook preset, fixed to 4. |
| **Abandonment Limit** | Maximum consecutive unimproved trials before abandoning a source ($limit \ge 1$). In Textbook preset, fixed to 3. |
| **Max cycles** | Termination criterion ($cycles \ge 1$). Default: 30. |
| **Seed** | Seed for `mulberry32` PRNG. Ensures deterministic colony initialization and perturbation sequences. |
| **Speed Slider** | Controls automatic playback speed (100 ms to 2000 ms per step). |
| **Play / Pause** | Toggles automatic stepping. Automatically stops at max cycles with "Finished" status. |
| **Step** | Advances execution by one unit (one phase or one single bee, depending on granularity). |
| **Reset** | Re-initializes the colony with the current seed and parameters. |
| **Granularity** | Switch between **Phase** (Employed $\rightarrow$ Onlooker $\rightarrow$ Scout) and **Single bee** (one bee at a time). |
| **Compare with Random Search** | Toggles simultaneous evaluation of a pure random search baseline with identical evaluation count. |
| **Run Self-Test** | Validates the three mathematical invariants on the textbook benchmark. |

---

## How the Hand-Check Works

Whenever an employed or onlooker bee perturbs its position, the **Hand-check** card computes and renders:
1. **Active Bee & Partner:** Identifies active bee index $i$, partner index $k$, and dimension $j$ ($x_1$ or $x_2$).
2. **Formula Substitution:**
   $$v_{i,j} = x_{i,j} + \phi \times (x_{i,j} - x_{k,j})$$
   Substitutes the exact floating-point coordinates and random perturbation coefficient $\phi \in [-1, 1]$.
3. **Boundary Check:** Indicates whether the candidate value was within $[-B, B]$ or clipped at the boundary.
4. **Objective Calculation (Sphere Expansion):**
   $$f(\vec{x}_{\text{old}}) = (x_{i,1})^2 + (x_{i,2})^2 = a + b = \text{old } f$$
   $$f(\vec{v}_{\text{new}}) = (v_{i,1})^2 + (v_{i,2})^2 = c + d = \text{new } f$$
5. **Greedy Decision:** Compares $f(\vec{v})$ with $f(\vec{x})$. If $f(\vec{v}) < f(\vec{x})$, displays green **✔ keep new position** (trials reset to 0); otherwise displays red **✖ discard, trials + 1** (position retained).

---

## Textbook Preset & Invariants Self-Test

Selecting **Textbook example (hand-check)** configures the canonical demonstration problem:
- **Objective Function:** Sphere $f(x_1, x_2) = x_1^2 + x_2^2$ on domain $[-5, 5]^2$.
- **Colony Size:** $SN = 4$ (inputs locked).
- **Abandonment Limit:** $\text{limit} = 3$ (inputs locked).
- **Starting Coordinates:**
  - Source #1: $(3.0, 2.0) \implies f = 3^2 + 2^2 = 13$
  - Source #2: $(-1.0, 4.0) \implies f = (-1)^2 + 4^2 = 17$
  - Source #3: $(2.0, -3.0) \implies f = 2^2 + (-3)^2 = 13$
  - Source #4: $(-4.0, 1.0) \implies f = (-4)^2 + 1^2 = 17$

Clicking **Run self-test** runs 1 complete cycle under Seed 1 and outputs a PASS/FAIL card verifying algorithmic correctness.

---

## Local Development & How to Run

Because ABC Lab is written in standard vanilla HTML, CSS, and JavaScript with zero external dependencies:

### Option 1: Open directly in your browser
Simply open `index.html` in any modern web browser:
```bash
# On Linux
xdg-open index.html

# On macOS
open index.html

# On Windows
start index.html
```

### Option 2: Run a lightweight static web server
```bash
# Using Python 3
python3 -m http.server 8000

# Using Node.js npx
npx serve .
```
Navigate to `http://localhost:8000` in your web browser.

---

## Deployment

The repository includes a ready-to-deploy `vercel.json` file. To deploy to Vercel:

```bash
# Using Vercel CLI
vercel --prod
```

Or connect the repository to [Vercel](https://vercel.com) / [GitHub Pages](https://pages.github.com) / [Cloudflare Pages](https://pages.cloudflare.com) for instantaneous one-click static hosting.

