Demo Experiment Classes
=======================

Now that we've seen how the base ``Experiment`` works, let's take a look at how some of the demos that come packaged with Dallinger modify the base ``Experiment`` Class to produce their own unique ``Experiment`` subclasses.

First, you'll need to know how to find the demo files. First go to *Dallinger/demos/dlgr/demos* and in there you should see a bunch of folders that correspond to various different experiments. We'll go through a few of them now.

Bartlett1932
------------

This is the first demo we ever made for Dallinger. It was created very early on in development and served as a means for us to test that Dallinger worked at all. As Dallinger development continued the Bartlett1932 code continued to be updated to take advantage of the new functionality we added to Dallinger. As such its a bit like the Ship of Theseus - at some point we've changed pretty much every last bit of it, nonetheless, it some senses its as old as Dallinger itself.

Anyway, so what is Bartlett1932? Well, it's the Dallinger version of an experiment originally carried out by the British psychologist Frederic Bartlett, and published in his 1932 book remembering. The purpose of the experiment was to explore some of the dynamics of memory and the repeated remembering of information. The structure of the experiment is very similar to the children's game "telephone" or "Chinese whispers", though in academic circles its more often called a transmission chain. At the start of the experiment some information was given to a participant, and they wre asked to remember and recall it. Whatever they recalled was then passed on to another participant with the same instructions, and this process was repeated forming a chain of participants. At the end of the chain, the resultant information could be radically different to what was originally provided. In Bartlett's case he started chains with traditional folk stories.

The two files we are going to look inside are *models.py* and *experiment.py*. Let's start with *models.py*.

models.py
^^^^^^^^^

To understand the existence of models.py, remember that all experiments require a new subclass of ``Experiment`` to be built that overwrites some of the functions in the base ``Experiment`` class, but that specific experiments might also require the creation of newsubclasses of other Dallinger objects, like Nodes, or Infos. A convention (though one that can be violated if you so wish) is to place the experiment specific subclass of ``Experiment`` in *experiment.py* but to place experiment specific subclasses of any other objects in *models.py*. Bartlett1932 makes one new kind of object, so let's take a look at it, starting with the class declaration:
::

	class WarOfTheGhostsSource(Source):
    	"""A Source that reads in a random story from a file and transmits it."""

The first thing to note is that the newly created class (``WarOfTheGhostsSource``) overwrites the class ``Source``, so its immediately going to inherit all the behavior of a Source. You can go back to the Nodes page and re-read the Source bit to catch up, but the short of it is that Sources cannot receive Transmissions and they have some rather complicated behavior associated with their ``transmit`` function that we'll cover shortly.

You might also be wondering why the newly made subclass has such a strange name. Well, ultimately it doesn't really matter what you call your custom classes (though probably best not to call them names that are already in use like "Info"), but in this case the source is named after one of the folk stories used by Bartlett in his original experiment.

Next, the class has that rather clunky ``__mapper_args__`` bit, which, if you can remember, simply tells the database what value to put in the *type* column of the corresponding table in the database when a WarOfTheGhostsSource is created, in this case its *war_of_the_ghosts_source*.
::

    __mapper_args__ = {
        "polymorphic_identity": "war_of_the_ghosts_source"
    }

After this the code overwrites just a single function, ``_contents()``. To understand what this function does you really might want to go back an dre-read the seciton on Sources in the Node page. But the short of it is as follows:

1. When Node of any kind (including a Source) is told to ``transmit()`` the caller can provide arguments telling the Node *what* to transmit and *to_whom* to transmit to.
2. If *what* is not specified the Node calls its function ``_what()`` to check its default behavior.
3. For the base class ``Node`` this returns ``Info``, i.e. the Node should transmit all its Infos. But the ``Source`` class overwrites this, such that ``_what()`` just calls ``create_information()`` a new function that only Sources have.
4. ``create_information()`` makes a new Info and returns it to ``_what()`` (which returns it to ``transmit()``) so the default behavior of a Source is a transmit a newly made Info.
5. But, before ``create_information()`` makes the Info, it calls two more functions.
6. First it calls ``_info_type()`` to determine what kind of Info it should make. By default this function returns ``Info``, but it can be overwritten to return subclasses of ``Info``, like ``Gene`` or ``Meme``.
7. Next it calls ``_contents()`` to determine what the contents of the new Info should be. By default this function just raises an error because the contents of the Info is going to be experiment specific and so the user will have to overwrite it according to their experimental needs.

OK, so with this in mind we can now see what the ``WarOfTheGhostsSource`` is doing - it's overwriting ``_contents()`` to specifcy what the contents of newly made Infos should be when the WarOfTheGhostsSource is asked to transmit but not told what it should transmit. Note that it doesn't overwrite ``_info_type()``, so in this experiment we are happy to send generic Infos.

