# DIY VNA for low frequencies
How to measure complex impedance at low (audio) frequencies.

# Motivation
I was studying magnetism in general and the magnetic permeabilities of materials in particular. See projects

* transformer-cores
* ferrite-cores
* ferrite-beads

At HF/UHF frequencies I can measure reasonably everything and it all makes sense. Then I came across line power inductors/transformers, I salvaged a few from an old fluorescent tube starter circuit. These suppose to operate at 60Hz.

NanoVNA's lowest frequency is 50kHz. These look to me as transformer steel aka silicon steel aka magnetically soft steel. I was not able to measure coil impedance and compute the relative permeability of these iron cores that would match online records.

VNA measurement (see the transformer-cores project) shows a steady drop of relative permeability from ~90 down. 

<img src="iron-core.png" width=400/>

The literature I found on the internet has extensive coverage for ferrite, nanocrystalline, and amorphous core materials that operates at high frequencies. The datasheets start from 10kHz and other times from 100 kHz.Surprisingly, line power core materials like transformer iron silicon steel are not as popular. It's probably old tech and not a significant source of research/study.

There are tables like this being copied from one [source][2] to another

<img src="magnetic_materials.jpg" width=400/> 

But it is rather pointless, as the table does not depict the dependencies on frequencies and temperature. 
There are (chemically) different kinds of steels and irons that have (physically) different crystal structures that affect their magnetic properties. This page is a [click-bait, paid-for, source of great info on magnetic metallurgy][4]. Even from free snippets, one can gain a great insight into the complexities there.

There is a relevant study [Complex Permeability for Various Magnetic Materials][3] (also see PDF)

Good introduction from there

> The magnetic materials are essential for the magnetic energy storage of various applications such as power converters and wireless power transfer systems. As low operating frequency applications (50 Hz/60 Hz), silicon steel sheet materials are widely used for the utility transformers or pole transformers in various industries. On the other hands, most electric devices are designed based on the high-frequency magnetic materials, operating at several tens of kHz high operating frequency, for compact size of the magnetic components. In general, high-frequency ferrite cores are highly preferred due to high-frequency characteristics, low core loss, and easy manufacturing. Recently, amorphous cores, which are formed by multi layers of laminated cores, are also the alternative solutions for 60 Hz–100 kHz of operating frequency applications. Such amorphous cores have a high saturated magnetic field Bsat (≈1.5 T) and can reduce the eddy loss due to ultra-thin-laminated Fe-based plates. In order to miniaturize the magnetic materials, Nanocrystalline alloy, which is usually composed of Fe, Si, B with the other chemical combinations, can be utilized over 1 kHz–100 kHz of operating frequency applications. Such magnetic materials mentioned above have their own merits and demerits for various applications.

I also found this study on the [Complex Permeability of Silicon Steel][1] (also see PDF) that shows the permeability they measured. 

<img src="iron-core-from-paper.jpg" width=400/>

Notice the tail of the plot, painted yellow. These are the frequencies I was able to measure on the plot above. This portion matches my measurements.



All-in-all, I wanted to measure complex impedance at low frequencies to characterize salvaged cores.

# The impedance at low frequencies

Turns out that measuring impedance is not easy. Measuring impedance with precision is hard, and knowing where errors are coming from is even harder.
Here is one simple circuit, that requires a signal generator and AC voltmeters or oscilloscope. 

<img src="circuit.jpg" width=300/> 

<img src="scope_fixture.png" height=200/> 

Here the known resistance R1 is used to measure current, technically voltage, but assuming that R1 is purely resistive, it gives us the current waveform 
$$I = \frac V R$$ 
then the impedance Z is computed via the phase angle between voltage and current waveforms.

See [diy-vna.ipynb](diy-vna.ipynb)

Scope probes have finite resistance and stray capacitance. The measurement circuit model looks like this:

<img src="circuit2.jpg" width=300/> 

Another naive assumption is that the coil itself is purely inductive with some preset resistance across all frequencies. 

<img src="skin-effect.webp" height=100/> 

It is not the case. As frequency goes up, the copper wire becomes more and more resistant due to the skin effect. What is also counterintuitive, is that a thicker wire proportionally loses conductivity faster. A thin wire does not have enough "depth" to lose.

<img src="awg-skin-effect.png" width=300/> 

See [code here](skin-effect.ipynb)

Here is a quick reference table wire [AWG-to-Max frequency for 100% skin depth][6]. 

The somewhat good news is that for 100Hz..100kHz we can use thin wires AWG26+ and ignore this error.

