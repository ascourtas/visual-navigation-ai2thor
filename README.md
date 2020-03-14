# Target-driven Visual Navigation Model using Deep Reinforcement Learning
[Docker](https://hub.docker.com/repository/docker/denmonz/ai2thor)

## Problem Statement

Indoor visual navigation has many challenges. Where do you get the training data? How can you guarantee that your agent will generalize well? How can you ensure your agent will adapt well to dynamic environments? Our project is built off of the deep siamese actor-critic model introduced in the following paper:  <br><br>**[Target-driven Visual Navigation in Indoor Scenes using Deep Reinforcement Learning](https://arxiv.org/abs/1609.05143)**
<br>
[Yuke Zhu](http://web.stanford.edu/~yukez/), Roozbeh Mottaghi, Eric Kolve, Joseph J. Lim, Abhinav Gupta, Li Fei-Fei, and Ali Farhadi
<br>
[ICRA 2017, Singapore](http://www.icra2017.org/)
<br><br>
The agent aims for environmental, goal/target, and real-world generalizability. The original agent utilized the first version of AI2Thor, and we transitioned the agent into the latest version of [AI2Thor](https://ai2thor.allenai.org/).

## Input and Output

The model takes as input an observation and goal image, both of size (224x224x3), and the output is one of the four following actions to be taken by the agent at any time step: "Move Ahead", "Rotate Right", "Rotate Left", "Move Back".
We extract feature maps of the input images via the second to last layer of ResNet 50 (by utilizing [Keras](https://keras.io/applications/#resnet)). These features are dimensionally reduced representations of the original images, and are used for the deep learning component of our model.

<p align="center">
  <img src="/images/networkArchitecture.png" data-canonical-src="/images/networkArchitecture.png" width="450"/>
</p>

## Setup
This code is implemented in [Tensorflow API r1.0](https://www.tensorflow.org/api_docs/). You can follow the [online instructions](https://www.tensorflow.org/install/) to install Tensorflow 1.0. Other dependencies ([h5py](http://www.h5py.org/), [numpy](http://www.numpy.org/), [scikit-image](http://scikit-image.org/), [pyglet](https://bitbucket.org/pyglet/pyglet/wiki/Home)) can be install by [pip](https://pypi.python.org/pypi/pip): ```pip install -r requirements.txt```. This code has been tested with Python 2.7 and 3.5.

## Scenes
To facilitate training, we provide [hdf5](http://www.h5py.org/) dumps of the simulated scenes. Each dump contains the agent's first-person observations sampled from a discrete grid in four cardinal directions. To be more specific, each dump stores the following information row by row:

* **observation**: 300x400x3 RGB image (agent's first-person view)
* **resnet_feature**: 2048-d [ResNet-50](https://arxiv.org/abs/1512.03385) feature extracted from the observations
* **location**: (x, y) coordinates of the sampled scene locations on a discrete grid with 0.5-meter offset
* **rotation**: agent's rotation in one of the four cardinal directions, 0, 90, 180, and 270 degrees
* **graph**: a state-action transition graph, where ```graph[i][j]``` is the location id of the destination by taking action ```j``` in location ```i```, and ```-1``` indicates collision while the agent stays in the same place.
* **shortest_path_distance**: a square matrix of shortest path distance (in number of steps) between pairwise locations, where ```-1``` means two states are unreachable from each other.

Before running the code, please download the scene dumps using the following script:
```bash
./data/download_scene_dumps.sh
```
We are currently releasing one scene from each of the four scene categories, *bathroom*, *bedroom*, *kitchen*, and *living room*. Please contact me for information about additional scenes.
A ```keyboard_agent.py``` script is provided. This script allows you to load a scene dump and use the arrow keys to navigate a scene. To run the script, here is an example command:
```bash
# make sure the scene dump is in the data folder, e.g., ./data/bedroom_04.h5
python keyboard_agent.py --scene_dump ./data/bedroom_04.h5
```

These scene dumps enable us to train a (discrete) navigation agent without running the simulator during training or extracting ResNet features. Thus, it greatly improves training efficiency. The training code runs comfortably on CPUs (of my Macbook Pro). Due to legal concerns, our THOR simulator will be released later.

## Training and Evaluation
The parameters for training and evaluation are defined in ```constants.py```. The most important parameter is ```TASK_LIST```, which is a dictionary that defines the scenes and targets to be trained and evaluated on. The keys of the dictionary are scene names, and the values are a list of location ids in the scene dumps, i.e., navigation targets. We use a type of asynchronous advantage actor-critic model, similar to [A3C](https://arxiv.org/abs/1602.01783), where each thread trains for one target of one scene. Therefore, make sure the number of training threads ```PARALLEL_SIZE``` is *at least* the same as the total number of targets. You can use more threads to further parallelize training. For instance, when using 8 threads to train 4 targets, 2 threads will be allocated to train each target.

The model checkpoints are stored to ```CHECKPOINT_DIR```, and Tensorboard logs are written in ```LOG_FILE```. To train a target-driven navigation model, run the following script:
```bash
# train a model for targets defined in TASK_LIST
python train.py
```

For evaluation, we run 100 episodes for each target and report the mean/stddev length of the navigation trajectories. To evaluate a model checkpoint in ```CHECKPOINT_DIR```, run the following script:
```bash
# evaluate a checkpoint on targets defined in TASK_LIST
python evaluate.py
```

## Our Amendments
* Upgrades the agent's original AI2Thor environment to the latest AI2Thor environment.
* Implemented online feature extraction for agent evaluation.
* Updated keyboard_agent.py to navigate the new environment and capture goal images.

## Future Work
* Cache ResNet50 features for quicker training and evaluation.
* Replace ResNet for RedNet and utilize depth as input to the model.

## Acknowledgements
I would like to acknowledge the following references that have offered great help for me to implement the model.
* ["Asynchronous Methods for Deep Reinforcement Learning", Mnih et al., 2016](https://arxiv.org/abs/1602.01783)
* [David Silver's Deep RL course](http://www0.cs.ucl.ac.uk/staff/d.silver/web/Teaching.html)
* [muupan's async-rl repo](https://github.com/muupan/async-rl/wiki)
* [miyosuda's async_deep_reinforce repo](https://github.com/miyosuda/async_deep_reinforce)

## Citation
Please cite our ICRA'17 paper if you find this code useful for your research.
```
@InProceedings{zhu2017icra,
  title = {{Target-driven Visual Navigation in Indoor Scenes using Deep Reinforcement Learning}},
  author = {Yuke Zhu and Roozbeh Mottaghi and Eric Kolve and Joseph J. Lim and Abhinav Gupta and Li Fei-Fei and Ali Farhadi},
  booktitle = {{IEEE International Conference on Robotics and Automation}},
  year = 2017,
}
```

## License
MIT
