Dallinger's Javascript library
==============================

In the previous section we saw that dallinger comes with a range of front-end files that get added to your experiment at run time. These are intended to help you make functional experiments more quickly than would otherwise be the case. We walked through some of the html files, but the real utility, the bit that will save you the most time, is dallingers javascript library `dallinger2.js` (its called `dallinger2` because it replaced an earlier `dallinger.js`). In this section we'll walk through some of the key functions that this library provides, so get it open now (`Dallinger/dallinger/frontend/static/scripts/dallinger2.js`) and we'll get cracking.

Sending requests
----------------

Probably the nicest thing about the `dallinger2` js library is that it provides a function that masks most of the detail of sending requests from the front end to the back end. The base function is `ajax`:
::

	var ajax = function (method, route, data) {
	    var deferred = $.Deferred();
	    var options = {
	      url: route,
	      method: method,
	      type: 'json',
	      success: function (resp) { deferred.resolve(resp); },
	      error: function (err) {
	        console.log(err);
	        var rejection = dlgr.AjaxRejection(
	          {'route': route, 'method': method, 'data': data, 'error': err}
	        );
	        deferred.reject(rejection);
	      }
		};
		if (data !== undefined) {
		  options.data = data;
		}
		reqwest(options);
		return deferred;
	};

You really don't need to understand this function in detail, but you atleast need a sense of hot it works. First, note that it takes three arguments:

1. `method` is what route method you want to use, i.e. `get` or `post`.
2. `route` is the address of the route, e.g. `node/5/neighbors` for the neighbors of node 5.
3. `data` is something we haven't seen much of yet, but we'll see later that its a way to bundle additional information into the request, things like `contents` for Info post requests.

The rest of the code simply executes this request. Note also that at the end it returns the output of the request to whatever called the function:
::

	return deferred;

where `deferred` is the request object (I am not sure why it is called deferred, but it doesn't really matter). So, to summarize, this function allows user created experiments to send requests very quickly. Let's say you do want to get the neighbors of Node 5. With `dallinger2` imported you only need to do this:
::
	
	dallinger.ajax('get', 'node/5/neighbors')

Note that here we are not specifying anything for the `data` argument, and we call the `dallinger2` libarary by appending ``dallinger.`` to the front of the command (and not ``dallinger2.``). Here's another example where we are sending a request to create a new Info:
::

	dallinger.ajax('post', 'info/5', data = {"contents": "hello!"})

This sends a request to have Node 5 make an Info with the contents "hello!". But things get even nicer. Because `get` and `post` requests are very common, `dallinger2` provides handy wrappers for both:
::

	dlgr.get = function (route, data) {
    	return ajax('get', route, data);
    };

    dlgr.post = function (route, data) {
    	return ajax('post', route, data);
    };

You should be able to see that these are very basic wrappers around the `ajax` function that fill in the method for you. So with these, the above request to make an Info can be replaced with:
::

	dallinger.post('info/5', data = {"contents": "hello!"})

In fact `dallinger2` goes even further with wrappers for common types of requests. So there's the function `createAgent` to send a Node post request:
::

	dlgr.createAgent = function () {
    	return dlgr.post('/node/' + dallinger.identity.participantId);
    };

So now you can send a request to create a Node simply by executing ``dallinger.createAgent()``. You might be wondering what ``dallinger.identity`` is and we'll cover that shortly in the subsection on ``createParticipant``. There are other handy functions too: ``createInfo`` sends an Info post request:
::

	dlgr.createInfo = function (nodeId, data) {
    	return dlgr.post('/info/' + nodeId, data);
    };

If you look through the rest of the file you'll see there are other similar functions to get Infos, get received Infos, get incoming Transmissions, get Experiment properties, and so on. Some of the functions are bigger than others though, so we'll look through a couple of them now.

Create participant
------------------

In the previous chapter we saw there is a server route (`/participant/`) that can be pinged to prompt the creation of Participant objects. One the front-end side `dallinger2.js` includes the function `createParticipant()` that can be called to send a request to this route. But, its a bit more complicated that the other functions we've just been looking at, so let's go through it more carefully. We're not going to go through every line though, a lot of the code is a bit beyond what you need to know.

The first important bit is where the function gets the various parameters needed for participant creation, and uses them to create the route address:
::

	hit_params = get_hit_params();
    url = "/participant/" + hit_params.worker_id + "/" + hit_params.hit_id +
      "/" + hit_params.assignment_id + "/" + hit_params.mode + "?fingerprint_hash=" +
      (hit_params.fingerprint_hash) + '&recruiter=' + hit_params.recruiter;

The function `get_hit_params()` is what loads the parameters, and they are originally sent to the participants machine by the recruitment service (i.e. mechanical turk) itself.

Next the function uses the `post()` function to actually send the request and opens a ``done`` function that dictates what the function should do `once the request has completed`:
::

	dlgr.post(url).done(function (resp) {

The next thing it does is save the `participant id` to the local variable ``dallinger.identity``:
::

	$('.btn-success').prop('disabled', false);
    dlgr.identity.participantId = resp.participant.id;

This is pretty much all the important stuff. The rest of the function checks how many other participants have arrived to determine if quorum has been reached, but we don't need to worry about that for now. So, to summarize, the create participant function does the following:

1. load the parameters that originally came from MTurk
2. create the url and send the request
3. save the result in the dallinger.identity variable

Submitting the Questionnaire and Assignment
-------------------------------------------

The other functions we'll look at happen at the very end of the experiment. The first is ``submitQuestionnaire()``. This is an optional function that you can call to submit questionnaire data the participant completes during debriefing (i.e. the data is saved in Question objects and not in Infos) provided that your questionnaire follows dallingers expected style (we'll see examples of this later). The code is pretty abstract though, so I'll just give you a high level overview. The first bit is what extracts all the questionnaire data:
::

	var formSerialized = $("form").serializeArray(),
		spinner = dlgr.BusyForm(),
		formDict = {},
		xhr;

	formSerialized.forEach(function (field) {
		formDict[field.name] = field.value;
	});

This data is then sent to the Question post route:
::

	xhr = dlgr.post('/question/' + dlgr.identity.participantId, {
        question: name || "questionnaire",
        number: 1,
        response: JSON.stringify(formDict)
    });

For simplicity (although somewhat ignoring Dallingers implicit style) this function saves all the data in a single Question object, where the `response` column actually contains a full description of all the questions and answers. After this the ``submitAssignment()`` function is called:
::

	xhr.done(dlgr.submitAssignment);

The first thing this function does is ask the server for all the participants details. The front end knows its `participant id`, but in order to submit the assignment back to the recruiter it needs to look back up all its recruiter specific values, things like `worker_id` and so on:
::

	dlgr.get('/participant/' + dlgr.identity.participantId).done(function (resp) {

Once this request is complete sends another request to the ``worker_complete`` route:
::

	dlgr.get('/worker_complete', {
        'participant_id': dlgr.identity.participantId
    })

If this request completes successfully the participant is sent to the ``complete`` page that returns them to the recruiters website where they can submit their work and get paid and so on:
::

	dallinger.allowExit();
    window.location = "/complete";

And this ends the experiment.