Yet another source of errors is the wire proximity effect between the windings. Which I'm also going to ignore.

<img src="proximity-effect.jpeg" width=200/>  

I am going to space wire loops apart and ignore this error.

## 3 measurements method

For impedance measurement, we need an AC voltmeter. An oscilloscope is a relatively imprecise voltmeter. ADC is 8 bits as it is, then there is offset and actual % error. This is another source of error. Yet, the Rigol ds1054z is the only instrument I have.

Here is a [good source for circuits][5] for cheap DIY impedance measurement. 

Here is a [SPICE](diy-vna.asc) simulation. The computations in [diy-vna.[ipynb](diy-vna.ipynb) are sometimes called the "simple 3 measurements method"

Notice, that measuring impedance at frequencies like 10kHz  with a preset resistor R1 of 100 Ohm introduces another issue. The phase angle is a fraction of a degree. Many sources mention in passing that one needs to select a resistor that roughly matches the unknown impedance Z. This is where it comes from.
Also notice that the phase angle depends on both R1 and frequency. For really small values of inductance/capacitance and very low frequencies like 100 Hz the resistor needs to be in single digits mOhm range with 1% tolerance. At these ranges, wire resistance and stray inductance/capacitance become a factor. 
Since there is no calibration there is no way to compensate for them.

Better than nothing but useless for transformer iron cores. The inductance I need to measure is small and the frequencies are low.

## Better methods

Read this paper [An LMS Impedance Bridge][7] on other methods to improve accuracy.

Turns out silicon iron per

# Notes

Ferrite using MgZn can store energy better at high f but saturate at low f. While CRGOS cores are magnetized with 10% rated current to couple far greater energy than the reactive energy stored... The disadvantage is that it is
easily saturated (its saturation flux density is typically <
0.5 T).


As the name indicates, ferromagnetic powder cores are
not ferrites, but cores made up of oxidised ferromagnetic powder, moulded in a binding agent. The advantage of
this material is that the core is not as easily saturated as
in the case of ferrites. The saturation flux density is as
high as approx. 1.5 T, making it possible to operate with
high direct currents. Such cores also cope with high frequencies and are thermally stable.
The greatest disadvantage of ferromagnetic powder cores is that they have low permeability. This is due to the
spacing between the iron particles, which adds up to a
large air gap (known as a distributed air gap). Such cores are mainly used in reactors for energy storage and
filtering at low frequencies. They are also used for HF
impedance adjustment. Ferromagnetic powder cores are
produced virtually exclusively in the form of toroidal cores.


. But my point is that the transformer core is not involved in limiting the power delivered. This limit comes from the windings, and has two faces: One is the voltage drop, which is proportional to load current and will at some point be so much that the voltage is no longer sufficient for your load, and the other is heating. As the load current increases, the power dissipation in the windings increases to the square, and if you draw enough power from the transformer for enough time, you will burn it up.


All the above makes clear that a transformer's power handling ability depends as much on its magnetic cross section (because more section requires fewer turns, allowing thicker wire to be used), and on its window size, that is, the cross-sectional area of the space where the windings go. But there is no linear formula relating the product of the two areas to power! As the transformer grows larger, the path length for heat evacuation becomes longer, and more importantly, the heat-generating volume of material increases at a faster rate than the heat-dissipating surface. So the increase in power handling ability isn't linearly related to the area product, nor to total volume, weight nor surface area. The best approximation for the rate of change of power capability when transformer size changes, is that the power changes at a slightly slower rate than the volume of the transformer. 


For one, at frequencies much above a few hundred Hertz, saturation is no longer the limiting factor when choosing the maximum flux density. The reason is that the losses in the magnetic material become so large that the flux density has to be kept much below saturation, simply in order to keep the core losses at an acceptable level!


[1]: <https://ieeexplore.ieee.org/document/4475319/> "Determination of Complex Permeability of Silicon Steel for Use in High-Frequency Modeling of Power Transformers"
[2]: https://www.microwaves101.com/encyclopedias/magnetic-materials
[3]: https://www.mdpi.com/2079-9292/10/17/2167
[4]: https://www.sciencedirect.com/topics/materials-science/silicon-steel
[5]: https://www.nutsvolts.com/magazine/article/a_low_cost_rf_impedance_analyzer
[6]: https://kaizerpowerelectronics.dk/theory/wire-size-table/
[7]: steber-lms-bridge1.pdf
[8]: steber-lms-bridge2.pdf