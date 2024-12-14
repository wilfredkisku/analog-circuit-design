# Analog Circuit Design

[![Quarto Publish](https://github.com/iic-jku/analog-circuit-design/actions/workflows/quarto-publish.yml/badge.svg?branch=main)](https://github.com/iic-jku/analog-circuit-design/actions/workflows/quarto-publish.yml)
[![DOI](https://zenodo.org/badge/830446772.svg)](https://doi.org/10.5281/zenodo.14387481)

**(c) 2024 Harald Pretl and co-authors, Institute for Integrated Circuits (IIC), Johannes Kepler University, Linz (JKU)**

This is the material for an intermediate-level MOSFET circuit design course, held at JKU under course number 336.009 ("KV Analoge Schaltungstechnik"). Follow this [link](https://iic-jku.github.io/analog-circuit-design) to access the material.

The course makes heavy use of circuit simulation, using **Xschem** for schematic entry and **ngspice** for simulation. The 130nm CMOS technology **SG13G2** from IHP Microelectronics is used.

Tools and PDK are integrated in the **IIC-OSIC-TOOLS** Docker image, which will be used during the coursework.

All course material is made publicly available and shared under the Apache-2.0 license.

**We happily accept PR to fix typos or add content! If you want do discuss something that is not clear, please open an issue!**








########

::: {.content-hidden}
Copyright (C) 2024 Harald Pretl and co-authors (harald.pretl@jku.at)

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
:::

# A Basic 5-Transistor OTA {#sec-basic-ota}

Suited with the knowledge of basic transistor operation (@sec-first-steps and @sec-gmid-method) and the working knowledge of the current mirror (@sec-mosfet-diode and @sec-current-mirror) as well as the differential pair (@sec-diff-pair) we can now start to design our first real circuit. A fundamental (simple) circuit that is often used for basic tasks is the 5-transistor operational transconductance amplifier (OTA). A circuit diagram of this 5T-OTA is shown in @fig-basic-ota.

{{< include ./figures/_fig_basic_ota.qmd >}}

::: {.callout-warning title="Refresh MOSFET Basic Circuits"}
While we repeat the basics of elementary MOSFET amplifier stages (like common-source stage, common-gate stage, and current mirror) in this course material, the following compendium [@murmann_mos_stages_2013] is recommended for review. It is freely available at <https://github.com/bmurmann/Book-on-MOS-stages>.

In addition, we can highly recommend these references [@Gray_Mayer_5th_ed, @Razavi_Analog_CMOS] for further study.
:::

The operation is as follows: $M_{1,2}$ form a differential pair which is biased by the current source $M_5$. $M_{5,6}$ form a current mirror, thus the input bias current $I_\mathrm{bias}$ sets the bias current in the OTA. The differential pair $M_{1,2}$ is loaded by the current mirror $M_{3,4}$ which mirrors the output current of $M_1$ to the right side. Here, the currents from $M_4$ and $M_2$ are summed, and together with the conductance effective at the output node a voltage builds up.

We note that $M_{1,2}$ and $M_{3,4}$ need to be symmetric, thus will have the same $W$ and $L$ dimensioning. $M_{5,6}$ we scale accordingly to set the correct bias current in the OTA.

As this is an OTA the output is a current; if the load impedance is high (i.e., purely capacitive, which is often the case in integrated circuits when driving MOSFET inputs) then the voltage gain of the OTA can be high (of course, in this simple OTA it is limited). With a high-impedance loading this OTA can provide a voltage output, and this is actually how OTAs are mostly operated.

## Voltage Buffer with OTA

In order to design an OTA we need an application, and from this we need to derive the circuit specifications. We want to use this OTA to realize a voltage buffer which lightly loads a voltage source and can drive a large capacitive load. Such a configuration is often used to, e.g., buffer a reference voltage that is needed (and thus loaded) by another circuit. The block diagram of this configuration is shown in @fig-voltage-buffer-ota.

{{< include ./figures/_fig_voltage_buffer_ota.qmd >}}

If the voltage gain of the OTA in @fig-voltage-buffer-ota is high, then $V_\mathrm{out} \approx V_\mathrm{in}$. We now want to design an OTA for this application for the following specification values (see @tbl-voltage-buffer-spec). These values are rather typical of what could be expected for such a buffer design.

| Specification                                           | Value                              | Unit    |
|:--------------------------------------------------------|:----------------------------------:|:-------:|
| Supply voltage                                          | $1.45 < \underline{1.5} < 1.55$    | V       |
| Temperature range (industrial)                          | $-40 < \underline{27} < 125$       | degC    |
| Load capacitance $C_\mathrm{load}$                      | $50$                               | fF      |
| Input voltage range (for buffering 2/3 bandgap voltage) | $0.7 < \underline{0.8} < 0.9$      | V       |
| Signal bandwidth (3dB)                                  | $>10$                              | MHz     |
| Output voltage error                                    | $<3$                               | %       |
| Total output noise (rms)                                | $<1$                               | mV~rms~ |
| Supply current (as low as possible)                     | $<10$                              | µA      |
| Stability                                               | stable for rated $C_\mathrm{load}$ |         |
| Turn-on time (settled to with 1%)                       | $<10$                              | µs      |
| Externally provided bias current (nominal)              | $20$                               | µA      |

: Voltage buffer specification {#tbl-voltage-buffer-spec}

## Large-Signal Analysis of the OTA {#sec-basic-ota-large-signal}

The first step when receiving a design task is to look at the specifications, and see whether they make sense. Detailed performance of the design will be the result of the circuit simulation, but before we step into sizing we need to do a few simple calculations to (a) allows to do back-of-the-envelope gauging if the specification makes sense, and (b) the derived analytical equations will serve as guide for the sizing procedure.

- In terms of large-signal operation, we will now check whether the input and output voltage range, as well as the settling time can be roughly met.
- When the input is at its maximum of $0.9\,\text{V}$, we see that we need to keep $M_1$ in saturation. We can calculate that $\VDS[1] = \VDD - |\VGS[3]| + \VGS[1] - V_\mathrm{in} = 1.45 - 0.6 + 0.6 - 0.9 = 0.55\,\text{V}$, which leaves enough margin.
- When the input is at its minimum of $0.7\,\text{V}$, we see that the $\VDS[5]$ of $M_5$ is calculated as $\VDS[5] = V_\mathrm{in} - \VGS[1] = 0.7 - 0.6 = 0.1\,\text{V}$, so this leaves little margin, but likely $\VGS[1]$ will be smaller, so it should work out.
- For the output voltage, when the output voltage is on the high side, it leaves $|\VDS[4]| = \VDD - V_\mathrm{out} = 1.45 - 0.9 = 0.55\,\text{V}$, which is enough margin.

In summary, we think that we can make an NMOS-input OTA like the one in @fig-basic-ota work for the required supply and input- and output voltages. If this would not work out, we need to look for further options, like a PMOS-input OTA, or a NMOS/PMOS-input OTA.

Another large-signal specification item that we can quickly check is the settling time. Under slewing conditions, the complete bias current in the OTA is steered towards the output (try to understand why this is the case), so when the output capacitor is fully discharged, and we assume just a linear ramp due to constant-current charging of the output capacitor, the settling time is
$$
T_\mathrm{slew} \approx \frac{C_\mathrm{load} V_\mathrm{out}}{I_\mathrm{tail}} = \frac{50 \cdot 10^{-15} \cdot 1.3}{10 \cdot 10^{-6}} = 6.5\,\text{ns}
$$
so this leaves plenty of margin for additional slow-signal settling due to the limited bandwidth, as well as reducing the supply current.

The small-signal settling (assuming one pole at the bandwidth corner frequency) leads to an approximate settling time (1% error corresponds to $\approx 5 \tau$) of
$$
T_\mathrm{slew} \approx \frac{5}{2 \pi f_\mathrm{c}} = \frac{5}{2 \pi \cdot 1 \cdot 10^{-6}} = 0.8\,\mu\text{s}.
$$
which also checks out.

## Small-Signal Analysis of the OTA {#sec-basic-ota-small-signal}

In order to size the OTA components we need to derive how MOSFET parameters define the performance. The important small-signal metrics are

- dc gain $A_0$
- gain-bandwidth product (GBW)
- output noise

The specification for GBW is given in @tbl-voltage-buffer-spec, the dc gain we have to calculate from the voltage accuracy specification. For a voltage follower in the configuration shown in @fig-voltage-buffer-ota the voltage gain is given by
$$
\frac{V_\mathrm{out}}{V_\mathrm{in}} = \frac{A_0}{1 + A_0}.
$$ {#eq-voltage-buffer-tolerance}

So in order to reach an output voltage accuracy of at least 3% we need a dc gain of $A_0 > 30.2\,\text{dB}$. To allow for process and temperature variation we need to add a bit of extra gain as margin. 

### OTA Small-Signal Transfer Function

In order to derive the governing equations for the OTA we will make a few simplifications:

- We will set $\gmb = 0$ for all MOSFETs.
- We will further set $\Cgd = 0$ for all MOSFETs except for $M_4$ where we expect a Miller effect on this capacitor, and we could add its effect by increasing the capacitance at the gate node of $M_{3,4}$ (for background please see @sec-miller-theorem). However, as this does not create a dominant pole in this circuit, we consider this a minor effect (see @eq-simple-ota-gain). Thus, only $\Cgs[34]$ is considered at the gate node of the current mirror load.
- We assume $\gm \gg \gds$, so we set $\gds[1] = \gds[3] = 0$.
- The drain capacitance of $M_2$ and $M_4$, as well as the gate capacitance of $M_2$ we can add to the load capacitance $C_\mathrm{load}$. Note that $\Cgs[2]$ can be added because of the feedback connection between the inverting input and the output. However, this is not shown in the small-signal equivalent circuits below, because we are interested in the open-loop transfer function.

The resulting small-signal equivalent circuit is shown in @fig-basic-ota-small-signal.

::: {.callout-warning title="Refresh MOSFET Small-Signal Model"}
Please review the MOSFET small-signal equivalent model in @fig-mosfet-small-signal-model at this point. For the PMOS just flip the model upside-down.
:::

{{< include ./figures/_fig_basic_ota_small_signal.qmd >}}

We can further simplify the output side by recognizing that the impedance looking from the output down we have $\gds[2]$ in series with $\gds[5] + \gm[12]$ (since we treat $M_1$ as a common-gate stage when looking from the output, and since it is loaded by a low impedance of $\gm[34]^{-1}$ we can approximate the impedance looking into the source of $M_1$ with $\gm[12]^{-1}$). With the approximation that $\gm \gg \gds$ the parallel connection of $\gm[12]$ and $\gds[5]$ is dominated by $\gm[12]$ and series connection by $\gds[2]$. Therefore, we can move $\gds[2] + \gds[4]$ in parallel to $C_\mathrm{load}$. Further, assuming a differential drive with a virtual ground at the tailpoint we can remove $\gds[5]$. The current source $\gm[34] \vgs[34]$ is replace with the equivalent conductance $\gm[34]$. This results in the further simplified equivalent circuit shown in @fig-basic-ota-small-signal-simplified.

{{< include ./figures/_fig_basic_ota_small_signal_simplified.qmd >}}

In the simplified circuit model in @fig-basic-ota-small-signal-simplified we can see that we have two poles in the circuit, one at the gate note of $M_{3,4}$, and one at the output. Realizing that $v_\mathrm{in,p} = v_\mathrm{in}/2$ and $v_\mathrm{in,n} = - v_\mathrm{in}/2$ we can formulate KCL at the output node to
$$
-\gm[34] \Vgs[34] - \left( -\gm[12] \frac{V_\mathrm{in}}{2} \right) - V_\mathrm{out} (\gds[2] + \gds[4] + s C_\mathrm{load}) = 0.
$$ {#eq-simple-ota-eq1}
 We further realize that
 $$
\Vgs[34] = -\gm[12] \frac{V_\mathrm{in}}{2} \frac{1}{\gm[34] + s \Cgs[34]}.
 $$ {#eq-simple-ota-eq2}

By combining @eq-simple-ota-eq1 and @eq-simple-ota-eq2 and after a bit of algebraic manipulation we arrive at
$$
A(s) = \frac{V_\mathrm{out}}{V_\mathrm{in}} = \frac{\gm[12]}{2} \frac{2 \gm[34] + s \Cgs[34]}{(\gm[34] + s \Cgs[34]) (\gds[2] + \gds[4] + s C_\mathrm{load})}.
$$ {#eq-simple-ota-gain}

When we now inspect @eq-simple-ota-gain we can see that for low frequencies the gain is
$$
A(s \rightarrow 0) = A_0 = \frac{\gm[12]}{\gds[2] + \gds[4]}
$$ {#eq-simple-ota-gain-dc}
which is plausible, and confirms the requirement of a high impedance at the output node. For very large frequencies we get
$$
A (s \rightarrow \infty) = \frac{\gm[12]}{2 s C_\mathrm{load}}
$$ {#eq-simple-ota-highf}
which is essentially the behavior of an integrator, and we can use @eq-simple-ota-highf to calculate the frequency where the gain drops to $1$:
$$
f_\mathrm{ug} = \frac{\gm[12]}{4 \pi C_\mathrm{load}}
$$

when looking at @eq-simple-ota-gain we see that we have a dominant pole at $s_\mathrm{p}$ and a pole-zero doublet with $s_\mathrm{pd}$/$s_\mathrm{zd}$:
$$
s_\mathrm{p} = -\frac{\gds[2] + \gds[4]}{C_\mathrm{load}}
$$
$$
s_\mathrm{pd} = -\frac{\gm[34]}{\Cgs[34]}
$$
$$
s_\mathrm{zd} = -\frac{2 \gm[34]}{\Cgs[34]}
$$

### OTA Noise

For the noise analysis we ignore the pole-zero doublet due to $\Cgs[34]$ (we assume minor impact due to this) and just consider the dominant pole. For the noise analysis at the output we set the input signal to zero, and thus we arrive at the simplified small-signal circuit shown in @fig-basic-ota-small-signal-noise.

{{< include ./figures/_fig_basic_ota_small_signal_noise.qmd >}}

We see that
$$
\overline{\Vgs[34]^2} = \frac{1}{\gm[34]^2} \left( \overline{I_\mathrm{n1}^2} + \overline{I_\mathrm{n3}^2} \right).
$$

::: {.callout-important title="Noise Addition"}
Remember that **uncorrelated** noise quantities need to be power-summed (i.e., $I^2 = I_1^2 + I_2^2$)!
:::

We can then sum the output noise current $\overline{I_\mathrm{n}}$ as
$$
\overline{I_\mathrm{n}^2} = \overline{I_\mathrm{n2}^2} + \overline{I_\mathrm{n4}^2} + \gm[34]^2 \frac{1}{\gm[34]^2} \left( \overline{I_\mathrm{n1}^2} + \overline{I_\mathrm{n3}^2} \right) = 2 \left( \overline{I_\mathrm{n12}^2} + \overline{I_\mathrm{n34}^2} \right).
$$

As a next step, let us rewrite the OTA transfer function $A(s)$ (see @eq-simple-ota-gain) by getting rid of the pole-zero doublet as a simplifying assumption to get
$$
A'(s) = \frac{\gm[12]}{\gds[2] + \gds[4] + s C_\mathrm{load}}.
$$ {#eq-simple-ota-gain-simplified}

Inspecting @eq-simple-ota-gain-simplified we can interpret the OTA transfer function as a transconductor $\gm[12]$ driving a load of $Y_\mathrm{load} = \gds[2] + \gds[4] + s C_\mathrm{load}$. We can thus redraw @fig-voltage-buffer-ota in the following way, injecting the previously calculated noise current into the output node. The result is shown in @fig-voltage-buffer-ota-noise.

{{< include ./figures/_fig_voltage_buffer_ota_noise_zout.qmd >}}

::: {.callout-note title="Output Impedance of the Voltage Buffer"}
First we short the input terminal to ground and then we connect a current source $I_\mathrm{out}$ at the output terminal, see @fig-voltage-buffer-ota-noise-zout. Since we can neglect the gate leakage current into the inverting  input terminal of the OTA, KCL at the output node is simply:
$$
I_\mathrm{out} + \gm[12]\left(-V_\mathrm{out}\right) = 0
$$
Thus, the output impedance is easily calculated.
$$
Z_\mathrm{out} = \frac{V_\mathrm{out}}{I_\mathrm{out}} = \frac{V_\mathrm{out}}{\gm[12]V_\mathrm{out}} = \frac{1}{\gm[12]}
$$

:::

{{< include ./figures/_fig_voltage_buffer_ota_noise.qmd >}}

We see that the feedback around the transconductor $\gm[12]$ creates an impedance of $1/\gm[12]$. We can now calculate the effective load conductance of
$$
Y'_\mathrm{load} = \gds[2] + \gds[4] + s C_\mathrm{load} + \gm[12] \approx \gm[12] + s C_\mathrm{load}.
$$ {#eq-basic-ota-output-noise-yload}

The output noise voltage is then (using @eq-mosfet-noise)
$$
\overline{V_\mathrm{n,out}^2}(f) = \frac{\overline{I_\mathrm{n}^2}}{|Y'_\mathrm{load}|^2} = \frac{\overline{I_\mathrm{n}^2}}{\gm[12]^2 + (2 \pi f C_\mathrm{load})2^2} = \frac{8 k T (\gamma_{12} \gm[12] + \gamma_{34} \gm[34])}{\gm[12]^2 + (2 \pi f C_\mathrm{load})^2}.
$$

We can use the identity @eq-integral-identity to calculate the rms output noise to
$$
V_\mathrm{n,out,rms}^2 = \int_0^\infty \overline{V_\mathrm{n,out}^2}(f) df = \frac{k T}{C_\mathrm{load}} \left( 2 \gamma_{12} + 2 \gamma_{34} \frac{\gm[34]}{\gm[12]} \right).
$$ {#eq-basic-ota-output-noise}

Inspecting @eq-basic-ota-output-noise we can see that the integrated output noise is the $k T / C$ noise of the output load capacitor, enhanced by the $\gamma_{12}$ of the input differential pair, plus a (smaller) contribution of the current mirror load $M_{3,4}$. Intuitively, this result makes sense.

::: {.callout-tip title="Exercise: Derivation of 5T-OTA Performance"}
Please take your time and carefully go through the explanations and derivations for the 5-transistor-OTA in @sec-basic-ota-large-signal and @sec-basic-ota-small-signal. Try to do the calculations yourself; if you get stuck, review the previous chapters.
:::

## 5T-OTA Sizing {#sec-basic-ota-sizing}

Outfitted with the governing equations derived in @sec-basic-ota-small-signal we can now size the MOSFETs in the OTA, we remember that we have to size $M_{1,2}$ and $M_{3,4}$ equally.

First, we need to select a proper $\gmid$ for the MOSFET. Remembering @sec-gmid-method we see that for the input differential pair we should go for a large $\gm$, thus we select a $\gmid = 10$. As $\gds$ of $M_2$ could limit the dc gain (@eq-simple-ota-gain-dc) we go with a rather long $L = 5\,\mu\text{m}$. For current sources a small $\gmid$ is a good idea, so we start with $\gmid=5$ (because we can not go too low because of $V_\mathrm{ds,sat}$) and also an $L = 5\,\mu\text{m}$. The $\gmid$ is also useful to estimate the required drain-source voltage to keep a MOSFET in saturation (i.e., keep the $\gds$ small) with this approximate relationship:

$$
\Vds[,sat] = \frac{2}{\gmid}
$$ {#eq-gmid-saturation}

::: {.callout-tip title="Exercise: 5T-OTA Sizing"}
Please size the 5T-OTA according to the previous $\gmid$ and $L$ suggestions. Please calculate the $W$ of $M_{1-6}$ and the total supply current. Please check wether gain error, total output noise, and turn-on settling is met with the calculated devices sizes and bias currents.
:::

The sizing procedure and its calculation are best performed in a Jupyter notebook, as we can easily look up the exact data from the pre-computed tables:

::: {.callout-tip title="Solution: 5T-OTA Sizing" collapse="true"}
{{< embed ./sizing/sizing_basic_ota.ipynb echo=true >}}
:::

## 5T-OTA Simulation {#sec-basic-ota-simulation}

With the initial sizing of the MOSFETs of the 5T-OTA done, we can design the 5T-OTA circuit and setup a simulation testbench to check the performance parameters. Since this is the first time we draw a more complex schematic, and use a hierarchical design, we should note that drawing a schematic is an art, and there exists a set of rules and recommendations how to name pins, how to use annotations, and so on. Please read @sec-designers-etiquette before you start into your design work.

::: {.callout-tip title="Exercise: 5T-OTA Design and Testbench"}
Please design the circuit of the 5T-OTA. Put the OTA circuit in a separate schematic, create a symbol for it, and use this symbol in a testbench you create in Xschem for this 5T-OTA used as a voltage buffer as shown in @fig-voltage-buffer-ota. Use typical conditions for the simulation, and check how well the specification in @tbl-voltage-buffer-spec is met, and how well the derivations in @sec-basic-ota-large-signal and @sec-basic-ota-small-signal fit to the simulation results.

If you get stuck, you can find the testbench and 5T-OTA schematic [here](./xschem/ota-5t_tb-ac.svg) (for the small-signal analysis) and [here](./xschem/ota-5t_tb-tran.svg) (for the large-signal settling simulation). For interested students, the loop gain analysis with Middlebrook's and Tian's method of the 5T-OTA can be found [here](./xschem/ota-5t_tb-loopgain.svg).
:::

## 5T-OTA Simulation versus PVT

As you have seen in @sec-basic-ota-simulation running simulations by hand is tedious. When we want to check the overall performance, we have to run many simulations over various conditions:

1. The supply voltage of the circuit has tolerances, and thus we need to check the performance against this variation.
2. The temperature at which the circuit is operated is likely changing. Also the performance against this has to be verified.
3. When manufacturing the wafers random variations in various process parameters lead to changed parameters of the integrated circuit components. In order to check for this effect, wafer foundries provide model files which shall cover these manufacturing excursions. Simplified, this leads to a slower or faster MOSFET, and usually NMOS and PMOS are not correlated, so we have the process corners **SS**, **SF**, **TT**, **FS**, and **FF**. So far, we have only used the **TT** models in our simulations.

The variations listed in the previous list are abbreviated as **PVT** (process, voltage, temperature) variations. In order to finalize a circuit all combinations of these (plus the variations in operating conditions like input voltage) have to be simulated. As you can imagine, this leads to a huge number of simulations, and simulation results which have to be evaluated for pass/fail.

There are two options how to tackle this efficiently:

1. As an experienced designer you have a very solid understanding of the circuit, plus based on the analytic equations you can identify which combination of operating conditions will lead to a worst case performance. Thus, you can drastically reduce the number of corners to simulate, and you run them by hand.
2. You are using a framework which highly automates this task of running a plethora of different simulations and evaluating the outcome. These frameworks are called simulation runners.

Luckily, there are open-source versions of simulation runners available, and we will use [CACE](https://github.com/efabless/cace) in this lecture. CACE is written in Python and allows to setup a datasheet in [YAML](https://yaml.org) which defines the simulation problem and the performance parameters to evaluate against which limits. The resulting simulations are then run in parallel and the simulation data is evaluated and summarized in various forms.

There is a CACE setup available for our 5T-OTA. The [datasheet](./cace/voltage-buffer-ota.yaml) describes the operating conditions and the simulations tasks. For each simulation a testbench template is needed, [this one](./cace/templates/ota-5t-ac.sch) is used for ac simulations, [this one](./cace/templates/ota-5t-noise.sch) is used for noise simulation, and [this one](./cace/templates/ota-5t-tran.sch) is used for transient simulation.

After a successful run, a documentation is automatically generated. The result of a full run of this [OTA design](./xschem/ota-5t.svg) is presented here:

::: {#nte-basic-ota-cace-result .callout-note title="CACE Summary for 5T-OTA" collapse="false"}

{{< include ./cace/_docs/ota-5t_schematic.md >}}
:::

### PVT Simulation Analysis

Looking at the CACE report in @nte-basic-ota-cace-result we see that (luckily) the specification is met for all parameters. This is great news! We now have a design that we carefully simulated across PVT and other corners, and which is ready for layout. Once we have the layout ready, we can extract the wiring parasitic ($R$ and $C$) as well as other layout-dependent effects like [well proximity](https://global.oup.com/us/companion.websites/9780195170153/pdf/proximityeffectmodels.pdf). Using this augmented netlist we can then again use CACE to check performance across conditions and parameter variations, and if we still pass all specification points then our design is finished.

