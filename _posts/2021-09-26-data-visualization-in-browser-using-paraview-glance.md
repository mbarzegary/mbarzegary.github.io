---
layout: post
title: Data visualizations inside a web browser using ParaView Glance
---

Have you ever been in demand for a solution to visualize (complex) data in a web browser (like Google Chrome or Mozilla Firefox) so that you can embed it in a website (web page)? We know that [ParaView](https://www.paraview.org/) is a great tool for doing such a task offline in your local machine, but what if we can employ the power of ParaView to do the visualization online and inside a web browser? 

Although it seems an impossible task at first, let me tell you that this is a super streightforward process by taking advantage of [ParaView Glance](https://kitware.github.io/paraview-glance/index.html), one of the tools provided by the [ParaView Web](https://www.paraview.org/web/) project. ParaView Glance is lightweight version of ParaView developed as a web application to run inside a web browser. It can also be used as a tool to build custom viewers on web. It is very exciting, isn't it? If you want to take a look at an example to see how it looks like, you may scroll down this page a little bit or refer to the [Research page]({% link research.md %}) and view the section in which I have provided a visualization of a degradation simulation as a glimpse into the kind of work I do during my PhD. 

For this post, I assume that we are going to host such a visulization in an online website, like what I did in this website. I say this because running this locally on the web browser installed on your machine becomes easier since you don't need to serve the visualizaion file using an online service (I have used GitHub for this). 

In order to make it up and running, you need to do 3 steps: 1) creating a special file (a scene) using ParaView, 2) a service or a public location to upload this special file to, and 3) write a couple of lines into the source of any web page to embed ParaView Glance into it. Let's go through this step by step.

The code you need to add to the source of the page is a couple of lines of HTML to load the corresponding JavaScript library and provide it with the scene file. The code looks like this: 

```html
<script>
    var app = "https://kitware.github.io/paraview-glance/app";
    var datadir = "https://raw.githubusercontent.com/mbarzegary/datasets-and-scenes/main/";
    var file = "degrading_screw.vtkjs";

    document.write("<iframe src='" + app + "?name=" + file + "&url=" +datadir + file + "' id='iframe' width='1100' height='900'></iframe>");
</script>
```

What does it do? It first defines a variable to point to the web app location (can be online or offline) as well as 2 variables to keep the directory in which the scene file is located and the name of the file. The latter is stored in 2 variables instead of 1 to allow us to provide the web app with a unique (instance) name, which is actually the name of the file in this case. The URL pointing to the scene file is then created by appending these 2 variables. After this, a special HTML element (an iframe) is created and the app is generated inside it. You need to change the second and the third variables accordingly to point to the path and name of your own files.

It's worth noting that as an alternative solution, you can choose to download the latest version of ParaView Glance and link the downloaded files with this. Instead of doing this, I usually prefer to load the latest version of things from their website directly, which is in this case the Kitware GitHub pages site (kitware.github.io). 

The result will be something like this:

<script>
    var app = "https://kitware.github.io/paraview-glance/app";
    var datadir = "https://raw.githubusercontent.com/mbarzegary/datasets-and-scenes/main/";
    var file = "degrading_screw.vtkjs";

    document.write("<iframe src='" + app + "?name=" + file + "&url=" +datadir + file + "' id='iframe' width='1100' height='900'></iframe>");
</script>