---
layout: post
title: Data visualizations inside a web browser using ParaView Glance
---

Have you ever been in demand for a solution to visualize (complex) data in a web browser (like Google Chrome or Mozilla Firefox) so that you can embed it in a website (web page)? We know that [ParaView](https://www.paraview.org/) is a great tool for doing such a task offline in your local machine, but what if we can employ the power of ParaView to do the visualization online and inside a web browser? 

Although it seems an impossible task at first, let me tell you that this is a super straightforward process by taking advantage of [ParaView Glance](https://kitware.github.io/paraview-glance/index.html), one of the tools provided by the [ParaView Web](https://www.paraview.org/web/) project. ParaView Glance is lightweight version of ParaView developed using [VTK.js](https://kitware.github.io/vtk-js/index.html) as a web application to run inside a web browser. It can also be used as a tool to build custom viewers on the web. It seems to be very exciting, doesn't it? If you want to take a look at an example to see how it looks like, you may scroll down this page a little bit or refer to the [Research page]({% link research.md %}) and view the section in which I have provided a visualization of a degradation simulation as a glimpse into the kind of work I do during my PhD. 

For this post, I assume that we are going to host such visualization on an online website, like what I did here. I say this because running this locally on your machine becomes easier since you don't need to serve the visualization file using an online service (for which I have used GitHub). 

In order to make it up and running, you need to take 3 steps: 1) creating a special file (a scene) using ParaView, 2) upload this special file to a service or a public location, and 3) write a couple of lines of code into the source of any web page to embed ParaView Glance into it. Let's go through this step by step.

First, we need a scence file, which can be created easily using ParaView. A scene file is a sort of snapshot from the current state of ypur visualization that you have created using a set of pipelines in ParaView. After setting up your visualization (including camera settings) in ParaView, select *Export Scene...* from the *File* menu, select ".VTKJS files" from the *File of types* list, and save your file. This creates a VTK.js scene file for you. You may check the validity of the file by opening it inside the [online version of ParaView Glance](https://kitware.github.io/paraview-glance/app/) provided by Kitware.

After checking the validity of the file, we are ready to upload it to any public location. I used GitHub for this purpose since it makes it easier to keep track of various changes I want to make to the list of scene files I have. I have created [a repository](https://github.com/mbarzegary/datasets-and-scenes/) for this purpose, to which I can upload my scene files. If you put a file named X.vtkjs inside a repository named Y, you can access the file directly using this URL (assuming you have put it in the branch "main"): "https://raw.githubusercontent.com/\<your_username\>/Y/main/X.vtkjs". You may choose to upload it anywhere else like a public cloud or a private website.

Cool, so it's time for the final step. The code you need to add to the source of the page is a couple of lines of HTML to load the corresponding JavaScript library and provide it with the scene file. The code looks like this: 

```html
<script>
    var app = "https://kitware.github.io/paraview-glance/app";
    var datadir = "https://raw.githubusercontent.com/mbarzegary/datasets-and-scenes/main/";
    var file = "degrading_screw.vtkjs";

    document.write("<iframe src='" + app + "?name=" + file + "&url=" +datadir + file + "' id='iframe' width='1100' height='900'></iframe>");
</script>
```

What does it do? It first defines a variable to point to the web app location (can be either online or offline) as well as 2 variables to keep the directory in which the scene file is located and the name of the file. The latter is stored in 2 variables instead of 1 to allow providing the web app with a unique (instance) name, which is actually the name of the file in this case. The URL pointing to the scene file is then created by appending these 2 variables. After this, a special HTML element (an iframe) is created and the app is generated inside it. You need to change the second and the third variables accordingly to point to the path and name of your own files.

It's worth noting that as an alternative solution, you can choose to download the latest version of ParaView Glance, upload it to your web server, and point to this new location for loading the app (the `app` variable in our code snippet). Instead of doing this, I usually prefer to load the latest version of things online from the corresponding website directly, which is in this case the Kitware GitHub pages site (kitware.github.io). 

That's all we need to do. The result will be something like below. Try to play with it with your mouse buttons and see it in action.

<script>
    var app = "https://kitware.github.io/paraview-glance/app";
    var datadir = "https://raw.githubusercontent.com/mbarzegary/datasets-and-scenes/main/";
    var file = "degrading_screw.vtkjs";

    document.write("<iframe src='" + app + "?name=" + file + "&url=" +datadir + file + "' id='iframe' width='1100' height='900'></iframe>");
</script>