# HyperBoard: A web-based dashboard for Deep Learning

HyperBoard is a simple visualization tool to facilitate hyperparameter tuning for Deep Learning players. It helps you to

- train on a remote server and visualize training curves on the local browser
- update curves in real time
- interactively compare hundreds of training curves for hyperparameter tuning, with filtering and visibility control
- save hundreds of training records on disk and re-load them when needed

HyperBoard resembles [Tensorboard](https://github.com/tensorflow/tensorflow/tree/master/tensorflow/tensorboard) and [Tensorboard for MXNet](https://github.com/dmlc/tensorboard). However, HyperBoard is independent from any specific Deep Learning platform and provides extra functions.

For now, HyperBoard only supports visualization of streaming scalar data (i.e. training curves). Two more features are being develeped:

- visualization of streaming tensors as interactive animation of histograms
- visualization of streaming tensors as interactive animation of scatter plots, using methods like t-SNE, Isomap, etc

**Watch a 3 minutes demonstration on [Youtube](https://youtu.be/sWmVZyRfejc) or [Bilibili](http://www.bilibili.com/video/av8215364/).**

## Installation

```shell
pip install flask flask-httpauth requests numpy # install dependencies
git clone https://github.com/WarBean/hyperboard.git
cd hyperboard/
python setup.py install
```

## Usage

HyperBoard is easy to use. It is composed of three components: **Server**, **Agent** and **Dashboard**. In general, when playing with deep neural networks on some remote server, you can

- launch **HyperBoard Server** on the same remote server
- setup your new hyperparameters, then call **HyperBoard Agent** to register some curves at the **HyperBoard Server** for this time
- start model training and for each K iteration, call **HyperBoard Agent** to send cross entropy, accuracy, BELU, etc to **HyperBoard Server**
- open **HyperBoard Dashboard** on your local browser, watch the curves growing

You can also run HyperBoard Server on your local PC. Currently, HyperBoard Server and Agent have been tested on Mac OS and Ubuntu. HyperBoard Dashboard has been tested on Firefox, Chrome and Safari.

Here's the details:

### Prepare a directory for storing records

```shell
cd ~
mkdir my_records
```

### Launch the server inside the directory

```shell
cd my_records
hyperboard-run --user <your_user_name> --password <your_password>
```

You can use HyperBoard without security authentication by simply using `hyperboard-run` without any arguments. Also use `hyperboard-run -h` for more help info.

### Start model training

For this part, you are recommended to run a simulation script [`example/run_agent.py`](https://github.com/WarBean/hyperboard/blob/master/example/run_agent.py) first. Detailed usage of HyperBoard Agent will be explained below.


### Open dashboard on you local browser

If `hyperboard-run` is run with argument `--local`, open `http://127.0.0.1:5000`, otherwise `http://<your_remote_server_address>:5000`.

The dashboard provides convenient interactive controls on visualization. Feel free to play with them - except the **Delete from Server** button :).

## HyperBoard Agent Usage

### Initialize an agent object:

```python
from hyperboard import Agent
agent = Agent(username = '', password = '', address = '127.0.0.1', port = 5000)
```

`username` and `password` can be omitted if you have launched HyperBoard Server without security authentication. `address` and `port` can be omitted to use default value.


### Register a curve on Server

```python
name = agent.register(hyperparameters, metric, overwrite = False)
```

`hyperparameters` should be a dictionary.

`metric` is a string label indicating the numerical range of the values you send next. Curves sharing the same `metric` will share one y-axis on the dashboard. It helps you properly visualize cross entropy and accuracy in the same graph.

`overwrite = False` lets Agent to ask you for confirm before removing existing records with the same hyperparameter setup.

### Send record during training iteration

```python
agent.append(name, index, value)
```

## Record Storage

Each curve is saved as a file with suffix `.record`, in the very directory where you launcher HyperBoard Server. The content of record file is simple:

```shell
$ head my_records/fd8e3e27e4ef661488932e9a58197d90.record
{"batch size": 256, "corpus": "PennTreeBank", "criteria": "train accu", "learning rate": 0.001, "momentum": 0.9, "num hidden": 300, "optimizer": "Adam"}
accuracy
0 -0.22079159783278235
1 -0.15177436116678278
2 -0.0847468825330926
3 -0.009928149024110766
4 0.07511021349995883
5 0.16286792223048174
6 0.22981841687923243
7 0.25812625639630005
```

The first line is hyperparameters. The second line is metric. Each line below contains the iteration index and the criteria value.

The next time you launch HyperBoard Server, it will reload all `.record` files (except those you delete manually) in to memory.
