# MARVELO meets FISSON
## FISSION - **F**ailover in w**I**rele**SS** d**I**stributed c**O**mputi**N**g

- [MARVELO meets FISSON](#marvelo-meets-fisson)
  * [FISSION - **F**ailover in w**I**rele**SS** d**I**stributed c**O**mputi**N**g](#fission-----f--ailover-in-w--i--rele--ss---d--i--stributed-c--o--mputi--n--g)
  * [Terminology and rough overview](#terminology-and-rough-overview)
    + [A **Client**](#a---client--)
    + [A **Server**](#a---server--)
    + [A **Job**](#a---job--)
  * [Setting up Client and Servers](#setting-up-client-and-servers)
    + [Client](#client)
    + [Server](#server)
  * [Configuring your Network](#configuring-your-network)
    + [Starting a Project and Network](#starting-a-project-and-network)
  * [Writing the first Jobs](#writing-the-first-jobs)
  * [Writing a custom Pipe](#writing-a-custom-pipe)
  * [Edit config.py](#edit-configpy)
  * [Acknowledgment:](#acknowledgment-)


This project is a Python-based solution for distributing a set of dependent tasks onto multiple devices and having a failover mechanism in place to prevent outage of your cluster.  
**MARVELO** uses **FISSION** to implement failover by making us of [**dispy**](http://dispy.sourceforge.net/)s heartbeat, monitoring and job cluster functionality.

For a quick ramp-up, please have a look at the [video](https://drive.google.com/file/d/1FhXeoun40hiU3WRceljSOrwO5wBEkZ5Y/view?usp=sharing) and [documentation](https://marvelo.readthedocs.io/en/latest)

---

## Terminology and rough overview

The four most important terms in **MARVELO** are: **Network**, **Client**, **Server** and **Job**.

> Coming from MARVELO:
>
> - Server -> Client
> - Daemon -> Server
> - Algorithm -> Job

Within a **Network** there is **one Client**, some number of **Server**s and a number of **Job**s.

### A **Client**

- **distributes** Jobs and their associated Files.
- **monitors** all Servers.
- **restarts** the Jobs of a Server within the Network in case of its failure.

### A **Server**

- **executes** provided Jobs
- **receives** input data from other Servers
- **sends** output data to other Servers

### A **Job**

- **defines** a program and the Jobs it communicates with

## Setting up Client and Servers

**Your Python version has to be 3.6 or greater!**

### Client

The Client is where you should copy the repository to and is the "supervisor" of your cluster, monitoring all servers and managing the failover.

**FISSION** depends on named pipes and therefore can not run on Windows.

**FISSION** is installable via pip. We recommend setting up a virtual environment and activating it:

```bash
python3 -m venv <my-virtual-environment>
source <my-virtual-environment>/bin/activate
```

Next install fission via pip:

```bash
pip3 install <fission-root-directory>
```

When developing we recommend to make an editable install by adding the `-e` flag.  

That's it, the Client is ready to run!

### Server

If you haven't already, install ansible. It is used to set up your servers automated, you only have to tweak a few setting.

> **On Windows**:
> There is no Windows version of ansible, but on Windows 10, you can simply install WSL (e.g., [Ubuntu](https://www.microsoft.com/de-de/p/ubuntu-1804-lts/9n9tngvndl3q)),
> which gives you a Linux subsystem running on top of your Windows machine.
> Then, you can open the Linux terminal and follow the instructions below, as if you were on Linux.

> **On Ubuntu**:
> [ansible documentation](http://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#latest-releases-via-apt-ubuntu)  
> Commands:
>
>```bash
> sudo apt-get update
> sudo apt-get install software-properties-common
> sudo apt-add-repository ppa:ansible/ansible
> sudo apt-get update
> sudo apt-get install ansible make
> ```

> **On macOS**
>
> ```bash
> brew install ansible
> ```

You have to set all IPs of your servers in the ```ansible/inventories/production/hosts.yml``` file.
If the ip you are connecting to for installation differs from the one you wish you dispynode to serve just add a `dispy_ip` to the host.
You will find two examples, follow them.  
Now head to ```ansible/inventories/production/group_vars/fissionservers/setting.yml``` and change ```ansible_user``` to what ever user you set up on your servers. At this point you can also specify any additional python3 packages you wish to install for your jobs.

Run ```make servers_pw```. This will run the ansible playbook and ask for an ssh password and a sudo password.  
If you already copied an id to the server by running ```ssh-copy-id <server-address>``` (you maybe have to run ```ssh-keygen``` first to create a key), you can run ```make servers``` and it will only ask for a sudo password.

The following commands are also supported by the make file:

- `make start` starts the service running a dispynode on each node (`make servers starts the service automatically)

- `make stop` stops the service running a dispynode on each node

- `make renew` copies a new version of the fission.service file to the nodes and restarts the fission service.
  This is useful if you want to change the ip the dispynode serves or changed something in the `ansible/roles/setup/templates/fission.service.j2` file.
  E.g. you could ad `-d` flag to run the nodes in debug mode if you are experiencing issues.

**Make sure you run the same dispy versions on the client and all nodes, otherwise they will not detect each other.**

Your Network is ready now.

## Configuring your Network

### Starting a Project and Network

To start a **FISSION** project just run:

```bash
fission_admin.py <my-project> startproject
```

This will result in the following structure:

```
<my-project>/
├── manage.py
└── shared
    ├── __init__.py
    ├── jobs
    │   ├── __init__.py
    │   └── jobs.py
    ├── nodes
    │   ├── __init__.py
    │   └── nodes.py
    ├── pipes
    │   ├── __init__.py
    │   └── pipes.py
    └── source
```

By default a `shared` network is started.
It contains no `config.py` file and there for can't be directly executed.  
It's purpose is to hold jobs, nodes, pipes and files you may want to use in multiple networks within you project.

Next head into ```<my-project>``` and start a network:

```bash
cd <my-project>
python3 manage.py <my-network> startnetwork
```

The folder structure changes to:

```
<my-project>/
├── manage.py
├── shared
│   ├── __init__.py
│   ├── jobs
│   │   ├── __init__.py
│   │   └── jobs.py
│   ├── nodes
│   │   ├── __init__.py
│   │   └── nodes.py
│   ├── pipes
│   │   ├── __init__.py
│   │   └── pipes.py
│   └── source
└── <my-network>
    ├── config.py
    ├── __init__.py
    ├── jobs
    │   ├── __init__.py
    │   └── jobs.py
    ├── nodes
    │   ├── __init__.py
    │   └── nodes.py
    ├── pipes
    │   ├── __init__.py
    │   └── pipes.py
    └── source
```

Notice how our newly created network provides a `config.py` file, meaning we can actually run it.

We'll return to the config later, first a quick explanation of what's what:

- ```config.py``` is the part of your network where you define a few general things, but also how and with what pipes your jobs are connected.

- ```jobs``` is a directory / python module where you are meant to define your jobs in.

- ```nodes``` is a directory / python module where you are meant to define your nodes in.

- ```pipes``` is a directory / python module where you are meant to define, you guessed it, your pipes in.

- ```source``` is a directory where you are meant to put your job's dependencies like executables or in general files you job depends on.

So it's time to think about our first Network, we will start of by building this:

```mermaid
graph LR
  subgraph network
    2(SourceJob 2) -->|101| 5(JoinJob 5)
    3(SourceJob 3) -->|102| 5(JoinJob 5)
    4(SourceJob 4) -->|103| 5(JoinJob 5)
    5(JoinJob 5)-->|105| 6(ReduceJob 6)
  end
```

## Writing the first Jobs

Assuming you already started a project and network let's start by implementing our first job:

The following is al written to `jobs/jobs.py` in your network.

The `SourceJob`:

```python
from fission.core.jobs import PythonJob
import time

class SourceJob(PythonJob):
    def run(self):
        value = 1
        while True:
            print(f"{self.id} -> {value}")
            yield value
            time.sleep(1)
            value += 1
```

We start by importing `PythonJob` from `fission.core.jobs`.
This is necessary for all following jobs but I will take this as given.

We will use the `time` module to limit the output of our job to once per second.

Let's have a look at the following lines one by one:
First we start a new class and inherit from `PythonJob`.

```python
class SourceJob(PythonJob):
```

Next we override the `run` method, only taking `self` as an argument as this is a source job and there not expected to receive an data.  
We also initiate a counter name `value` to one. You may wonder why we this when `run` is called per pass, we'll see in a second.

```python
    def run(self):
        value = 1
```

Now we define a `while True`-Loop, add a small print statement for debugging purposes and `yield` values.  
So we are making a `generator`. This answers why we for one define a `while True`-Loop and why we can simply define `value` in `run`.  
Overriding `run` with a `generator` is only possible for source jobs (jobs with no inputs) and allows to easily keep context between calls without having to us object attributes(`self`).

Then we sleep for a second, increment our counter and loop for ever.

We in our flow chart above each job only has one output by this does not apply to all.  
How multiple outputs work will be discussed in the [documentation](https://git.cs.upb.de/js99/afdwc/-/wikis/Examples#demo-network).

```python
        while True:
            print(f"{self.id} -> {value}")
            yield value
            time.sleep(1)
            value += 1
```

Lets head to the `JoinJob`:

```python
class JoinJob(PythonJob):
    def run(self, in_1, in_2, in_3):
        result = in_1 * in_2 * in_3
        print(f"{self.id} -> {result}")
        return result
```

So this time we take 3 arguments as we have 3 inputs.
We calculate the product, print it for debugging and return it.  
Easy enough right? So let's make our last job:

The `ReduceJob`:

```python
class ReduceJob(PythonJob):
    def run(self, L):
        result = sum(L)
        print(f"{self.id} -> Sum over {L} is {result}")
```

It might be a little bit confusing, that we call sum on our only argument as just return a integer in `JoinJob`.  
We will see what's up with that when we make a custom pipe and this is what we are going to do next as our jobs are done.

## Writing a custom Pipe

Now we head to `pipes/pipes.py` in your network.

We write:

```python
from fission.core.pipes import PicklePipe

class ReducePipe(PicklePipe):
    BLOCK_COUNT = 5
```

First we import `PicklePipe` from `fission.core.pipes`.
This pipe uses the pickle stack protocol, meaning you can throw nearly any Python object at it and it will bring it to the other end.

We also define a new pipe called `ReducePipe` and override the class attribute `BLOCK_COUNT` to 5.
Setting this parameter means that **FISSION**  buffers 5 inputs on this pipe before passing them at once as a list.

## Edit config.py

I will just throw the complete config file at you and afterwards step through its:

```python
# ----------------------------------------------------
# FISSION CONFIG FILE
# ----------------------------------------------------

from fission.core.pipes import PicklePipe
from fission.core.nodes import BaseNode

from demo_network_v1.jobs.jobs import SourceJob, ReduceJob, JoinJob
from demo_network_v1.pipes.pipes import ReducePipe

# Enter the user on the remote machines
USER = "pi"

# The directory all the action is done on the remote machines
REMOTE_ROOT = f"/home/{USER}/fission/"

# Logging
LOG_LEVEL = "INFO"
LOG_FILE = "FISSION.log"

# Enter the clients ip within the network, must be visible for nodes
CLIENT_IP = "192.168.4.1"

# Debug window
# Redirects stdout to console
DEBUG_WINDOW = True

# A list of jobs to be executed within the network.
# This can be an instance of BaseJob or any of its subclasses.
JOBS = [
    SourceJob(
        outputs=[
            PicklePipe(101),
        ]
    ),
    SourceJob(
        outputs=[
            PicklePipe(102),
        ]
    ),
    SourceJob(
        outputs=[
            PicklePipe(103),
        ]
    ),
    JoinJob(
        inputs=[
            PicklePipe(101),
            PicklePipe(102),
            PicklePipe(103),
        ],
        outputs=[
            ReducePipe(105),
        ]
    ),
    ReduceJob(
        inputs=[
            ReducePipe(105),
        ]
    ),
]

# A list of nodes to be included in the network
# This can be an instance of BaseNode or Multinode
# or a path to csv file defining nodes (see documentation)
NODES = [
    BaseNode("192.168.4.2"),
    BaseNode("192.168.4.3"),
    BaseNode("192.168.4.4"),
]

# Whether or not files (dependencies) for every job should be copied
# to every node. If not files will be copied before the job starts.
# Turning it on results in higher setup time when starting the network
# but reduces the delay in case of failover. It also demands more disk
# space on the nodes.
PRE_COPY = False

# Optional XML file defining the whole network or a part of it
# This is not recommended and only exists for specific use cases
XML_FILE = None
```

First we import `PicklePipe` to connect our `SourceJob`s with out `JoinJob` and `BaseNode` to define the nodes in our network.  
Next we import our jobs and pipe from our network, in my case this is `demo_network_v1`.  

`USER` is set to "pi" in our example as we work with Raspberry Pis and this is the default user. Change this to what ever your user is.

`REMOTE_ROOT` defines where **FISSION**'s root directory should be on the remote machines, this does not have to changed.

`LOG_LEVEL` is a string indicating a log level.
Every level as provided in the `logging` module is fine but we recommend "INFO" or "DEBUG" if something is not working.

`LOG_FILE` is the path to the log file. By default this is "FISSION.log" in the project folder.

`CLIENT_IP` is the IP of the client and must be reachable by the servers.

`DEBUG_WINDOW` indicates whether a debug window should be launched attaching to the `stdout` and `strerr` of all jobs.  
This defaults to `False` but for this demo we set it to `True`.

`JOBS` is where it becomes interesting:  
You basically create an object for every job you need in this list and pass pipe objects to the inputs and/or outputs.  
You pass a id to the pipe (this has to be an integer) to identify it.
**FISSION** caches all pipe objects so calling a pipe multiple times with the same id will provide the exact same object.  
In addition the order in which you pass the pipes to a job matters.
So the pipe with id 101 will be passed to to the `in_1` argument of `JoinJob`, id 102 to `in_2` and so on.  

The reason we picked a relatively high number for the ids is because we are going to extend this example in the documentation and having a prefix to the pipe ids helps to keep track.

If you make any mistakes like having more than one end to a pipe or only one you will receive according errors.
Also once a pipe has been initiated with an id you will receive and error you try to call the same id with a different type of pipe.

`NODES` is just a list of `BaseNodes` in this case.

`PRE_COPY` is already explained quite extensively in its comment.

`XML_FILE` has a very specific use case and therefore won't be explained at this point.

So what's next?

Before deploying the network lets make a test run:  
`python3 manage.py demo_network_v1 simulate`

This should run without errors and show the print statements we defined earlier.  
The `ReduceJob` is set of by one "round" due to the buffering going on while checking SQN's.  
Hit "Ctrl + C" to cancel the simulation.  

The simulation created an additional folder in your network called "simulation". It is cleared before each simulation so store data somewhere else before rerunning the simulation if you want to.
You will find a folder for each job named after it id.
`info.txt` may help to find the right one but in general they are just enumerated in the order they are define in the config's `JOBS`.
There is also a additional log file how it would be created on a node with DEBUG log level.

Assuming you developed on the machine which will act as a client just run:  
`python3 manage.py demo_network_v1 run`  

If you didn't, use e.g. `scp` to copy the project to the client, connect to it via `ssh -X user@address` to allow for additional consoles to be opened, activate you venv if there is one and then execute the command.

Congratulations, these are the very basics of the **FISSION** framework.
All files for this example can be found under `examples/demo_network_v1` in the git repository.


[comment]: <> (This example will be extended to the following the the [examples section of the documentation]https://git.cs.upb.de/js99/afdwc/-/wikis/Examples#demo-network)

```mermaid
graph LR
  subgraph network
    2(SourceJob 2) -->|101| 5(JoinJob 5)
    3(SourceJob 3) -->|102| 5(JoinJob 5)
    4(SourceJob 4) -->|103| 5(JoinJob 5)
    5(JoinJob 5)-->|105| 6(ReduceJob 6)
  end
  subgraph locals
    1(InteractiveJob 1) -->|104| 5(JoinJob 5)
    5(JoinJob 5) -->|204| 7(CSVSinkJob 7)
    5(JoinJob 5) -->|205| 7(CSVSinkJob 7)
    4(SourceJob 4) -->|203| 7(CSVSinkJob 7)
    3(SourceJob 3) -->|202| 7(CSVSinkJob 7)
    2(SourceJob 2) -->|201| 7(CSVSinkJob 7)
    6(ReduceJob 6) -->|301| 8(CSVSinkJob 8)
  end
```


## Acknowledgment:

- [Deutsche Forschungsgemeinschaft - DFG-FOR 2457](https://www.uni-paderborn.de/asn/)

![img](https://www.uni-paderborn.de/fileadmin/_processed_/9/2/csm_ASNLogo_c443ce161b.png)

- Teamprojekt FISSION at Paderborn University

## contact:
Haitham Afifi

haitham.afifi@ieee.org

haitham.afifi@hpi.de

Holger Karl

holger.karl@hpi.de
