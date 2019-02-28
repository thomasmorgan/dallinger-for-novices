Demo Front Ends
===============

Armored with our knowledge of the shared front-end files, let's delve back into the demos to see how they use these files, in combination with their own experiement specific code, to get their jobs done. As before we'll start with the Bartlett1932 demo, so go to *Dallinger/demos/dlgr/demos/Bartlett1932* and we'll go from there.

Bartlett1932
------------

As a quick recap (see the Demo Experiment Classes page for more details), Bartlett1932 is a text-based transmission chain experiment in which participants are shown and then asked to recall short stories, and their recollections are then shown to the next participant. As more participants take part they form a chain.

In the `/bartlett1932` folder we've already gone though `experiment.py` and `models.py`. In this section we'll go through the folders `static` and `templates` which (if you recall form the previous page) are the conventional names for the folders that store scripts and html pages respectively. Let's start with the `templates` folder.

Templates
^^^^^^^^^

There aren't that many files with `templates`, and once you open them, you'll see that they don't contain that much either. We'll start with `ad.html` which is the page that users see when they are browing for potential things to do on the recuiting service. It turns out that Bartlett's `ad.html` does nothing other than extend the base `ad.html` from the shared front end files:
::

	{% extends "base/ad.html" %}

You can go back and look at that file if you want, but the base ad is basically a very generic statement inviting the viewer to take part in "a study" with the names of institutions and researchers replaced with XXXXXX. For an actual experiment this would obviously need tweaking, but for a demo packaged with Dallinger its fine.

The consent page similarly only extends the base consent page, but `layout.html` and `exp.html` do a little bit more. Let's start with `layout.html`:
::

	{% extends "base/layout.html" %}

	{% block title -%}
	    Psychology Experiment
	{%- endblock %}

	{% block libs %}
	    <script src="/static/scripts/store+json2.min.js" type="text/javascript"> </script>
	    <script src="/static/scripts/markdown.min.js" type="text/javascript"> </script>
	    {{ super() }}
	    <script src="/static/scripts/experiment.js" type="text/javascript"> </script>
	{% endblock %}

So the first thing to note is that it extends the base layout page. If you remember from a couple of sections back, the base layout page sets a title, imports some css files and some js files, and checks for adblockers. It also sets up a bunch of code blocks that experiments can modify individually. So right off the bat Bartlett's layout page gets all this functionality, but we can also see that the next thing it does is change the title:
::

	{% block title -%}
	    Psychology Experiment
	{%- endblock %}

See how it's using "blocks" to edit just the title. Here's the code from the base layout page that sets up the block in the first place:
::

    <title>{% block title %}Experiment{% endblock %}</title>

So the base layout page creates an html `<title>` section, that by default contains the word "Experiment". But, because it gives it the block name "title", Bartlett can modify this block to change the contents of the html `<title>` section to "Psychology Experiment".

The last thing Bartlett's `layout.html` page does is add some extra javascript files to import:
::

	{% block libs %}
	    <script src="/static/scripts/store+json2.min.js" type="text/javascript"> </script>
	    <script src="/static/scripts/markdown.min.js" type="text/javascript"> </script>
	    {{ super() }}
	    <script src="/static/scripts/experiment.js" type="text/javascript"> </script>
	{% endblock %}

Note again that using referencing a code block in the base layout page and modifying it. It's using the statement ``{{ super() }}`` to mean that the javascript files imported by `base/layout.html` should still be imported, but that the extra ones listed should be imported too. The extra files are "store+json2.min.js" (which is actually imported by the base layout anyway and so doesn't need to be here), "markdown.min.js" which is a javascript library used to read the stories that are formatted as markdown files and finally "experiment.js" which is Bartlett's custom javascript file.

So, Bartlett's `layout.html` changes the title and adds some extra javascript files to import. As long as the rest of Bartlett's html files extend `layout.html` (either immediately, or indirectly as in the case of the consent page that extends a page that extends layout) then they will also get these javascript libraries.

