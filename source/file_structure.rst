Experiment structure
====================

Before we press on any further I want to take quick detour to the demo experiments. The point is to let you see that the structure we have just talked about (front-end vs back-end, and so on) is manifest in the file structure of each indiviual experiment. So let's start by navigating to one of the demos, Bartlett1932 which can be found in `Dallinger/demos/dlgr/demos/bartlett1932`. We'll talk through some of the files and folders now.

The Front-end
-------------

One of the units in the diagram we saw in the previous chapter was a chunk labelled "front-end", this is everything that runs on the participants computer. In the file structure of the experiments this code is stored in files found within the folders called `static` and `templates`. The `static` folder is subdivided into `scripts`, `images`, `css` and so on. We don't need to worry too much about this stuff yet, but these are various resources that your experiment will call on at run time. Probably the most important are the files within the script folder as these determine your experiments behavior (what happens when buttons are clicked and so on). Meanwhile the `templates` folder contains html pages that determine the basic structure of each page (titles, text, button placement and so on). Its worth noting that this structure is neither necessay, nor unique to Dallinger. The division between static and templates is standard practice in web design, but users are free to ignore this and use whatever directory structure they feel like for any individual experiment. Nonetheless, this structure is standard practice for a reason - it works well for more purposes - and so unles you have very specific requirements you will probably want to stick with it.

The Back-end
------------

The Experiment
^^^^^^^^^^^^^^

In the main experiment directory you'll see a file called `experiment.py`. As you might have guessed this is where the code that creates the experiment goes. You can open it now and have a quick look, but it probably won't make sense until later (we'll come back to it in a later chapter anyway). This isn't the only file that serves this purpose though. There's also `models.py` which contains code descriptions of customs types of objects (Nodes, Vectors etc) the experiment needs, as well as `config.txt` that contains parameters that control various experiment properties. We'll come to all these files in due course.

The Server Routes
^^^^^^^^^^^^^^^^^

Server routes were a part of the diagram on the previous page, but they are missing here. This is because the server routes are provided by core Dallinger and so they do not need to be a part of each individual experiment. Nonetheless you can create custom server routes if you really want to, but this is an advanced feature that we'll come to much later.

The Database
^^^^^^^^^^^^

The database appears to be missing too, but again this is because the database structure is determined by the core dallinger code as opposed to being an experiment specific thing.