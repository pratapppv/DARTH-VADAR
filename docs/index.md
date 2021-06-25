# Welcome to Darth VADAR- a simple 2.4GHz ISM band FMCW RADAR

Sometime back on early 2019, I took the plunge into RF circuits and the first RF design I've always wanted to do was a simple RADAR. [Hforsten](https://hforsten.com) was a big inspiration for this and he frankly made it seem simple enough.
This post is supposed to be a mix of a tutorial and documentation for whatever RADAR me and a couple of my friends built.

As I wasn't too picky about the range or the resolution, I decided to choose to operate the RADAR at a center frequency of $$f_0 = 2.45GHz$$ and a maximum bandwidth of $$\Delta f = 100MHz$$ setting the RADAR to work completely within the allocated ISM band. As this band is also where WiFi and bluetooth reside, finding chips which were'nt exorbidantly priced was easy. Another advantage of choosing this frequency was that the dielectric loss of FR4 at this frequency was not too high and could be easily used. This meant that no exotic substrates would be required and any standard PCB fab house could make the PCB's.

## Obligatory introduction to FMCW RADAR
An FMCW RADAR is a specific type of RADAR which is most commonly used to determine the distance and velocity of a target. Due to the high propagation velocity of the electromagnetic waves used, the FMCW RADAR is designed to convert the distance into frequency. The measurement of the velocity of the object varies. For the rest of this introduction only the measurement of distance will be considered.

Consider a single object with velocity $$v = 0$$ hence static at a distance $$d$$ from the RADAR. To detect the object, let an EM wave pulse be transmitted to irradiate the object. Due to the finite propagation velocity of an EM wave in free space, the time taken for incidence on the target is $\frac{d}{c}$ where $c$ is the speed of light (medium is considered to be air). For a monostatic RADAR with the transmitter and receiver located at the same position, the cumulative round trip time delay is  $$\frac{2d}{c}$$. As this delay is in the order of nanoseconds, accurate and precise measurement is not feasible. Therefore, let the transmitted pulse be frequency modulated by a sawtooth waveform with a timeperiod $$T_{ch}$$. Therefore the frequency can be written as $ f(t) = f_0 + \frac{BW t}{T_{ch}} $. Now a frequency modulated signal can be written as 

\begin{equation}
   Tx(t) = A_{Tx}cos(\int 2\pi f(t)dt)
\end{equation}

On integrating and substituting the expression, we get $ Tx(t) = A_{Tx}cos(2\pi f_0t + \pi \frac{BWt^2}{T_{ch}}) $. As multiple chirps are transmitted, the $ m^{th} $ chirp will have the following equation.

\begin{equation}
  Tx(t) = A_{Tx}cos(2\pi f_0t + \pi (\frac{BW(t - mT_{ch})^2}{T_{ch}})^2)
\end{equation}

In the above equations,
$ f_0 $ is the minimum frequency transmitted
$ BW $ is the bandwidth of the chirp
$ A_{Tx} $ is the amplitude of the transmitted signal

Now once transmitted, the radiated chirp will reflect off a target and will be received by the receiver section of the RADAR. The round trip propagation delay is given by $ t_d $. The received signal can be mathematically expressed as an attenuated and delayed version of the transmitted chirp. Therefore the received signal can be expressed as follows.

\begin{equation}
   Rx(t) = \alpha A_{Tx}cos(2\pi f_0(t - t_d) + 2\pi \frac{BW(t - t_d - mT_{ch})^2}{T_{ch}})
\end{equation}

Now to determine the round trip time delay, the transmitted and received equations are multiplied(mixed). This operation gives two frequencies of interest, a sum component and another the difference component. The difference component is one of interest and can easily be isolated by lowpass filtering the spectrum obtained from the post mixing operation. On taking the fourier transform of this difference signal, the spectral component is given as follows

\begin{equation}
      M(j\omega) = 2\pi\alpha A_{Tx}^2 (\delta(\omega + \frac{t_d BW}{T_{ch}}) + \delta(\omega - \frac{t_d BW}{T_{ch}}))
\end{equation}

Clearely, the measured frequency denoted by $ f_{diff} $ is calculated by the following equation

\begin{equation}
      f_{diff} = \frac{t_d BW}{T_{ch}}
\end{equation}

This can further be re-written to express $ f_{diff} $ as a function of the one way distance between the RADAR and the target denoted by $ d $. On solving for $ d $, we get the following equation

\begin{equation}
   d = \frac{cf_{diff}T_{ch}}{2BW}
\end{equation}

Where $ c $ is the speed of light (we assume the propagating medium to be air)

From the above described equations, the block-diagram of a FMCW RADAR is pretty easily arrived at and is shown below  

![block-diagram](/img/BD)

## Design of RF Front End
The block diagram is rather easily and directly translated into it's circuit level equivalent. To simplify matters, fully integrated components were used, condensing each block into a single IC and a handfull of passives. The chirp is generated off board and is not discussed. For the remainder of the post, focus would be towards some of the calculations performed to determine the range of the RADAR and the PCB layout considerations.

The Schematic for the RADAR is attached [here](./docs/schematic.pdf)

### RF Chain explanation
The RF chain begins with the Voltage Controlled Oscillator(VCO) chosen to be the [MAX2750EUA+](https://datasheets.maximintegrated.com/en/ds/MAX2750-MAX2752.pdf) which requires essentially no external components and provides an AC coupled output. As the output power of the VCO is a paltry $$-3dbBm$$, a power amplifier follows next. The power amplifier chosen is a [GVA-60+](https://www.minicircuits.com/pdfs/GVA-60+.pdf) from Mini-Circuits. This PA is rated to have a power gain of $$+20dB$$ wich should boost the signal level to $$17dBm$$. The PA also requires a teensy bit more effort to get up and running than the VCO as it requires a couple of RF Chokes and DC blocking capacitors. The output of the PA is split into two equal paths via a lumped element Wilkinson power divider( ) from Anaren. Normally, a distributed element Wilkinson divider is used, but in the current band of interest, a lumped element one is far more compact than a distributed element one. This divider from Ananren requires an external $$100\Omega$$ resister. A 0.1% tolerence $$100\Omega$$ high frequency RF resistor was specifically chosen for this which was probably a bit overkill, but I didnot want to take any chances. As the power divider chosen splits the power into two equal half, each port delivers a power of $$14dBm$$ to the load attached to it. As the transmitting antenna is connected to one port of the power divider, the net power delivered to the antenna is $$14dBm$$ which will become important later on while determining the range of the RADAR. The other port of the power divider is connected to the mixer, delivering $$14dBm$$ of power to it which is the main driving signal for the double balanced passive mixer. The mixer used is the [MDB-73H+](https://www.minicircuits.com/pdfs/MDB-73H+.pdf) so selected as it requires a typical driving signal power of $$15dBm$$.
The other input to the mixer is the signal reflected by the target which is received by the receiving antenna. As the signal power of this received signal is pretty low, it is amplified by a Low Noise Amplifier, abbreviated as LNA. The LNA used is the [BGA622H6820XTSA1](https://www.mouser.in/datasheet/2/196/infineon_infns12498-1-1735744.pdf) from Infineon semiconductor. Inorder to improve the insertion loss of the LNA, an LC matching network is placed between the receiving antenna's output and the LNA's input which is as described in the datasheet.

The output spectrum of the mixer contains two different frequencies which is the sum and difference of the two inputs. The sum component lies in the range of $$4.8GHz$$ to $$5GHz$$ which can be filtered out. The difference component lies in the $$0Hz$$ to $$100MHz$$ range. This baseband signal is sampled by an ADC and the fourier transform is computed by an external FPGA.