exp.html
^^^^^^^^

The last html page in this directory is `exp.html` which is where the participants browsers will be pointing when they are actually taking part in the experiment. Like the other pages it extends `layout.html`:
::

	{% extends "layout.html" %}

After that it changes the "body" block of the page, which is the actual contents of the page (html pages are often split into the body, which is the actual contents, and the header, which is imports and so forth). Here's the new body code:
::

    <div class="main_div">
        <div id="stimulus">
            <h1>Read the following text:</h1>
            <div><blockquote id="story"><p>&lt;&lt; loading &gt;&gt;</p></blockquote></div>
            <button id="finish-reading" type="button" class="btn btn-primary">I'm done reading.</button>
        </div>

        <div id="response-form" style="display:none;">
            <h1>Now reproduce the passage, verbatim:</h1>
            <p><b>Note:</b> Your task is to recreate the text, word for word, to the best of your ability.<p>
            <textarea id="reproduction" class="form-control" rows="10"></textarea>
            <p></p>
            <button id="submit-response" type="button" class="btn btn-primary">Submit response.</button>
        </div>
    </div>

First note the whole thing is set within a div called `main_div`:
::

    <div class="main_div">
        contents...
    </div>

And the contents of the main_div is further split into two other divs, the "stimiulus" div and the "response-form" div:
::

    <div class="main_div">
        <div id="stimulus">
            stimulus contents...
        </div>

        <div id="response-form" style="display:none;">
            response-form contents...
        </div>
    </div>

Note also that the `response-form` div is initially set to be hidden (``style="display:none;"``). This means that when the page loads the `response-form` won't be visible to participants, only the `stimulus` will be. Infact when we run through Bartlett in the next seciton you'll see that the experiment basically switches these two divs on and off as the participant proceeds.

The `stimulus` div contains a header saying "Read the following text" and then creates a `blockquote` item where the story will appear (its empty by deafult though) and then creates a button called `finish-reading` labelled "I'm done reading":
::

        <div id="stimulus">
            <h1>Read the following text:</h1>
            <div><blockquote id="story"><p>&lt;&lt; loading &gt;&gt;</p></blockquote></div>
            <button id="finish-reading" type="button" class="btn btn-primary">I'm done reading.</button>
        </div>

So, straight away the participant would see something vaguely like the experiment, just with no story yet shown. You might be wondering what `class="btn btn-primary"` means in the button code. To understand this requires a few steps:

1. Remember that the base layout.html page imports some css files, and that css controls the general "look-and-feel" of a page.
2. One of these files is bootstrap.css, a very common css library that gives a pleasant, if generic, appearance.
3. Css changes the appearance of elements according to their classes, and `btn` and `btn-primary` are classes that bootstrap looks for.
4. `btn` probably does basic button functionality, but I know for a fact that `btn-primary` makes the button blue.
5. So by using these classes here we can make sure the button looks perfectly fine (if a little generic) with no further work.

For more bootstrap see here: https://getbootstrap.com/docs/3.4/css/

The `response-form` div is structurally pretty similar:
::

    <div id="response-form" style="display:none;">
        <h1>Now reproduce the passage, verbatim:</h1>
        <p><b>Note:</b> Your task is to recreate the text, word for word, to the best of your ability.<p>
        <textarea id="reproduction" class="form-control" rows="10"></textarea>
        <p></p>
        <button id="submit-response" type="button" class="btn btn-primary">Submit response.</button>
    </div>

It contains a header, plus a slightly more detailed subheader, a `textarea` (as opposed to `blockquote`) where participants can enter their response, and then a submit button.

Note that at this point neither the `submit-response` button nor the `finish-reading` button will actually do anything. They are added to the page by html, and their appearance is adjusted by css, but to actually have clicking them cause anything to happen we need javascript.

instruct-ready.html
^^^^^^^^^^^^^^^^^^^

