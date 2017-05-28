**********
NDlib-REST
**********


The simulation facilities offered by NDlib are specifically designed for those users that want to run experiments on their local machine. 
However, in some scenarios, e.g. due to limited computational resources or to the rising of other particular needs, it may be convenient to separate the machine on which the definition of the experiment is made from the one that actually executes the simulation.

In order to satisfy such needs, we developed a RESTfull service, NDLIB-REST, that builds upon NDLIB an experiment server queryable through API calls.

**Project Website**: https://github.com/GiulioRossetti/ndlib-rest


=========
Rationale
=========

The simulation web service is designed around the concept of **experiment**. 

An experiment, identified by a unique identifier, is composed of two entities: (i) a network and (ii) one (or more) configured models. 

Experiments are used to keep track of the simulation definition, to return consecutive model iterations to the user and to store - locally on the experiment server - the current status of the diffusion process.


The last action, involving the destruction of the experiment, is designed to clean the serialization made by the service of the incremental experiment status. 

If an experiment is not explicitly destroyed its data is removed, and the associated token invalidated, after a temporal window that can be configured by the service administrator. 

NDlib-REST is shipped as a Docker container image so to make it configuration free and easier to setup. Moreover, the simulation server is, by default, executed within a Gunicorn instance allowing parallel executions of multiple experiments at the same time. 

NDlib-REST is built using Flask and offers a standard online documentation page that can also be directly used to test the exposed endpoints both configuring and running experiments.


-------------
API Interface
-------------

As a standard for REST services, all the calls made to NDlib-REST endpoints generate JSON responses. 

The APIs of the simulation service are organized in six categories so to provide a logic separation among all the exposed resources. 
In particular, in NDlib-REST are exposed endpoints handling:


By default, when such parameter is not specified, all the models are executed and their incremental statuses returned. 

A particular class of endpoints is the **Exploratories** one. 
Such endpoints are used to define the access to pre-set diffusion scenarios. 
Using such facilities the owner of the simulation server can describe, beforehand, specific scenarios, package them and make them available to the service users. 

From an educational point of view such mechanism can be used, for instance, by professors to design emblematic diffusion scenarios (composed by both network and initial node/edge statuses) so to let the students explore their impact on specific models configurations (e.g. to analyze the role of weak-ties and/or community structures).

============
Installation
============

The project provides:

- The REST service: ndrest.py
  - Web API docs: http://127.0.0.1:5000/docs
  - Unittest: ndlib-rest/service_test
- Python REST client: ndlib-rest/client

------------------
REST service setup
------------------

Local testing

.. code-block:: python

    python ndrest.py


Local testig with multiple workers (using [gunicorn](http://gunicorn.org/) web server):

.. code-block:: python

    gunicorn -w num_workers -b 127.0.0.1:5000 ndrest:app


In order to change the binding IP/port modify the apidoc.json file.
To update the API page run the command:

.. code-block:: python

    apidoc -i ndlib-rest/ -o ndlib-rest/static/docs


----------------
Docker Container
----------------

The web application is shipped in a Docker (https://www.docker.com/) container.
You can use the Dockerfile to create a new image and run the web application using the gunicorn application server.

To create the Docker image, install Docker on your machine.
To create the image execute the following command from the local copy of the repository

.. code-block:: python

    docker build -t [tagname_for_your_image] .

The command create a new image with the specified name. Pay attention to the **.** a the end of the command.

.. code-block:: python

    docker run -d -i -p 5000:5000 [tagname_for_your_image] 

This command execute a container with the previous image, bind the local port 5000 to the internal port of the container. 
The option **-d** make the container to run in the background (detached)

To have a list of all active container

.. code-block:: python

    docker ps -al


To stop a container 

.. code-block:: python

    docker stop container_name


==============================
Configuration and Dependencies
==============================

------------
Configurtion
------------

In ndrest.py are specified limits for graph sizes. 
In particular it describes the minimum and maximum numbers of nodes (for both generators and loaded networks) as well as the maximum file sizes for upload.

.. code-block:: python

    app.config['MAX_CONTENT_LENGTH'] = 50 * 1024 * 1024  # 50MB limit for uploads
    max_number_of_nodes = 100000
    min_number_of_nodes = 200 # inherited by networkx


- The "complete graph generator" endpoint represents the only exception to the specified lower bound on number of nodes: such model lowers the minimum to 100 nodes. Indeed, the suggested limits can be increased to handle bigger graphs.
- When loading external graphs nodes MUST be identified by integer ids.

------------
Dependencies
------------

Python 2.7 required dependencies:

- flask==0.12
- flask-cors==3.0.2
- flask_restful==0.3.5
- flask_apidoc==1.0.0
- networkx==1.11
- numpy==1.12.0
- scipy==0.18.1
- ndlib==1.0b
