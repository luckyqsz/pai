<!--
  Copyright (c) Microsoft Corporation
  All rights reserved.

  MIT License

  Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated
  documentation files (the "Software"), to deal in the Software without restriction, including without limitation
  the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and
  to permit persons to whom the Software is furnished to do so, subject to the following conditions:
  The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

  THE SOFTWARE IS PROVIDED *AS IS*, WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING
  BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND
  NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM,
  DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
  OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
-->


# TensorFlow on OpenPAI

This guide introduces how to run [TensorFlow](https://www.tensorflow.org/) job on OpenPAI.
The following contents show some basic TensorFlow examples, other customized TensorFlow code can be run similarly.


## Contents

1. [TensorFlow CIFAR-10 image classification](#tensorflow-cifar-10-image-classification)
2. [TensorFlow ImageNet image classification](#tensorflow-imagenet-image-classification)
3. [Distributed TensorFlow CIFAR-10 image classification](#distributed-tensorflow-cifar-10-image-classification )
4. [TensorFlow Tensorboard](#tensorflow-tensorboard)

# TensorFlow examples

To run TensorFlow examples in OpenPAI, you need to prepare a job configuration file and submit it through webportal.

OpenPAI packaged the docker env required by the job for user to use. User could refer to [DOCKER.md](./DOCKER.md) to customize this example docker env. If user have built a customized image and pushed it to Docker Hub, replace our pre-built image `openpai/pai.example.tensorflow` with your own. 

Here're some configuration file examples:

### TensorFlow CIFAR-10 image classification 

```js
{
  "jobName": "tensorflow-cifar10",
  "image": "openpai/pai.example.tensorflow",

  "dataDir": "/tmp/data",
  "outputDir": "/tmp/output",

  "taskRoles": [
    {
      "name": "cifar_train",
      "taskNumber": 1,
      "cpuNumber": 8,
      "memoryMB": 32768,
      "gpuNumber": 1,
      "command": "git clone https://github.com/tensorflow/models && cd models/research/slim && python download_and_convert_data.py --dataset_name=cifar10 --dataset_dir=$PAI_DATA_DIR && python train_image_classifier.py --batch_size=64 --model_name=inception_v3 --dataset_name=cifar10 --dataset_split_name=train --dataset_dir=$PAI_DATA_DIR --train_dir=$PAI_OUTPUT_DIR"
    }
  ]
}
```

### TensorFlow ImageNet image classification 

```js
{
  "jobName": "tensorflow-imagenet",
  "image": "openpai/pai.example.tensorflow",

  // prepare imagenet dataset in TFRecord format following https://git.io/vFxjh and upload to hdfs
  "dataDir": "$PAI_DEFAULT_FS_URI/path/data",
  // make a new dir for output on hdfs
  "outputDir": "$PAI_DEFAULT_FS_URI/path/output",
  // download code from tensorflow slim https://git.io/vFpef and upload to hdfs
  "codeDir": "$PAI_DEFAULT_FS_URI/path/code",

  "taskRoles": [
    {
      "name": "imagenet_train",
      "taskNumber": 1,
      "cpuNumber": 8,
      "memoryMB": 32768,
      "gpuNumber": 1,
      "command": "python code/train_image_classifier.py --batch_size=64 --model_name=inception_v3 --dataset_name=imagenet --dataset_split_name=train --dataset_dir=$PAI_DATA_DIR --train_dir=$PAI_OUTPUT_DIR"
    }
  ]
}
```

### Distributed TensorFlow CIFAR-10 image classification 

```js
{
  "jobName": "tensorflow-distributed-cifar10",
  "image": "openpai/pai.example.tensorflow",

  // download cifar10 dataset from http://www.cs.toronto.edu/~kriz/cifar.html and upload to hdfs
  "dataDir": "$PAI_DEFAULT_FS_URI/path/data",
  // make a new dir for output on hdfs
  "outputDir": "$PAI_DEFAULT_FS_URI/path/output",
  // download code from tensorflow benchmark https://git.io/vF4wT and upload to hdfs
  "codeDir": "$PAI_DEFAULT_FS_URI/path/code",

  "taskRoles": [
    {
      "name": "ps_server",
      "taskNumber": 2,
      "cpuNumber": 2,
      "memoryMB": 8192,
      "gpuNumber": 0,
      "command": "pip --quiet install scipy && python code/tf_cnn_benchmarks.py --local_parameter_device=cpu --batch_size=32 --model=resnet20 --variable_update=parameter_server --data_dir=$PAI_DATA_DIR --data_name=cifar10 --train_dir=$PAI_OUTPUT_DIR --ps_hosts=$PAI_TASK_ROLE_ps_server_HOST_LIST --worker_hosts=$PAI_TASK_ROLE_worker_HOST_LIST --job_name=ps --task_index=$PAI_CURRENT_TASK_ROLE_CURRENT_TASK_INDEX"
    },
    {
      "name": "worker",
      "taskNumber": 2,
      "cpuNumber": 2,
      "memoryMB": 16384,
      "gpuNumber": 4,
      "command": "pip --quiet install scipy && python code/tf_cnn_benchmarks.py --local_parameter_device=cpu --batch_size=32 --model=resnet20 --variable_update=parameter_server --data_dir=$PAI_DATA_DIR --data_name=cifar10 --train_dir=$PAI_OUTPUT_DIR --ps_hosts=$PAI_TASK_ROLE_ps_server_HOST_LIST --worker_hosts=$PAI_TASK_ROLE_worker_HOST_LIST --job_name=worker --task_index=$PAI_CURRENT_TASK_ROLE_CURRENT_TASK_INDEX",
      "minSucceededTaskCount": 2
    }
  ],
  "retryCount": 0
}
```

### TensorFlow Tensorboard

```js
{
  "jobName": "tensorflow-tensorboard",
  "image": "openpai/pai.example.tensorflow",

  // prepare checkpoint and log to be visualized and upload to hdfs
  "dataDir": "$PAI_DEFAULT_FS_URI/path/data",
  // prepare visualization script tensorboard-example.sh and upload to hdfs
  "codeDir": "$PAI_DEFAULT_FS_URI/path/code",

  "taskRoles": [
    {
      "name": "tensorboard",
      "taskNumber": 1,
      "cpuNumber": 2,
      "memoryMB": 4096,
      "gpuNumber": 0,
      "command": "/bin/bash code/tensorflow-tensorboard.sh"
    }
  ]
}
```

For more details on how to write a job configuration file, please refer to [job tutorial](../../docs/job_tutorial.md#json-config-file-for-job-submission).