Now open up the last remaining Bartlett html page, which is in its own folder called "instructions". It doesn't need to have its own folder, this is done just for tidiness (often instructions comprise many html pages, not just one). Unlike all the other html pages, however, this page does have a small amount of javascript embedded in it. Let's skip the first few rows and look at the button:
::

    <div class="row">
        <div class="col-xs-10"></div>
        <div class="col-xs-2">
            <button type="button" class="btn btn-success btn-lg" onClick="dallinger.allowExit(); dallinger.goToPage('exp');">
            Begin</button>
        </div>
    </div>

So, there's a bunch of html (mostly divs) that are assigned to bootstrap classes to adjust their appearance. You might want to know about `col-xs-`: bootstrap lets you divide your page into columns of different widths. The extra-small columns (i.e. `col-xs-`) are narrow enough that exactly 12 fit across the screen. In this example we create a row, put an empty div that fills the first 10 columns, and then add another div that fills the last two columns and contains a button. This basically means the button ends up towards the right hand side of the screen. But, let's look at the button itself. First note that it uses slightly different css that the previous buttons being of the class `btn-success` which makes it green (as opposed to blue). But after this som javascript gets added to the button:
::

	onClick="dallinger.allowExit(); dallinger.goToPage('exp');"

This means that when the button is clicked (`onClick`) the code within the quote marks is executed. The code is simple and it calls two functions from within the dallinger2.js library (though note again these functions are called with `dallinger.`, and NOT `dallinger2.`). The first function `allowExit()` allows the participant to leave the current page. Dallinger can't actually stop the participant leaving as browsers (sensibly) won't let websites take control of the user's computer, but Dallinger is allowed to create a popup sayin "Are you sure you want to leave?" when the partcipant tries to close the tab or navigate elsewhere. Seeing as this is a case where we want to move the participant on (they have clicked the "Begin" button) we need to disable this system before moving them.

