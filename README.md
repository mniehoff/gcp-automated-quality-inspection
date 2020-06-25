# Google Cloud Platform Automated Image Quality Inspection
This repository is showcase about how to leverage google cloud platform to quickly train and deploy a machine learning project. We are using different components of gcp such as [App Engine](https://cloud.google.com/appengine), [AutoML](https://cloud.google.com/automl), [Cloud Storage](https://cloud.google.com/storage), [Cloud Pub/Sub](https://cloud.google.com/pubsub) and [Cloud Functions](https://cloud.google.com/functions) are used to implement an end-to-end machine learning project. Based on this [dataset](https://www.kaggle.com/ravirajsinh45/real-life-industrial-dataset-of-casting-product) we train an AutoML image classification model and deploy it through AutoML.

## Overview
- [app_engine](app_engine/): Demo application for deployment. For the implementation we use [fastapi](https://fastapi.tiangolo.com/) and [React](https://reactjs.org/) and deploy it on the [App Engine](https://cloud.google.com/appengine).
- [automl](automl/): All the code and instructions that are necessary to prepare for [AutoML](https://cloud.google.com/automl) image classification training.
- [cloud_functions](cloud_functions):
- [data](data/): All the data that is used for this project. We used [product image data for quality insepection](https://www.kaggle.com/ravirajsinh45/real-life-industrial-dataset-of-casting-product) from [kaggle](https://www.kaggle.com/) for this project. 
- [docs](docs/): Find the documentation images. 


## Prerequirements
You need access to the Google Cloud Platform. Create a new project and get access to the project with your local shell.
```sh
gcloud init
gcloud auth application-default login
```

Enable the APIs for AutoML, GCS, Cloud Functions, App Engine and Pub/Sub.

## Tutorial
There are X parts in this tutorial. We need to set the following configurations:
```sh
export PROJECT_ID="vigilant-shift-280708"
export BUCKET_LOCATION="US-CENTRAL1"
export BUCKET_NAME="product-quality"

```

### 1. AutoML Preparation, Training and Deployment
1. Download the [dataset](https://www.kaggle.com/ravirajsinh45/real-life-industrial-dataset-of-casting-product) and put it inside the [data](data/)-folder. Extract the zip file.
```
data
└── casting_data
    ├── test
    │   ├── def_front
    │   │   ├── ....
    │   │   └── new__0_9334.jpeg
    │   └── ok_front
    │       ├── ....
    │       └── cast_ok_0_9996.jpeg
    └── train
        ├── def_front
        │   ├── ...
        │   └── cast_def_0_9997.jpeg
        └── ok_front
            ├── ...
            └── cast_ok_0_9998.jpeg
```

2. Create a GCS bucket and upload the data:
```sh
gsutil mb -p $PROJECT_ID  -l $BUCKET_LOCATION -b on gs://$BUCKET_NAME
gsutil -m cp -r ../data gs://$BUCKET_NAME
```

3. Prepare CSV file for AutoML classification. The CSV file consists of the three columns: ,  and `Label`.
- `SET`: This is an optional field with fixed values to decide which sample belongs in which set. The fixed values are `TRAIN`, `VALIDATION` and `TEST`. If we don't assign this field AutoML will divide the dataset into 8:1:1. If we assign this field, it is necessary to use all of these values.
- `GCS Location`: The location of the image on the GCP.
- `LABEL`: The label of a sample.

We wrote a script [prepare.py](automl/prepare.py) to generate this CSV file based on the blobs in the specified bucket. 


4. Create a dataset in AutoML Vision. Assign it a name and select `Single-Label Classification` 
![AutoML dataset](docs/images/automl_dataset.png)


### 