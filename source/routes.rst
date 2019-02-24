Server Routes
=============

The first step in learning how to manage communication between the front and back ends of the experiment is to learn about server routes. These act as individual lines of communcation each of which is designed to accept a specific kind of message. Once you know what the available routes are and what kinds of message each route accepts you'll be able to design your experiment such that it contacts appropriate routes according to your needs.

What are routes?
----------------

To help you understand what routes are, let's take a look at a few examples. Here's the url to the main Dallinger docs website:

	https://dallinger.readthedocs.io

What that url is is the address of the server that hosts the dallinger docs. Just like your experiment server this server has routes. Routes are appended to the server address and are separated by slashes. If you go to that link you'll notice that the url of the page changes once you get there, it's now:

	https://dallinger.readthedocs.io/en/latest/

So we've gone from the main server address to the specific route called `/en/latest/`. Presumably `en` stands for `English` and `latest` just means the latest version of the docs. Note that the main server address itself if a route. The server url is equivalent to:

	https://dallinger.readthedocs.io/

(note that there's a single slash appened to the end of the server url). i.e. specifying no route is equivalent to going to the route `/`. A common feature of routes is that the final `/` is implied and so doesn't need to be typed in. So the route `/en/latest/` is the same as `/en/latest`, and `www.google.com` is the same as `www.google.com/`. You can check this now by removing the final `/` from the url of the page in your brower and hitting return/enter. The page will refresh and your browser will automatically put the final `/` back for you.

If you navigate among the pages you can see that the routes change accordingly, so the first page "Installation" corresponds to the route `/en/latest/installing_dallinger_for_users.html`. There's a couple of things to note here:

1. the final bit of the url is now a file name, with the extension .html. HTML is a language used to construct user interfaces, most commonly websites and you will probably use it yourself to make your experiments. That the url ends in a file name indicates that you requested a specific file from the server and it gave it to you.
2. Note that this time there is no final `/` on the url. That's because `/`s can be followed with more route details (e.g. `en/latest`) but a file name is terminal and so there is no `/` to allow anything else to be added to the url.

OK, so hopefully you now get a sense of one way in which you can use routes: you type them in to your browser. But this doesn't tell you how or why typing in routes gives you intelligible responses from the server. For this to happen, there must be some code on the server that is basically a list of all the routes on the server and what to do if someone pings them. And indeed there is and now we'll open up this bit of Dallinger's code to see what routes are available.

Dallinger's routes
------------------

The code that describes all of Dallinger's routes and what they do is located in the file `experiment_server.py` inside `Dallinger/dallinger/experiment_server/`. Open it now. The code here will look pretty fearsome, and that's ok, we'll go through bits of it slowly today, but in future you won't need to return here very often. As we'll see later, Dallinger provides some helpful functions that bury a lot of this complexity, and even if you do want to see a list of all the available routes, once you are more comfortable with Dallinger you'll be able to use the main documentation which contains a page listing all the routes:

	https://dallinger.readthedocs.io/en/latest/web_api.html

But for now, let's stare at the code until it makes sense.

Lot's of the early code in this file does things that I don't fully understand and that a Dallinger user definitely doesn't need to understand. For now, let's skip all the way down to around line 700 to look at the consent route:
::

	@app.route("/consent")
	def consent():
	    """Return the consent form. Here for backwards-compatibility with 2.x."""
	    config = _config()
	    return render_template(
	        "consent.html",
	        hit_id=request.args['hit_id'],
	        assignment_id=request.args['assignment_id'],
	        worker_id=request.args['worker_id'],
	        mode=config.get('mode')
	    )

The first thing the code does is declare the address of the route:
::

	@app.route("/consent")

What this means is that once your experiment is live (and let's say the server address is `www.myexperiment.com`) you could go to `www.myexperiment.com/consent` and that would trigger the function immediately below the route address declaration to run.

The function itself is pretty straight forward. It looks up a html page called `consent.html` and then "renders" it. Rendering basically means that it fills in certain value in the page to customize it for the current request. Think about a large site with many users, facebook, for instance. When a user logins in they probably go to a welcome page (and they perhaps get there by hitting the `/welcome` route). This returns the same welcome page for everyone, but often with some minor customization (e.g. the page might say "Welcome John!" when John logs in or "Welcome Jane" when Jane logs in). To have this information baked into the page itself would require a separate page for every user, which would be very wasteful. So instead there is a single template with fields that get filled in with specifics dependent on the user that requests it. We'll see how you can create/use templates like this yourself later, but in this specific case we can see that the values that are getting inserted are boring technical things like `assignment_id`:
::

	assignment_id=request.args['assignment_id'],

Note that the values that are being inserted are got from an object called `request`. This object is the request that the route received from the outside world.

So the above route returns the page "consent.html", but any given experiment is going to involve its own html pages that correspond to the various parts of the experiment. Does this mean that users will have to create their own routes corresponding to each page? Fortunately the answer is no, and the reason can be seen a few lines above, with the following route:
::

	@app.route("/<page>", methods=["GET"])
	def get_page(page):
	    """Return the requested page."""
	    try:
	        return render_template(page + ".html")
	    except TemplateNotFound:
	        abort(404)

First notice that the address of the route includes the symblos `<` and `>`. These are not literally part of the route, rather they demarcate a section of the route that could contain anything, but that the route will refer to as `page`. In short, this means that if a user tries to go to some randomly typed route consisting of a single `/` followed by a single word, this route will be activated and the function ``get_page(page)`` will run where `page` is whatever word was entered into the url. The ``get_page(page)`` function simply tried to find a html file with the same name as the value of the page variable and then return it. So, if a user manually edits their url bar in their brower to go to the route `www.myexperiment.com/xXx__420__xXx` then this function will be activated and the server will look for a file called `xXx__420__xXx.html`, which probably won't be there (unless you made it) in which case the function aborts and the user will see a 404 page not found error.

You might have wondered how the server knows to use the dedicated `/consent` route when a request to `www.myexperiment.com/consent` comes in, and not the generic route `/<page>`. This is because the generic routes have the lower priority and so incoming requests are preferntially handled by dedicated routes, and only if those don't exist are the requests passed on to generic routes.

Request parameter
-----------------

Shortly below these routes is a large function called `request_parameter`. We are not going to go through this function in detail (though feel free to if you want to), but we probably should cover what it does as most of the remaining functions call it.

We have already seen that an incoming request often comes with additional parameters. For instance a request to the `/consent` route appears to come with a `worker_id` parameter that is then used by the route to render the `consent.html` page. In Dallinger these request parameters are often things other than text. For instance they might be a number, a boolean (i.e. True or False) or even a Class. Nonetheless, at the moment a request is sent/received, all its contents are sent as text and so the typing of parameters is lost. We'll see this in action in some of the other routes, but the long and short of it is that Dallinger needs a way to extract these parameters from the request and return them to their correct typing. That's what the `request_parameter()` function does. Later routes will call it, give it an incoming request and tell it to extract a parameter with a certain name and of a certain type. The routes can also tell let the `request_parameter` function know if a default value should be used if the parameter was never sent. The function will do this to the best of its ability, but if the requested parameter is of a form that cannot be converted to the appropriate type it will raise an error. So if a route asks the function to get the parameter "score" from the request and says it should be an integer, but the function sees that it has the value "Hello" which cannot be turned into an integer an error will be raised. The consequences of this is that the server will send a message back to the user telling them their request could not be processed and an error will appear on their screen, but this shouldn't happen if you code your experiment to only send parameters with approriate values.

Routes that do things
---------------------

Now let's start looking at some of the routes that do the sorts of things we'll want during out experiment. We'll start with one of the most complex routes, `create_participant` which is just below the `request_parameter` function.

Create participant
^^^^^^^^^^^^^^^^^^

The first thing to look at is the "decorator" that specifies the address of the route (decorator is a term usd in python programming, put simply it refers to lines that start with an `@` and that are placed just above functions).
::

	@app.route("/participant/<worker_id>/<hit_id>/<assignment_id>/<mode>",
           methods=["POST"])

So we can see this routes address is `/participant/` followed by four more variables (the `worker_id` and so on) each separated by slashes. Note also that the parameter methods is specified as `POST`. In the route we looked at above the methods parameter was `GET`. The method of a route let's you know roughly that the route does. Specifically a `GET` route normally gets some information from the database and returns it to the user, while `POST` routes typically involve the user submitting some information that gets added to the database. Sometimes a single route can have multiple methods (note that `methods` is a list). You can also have multiple versions of the same route, each with a different method. So, if we wanted, we could have another route with the same address as long as it had a different method. Because this route has the `POST` route this tells us that it is adding information to the database. Moreover, because it starts with `/participant/` this is a good indication that its adding information to the participant table - i.e. this is the route that participants send a request to very early on, creating the entry in the participant table that corresponds to them.

Back to this route, it now has another decorator:
::

	@db.serialized

This decorator tells the server what to do is multiple participants send requests to this route at the same time. Normally everytime a request comes in the server handles it with a separate process. This allows the server to be very efficient because even if 10 requests all come in at the same time it just handles them all simulatenously. But there are some routes where you don't want this to happen, and we need to tell the server that is multiple requests come in to that route at the same time they should be processed one at a time and not in parallel. This is one of those routes and the decorator `@db.serialized` tells the server to treat it as such.

To understand why this is we need to think about what the route does. Remember that it is going to create a new entry in the participant table. Remember also that in each table, each row has a unique `id`. So now imagine that we already have three participants taking part in the experiment, they will have ids 1, 2 and 3. The next time a participant starts the experiment and sends a request to this route they will create Participant 4. But what happens if two requests come in at the same time? If they are handled in parallel each will be managed by a separate process. Because there is necessarily some time lag between the moment at which each process starts editing the database and the moment these edits become visible to other processes, it is possible that both processes will try to create participant 4. This will cause an error which, at the minimum, will cause which ever process started slightly later to fail. The solution is to force the two requests to be processed one after another such that which ever request came in second has to wait for the first request to complete processing. This way we pre-empt the risk of parallel processing of near-simultaneous requests to the same route leading to errors.

OK, let's get on with what the route actually does. As mentioned above this route is unusually complicated, so we'll skip over most of it. But there's a couple of highlights to look out for. Here's the code where it makers sure the participant's `worker_id` isn't already associated with a different Participant entry in the database:
::

	    already_participated = models.Participant.query.\
	        filter_by(worker_id=worker_id).one_or_none()

	    if already_participated:
	        db.logger.warning("Worker has already participated.")
	        return error_response(
	            error_type="/participant POST: worker has already participated.",
	            status=403)

To do this the code first executes a query over the participant table in the database (all routes can access the database at any time) and if a Participant with a matching `worker_id` is found and error is returned.

This route also checks to see if Mechanical Turk has been re-using `assignment_ids`.

Once all checks are passed the route creates the Participant:
::

	# Create the new participant.
    participant = models.Participant(
        recruiter_id=recruiter_name,
        worker_id=worker_id,
        assignment_id=assignment_id,
        hit_id=hit_id,
        mode=mode,
        fingerprint_hash=fingerprint_hash,
    )

It then uses the ``__json()__`` method of the newly created Participant to generate a text description of the Participant:
::

	result = {
        'participant': participant.__json__()
    }

And finally returns this to the client that made the request:
::

	return success_response(**result)

Now not only is the Participant created, but the front-end that requested it knows which Participant they are because they have been sent all the details of the row entered into the database.

Get Participant
^^^^^^^^^^^^^^^

The next route is the `GET` equivalent of the post route above, it gets and returns a row from the participant table:
::

	@app.route("/participant/<participant_id>", methods=["GET"])
	def get_participant(participant_id):
	    """Get the participant with the given id."""
	    try:
	        ppt = models.Participant.query.filter_by(id=participant_id).one()
	    except NoResultFound:
	        return error_response(
	            error_type="/participant GET: no participant found",
	            status=403)

	    # return the data
	    return success_response(participant=ppt.__json__())

This route is much simpler than the `POST` equivalent and hopfully you can follow it. There's a few things to note though:

1. The parameters the route takes is just the `participant_id` and not all the `worker_id` etc that we had to deal with before. That's because once a Participant is created the `id` is all you need to identify it.
2. The query that gets the requested participant is in a ``try`` statement. This means the the route knows that it might fail, i.e. the Participant might not exist. If failure occurs the code in the ``except`` statement will run.
3. If the request is successful a `success_response` is returned, containing a json description of the Participant object that was requested.

Node neighbors
^^^^^^^^^^^^^^

After this are some more basic routes; a route to get networks, a route to create questions and so on. We'll take a look at a more interesting route; a route to get a Node's neighbors. Remember back in the section on Nodes that Nodes have a function called `neighbors` that returns a list of all the Nodes that the focal Node has a connection to. However, that function is part of the back-end code, so its not readily available to the front end. Nonetheless, it's very plausible that a participant taking part might need to know which other participants it has a connection to, say they are taking part in a team-based task and each participant needs to know who their team mates are. The easiest way to do is is to create a route that hooks into the `neighbors` function of the ``Node`` class. That's what this route is.

The route declaration tells you where you need to address requests to to hit this route:
::

	@app.route("/node/<int:node_id>/neighbors", methods=["GET"])
	def node_neighbors(node_id):

So if you are playing as node 8 and you need to get your neighbors, you would send a `GET` request to `/node/8/neighbors`.

After this the route extracts some important parameters using the `request_parameter` function discussed above:
::

    node_type = request_parameter(parameter="node_type",
                                  parameter_type="known_class",
                                  default=models.Node)
    connection = request_parameter(parameter="connection", default="to")
    failed = request_parameter(parameter="failed",
                               parameter_type="bool",
                               optional=True)

So the parameters are (1) `node_type`, which is a `known_class` (i.e. a Class added to the Experiment's dictionary of known classes - see the Experiment class page for more information) and it defaults to ``Node``, (2) `connection` which defaults to `to`, and (3) `failed` which is a boolean and is optional. You might be wondering where these parameters are being entered (they're not in the route address) and we'll see this shortly.

Next the route gets the Node which is asking for its neighbors:
::

	node = models.Node.query.get(node_id)

It checks to make sure the Node isn't `failed` as `failed` Nodes are not allowed to make requests. If this check passes it gets the neighbors:
::

	nodes = node.neighbors(type=node_type, direction=connection)

It then notifies the Experiment that the request has been received and (almost) completed using the `node_get_request()` function:
::

	exp.node_get_request(
                node=node,
                nodes=nodes)
            session.commit()

By default this function does nothing (see the Experiment class chapter for more information) but individual Experiments can overwrite it to add extra functionality as needed.

Last of all the route returns a `success_response` including a list of all the neighbors, each of which has been turned into a json description:
::

	return success_response(nodes=[n.__json__() for n in nodes])

Creating Nodes
^^^^^^^^^^^^^^

After this the same sort of code is repeated time and time again to add even more routes which can received requests. As with the `neighbors` route, a lot of the time they are simply adding routes that call specific functions in one of the classes. This was you can use requests to call the same functions from the front-end that you routinely use when designing the back-end of your experiment. Let's rush through a few more:

There's a route to create Nodes:
::

	@app.route("/node/<participant_id>", methods=["POST"])
	@db.serialized
	def create_node(participant_id):

Note it requires a participant id, and, just like the route that creates Participants it is serialized to prevent conflicts arising when multiple requests come in at the same time. This function does its work by calling a whole series of functions. First it uses ``get_network_for_participant()`` to identify if there are any available Networks that the Participant's Node could be put into:
::

	network = exp.get_network_for_participant(participant=participant)

Then it calls the Experiment's ``create_node()`` function to make a Node of the appropriate type:
::

	node = exp.create_node(participant=participant, network=network)

Then it passes the Node to the ``assign_properties()`` function. This function is near the end of the file, but in short it checks the request to see if it included values for the wildcard columns (`property1`, `property2` and so on) and if they are present it adds them to the Node:
::

	assign_properties(node)

Then it calls ``add_node_to_network()`` which is another Experiment function to connect the Node up as required by the Experiment:
::

	exp.add_node_to_network(node=node, network=network)

Finally it calls ``node_post_request()`` to give the Experiment an opportunity to add some extra instructions to this process as necessary:
::

	exp.node_post_request(participant=participant, node=node)

Last of all, it returns a json description of the newly created node back to whoever sent the request:
::

	return success_response(node=node.__json__())

It's worth going over this a couple more times to get a sense of what happens. But essentially it means that a single request from a participant to have a new Node created for them triggers a whole series of events:

1. A suitable Network is found.
2. A Node is created and is assigned any properties.
3. The Node is added to the Network and connected up as necessary.

The Experiment can intervene at any stage of this process by overwriting the involved functions, and many of the demos do exactly this.

Other routes
^^^^^^^^^^^^

The next few routes are pretty easy to follow and most of them require a Node's `id` as a parameter. This is because although participants are associated with a Participant object, at any one time they are typically playing the role of a Node. Remember than Participants do not exist within Networks, only Nodes do, and so to take part in the experiment proper Participants need to be assigned a Node. So there's a route to get a Node's Vectors:
::

	@app.route("/node/<int:node_id>/vectors", methods=["GET"])
	def node_vectors(node_id):

A route to connect one Node to another (note this route needs both the id of the connecting Node and the Node you want to connect to/from:
::

	@app.route("/node/<int:node_id>/connect/<int:other_node_id>",
    	       methods=["POST"])
		def connect(node_id, other_node_id):

The direction of the connection is passed as part of the request, but not in the URL (we'll see how to do this shortly):
::

	direction = request_parameter(parameter="direction", default="to")

A route to get a specific Info:
::

	@app.route("/info/<int:node_id>/<int:info_id>", methods=["GET"])
	def get_info(node_id, info_id):

For this one you need to specify both your Node id and the id of the Info you want. The route checks if that Node is allowed to have access to the Info before it returns it. Permission is granted only if the Node created the Info, or if the Node has been sent the Info by another Node.
::

    elif (info.origin_id != node.id and
          info.id not in
            [t.info_id for t in node.transmissions(direction="incoming",
                                                   status="received")]):
        return error_response(error_type="/info GET, forbidden info",
                              status=403,
                              participant=node.participant)

A route to get all the Infos made by a Node:
::

	@app.route("/node/<int:node_id>/infos", methods=["GET"])
	def node_infos(node_id):

And another route to get all the Infos sent to a Node. This one is used quite a lot as its common for a participant to need to see information they have been sent:
::

	@app.route("/node/<int:node_id>/received_infos", methods=["GET"])
	def node_received_infos(node_id):

Creating Infos
^^^^^^^^^^^^^^

Perhaps the most commonly used route is the route to create Infos. This is because participant responses will almost always be stored as the contents of an Info. So in a questionnaire experiment, for instance, every time a question is responded to, the front-end will sent a request back to the back-end to have the response stored as an Info:
::

	@app.route("/info/<int:node_id>", methods=["POST"])
	@crossdomain(origin='*')
	def info_post(node_id):

I'm not entirely sure what the `@crossdomain` decorator means. You do need to know though that the route checks the request for some additional paramters:
::

    contents = request_parameter(parameter="contents")
    details = request_parameter(parameter="details", optional=True)
    info_type = request_parameter(parameter="info_type",
                                  parameter_type="known_class",
                                  default=models.Info)

(1) the `contents` which will be stored as the contents of the Info.
(2) the details, which will be stored in the details column of the Info.
(3) the `info_type` which is the class of Info to be made, say a Meme, or a Gene, or a Response. In the latter case the default value is Info, so if you don't specify an `info_type` you'll get generic Infos which is perfectly fine for many purposes.

I'll leave the rest of the routes up to you to look though, but they're more of the same sort of stuff we've already seen. Just remember that all of them hook into various Experiment functions that can be overwritten to change some of the functionality of these routes. Once you a vaguely comfortable with them you'll probably find it much easier to look at the main docs, as opposed to this guide or the back-end code.