---
layout: post
title: Generate volume mesh from a surface mesh using GMSH
---

Creating a volume mesh out of a surface mesh is a quite common task in computational sciences, especially when you deal with a sort of image segmentation resulting in a surface mesh (usually in STL format). The surface mesh is suitable for printing or demonstration purposes, but if one wants to go for a computational analysis, like a finite element analysis (FEA) for structural and heat transfer simulations or a computational fluid dynamics (CFD) simulation, then a volume mesh is required.

[GMSH](https://gmsh.info/) is a powerful software to make such volume mesh. We can simply open a mesh in it (`File -> Open`), define a volume (`Physical groups -> Add -> Volume`), and then generate a mesh on it (`Mesh -> 3D`). We may also need to adjust the global mesh size before meshing the volume (`Tools -> Options -> Mesh -> General -> Element size factor`). 

But, there will be a big problem here if the surface mesh has a complex morphology, leading to a long waiting time for each of the above steps to complete on the GUI. The solution is to take advantage of the GMSH scripting language. Here is a very simple code to accomplish this:

```
Merge 'mesh_file.stl';
Surface Loop(1) = {1};
Volume(1) = {1};
```

We should save this in a `.geo` file (let's say `meshing.geo`) and then run this command to generate the volume mesh (which will be saved as `output.mesh`):

```bash
gmsh meshing.geo -3 -o output.mesh
```
 The `-3` flag denotes that it's a 3D mesh.