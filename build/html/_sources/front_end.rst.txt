Front end files
===============

	Note: before proceeding past this point it is recommended that you learn a little bit about HTML, css and javascript. Enough to know the basics of using each language, and what they are used for in basic web design. If you don't have this knowledge I strongly suggest finding some online tutorials to give you a quick introduction.

Back in the early days of Dallinger, Dallinger itself was just a back end with routes. The front-end, including how participants could send requests to routes, was left entirely up to the experimenter to design and create. This was for a couple of reasons. A pragmatic one was that we were sufficiently busy building the back-end that we didn't have time to think about front-end support. But we were also interested in keeping Dallinger as open as possible and didn't want to inadvertently place any constraints on possible front ends that users might want to build. A final reason, that I think was more important to me than to most of the other devs, was that I didn't want to hide any of the complexity of Dallinger. In order to create a working front end from scratch you need a good knowledge of Dallinger, and if we gave users too much help they might try to get cracking without a firm grounding in how Dallinger works. This would be ok in the short term, but my concern was that it could lead to users making errors and underestimating the range of experimental designs that Dallinger supports.

To a large extent this vision of Dallinger (powerful, but with a nasty learning curve) remains accurate (hence why I'm writing docs like these). For instance, things like drag-and-drop UI builders are nowhere to be found (yet, at least). Nonetheless, Dallinger does provide some front end support. This takes the form of a handful of html files and a javascript library. Collectively these can make the experimenter's job of building a new experiment much easier. Moreover, they don't reduce Dallinger's scope becuse they are all entirely optional. A user is entirely free to ignore them and genuinely build a front end from scratch. But after creating several Dallinger experiments we noticed a common pattern of shared code that it just make sense to make available across all experiments. We'll go through the javascript library in the next chapter, but for now we'll do a quick tour of the other files.

Dallinger/frontend
------------------

To see all the front end files that are made available to experiments you should navigate inside `Dallinger/dallinger/frontend`. When you run an experiment, either locally with ``dallinger debug`` or live with ``dallinger deploy``, the contents of this folder are copied in to your experiment directory. So they're there whether or not you want them. That said if your experiment contains any files with the same names as the shared Dallinger files, your versions will "win" (but a warning that you are conflicting with some of the shared files will be printed in the terminal just so you know its happening).

Inside `frontend` are two more folders: `static` and `templates`. This is exactly the same structure we see inside most experiments and that we covered way back in the section on Experiment structure. The `static` directory contains four things:

1. A folder called `scripts` which contains the dallinger javascript library as well as some other libraries required by the dallinger library.
2. A folder called `css` this contains the dallinger.css style sheet that can give your web pages a basic, but decent look.
3. A folder called `images` that contains nothing other than the icon that will appear on the tab in users browsers.
4. A file called `robots.txt`. This is provided to any search engine bots that happen to stumble in to your experiment as it runs.

The `templates` folder contains a series of html files (and a bunch more in a folder called `base`) that can act as templates for your later web pages, we'll go through a few of these now.

Layout.html
-----------

Let's start by opening the layout.html page. Upon inspection you might be a little disappointed as all it contains is the following line:
::

	{% extends "base/layout.html" %}

What this line means is that this file inherits all the contents of the file `base/layout.html`. It's a form of inheritance. Just like how the ``Agent`` class in the back end extends the ``Node`` class with the statement ``class Agent(Node)`` and thereby gets access to all its functionality.

So if we want to understand what `layout.html` does we'll need to open `base/layout.html`. If you do that now you'll see a bunch of html. This provides a very basic structure to a page. For instance it has a title containing the word "Experiment":
::

	<title>{% block title %}Experiment{% endblock %}</title>

The code in curly brackets is a way of labelling sections (a.k.a. "blocks") of code. So, in plain English, this code means "This page has a title that by default contains the word Experiment, and the title can be referred to with the word 'title'". Setting up blocks like this allows other html pages that extent this one to edit certain bits of it very easily. For instance, if we changed `layout.html` (the empty one in templates) so it looked like this, it would create pages that looked just like the base layout page, but with a title of "Amazing Experiment":
::

	{% extends "base/layout.html" %}
	{% block title %}Amazing Experiment{% endblock %}

`base/layout.html` also imports the Dallinger css files so, as long as you extend this file (or another file that extends it already, like `layout.html`) you won't need to import them yourself:
::

    {% block replace_stylesheets %}
        <link rel="stylesheet" href="/static/css/bootstrap.min.css" type="text/css">
        <link rel="stylesheet" href="/static/css/dallinger.css" type="text/css">
        {% block stylesheets %}
        {% endblock %}
    {% endblock %}

Notice it creates two more blocks (`replace_stylesheets` and `stylesheets`) and you can overwrite these in your own html pages to either list *more* css files to import (`stylesheets`) or list css files to import *instead* of the core dallinger ones (`replace_stylesheets`).

This page also imports the various javascript libraries that dallinger typically needs to function:
::

    {% block libs %}
        <script src="https://code.jquery.com/jquery-2.2.4.min.js" integrity="sha256-BbhdlvQf/xTY9gja0Dq3HiwQF8LaCRTXxZKRutelT44=" crossorigin="anonymous"></script>
        <script src="/static/scripts/reqwest.min.js" type="text/javascript"> </script>
        <script src="/static/scripts/reconnecting-websocket.js"></script>
        <script src="/static/scripts/spin.min.js" type="text/javascript"></script>
        <script src="/static/scripts/store+json2.min.js" type="text/javascript"></script>
        <script src="https://cdnjs.cloudflare.com/ajax/libs/fingerprintjs2/1.5.1/fingerprint2.min.js" type="text/javascript"></script>
        <script src="/static/scripts/dallinger2.js" type="text/javascript"> </script>
    {% endblock %}

Again this is for user convenience: there is a good chance you would want to import these javascript libraries and so its often easier just to have your html pages extend `layout.html` that have to list them all yourself. If you look at the src value of the libraries you can see it points inside `/static/scripts/` and if you go there now you will see all the javascript libraries as individual files.

The last bit of the file is a block for scripts that by default just checks for ad blockers and warns users that they can interfere with Dallinger experiments.

The advert
----------

Let's look now at `base/ad.html`. This is the page that potential participants see when they are browsing recruitment services like mechanical turk. The first thing to notice is that it extends `layout.html` which means it acquires all the contents of `base/layout.html`. Which isn't much, but it does mean that this file doesn't need to import the various css and javascript files handled by `layout.html`.

The file also includes some css to adjust appearance:
::

	{% block stylesheets %}
	    <style>
	        body {
	            padding:0px;
	            margin: 0px;
	            background-color: white;
	            color: black;
	            font-weight: 300;
	            font-size: 13pt;
	        }
	        #adlogo {
	            float: right;
	            width: 140px;
	            padding: 2px;
	            border: none;
	        }
	        #container-not-an-ad {
	            position: absolute;
	            top: 0px; /* Header Height */
	            bottom: 0px; /* Footer Height */
	            left: 0px;
	            right: 0px;
	            padding: 100px;
	            padding-top: 5%;
	            border: 18px solid #f3f3f3;
	            background: white;
	        }
	    </style>
	{% endblock %}

