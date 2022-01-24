---
layout: page
title: Research Portfolio
---

[My Research Interests](#interests)

[Research Projects I've Been Involved In](#projects)

[My PhD Project](#phd_project)

## <a name="interests"></a> My Research Interests

* Scientific Computing
* Computational Engineering
* GPU Programming and High-Performance Computing
* Machine Learning and Computational Intelligence
* Computational Biomechanics
* Computational Materials Science

## <a name="projects"></a> Research Projects I've Been Involved In

#### Computational Tissue Engineering

<!-- add atlas work -->

Mathematical modeling and numerical simulation of biodegradation behavior of metallic implants and medical devices\
*KU Leuven*, 2018--2021
<!-- Development of open-source software BioDeg for massively-parallel simulation of biodegradation and corrosion of metallic biomaterials\
*KU Leuven*, 2020--2021 -->

Mathematical modeling and numerical simulation of bone tissue growth process\
*KU Leuven*, 2019--2021

Contribution to the development of open-source software ASLI for creating TPMS-based functionally graded scaffolds and implants\
*KU Leuven*, 2020--2021

Numerical modeling of oxygen consumption and cell viability for pancreatic islets\
*KU Leuven & Maastricht University*, 2019--2021

Development of coupled models of topology optimization and metals corrosion for optimizing the shape of biodegradable medical devices\
*KU Leuven & Kyoto University*, 2021

#### Computational Fluid Dynamics

Numerical modeling of foam formation process using Lattice Boltzmann method and multiphase fluid simulation\
*Amirkabir University of Technology*, 2013--2017

Development of in-house CFD codes for simulating fluid flow and heat transfer in metal casting process\
*Amirkabir University of Technology*, 2008--2011

#### Computational Biomechanics

Contribution to the development of open-source software TFMLab for traction force microscopy calculations of cellular movements\
*KU Leuven* 2020

Implementation of Fluid-Structure Interactions models to simulate fluid dynamics of human body fluids\
*University of Tehran*, 2012--2014
<!-- Implementation of ANN models to investigate complex parameters of urology diseases\
*University of Tehran*, 2013--2014 -->


#### Computational Materials Science

Development of dendrite and microstructure growth models to simulate the solidification process of metals\
*Amirkabir University of Technology*, 2009--2011
<!-- CFD aspects -->
<!-- Development of coupling simulation software packages to link multiphysics CFD models and AI\
*Amirkabir University of Technology*, 2010--2011 -->
<!-- Implementation of ANN models to investigate relations between porous media parameters and permeability\
*Amirkabir University of Technology*, 2010--2011 -->

#### Machine Learning and Computational Intelligence

Development of physics-informed neural network models to solve governing equations of tissue engineering processes (cell growth and oxygen consumption)\
*KU Leuven* 2020--2021

Development of Privacy-Preserving Deep Learning models using Federated Learning and Differential Privacy for healthcare IoT systems\
*KU Leuven & Duke University*, 2019--2020

Implementation of Machine Learning models for signal processing and anomaly detection of EEG and ECG signals\
*KU Leuven & Imec*, 2018--2019

## <a name="phd_project"></a> My PhD Project

* **Title**: Computational Multiscale Modeling of Biodegradation Behavior of Personalized Printed Implants

* **Summary**: Beside mechanical functionality and stability, biodegradation plays an important role in implant design and optimization. In this project, we will develop a 3D mathematical model of biodegradation of printed implants, which provides a framework for assessment of degradation and corrosion of implants in the bioreactor (in-vitro) and body (in-vivo) environments. The mathematical models will cover a variety of materials. In addition, chemical and biological interaction of the implant with the bioreactor environment will be modeled. The mathematical models will be simulated using proper numerical schemes. The developed code will be linked with structure optimization models to obtain more accurate predictions of the implant behavior over time.

* **Glimpse into the results**: The following visualization is a typical output of simulations that I perform using the model I have developed (it's an interactive ParaView Glance session, so you can explore it using your mouse buttons and modify it using the panel on the left).

<script>
    var app = "https://kitware.github.io/paraview-glance/app";
    var datadir = "https://raw.githubusercontent.com/mbarzegary/datasets-and-scenes/main/";
    var file = "degrading_screw.vtkjs";

    document.write("<iframe src='" + app + "?name=" + file + "&url=" +datadir + file + "' id='iframe' width='1100' height='900'></iframe>");
</script>
