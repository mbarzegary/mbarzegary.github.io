---
layout: page
title: My PhD Project
tags: child
---

* **Title**: Mathematical and computational modeling of metallic biomaterials biodegradation

* **Public defense**: The recorded version of the public PhD defense, held on 15 June 2023, Leuven, Belgium, can be found here:\\
[![presentation](http://img.youtube.com/vi/SORPl5E_K9k/0.jpg){: style="max-width: 300px; height: auto;"}](https://www.youtube.com/watch?v=SORPl5E_K9k)

* **(Not so short) summary**: Degradable metallic materials are gaining popularity in a wide variety of applications. In the biomedical field, biocompatibility, biodegradability and positive impact on biological processes are the critical properties that nominate a metal as an applicable option. Taking this into account, magnesium (Mg), iron (Fe), and zinc (Zn) are usually considered as biodegradable metallic (bio)materials. Due to their mechanical properties, metallic biomaterials are appropriate candidates for load-bearing conditions in various bone healing and cardiovascular applications. Biodegradable metals meet the (often temporary) need for mechanical support while avoiding stress-shielding (in orthopedic applications) in the long term and omitting the need for revision surgery as required for permanent materials. Despite the advantages of using biodegradable metals in implant design, their fast degradation and uncontrolled ion release remain a challenge in practical applications. Beside experimental approaches to investigate the properties of biodegradable metallic implants and scaffolds, computational modeling of the biodegradation process and behavior can act as an efficient tool to design the next generation of medical devices and implants. A validated computational model of the degradation process can facilitate tuning of biodegradation properties and optimizing the design for specific applications.<br/><br/>
In this research, we have developed a mathematical and computational model to predict the biodegradation behavior of biodegradable metallic biomaterials, focusing on Mg. Our developed model captures the release of metallic ions, changes in pH, the formation of a protective film, the effect of different ions in the environment, and the effect of perfusion of the surrounding fluid, when applicable. This has been accomplished by deriving a system of time-dependent reaction-diffusion-convection partial differential equations from the underlying oxidation-reduction reactions and solving them using the finite element method. The level-set formalism was employed to track the biodegradation interface between the biomaterial and its surroundings.<br/><br/>
Tracking the moving front at the diffusion interface requires high numerical accuracy of the diffusive state variables. Improving the accuracy requires a refined computational mesh, leading to a more computation-intensive simulation. To overcome this challenge and yield interactable simulations in more feasible turnaround times, scalable parallelization techniques were implemented, making the model capable of being run on massively parallel systems to reduce the simulation time. Subsequently, the scaling behavior of the models was evaluated on hundreds to thousands of CPU cores in high-performance computing environments. Additionally, the core biodegradation model was coupled with fluid flow models to enable capturing the effect of hydrodynamics and perfusion conditions. Finally, the model was employed in a couple of multi-physics use-cases as the biodegradation compartment to demonstrate the ability of the model to be integrated in other modeling workflows in biomedical engineering.<br/><br/>
Taken together, this PhD work has developed a broad range of mathematical and computational tools in the field of degradable biomaterials, demonstrating the potential of integrating \textit{in silico} technologies in the design and optimization of novel biomaterial-based implants.



* **Glimpse into the results**: The following visualization is a typical output of simulations that I perform using the model I have developed (it's an interactive ParaView Glance session, so you can explore it using your mouse buttons and modify it using the panel on the left).

<script>
    var app = "https://kitware.github.io/paraview-glance/app";
    var datadir = "https://raw.githubusercontent.com/mbarzegary/datasets-and-scenes/main/";
    var file = "degrading_screw.vtkjs";

    document.write("<iframe src='" + app + "?name=" + file + "&url=" +datadir + file + "' id='iframe' width='1100' height='900'></iframe>");
</script>