Let's take a look at how the contents is determined. First the sources makes a list called *stories* and populates it with a bunch of file names (*.md* is the extension for markdown files), it then chooses one of these stories at random:
::

        stories = [
            "ghosts.md",
            "cricket.md",
            "moochi.md",
            "outwit.md",
            "raid.md",
            "species.md",
            "tennis.md",
            "vagabond.md"
        ]
        story = random.choice(stories)

At this point the variable *story* will contain the name of one of the files from the *stories* list. After this there's some quite hard-to-follow code that tells the experiment to actually go find that file and read its contents, then return this to whatever called the ``_contents()`` function (i.e. the ``create_information()`` function):
::

        with open("static/stimuli/{}".format(story), "r") as f:
            return f.read()

You're probably wondering where these stories are, and if you look at the above code you can see their directory: *static/stimuli/*. So if you look in the Bartlett1932 folder you'll see another folder called static, within that one called stimuli and within that are the story .md files. Go take a look! These are all the same stories used by Bartlett in the 1930s and so the language is often quite fun.

At this point you can probably think of some ways to modify the experiment for other purposes. For instance you could change the contents of the *.md* files to change which stories are sent to participants. You could also create new stories by adding new *.md* files in the *stimuli* directory and adding them to the *stories* list in the ``WarOfTheGhostsSource``. Finally, notice that the ``WarOfTheGhostsSource`` picks a story at random, but maybe you want it to be more deterministic. Let's say you want to run 8 parallel chains and you want to make sure each chain uses a different story such that each story gets used once and only once. Well, in Dallinger the quickest way to have repeat chains is to have multiple repeat networks (we'll see this shortly), each of which will contain its own WarOfTheGhostsSource. Because all Networks have a unique *id* that starts at 1 and counts upward we can use this as an index to pick the appropriate story for each Network:
::

        stories = [
            "ghosts.md",
            "cricket.md",
            "moochi.md",
            "outwit.md",
            "raid.md",
            "species.md",
            "tennis.md",
            "vagabond.md"
        ]
        story = stories[self.network_id-1]

So, here, in order to decide what story to use the ``WarOfTheGhostsSource`` looks up its *network_id* (``self.network_id``) and uses this as an index to pick a story. One must be subtracted from the id because python counts from 0 (i.e. the first story is at index 0).

experiment.py
^^^^^^^^^^^^^

OK, let's open *experiment.py*. Skip the various imports for now and head straight to the class declaration:
::

	class Bartlett1932(Experiment):
	    """Define the structure of the experiment."""

From this we can see that we're making a class called *Bartlett1932* and that it is a subclass of ``Experiment``. Note that there's no ``__mapper_args__`` funny business this time because there is not Experiment table.

The first function to be overwritten is ``__init__()``, let's go through it step-by-step. The very first bit looks odd, but is actually quite straightforward:
::

	    super(Bartlett1932, self).__init__(session)

This is simply the python way of saying "first run the ``__init__()`` function of the base ``Experiment`` class". If you go back to the section on Experiments you'll see that this mostly just sets a bunch of variables to appropriate values. By calling this we avoid having to duplicate this whole function and we can then just write out the bits that make a ``Bartlett1932`` differ from an ``Experiment``.

Next ``Bartlett1932`` imports out *models.py* page and saves it as the variable ``self.models``:
::

        from . import models
        self.models = models

We'll see this in action shortly when we need to make the WarOfTheGhostsSource.

Next Bartlett1932 changes the value of *self.experiment_repeats* and *self.initial_recruitment_size*. Thee correspond to how many Networks to make and how many participants to start off with, respectively. If you want more Networks increase the value of *self.experiment_repeats*, but don't forget to increase *self.initial_recruitment_size* accordingly otherwise no-one will be recruited for the extra Networks.

Finally, ``__init__()`` calls ``setup()``:
::

        if session:
            self.setup()

Don't worry too much about what ``session`` is, but you can think of it as the connection to the database tables. Because the purpose of ``setup()`` (which we covered in the chapter on the base Experiment class) is to create Networks, the function cannot run properly unless it has access to the database tables in order to create the Networks. There is a default ``setup()`` function in ``Experiment`` and this does the Network creation:
::

    def setup(self):
        """Create the networks if they don't already exist."""
        if not self.networks():
            for _ in range(self.practice_repeats):
                network = self.create_network()
                network.role = "practice"
                self.session.add(network)
            for _ in range(self.experiment_repeats):
                network = self.create_network()
                network.role = "experiment"
                self.session.add(network)
            self.session.commit()

