# Object Detection in X-ray Images for a Large-scale Security Inspection

### CMPT-733 Programming for Big Data 2 (Simon Fraser University)
### Group members: Nattapat Juthaprachakul, Rui Wang, Siyu Wu, Yihan Lan

#### Goals:
* The goal of this project is to use multiple algorithms, train multiple models, and report on comparative performance of each one. Performance of the models is described by mean average precision scores(Object Detection metrics), also including accuracy and recall scores.
* The program is written in Python.

#### Medium post:
https://medium.com/sfu-cspmp/object-detection-in-x-ray-images-414a4fb06dff

#### Video Presentation:
https://www.youtube.com/watch?v=XMQA4P7eYY0

#### Requirements:
1. Tensorflow Object Detection API (https://github.com/tensorflow/models/tree/master/research/object_detection)
2. Tensorflow 1.15 (Tensorflow 2.0 does not work properly with Object Detection API)
3. NumPy version lower than 1.18 as it will have problem with COCO API
4. COCO API (for evaluation metric and all of our models used are pre-trained on COCO Dataset)
5. Google Cloud Platform for training models and evaluating results with new images (you can use Compute Engineer VM or Google Deeplearning VM)
6. Some other basic library for Data Science like scikin-learn, pandas, numpy, matplotlib, ..

**The Main code is in Tensorflow Object Detection API**

#### Models:
* Download pre-trained models from Model Zoo (https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/detection_model_zoo.md)

* Download Config files for each model (https://github.com/tensorflow/models/tree/master/research/object_detection/samples/configs)

#### In this project, we are using the following models:
1. SSD_mobilenet_v1
2. SSD_mobilenet_v1_fpn
3. SSD_inception_v2
4. SSD_resnet50
5. RFCN_resnet101
6. Faster_RCNN_resnet50
7. Faster_RCNN_resnet101
8. Faster_RCNN_inception_v2

#### Dataset:
* SIXray dataset consists of X-ray images from Beijing subway (https://github.com/MeioJane/SIXray)
* Credit: (@INPROCEEDINGS{Miao2019SIXray,
    author = {Miao, Caijing and Xie, Lingxi and Wan, Fang and Su, chi and Liu, Hongye and Jiao, jianbin and Ye, Qixiang },
    title = {SIXray: A Large-scale Security Inspection X-ray Benchmark for Prohibited Item Discovery in Overlapping Images},
    booktitle = {CVPR},
    year = {2019} })

#### Codes and Pipeline:
* **Obtaining Dataset**: SIXray dataset contains Positive samples(the images containing objects of interest namely prohibited items that we want to localize/classify) and Negative samples (the images containing non-prohibited items) which are later used for evaluating our models. (Downloading SIXray from links above)
* **Preprocessing Dataset**: We use a subset of Positive samples for Training and another subset of Positive samples combining with Negative samples for Testing/Evaluating.

1. In this project, we do not use the whole SIXray dataset due to limitations on computation cost and power. Since we are using a subset of the main dataset, we need to get new labels. Later, these labels are used for Testing/Evaluating of models.

2. We turn image and location file(.xml) of Positive samples into Tensorflow Record for Training.

* **Training**: our training is done by Tensorflow Object Detection API where we can download and install from the link mentioned above together with Config files and Pre-trained Models. (we have tried Classification model but it does not work well; therefore, we change to Object Detection model instead.)
* **Testing/Evaluating**: We create Inference graph from our trained model and use it to evaluate with another subset of Positive samples and whole set of Negative samples. The performance is measured by precision-recall score and mean Average Precision scores (mAP).
* **Visualization**: we compare and plot the performance of our different models.


#### COPY from Local to Google Cloud Platform:
```
* single file: $ gcloud compute scp positive_train.record username@tensorflow-1-vm:. --zone us-west1-b
* folder: $ gcloud compute scp --recurse rfcn_resnet101 username@tensorflow-1-vm:./model/ --zone us-west1-b
```

#### COPY from Google Cloud Platform to Local:
```
* single file:
$ gcloud compute scp username@tensorflow-1-vm:model.ckpt-200000 . --zone us-west1-b
* folder:
$ gcloud compute scp --recurse username@tensorflow-1-vm:trained_rfcn_resnet101 . --zone us-west1-b
```

#### Training code: use Tensorflow Object Detection API (model/research/object_detection)
```
$ python model_main.py --logtostderr --model_dir=training/ --pipeline_config_path=training/pipeline.config
```

#### Viewing Training Progress via Tensorboard:
```
$ gcloud compute firewall-rules create tensorboard-port --allow tcp:8008
$ tensorboard --logdir=trained_model --port=8008
```
* Then look at the progress at http://[external-ip-of-Google-Cloud-VM]:6006

#### For starting Google Cloud VM in order to run Tensorflow Object Detection API
* Tensorflow Object Detection API (model/research)
```
$ protoc object_detection/protos/*.proto --python_out=.
$ export PYTHONPATH=$PYTHONPATH:`pwd`:`pwd`/slim
$ python object_detection/builders/model_builder_test.py (testing whether it works or not)
```
