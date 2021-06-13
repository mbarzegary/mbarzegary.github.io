---
layout: post
title: Separating different volume mesh blocks (regions) using FreeFEM
---

Lots of mesh generation routines may result in a multi-region mesh, out of which you want to extract a certain region (block). These kinds of mesh files are commonly found in the output of code-based mesh generation processes, i.e you generate a mesh using a mesh manipulation or computational geometry library such as [Mmg](https://www.mmgtools.org/) (used to generate the mesh below) or [CGAL](https://www.cgal.org/). 

<img src="/public/blog/separate-both.jpg" alt="color photo ftl" width="100%" height="auto" />

As can be seen in the figure (visualized using [GMSH](https://gmsh.info/)), in addition to several surfaces and curves, there are 2 volume sets in the generated mesh. We are interested to extract different regions into separate mesh files while preserving the surface sets. One possible solution is to write a code using Mmg that does this, but this requires dealing with the Mmg data structures. A much simpler approach is taking advantage of easy-to-use meshing interface in [FreeFEM](https://freefem.org/). 

The mesh above belongs to a tissue engineering scaffold in a mixed gyroid shape. These scaffolds are widely used for culturing cells. It consists of a volume block for the scaffold itself and a block for the void space around it (in which the cells start to proliferate). The goal is to separate these two parts. The code for doing that will be as simple as this:

```cpp
load "msh3"
include "getARGV.idp"

string fileName = getARGV("-mesh", "mesh_file_name"); 

mesh3 Mesh = readmesh3(fileName+".mesh");

mesh3 MeshScaffold = trunc(Mesh, region==3);
mesh3 MeshVoid = trunc(Mesh, region==2);

savemesh(MeshScaffold, fileName + "-scaffold.mesh");
savemesh(MeshVoid, fileName + "-void.mesh");
```

The code assumes that the mesh is in the medit format (.mesh extension). In other cases, you can use GMSH to convert it to medit. By running the code, the mesh is divided into the scaffold part (region number 3) and the void space (region number 2). The resulted mesh for region #3 will be this:

<img src="/public/blog/separate-scaffold.jpg" alt="color photo ftl" width="100%" height="auto" />