but ``Bartlett1932`` overrides this as follows:
::

    def setup(self):
        """Setup the networks.

        Setup only does stuff if there are no networks, this is so it only
        runs once at the start of the experiment. It first calls the same
        function in the super (see experiments.py in dallinger). Then it adds a
        source to each network.
        """
        if not self.networks():
            super(Bartlett1932, self).setup()
            for net in self.networks():
                self.models.WarOfTheGhostsSource(network=net)

So, first we can see that although ``Bartlett1932`` overrides the function, the first thing it does is call its *super* (the base ``Experiment`` class) and run its setup function anyway. This makes sure the Networks are created in the usual fashion. The onyl difference is that after this Bartlett1932 goes though each network and adds a WarOfTheGhostsSource to it. Note how it's using ``self.models`` to access the WarOfTheGhostsSource, this relies on the import we discussed earlier.

Not, remember that when the base ``Experiment`` class runs its ``setup()`` function (which is called by Bartlett1932's ``setup()`` function) it calls the ``create_network()`` function to determine what sort of Network it should be making. By default, the base Experiment class makes an Empty Network, but this is no good for Bartlett which needs a Chain. Accordingly, the ``create_network()`` function is also overwritten:

    def create_network(self):
        """Return a new network."""
        return Chain(max_size=5)

Note that the max_size of the Chain is set to 5, which includes the Source. If you want longer chains you would need to change this value.

The next function is ``add_node_to_network()``. A minimal version of this function already exists in the base ``Experiment`` class:
::

    def add_node_to_network(self, node, network):
        """Add a node to a network.

        This passes `node` to :func:`~dallinger.models.Network.add_node()`.

        """
        network.add_node(node)

From the chapter on Networks you may be able to remember that each type of Network has a function that defines how it grows as each Node is added. This function is called ``add_node()`` and as you can see the function of ``add_node_to_network()`` in the base Experiment class is simply to call this function. For a quick reminder, here's the ``add_node()`` function from the ``Chain`` Network:
::

    def add_node(self, node):
        """Add an agent, connecting it to the previous node."""
        other_nodes = [n for n in self.nodes() if n.id != node.id]

        if isinstance(node, Source) and other_nodes:
            raise Exception(
                "Chain network already has a nodes, "
                "can't add a source."
            )

        if other_nodes:
            parent = max(other_nodes, key=attrgetter('creation_time'))
            parent.connect(whom=node)

While this function correctly adds the new Node to the end of the Chain, it is missing one key thing necessary for Bartlett1932: it doesn't transmit the story recalled by the previous Node to the new Node. Accordingly, Bartlett1932 edits ``add_node_to_network()`` to add this functionality:
::

    def add_node_to_network(self, node, network):
        """Add node to the chain and receive transmissions."""
        network.add_node(node)
        parents = node.neighbors(direction="from")
        if len(parents):
            parent = parents[0]
            parent.transmit()
        node.receive()

So, first this function adds the Node in the usual fashion. It then gets the previous Node in the Chain (the "parent") by asking the new Node for a list of its neighbors and taking the 0th Node in this list (there will only ever be one neighbor). In then tells the parent to transmit. Note that we don't need to specify *what* or *to_whom*. The former defaults to ``Info`` (i.e. all Infos the parent has made) but thye have only made one - their recollection of the story - so that is the only thing that gets sent. Meanwhile, *to_whom* defaults to ``Node`` (i.e. all Nodes you have a connection to) and the new Node is the only Node the parent has a connection to.

The last function of ``Bartlett1932`` is ``recruit()``. Again, the base Experiment class has this function, but it defaults to never recruiting. For Bartlett 1932 this needs to be different, when each participant finishes we need to recruit the next one:
::

    def recruit(self):
        """Recruit one participant at a time until all networks are full."""
        if self.networks(full=False):
            self.recruiter.recruit(n=1)
        else:
            self.recruiter.close_recruitment()

Note that this function would work even if you had multiple Networks, but it would be slow because you would only have one participant taking part at a time. To improve the efficiency of your experiment you would want to have one participant active in each Network at any given time. There are a couple of ways you could do this,

Option 1:
1. Increase *initial_recruitment_size* to the same value as the number of Networks
2. Leave recruitment as it is, recruiting one more participant after each one finishes.

Option 2:
1. Increase *initial_recruitment_size* to the same value as the number of Networks
2. Change recruitment so you recruit participants in batches the same size as the number of Networks, recruiting each batch after everyone in the previous batch has finished.

Option 1 is maximally efficient, but it becomes very difficult to make sure two participants don't get put in the same Network at the same time. Option 2 is easier to implement, but a little inefficient.