It also modifies the script block to include the following function:
::

	{% block scripts %}
	<script>
	    function openwindow(event) {
	        popup = window.open('{{ server_location }}/consent?recruiter={{ recruiter }}&hit_id={{ hitid }}&assignment_id={{ assignmentid }}&worker_id={{ workerid }}&workerId={{ workerid }}&mode={{ mode }}','Popup','toolbar=no,location=no,status=no,menubar=no,scrollbars=yes,resizable=no,width='+1024+',height='+768+'');
	        event.target.setAttribute("disabled", "");
	    }

	    document.getElementById("begin-button").onclick = openwindow;
	</script>
	{% endblock %}

This is a little hard to follow, but it defines a new function called `openwindow` and then assigns it to the page element with the id `begin-button`. We'll see below that the `begin-button` is a button with the word "Begin" written on it. Morevoer, if you look at the `openwindow` function you can see it directs the brower to the `/consent` route, so if a participant clicks "Begin" they get sent to our consent page.

This code is actually much uglier than what you will mostly be dealing with, and this is because the advert is embedded inside an iframe within the recruiting service's own webpage and for security reasons they block the importing of javascript libraries so we have to write out code long hand. Once participants get to the consent page the experiment will be open in a tab all of its own and so this constraint will be lifted.

The rest of the html page actually determines what participants see. If they don't yet have an `assignment id` (i.e. they have not yet accepted the HIT on MTurk) they see an advert:
::

	{% if assignmentid == "ASSIGNMENT_ID_NOT_AVAILABLE" %}
	    {% block no_assignment %}
	        <h1>Call for participants</h1>
	        <p>
	            The XXX Lab at XXXXX University is looking for online participants
	            for a brief psychology experiment. The only requirements
	            are that you are at least 18 years old and are a fluent English
	            speaker.  The task will take XXXXX minutes and will pay XXXXX.
	        </p>
	        <p>
	            Please click the "Accept HIT" button on the Amazon site
	            above to begin the task.
	        </p>
	    {% endblock no_assignment %}

Note that the contents is within the block `no_assignment` so you can extend `base/ad.html` and overwrite just this block to customize the advert text.

But if an `assignment id` is available (so the participant has accepted the HIT on MTurk) the display changes to show a "thanks for taking part" message:
::

    {% else %}
        {% block live %}
            <h1>Thanks for accepting this HIT.</h1>
            <p>
                By clicking the following URL link, you will be taken to the experiment,
                including complete instructions and an informed consent agreement.
            </p>
            <div class="alert alert-warning">
                <b>Warning</b>: Please disable pop-up blockers before continuing.
            </div>

            <button type="button" id="begin-button" class="btn btn-primary btn-lg">
            Begin Experiment
            </button>
        {% endblock live %}
    {% endif %}

Again, this is given a block name, so it can be overwritten.

At this point you might want to look at `base/consent.html` as its broadly similar in terms of how it works. You can also look at the other templates. `error.html` is what participants are shown when a server error occurs (this indicates a bug in your experiment, but its good practice to show participants a friendly message letting them know!). `complete.html` and `thanks.html` are pages commonly shown at the end of experiments when everything is done.