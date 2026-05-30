# MeanFlowNav: Single-Pass Visual Navigation with Refined Average-Velocity Training

> [!NOTE]
> This source code was based on
>
> [IROS25 Combining Flow Matching and Depth Priors for Efficient Navigation](https://github.com/utn-air/flownav)
>
> [NoMaD: Goal Masking Diffusion Policies for Navigation and Exploration](https://general-navigation-models.github.io/nomad/index.html).

## Installation

create a virtual env and clone our repo:

```
conda create -n MFN python=3.10 -y
conda activate MFN
git clone --recurse-submodules --[we will publish our url later]
```

Install the dependencies:

```bash
pip install -r requirements.txt
```

## Download data and weights

1) Download and process the datasets according to [NoMaD&#39;s instructions](https://github.com/robodhruv/visualnav-transformer?tab=readme-ov-file#data-wrangling)
2) Download [DepthAnything-V2 ViT-s weights](https://huggingface.co/depth-anything/Depth-Anything-V2-Small/resolve/main/depth_anything_v2_vits.pth?download=true)

## Training

To train the model, you need to adjust the in the configuration YAML [`meanflownav.yaml`](meanflownav/config/meanflownav.yaml) as:

1) `train` to `True`
2) `depth/weights_path` to the DepthAnythingV2 checkpoint path
3) `datasets/<DATASET>/data_folder`, `datasets/<DATASET>/train` and `datasets/<DATASET>/test` to the folders generated during the (data processing step)[#-download-data-and-weights]

### For origin MeanFlow training, run the following command:

```bash
./train_meanflow.sh
```

### For improved Meanflow training , go to the iMF branch by running:

```
git checkout feat/iMF
```

and then replace the file `ctm_unet.py` in `consistency-policy\consistency_policy` with our modified `ctm_unet.py` in `meanflownav/training`

then run:

```
./train_meanflow.sh
```

If you want to use [wandb](https://wandb.ai/) to log the training, you can set the `use_wandb` flag in the configuration YAML to `True` and  the `project` and `entity` to your desired project and entity (usually your username). Don't forget to login first:

```bash
wandb login
```

## Deployment

### Hardware

##### Chassis

We have deployed on our **self-developed steering wheel chassis**; the website will be shared at [to be announced].

##### Camera

For the camera, we use an Intel RealSense; the link is at:  [realsenseai/realsense-ros: ROS Wrapper for RealSense™ Cameras](https://github.com/realsenseai/realsense-ros)

Follow the tutorial to complete the deployment , then run:

```
roslaunch realsense2_camera rs_camera.launch color_width:=640 color_height:=480 color_fps:=5
```

### Running MeanFlowNav

The ROS2 deployment details can be found at [Flownav](https://github.com/utn-air/flownav);

 here we use the ROS version.

#### Navigation Mode

- Create a topological map using the following script. The topological map is saved in `topomaps/<topomap_directory>`.

```bash
cd deployment/src/navigation
python create_topomap.py --dt 2 --dir path/to/topomaps/<topomap_directory>
```

- Run the Navigation Node:

```bash
cd deployment/src/navigation
export PYTHONPATH="${PYTHONPATH}:$(pwd):$(pwd)/consistency-policy:$(pwd)/py-meanflow"
python navigate-ros1.py --model flownav --dir path/to/topomaps/<topomap_directory> --ckpt path/to/weights
```

#### Exploration Mode

Exploration does not require a topological map.

- Run the Exploration Node:

```bash
cd deployment/src/exploration
python explore-ros1.py --model flownav --dir path/to/topomaps/<explore_topomap_directory> --ckpt path/to/weights
```

#### Run Nav Mode with smoothing

The main branch contains two deployment folders: `deployment` and `deployment-smoothing`. The latter optimizes the processing of the model's output by **utilizing all output waypoints** and applying **CCR smoothing**, which enables smooth output even when inference runs on lower-compute platforms. They are used in exactly the same way — simply change the path to switch between them.
