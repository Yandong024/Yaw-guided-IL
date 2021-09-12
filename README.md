# Yaw-guided-IL
Yaw guided imitation learning for autonomous driving in urban environments.

<img src="/home/yandong/Documents/um-data/dong/Github/pic/carla-il.png" alt="carla-il" style="zoom:67%;" />



## Dependences

- pillow==8.2.0
- scikit-image==0.14.0
- scipy==1.1.0
- setuptools==39.0.0  (update to 54.0.0) easy_install carla.egg
- matplotlib==3.0.0
- pygame==2.0.1
- networkx == 2.2
- numpy==1.15.2
- [Carla0.9.11](https://github.com/carla-simulator/carla/releases/tag/0.9.11)



## Collecting the training data

### Methods



<img src="/home/yandong/code_local/benchmark_carla_09/collect_data/town01_spawn_map1.png" alt="town01_spawn_map1" style="zoom: 50%;" />

We collect the expert data in **Carla0.9.11-Town01-four weather** ( 1: clear-noon  weather 3: wet-noon  weather 6: hardrain-noon 8: clear-sunset) conditions.

How to get the Expert experience? Tow methods:

 (1) Human driving; (2) Carla Agent AI;

We use the Carla agent AI to control the ego vehicle (Max speed 20 km/h) to collection data: 

(1) Global planning; (2) Perception; (3) Local planning; (4) PID control ;

We consider the different situation to satisfy the data diversity:

(1) Blocking vehicle and pedestrians;

(2) Follow lane left and right;

(3) Straight, left, right;

(4) All training paths;

### Image and label

The raw image size  600x800 -> clip [115, 510] -> scale[88, 200]

Keep environmental information and limit the amount of calculation

<img src="/home/yandong/code_local/benchmark_carla_09/updata_github/collect_data/image_raw.png" alt="image_raw" style="zoom: 33%;" />

<img src="/home/yandong/code_local/benchmark_carla_09/updata_github/collect_data/image_clip.png" alt="image_clip" style="zoom: 33%;" />

<img src="/home/yandong/code_local/benchmark_carla_09/updata_github/collect_data/image_scale.png" alt="image_scale" style="zoom: 33%;" />

labels: speed, yaw, control (steer, throttle, brake)

### Control noise

To keep the model robust, add noise to the control. Record the real PID control.

10 % noise steer [-0.5, 0.5] uniform noise

### Tricks

(1) Blocking situation, skip frames;

(2) Obstacle vehicle, pedestrians, traffic light, restart data;

(3) Straight (follow lane), skip many frames;

(4) Benchmark training condition tasks are in collected dataset;

### Dataset

Here you can download the data needed to train the model (collected in Carla version 0.9.11) **https://pan.baidu.com/s/1Hwfqsul75J01khvU38BJsg**  password: **bnf5**；Data collected in *Carla town01*, four different weather conditions ('clear_noon', 'wet_noon', 'hardrain_noon', and 'clear_sunset'). A total of 197470 data in the training set and 2782 data in the evaluetion set.  Data distribution

![data_distribution](/home/yandong/Documents/um-data/dong/Github/pic/data_distribution.png)





## Training Model

## Architecture

<img src="/home/yandong/code_local/benchmark_carla_09/updata_github/train_ARYIL/architecture.png" alt="architecture" style="zoom: 50%;" />

The YGAtt architecture consists of three part: Perception, Action, and Speed.

- Speed: predict the vehicle speed based on the perception information. Avoid stopping when restarting;
- Action: cat the perception information and vehicle measurement information to predict the control (steer, throttle, brake);
- Perception: Combination of ResNet34 and Attention to percept the environment;

###  Input data augmenter

<img src="/home/yandong/code_local/benchmark_carla_09/updata_github/train_ARYIL/training_data_auge.png" alt="training_data_auge" style="zoom: 80%;" />

In order to improve the generalization ability of  the model, we use the data augmenter.

(1)Blur; (2)Noise; (3)Dropout; (4)Contrast

### Hyper-parameter

![loss-func](/home/yandong/code_local/benchmark_carla_09/updata_github/train_ARYIL/loss-func.png)

```
epoch = 100
minibatch = 512
learning_rate = 0.001 # The learning rate is halved every 10 epochs	
control_loss_weight = 1.0
speed_loss_weight = 0.1
```



## Benchmark

Benchmark is divided into two types of  scenarios: Training condition and New Town & Weather ( WetCloudyNoon, SoftRainSunset). Navigation tasks are divided  into  five  categories  according  to  the  degree  of  difficulty:  Straight,  One  Turn,  Navigation  empty,  Nav.  Regular(Vehicles:  30,  Pedestrians:  10),  Nav.  Dense  (Vehicles:  50,Pedestrians:  30).  Each  type  of  navigation  task  contains  25 paths,  of  which  the  route  distance  in  Town01  is  not  less than 1 km, and the distance in Town02 is not less than 0.3 km. 



[Supplement video](https://www.youtube.com/watch?v=EuxlEzHYCGs)

