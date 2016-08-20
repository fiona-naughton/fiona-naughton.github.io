---
layout: posts
title: Wrapping the Universe
date: 2016-07-13
---

Hello again! I was away most of last week, hence the delay for this post, but I'm finally here to bring you your next dose of explanatory kitties!


![](/images/6SpaceKitty.png){: .kitty} 


This post I'm going to talk about what I've been doing for the second part of my project: a container for dealing with Umbrella Sampling (US) simulations (if you can't remember what US is all about, you can go back and read about it in [this post](http://fiona-naughton.github.io/blog/2016/05/25/What-is-this-MD-thing-anyway)). 

Originally, I envisioned this as a more specific 'Umbrella Class' for storing the simulation for each umbrella window and the associated restraint constants, restrained values, etc. However, it was pointed out that a more general way of storing a collection of Universes and associated *metadata* could be useful in a range of situations, including for US. *Metadata* here will likely largely be other data describing a simulation, but not associated with individual frames (as for 'auxiliary' data) - like the value of the reaction coordinate restrained to in US.

---

So here's the general idea of what we want our general 'Universe collection' container to do:


![](/images/6UniverseCollectionKitty.png){: .largekitty} 


- **Store a set of MDAnalysis Universes**, allowing iteration over all, access to each by index or name, and to add or remove. In the US case, there'll be one Universe for each of our umbrella windows.

- **Deal with auxiliary data**: check what auxiliaries all Universes have in common (e.g. to check all US universes have a 'pull force' auxiliary before we try running WHAM), and allow to add or alter (auxiliaries can be altered directly through each universe, but where set-up for each Universe is the same it will be convenient to do at once).

- **Deal with 'metadata'**: as for auxiliaries, check what metadata all Universes have in common (again, checking for analysis), and allow to add/alter 

---
We already have a place for auxiliary data, but we still need somewhere to store the metadata. We considered a couple of options, listed below. I made a quick example of how each might work in practice, which you can check out [here](https://gist.github.com/fiona-naughton/740f3911bebc16ca2b00614311ee8829). 

<br/>
![](/images/6DataStoreKitty.png){: .mediumkitty} 
<br/>

1. **Directly in the container**, as an attribute (alongside a dictionary of universes). We'd need to make sure we could match up the data with it's universe, and we can't associate metadata with a universe independently of the container.

2. **Part of the Universe**, in a new 'data' attribute. We'd now be able to use metadata without a Universe collection container, though we'll need to make sure our naming is consistent within it.

3. **In a newly-written wrapper for Universe**, to avoid cluttering up the universe itself. We'd have a new class that would basically store a Universe and its set of metadata. The Universe collection container would now store a dictionary of wrapped-universes.

4. **In an existing wrapper: [MDSynthesis](http://mdsynthesis.readthedocs.io/en/master/index.html)\* Sims**, which would allow storage of a universe and associated data (in a *categories* dictionary) without us having to write a wrapper ourselves. With MDSynthesis, we'd also be able to store our set of Sims as a *Bundle*, which already provides a lot of the functionality we want for our container. However, it requires MDSynthesis be installed to work.

    <sub>\**MDSynthesis is built on top of [**datreant**](http://datreant.org/), a Python library that allows us to identify, filter or group files across our file system from a Pythonic interface, with the help of tags and categories we can assign. It has persistent storage, so we can access our files/tags/categories again any time we start a new Python session. MDSynthesis builds on this for MD simulations, allowing us to directly reload an MDAnalysis universe without having to respecify the appropriate files and other arguments every session, and generally useful for keeping track of all our simulations. I recommend checking it out!.*</sub>

---

Of these, using MDSynthesis seems the most attractive option: the existing features do everything we want (and more). In fact, we needn't bother with making a container at all - we can just work directly with the existing Bundles. 


![](/images/6BundleKitty.png){: .mediumkitty} 


But if MDSynthesis does everything, what is there still to do? Firstly, since auxiliaries are something new that I'm trying to add, MDSynthesis doesn't deal with them yet - we can still add and read auxiliaries from a Sim's universe directly, but they won't be stored with the rest of the universe so we'll have to manual reload them each time. I've already added a couple of methods to the auxiliary stuff that'll allow us to, from a Universe, collect the necessary information from each added auxiliary that would allow us to replicate it. Next, we can add to MDSynthesis to be able to store this and reload our auxiliaries in any subsequent sessions.


![](/images/6AuxStoreKitty.png){: .largestkitty}


I'm also working on a function to let us get our Bundle set up more easily - passing in our simulations as trajectory files, Universes or Sims, and providing any auxiliaries or metadata we want to add straight way, and returning the appropriate Bundle. You can go have a look at the  [WIP on Github](https://github.com/MDAnalysis/mdanalysis/pull/900)!

---

I'll be back next post with another What I Learned About Python, so look forward to that coming soon!
