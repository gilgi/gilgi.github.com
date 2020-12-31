.. title: Code
.. slug: code
.. date: 2019-12-05
.. tags: 
.. category: 
.. link: 
.. description: 
.. type: text

Research software
=================
<div class="container-fluid"><div class="row"><div class="col-md-4 col-xs-12">
###lib5c

[Paper](https://www.ncbi.nlm.nih.gov/pubmed/30904376) |
[Source](https://bitbucket.org/creminslab/lib5c) |
[Docs](https://lib5c.readthedocs.io/) |
[PyPI](https://pypi.org/project/lib5c/) |
[Docker](https://hub.docker.com/r/creminslab/lib5c)

lib5c is a library for Chromosome Conformation Capture Carbon Copy (5C) data
analysis I wrote during my PhD. It forms the backbone of a good portion of
[my lab's](http://creminslab.com/) computational infrastructure. In addition to
an easily-customizable 5C analysis pipeline, lib5c also provides statistical
utilities and custom visualizations we use in nearly every project.
</div>
<div class="col-md-4 col-xs-12">
### hic3defdr

[Paper](https://www.ncbi.nlm.nih.gov/pubmed/32859248) |
[Source](https://bitbucket.org/creminslab/3defdr-hic) |
[Docs](https://hic3defdr.readthedocs.io/) |
[PyPI](https://pypi.org/project/hic3defdr/) |
[Docker](https://hub.docker.com/r/creminslab/hic3defdr)

hic3defdr is a genome-scale differential loop caller for Hi-C data, useful for
quantitatively understanding how loops in the genome change across conditions.
It implements a negative binomial statistcal model that accounts for technical
biases in Hi-C datasets. It also comes with a Hi-C dataset simulator that can be
used for benchmarking.
</div>
<div class="col-md-4 col-xs-12">
### treeshaker

[Source](https://github.com/sclabs/treeshaker) |
[PyPI](https://pypi.org/project/treeshaker/)

treeshaker is a tree-shaking library for Python projects - it identifies the
minimal subset of Python modules required by a specific target module, and
copies those files to an output directory. This is useful when your code is
developed in a large monolithic library that you want to keep private while
still releasing specific functionality to the public.
</div>
<div class="col-md-4 col-xs-12">
### iceflow

[Source](https://github.com/sclabs/iceflow) |
[PyPI](https://pypi.org/project/iceflow/)

iceflow is a meta-framework for training and evaluating neural networks I wrote
around [TensorFlow](https://www.tensorflow.org/) and [Sonnet](https://github.com/deepmind/sonnet)
back in 2017 when re-using architectures and running reproducible benchmarks was
considerably more difficult than it is today. iceflow was designed to make it
easy to connect reusable datasets and neural network models in a plug-and-play
manner, as shown off in example implementations of an [MLP](https://github.com/sclabs/iceflow/tree/master/examples/mlp)
and an [autoencoder](https://github.com/sclabs/iceflow/tree/master/examples/autoencoder).
</div>
<div class="col-md-4 col-xs-12">
### projectile

[Source](https://github.com/sclabs/projectile) |
[PyPI](https://pypi.org/project/projectile/)

projectile is a tile-on-demand tile server I built using [PIL](https://pillow.readthedocs.io/en/stable/)
and [Tornado](https://www.tornadoweb.org/en/stable/). projectile exposes parts
of a large image or matrix over a tile server-like API without requiring a tiling
or chunking step. This makes it attractive for use cases where the image data
changes frequently and the median number of expected requests per tile is close
to zero - a regime very different from typical map tiling.
</div></div></div>

Proprietary research tools
==========================
<div class="container-fluid"><div class="row"><div class="col-md-8 col-xs-12">
### hiclite

hiclite is a Python package for genome-wide loop calling in Hi-C datasets. It
re-implements parts of the
[Juicer/HiCCUPS Hi-C processing and loop calling pipeline](https://github.com/aidenlab/juicer),
but features a fast implementation of convolution on banded matrices, allowing
it to run quickly without GPU acceleration. Since it's written in Python, my lab
has been able to tweak hiclite to modify the expected modeling steps and call
stripes in Hi-C data. It has been used in [our 2019 Nature collaboration](https://www.ncbi.nlm.nih.gov/pubmed/31776509)
as well as a manuscript currently under revision.
</div>
<div class="col-md-4 col-xs-12">
### hiczoom

hiczoom is a Python package that provides a custom client for [projectile](https://github.com/sclabs/projectile)
that I built using [D3.js](https://d3js.org). It enables browser-based
visualization of genome-wide folding maps with seamless zooming and panning,
combined with overlayed reference genes, ChIP-seq signal tracks, and annotation
of called loops.
</div></div></div>

Just for fun: webapps and chatbots
==================================
<div class="container-fluid"><div class="row"><div class="col-md-4 col-xs-12">
### scootbot2.0

[Source](https://github.com/sclabs/scootbot2.0)

scootbot2.0 is a custom multi-function Slack chatbot built with [botkit](https://botkit.ai/)
and [Node.js](https://nodejs.org/en/). It is the spiritual successor to the
original [ScootBot](https://github.com/sclabs/scootbot) which was a Skype
chatbot written in C#. scootbot2.0 provides an interface to 14 different APIs
including [the gilgi.org cloud service](https://cloud.gilgi.org).
</div>
<div class="col-md-4 col-xs-12">
### sitestatus

[Source](https://github.com/sclabs/sitestatus) |
[Live example](https://status.gilgi.org)

sitestatus is a site status webapp built with [Django](https://www.djangoproject.com/)
that provides a simple way to display automatically-updating status information
for a list of websites or services that can be managed with a CMS-like interface
and [deployed to Google App Engine](https://github.com/sclabs/sitestatus-nonrel).
I use it to power the [gilgi.org network status page](https://status.gilgi.org).
</div>
<div class="col-md-4 col-xs-12">
### sccms

[Source](https://github.com/sclabs/sccms) |
[Live example](https://romanchickens.gilgi.org)

sccms stands for Simple-Comic-CMS and is a
[Django](https://www.djangoproject.com/)-based webapp that makes it easy to
display a simple webcomic on the web and manage it with a simple CMS-like
interface. I use it to power the [Roman Chicken webcomic](https://romanchickens.gilgi.org).
</div>
<div class="col-md-4 col-xs-12">
### gilgichat

[Source](https://github.com/sclabs/gilgichat-meteor) |
[Live pre-meteor tech demo](http://oldchat.gilgi.org)

gilgichat was a chat webapp I built using [Meteor](https://www.meteor.com/) and
[MongoDB](https://www.mongodb.com/) back in 2016. gilgichat had the basic
functionality of today's Slack, including channels, notification icons, status
indicators, and emoji support. It was based on [an earlier tech demo](https://github.com/thomasgilgenast/gilgichat)
that was built using [Socket.IO](https://socket.io/) in 2012 when websockets
were very cool.
</div>
<div class="col-md-4 col-xs-12">
### sclangular

[Source](https://github.com/sclabs/sclangular)

sclangular was an opinionated modular project structure and development
toolchain for building single-page web applications with AngularJS. I wrote it
back in 2014 before Angular 2 existed. It represents an early attempt to think
about how to design simple, scalable, and flexible development frameworks.
</div>
<div class="col-md-4 col-xs-12">
### shk

[Source](https://github.com/sclabs/shk)

shk was a [Django](https://www.djangoproject.com/)-based webapp I built back in
2012 when all my friends were playing an online strategy game called
[Stronghold Kingdoms](https://www.strongholdkingdoms.com/). We used the app as
a market to trade resources among ourselves, manage debt between players in the
form of IOUs, and track shipments between our cities.
</div></div></div>
