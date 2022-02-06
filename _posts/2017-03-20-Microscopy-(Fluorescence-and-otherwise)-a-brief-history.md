---
layout: post
date: 2017-03-20
title: Microscopy (Fluorescence and otherwise)
image: assets/images/fluorescencemicroscopy.jpg
categories: [ science, history ]
tags: [green, brown]
---
The development of microscopy has an exciting past & has been the subject of a few Nobel prize awards. Some notable contributes include..


* 1953 Physics Nobel awarded to Frits Zernike for his work on phase contrast microscopes;
* 1986 Physics Nobel awarded to Ernst Ruska for the electron microscope and Gerd Binnig with Heinrich Rohrer for the scanning tunneling microscope;
* 2014 Chemistry Nobel awarded to Eric Betzig, Stefan W. Hell and William E. Moerner for their development of super-resolved fluorescence microscopy techniques.
* Richard Zsigmondy – [details](https://www.nobelprize.org/educational/physics/microscopes/timeline/ )


We’ll skip past the development of lens grinding, past Anton van Leeuwenhoek and his peering into the previously unknown world of single-celled organisms (using globules of glass, no less: [see here](http://www.history-of-the-microscope.org/anton-van-leeuwenhoek-microscope-history.php)) and straight into fluorescence.


Attaching fluorescent molecules is a rather ingenious (if now obvious) innovation to increase the ability to see one’s needle when it’s stuck in a haystack. Biological experiments are typically just such a problem: observe one tiny process when it’s surrounded by a multitude. Fluorescence microscopy helps with this problem: if we can label our needle, we can be specific in finding it through all the hay and we can watch what it does.


##### How do we label our object of interest?


Chromophores are your typical dyes. What our eyes pick up - the brilliant hues of plants, cloth and foods are for a large part due to chromophores. These are molecules which absorb some wavelengths of light, leaving the unabsorbed wavelengths to bounce back (or through) and be seen. (What our eyes see is actually a product of many things: incident light; extent of reflection, scattering and absorption; translucency etc). Chromophores’ come in many forms, from chlorophyll to haemoglobin to the pH indicator you remember using during High School Chem. Their property as a dye comes from the energy difference between the ground state and excited state molecular orbitals – they can absorb photons in a particular energy range and cause electrons to be shift from one state to the other. The wavelengths of light that aren’t absorbed are the ones we see.


Fluorophores by contrast re-emit light upon excitation. Meaning, photons of a particular wavelength are absorbed and then photons of a lower energy (longer wavelength) are remitted. This makes them particularly useful as markers for use in fluorescent imaging and spectroscopy. This also makes them an incredible beaut as a form of luminescence and quite abundant in nature. Their use in labs ranges from standalone-use, to being a motif in proteins, peptides, compounds and so on. Green fluorescent protein (GFP) (derived originally from jellyfish) is one popular example.


##### How do we image what we’ve labelled?


Imagining we’ve attached fluorescent markers to our subject of interest, we can start imaging it. A simple explanation of how this works is in the schematic below. We use some light source (say, a xenon or mercury lamp) to excite our labelled subject. It re-emits photons (shifted to a longer wavelength due to Stokes shift) and we pick up the resultant emission. Now there are different imaging possibilities: we could just light up our whole sample with a wide-field of view and see what results, picking up the resulting image from a camera. This method is effective if we’re looking at very thin specimens and when we aren’t trying to resolve detailed structures sitting close together. It doesn’t however give us a good image within thick samples, or if we’re trying to gauge a structure in the z-direction. This is because (as we can see in picture (c)), the light excites fluorophores along the entire beam of light, resulting in scattered signals above the point we want to resolve.


##### More detail: laser scanning systems


A method that bypasses this is to focus our light to a small point and raster over the object we want to see. This requires the use of an excitation source such as a laser; scanning mirrors for sequential scanning; and photomultiplier tubes (PMTs) for detection. While it’s possible to view specimens by the naked eye using widefield systems, point source excitation requires more finesse to detect emitted photons.


##### Let’s take laser-scanning confocal microscopy as our example system.

As before an excitation source (laser) enters the system, is reflected by the dichroic mirror and excites the sample. As the light focuses on one point using scanning mirrors, emitted fluorescence light passes back through the objective, through the dichroic and to the PMTs. PMTs act as highly sensitive detectors which multiply the current produced by incident light and so, they allow sparse photon detection. To avoid the similar problem of unwanted emission along the path to our current point of imaging, confocal microscopy uses a pinhole aperture. The light from just below and just above our desired focal plane is blocked. By rastering our desired x,y-plane and shifting focal (z) plane we can assemble a 3D image of our specimen.


##### What are the issues with confocal systems?


Photobleaching, where the dye or fluorophore are damaged so that they cannot fluoresce is one typical problem. By excluding a large proportion of photons via the pinhole, we are exciting (yet not using) many fluorophores. Consequently the signal received might be low for one particular point, yet we’re damaging fluorophores in the way to that point.


Trying to improve this prompts the researcher to try some of the following things:

-	Experiment with PMT settings (increase gain or offset; potentially get a much noisier image)
-	Increase the averaging of the image (averaging images: just what you think)
-	Increasing the dwell time (laser stays on each point longer)
-	Increasing the pinhole size (Smaller pinhole = greater resolution but less photons. Larger pinhole = more out-of-focus light)
-	Ensuring laser wavelength, fluorophores, filter cubes and blocking filters are optimal


Unfortunately assuming our system is set up expertly, photobleaching is still one consequence of imaging if we want to maintain a good signal-to-noise ratio and decent resolution. Nonetheless, this system works brilliantly if you want a quick 3D image. The experimenter only really gets into trouble trying to do one of three things: extended live cell imaging such as time-lapse microscopy; imaging very deep into a sample; or imaging fast dynamics. This latter issue arises when imaging things like calcium dynamics, as the standard galvanometer scanners directing the laser simply aren’t fast enough.


##### Improving speed


Spinning disc confocal microscopy is a work-around to the image acquisition rate problem. Modifying our confocal microscope somewhat, we can use multiple points instead of just one to scan the sample. This is done by means of spinning discs with holes replacing the galvanometer scanners. The emitted light is then descanned through the pinhole as with regular microscopy, then passed through a dichroic and collected via a CCD camera. By virtue of multiple pinholes, dwell time is longer whilst lower excitation energy means less photobleaching, allowing for longer live cell imaging. One downfall here is resolution loss due to photon cross-interaction from multiple imaging points.


##### What about this “multi-photon” approach I keep hearing about?


Two photon microscopy takes on one of the major issues limiting imaging with confocal microscopes: depth penetration. Here, excitation only occurs when two photons are near-simultaneously (within about 0.5fs) absorbed by the fluorescent molecule. This requires a high density of photons which is achieved using lasers which focus short, high-powered pulses onto the diffraction-limited spot in the focal plane. For nonlinear processes, the probability of two photons arriving on the same molecule simultaneously is extremely low. Consequently, a higher laser intensity is required and the excitation light has to be highly concentrated in space and time. This is done using high powered, longer wavelength, mode-locked pulsed lasers (typically Ti:Sapphire). Longer wavelength light in the red range has the benefit of scattering less and has a lower likelihood of damaging tissue. For typically used fluorescent markers, multiphoton absorption occurs in the infrared range of 700-1000nm, with emission happening in the visible range.


##### What is this “scattering”, that becomes such a problem?


As we mention scattering a fair amount, it is useful to recap what we mean. For biological tissue imaged in the infrared range, scattering plays a much larger effect than absorption. Scattering is the deflection of a photon from its initial vector which, if the energy is unchanged is termed elastic scattering. This elastic scattering depends on refractive index inhomogeneities, which are strong in tissue due to the heterogenous mixture of molecules and structures with varying polarisabilities. The strength of the scattering is defined by the “mean free path” ls, which is the average distance between scattering events. The likelihood and angular distribution of scattering depends on the variation in refractive index, object size and wavelength.


For very small objects such as atoms or molecules in a gas, scattering is almost isotropic (same magnitude independent of direction) and strongly dependent on the wavelength i.e. Rayleigh scattering. For large objects that are comparable in size to the wavelength (as occurs in cells) scattering can be quantified by the anisotropy parameter (g) or the transport mean free path lt, where lt = (ls / (1-g).


Scattering is influenced by many variables. Within extracted tissue grey brain matter the scattering values are around 50-100µm using 630nm excitation, but at 800nm and in vivo this value goes to around 200µm. In general the anisotropy parameter for brain matter is quite high around 0.9, but of course there’s variability. Both ls (mean free path) and g (the anisotropy parameter) don’t depend solely on the tissue type, but also can change substantially with age (the brain becomes more “solid”) and after removing the tissue from the animal. Surface scattering can become increasingly important if the beam crosses mediums with substantially different refractive indices, such as in vivo imaging and through the skull / coverslips.


##### Ahuh, so multi-photon microscopy minimises scattering..


For nonlinear optical microscopy such as two photon microscopy, only ballistic (non-scattered) photons contribute to generating a signal in the focal volume. The ballistic power follows a Lambert-Beer-like exponential decline with the imaging depth. The equation below expresses the relationship between the ballistic power P to the power at the surface (Po), depth (z) and length constant (ls) as described above.


> P-ball = Po exp(-z/ls)


Due to the quadratic intensity-dependence in two photon microscopy, the fluorescence we read declines as


> F(2PE) α (exp(-z/ls))^2 = exp(-2z/ls)


The need for an exponentially increasing laser power to maintain the ballistic intensity at the focus (where we are imaging) explains the need for the high-powered, pulsed lasers. A major bonus is that we can use (unlike confocal) all the emitted light captured since we can be pretty certain of our emission point source plane. On a two photon microscope we can also move the PMTs closer to the specimen (we don’t need descanning, similar to laser scanning confocal) and so collect more photons after they’ve passed through the dichroic.


##### Two photon(+) microscopy still has its draw-backs


One of the limitations still experienced by two photon microscopy is that it is still a diffraction limited system. What this means is that with the perfect set-up of fluorescence indicator, lens, laser wavelength and so forth, the system still has the fundamental theoretical limit in distinguishing two points. For two-photon this falls around the 500-700nm range in the z (axial) plane. In the x,z (lateral) plane this resolution can approach 200nm with a powerful, well-set up system. There are some factors to consider in this limitation (such as diffraction) but in general scientists are pretty pleased with the resolution given by this.