The next function actually moves the participant on, using `dallinger.goToPage()` and it sends them to exp.html, the page we just looked at (note that you don't need to include ".html" in the function call).

Last of all the page calls one more function, again from the dallinger2.js library:
::

	{% block scripts %}
	    <script>
	        dallinger.createParticipant();
	    </script>
	{% endblock %}

This script isn't attached to an html element (like a button) so it just runs as soon as the page loads. Here its pinging the `createParticipant` function which we covered in earlier sections. This sends a request to the server to create an entry in the Participant table corresponding to this participant. You'll need to do this at some point in all experiments, but only once per participant, and before the experiment proper starts, `and` after participants have given their consent to take part. Given these constraints the first (and in this case, only) instructions page is ideal.

Static
^^^^^^

OK, let's get started with the `static` folder. First there a css folder containing `bootstrap.min.css`, this is the bootstrap css library. Interestingly enough the bootstrap library (at the time of writing) is not included in the shared frontend files (even though the shared front end templates attempt to import it!) and so because Bartlett wants to use it, it has to provide it itself. This is bad practice and in a future version of Dallinger it'll probably get moved into te shared frontend files so individual experiments don't need to provide it themselves.

The `images` folder contains a single image called `logo.png` which is a generic images of something like a certificate. If you search across all files for uses of this image you'll see it crops up in the base `ad.html`:
::

    {% block lab_logo %}
        <img id="adlogo" src="{{ server_location }}/static/images/logo.png" alt="Lab Logo" />
    {% endblock %}

So this image appears on the ad page, alongside the advert text. For individual experiments you can replace the image with your institution or research group logo to make things look a bit nicer.

Instide the `stimuli` folder (which is a folder unique to Bartlett) are all the stories shown to participants saved in individual markdown files.

Last of all is the `scripts` folder. This contains two files, one is the markdown library used to read in the stories (and, as we saw, important by Bartlett's `layout.html`). The other is Bartlett's `experiment.js` which contains three functions that enable the experiment front end to work. We'll go through them one by one. First is the ``(document).ready()`` function:
::

	$(document).ready(function() {

	  $("#finish-reading").click(function() {
	    $("#stimulus").hide();
	    $("#response-form").show();
	    $("#submit-response").removeClass('disabled');
	    $("#submit-response").html('Submit');
	  });

	  $("#submit-response").click(function() {
	    $("#submit-response").addClass('disabled');
	    $("#submit-response").html('Sending...');

	    var response = $("#reproduction").val();

	    $("#reproduction").val("");

	    dallinger.createInfo(my_node_id, {
	      contents: response,
	      info_type: 'Info'
	    }).done(function (resp) {
	      create_agent();
	    });
	  });

	});

We'll go through this line by line. The first bit simple means that this function should only run when the document (the webpage) is ready (i.e. it as loaded):
::

	$(document).ready(function() {

After that it assigns functionality to the different buttons that appear on various pages. First the `finish_reading` button which participant click when they think they have memorized the story is updated so it hides the story, shows the response form and enables the `submit-response` button:
::

	$("#finish-reading").click(function() {
	    $("#stimulus").hide();
	    $("#response-form").show();
	    $("#submit-response").removeClass('disabled');
	    $("#submit-response").html('Submit');
	});

Note that this code is using dollar signs ($) to reference specific html elements. This uses a javascript library called `jquery` which is ubiquitous across and the web and is included in Dallinger's shared front end files.

The `submit-response` button is given slightly more complex functionality:
::

	$("#submit-response").click(function() {
	    $("#submit-response").addClass('disabled');
	    $("#submit-response").html('Sending...');

	    var response = $("#reproduction").val();

	    $("#reproduction").val("");

	    dallinger.createInfo(my_node_id, {
	        contents: response,
	        info_type: 'Info'
	    }).done(function (resp) {
	        create_agent();
	    });
	});

So it first disables the button itself, stopping participants from clicking it repeatedly. It then reads whatever they typed in from the `reproduction` html element. The it uses the `createInfo` function to send this back to the server to save it. When this request is completed it calls `create_agent` again to ask for another node.

We can see the `create_agent` function immediately below:
::

	var create_agent = function() {
	  $('#finish-reading').prop('disabled', true);
	  dallinger.createAgent()
	    .done(function (resp) {
	      $('#finish-reading').prop('disabled', false);
	      my_node_id = resp.node.id;
	      get_info();
	    })
	    .fail(function (rejection) {
	      // A 403 is our signal that it's time to go to the questionnaire
	      if (rejection.status === 403) {
	        dallinger.allowExit();
	        dallinger.goToPage('questionnaire');
	      } else {
	        dallinger.error(rejection);
	      }
	    });
	};

``create_agent`` disables the `finish-reading` button, and then sends a request to the `createAgent` route on the server. If this request succeeds (``.done(``) the `finish-reading` button is re-enabled, the front ends makes a note of the `id` of the node, and the ``get_info()`` function is called. If the function failes though (``.fail(``) then the participant is sent to the questionnaire.

The final function is ``get_info()``:
::

	var get_info = function() {
	  // Get info for node
	  dallinger.getReceivedInfos(my_node_id)
	    .done(function (resp) {
	      var story = resp.infos[0].contents;
	      var storyHTML = markdown.toHTML(story);
	      $("#story").html(storyHTML);
	      $("#stimulus").show();
	      $("#response-form").hide();
	      $("#finish-reading").show();
	    })
	    .fail(function (rejection) {
	      console.log(rejection);
	      $('body').html(rejection.html);
	    });
	};

This sends a request to the server to get any of the node's received Infos. When the rquest is successful it gets the contents of the first Info (there should only be one) and displays it in the `story` html element.

That's it. At this point its probably still quite confusing how all these functions, html elements, server code and so on work together. So in the next section we'll go through the demos again, but this time walking through all their code together.
