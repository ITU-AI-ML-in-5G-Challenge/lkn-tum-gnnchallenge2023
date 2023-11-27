# LKN: Graph Neural Networking Challenge 2023


## Repository structure

- ckpt: folder containing the saved model weights for each epoch of the submitted model
- data: folder containing the preprocessed datasets for the train and test datasets
- docker: folder containing the docker-compose file for required Docker environment
- tensorboard: tensorboard path for the submitted model
- verification_files: folder containing files used by the predict.py script to verify the generated submissions
- [models.py](models.py): python module which contain our model
- [train.py](train.py): python script that can be used to train the baseline projects
- [predict.py](predict.py): python script used to generate predictions for the test dataset and challenge submissions


## Quickstart

### Requirements

In order to reproduce our measurement results, a working Docker version 24.0.2 and Docker Compose version v2.23.0 is required.

### Training the model

```bash
# Navigate to docker folder
cd docker

# Bring up the service 

# If you're training on CPU
docker compose up -d train-cpu

# If you're training on GPU
docker compose up -d train-gpu
```

Training will start after the service is up. The training status can be followed via the container logs and through the tensorboard. 
The model checkpoints will be saved to `ckpt` directory during training.

### Evaluating the model

Preprocessed test dataset can be found under `data/data_test` folder. For generating predictions, we follow the guidelines provided by the event organizers.

### Default features

The features used by default in the baseline are the following:
-  `flow_traffic`: the average traffic bandwidth per flow in bps
-  `flow_packets`: the number of generated packets per flow
-  `flow_packet_size`: the size of the generated packets per flow
-  `flow_type`: two-dimensional one-hot encoded feature used to identify the flow type of each flow
   -  `[1, 0]` indicates the flow is a Constant Bit Rate (CBR) flow
   -  `[0, 1]` indicates the flow is a Multi Burst (MB) flow
- `flow_length`: length of the physical path followed by each flow
- `link_capacity`: for each link, it indicates its bandwidth in bps
- `link_to_path`: for each flow, it indicates the links forming its path, in order
- `path_to_link`: for each link, it lists the flows that traverse it. It also includes the position of the link in each flow's path. For a given link the same flow can appear more than once if the link is traversed more than one in the same flow path

The target metric is `flow_delay`, the mean packet delay per flow, in mbps.

From these features, `flow_traffic`, `flow_packets`, `flow_packet_size` and `link_capacity` are normalized using min-max normalization.

Additionally, the following feature is computed at run time during training:
- `load`: for each link, the expected load is computed by combining the `flow_traffic` and `path_to_link` features.

Finally, three additional features are extracted from data in order to identify the samples. These are not used in the model itself, but are left for more insight during debugging and to correctly identify each node:
- `sample_file_name`
- `sample_file_id`
- `flow_id`

### Extended features

We add the following features:

-  `flow_ipg_mean`: mean inter packet gap per flow
-  `flow_ipg_var`: variance of inter packet gap per flow
-  `flow_ipg_percentiles`: list of every 10th percentile inter packet gap per flow
-  `sending_duration`: the sending duration of a flow in seconds (not used in the model)

Provided `data_generator.py` script is used and extended for the extraction of above features. We also provide the edited script under `data` folder in this repository.

### Model

We rely on the baseline model provided by the organizers and only extend the path hidden states with the new extended features as described above.

### Additional Notes

For training, we use the CBR - MB dataset, and preprocess it with the provided `data/data_generator.py` script. 
Our training script takes 200 samples from the dataset as validation dataset and keeps the rest for training.
Unfortunately, due to shuffling of the dataset, the reproduced results may not be the same, however, the obtained results should be very similar.
For the final model selection, we chose the model with the lowest validation MAPE.


### Answers to Questions

- Programming Language Used: Python
- ML Programming Framework: Tensorflow
- Dependencies: Docker version 24.0.2, Docker Compose version v2.23.0
- Our framework relies purely on the baseline RouteNet implementation.
- With our setup, one epoch took around 12 minutes.
- We used 1 Tesla T4 GPU during training.
