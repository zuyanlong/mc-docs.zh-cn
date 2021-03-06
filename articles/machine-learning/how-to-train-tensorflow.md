---
title: 训练和部署 TensorFlow 模型
titleSuffix: Azure Machine Learning
description: 了解如何使用 Azure 机器学习大规模运行 TensorFlow 训练脚本。
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: how-to
ms.author: maxluk
author: maxluk
ms.date: 08/20/2019
ms.custom: seodec18
ms.openlocfilehash: f78a8c987b0e89bfa4da61ab947f56159b3aeb40
ms.sourcegitcommit: 7320277f4d3c63c0b1ae31ba047e31bf2fe26bc6
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92117957"
---
# <a name="build-a-tensorflow-deep-learning-model-at-scale-with-azure-machine-learning"></a>使用 Azure 机器学习大规模构建 TensorFlow 深度学习模型


本文展示了如何使用 Azure 机器学习的 [TensorFlow 估算器](https://docs.microsoft.com/python/api/azureml-train-core/azureml.train.dnn.tensorflow?view=azure-ml-py&preserve-view=true)类大规模运行 [TensorFlow](https://www.tensorflow.org/overview) 训练脚本。 此示例使用深度神经网络 (DNN) 训练并注册 TensorFlow 模型来对手写数字进行分类。

无论是从头开始开发 TensorFlow 模型，还是将[现有模型](how-to-deploy-existing-model.md)引入云中，都可以使用 Azure 机器学习来横向扩展开源训练作业，以便构建、部署和监视生产级模型以及对其进行版本控制。

详细了解[深度学习与机器学习](concept-deep-learning-vs-machine-learning.md)。

## <a name="prerequisites"></a>先决条件

在以下任一环境中运行此代码：

 - Azure 机器学习计算实例 - 无需下载或安装

     - 在开始本教程之前完成[教程：设置环境和工作区](tutorial-1st-experiment-sdk-setup.md)以创建预先装载了 SDK 和示例存储库的专用笔记本服务器。
    - 在笔记本服务器上的示例深度学习文件夹中，导航到以下目录，查找已完成且已展开的笔记本：**how-to-use-azureml > ml-frameworks > tensorflow > deployment > train-hyperparameter-tune-deploy-with-tensorflow** 文件夹。 
 
 - 你自己的 Jupyter 笔记本服务器

    - [安装 Azure 机器学习 SDK](https://docs.microsoft.com/python/api/overview/azure/ml/install?view=azure-ml-py&preserve-view=true)。
    - [创建工作区配置文件](how-to-configure-environment.md#workspace)。
    - [下载示例脚本文件](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/ml-frameworks/tensorflow/deployment/train-hyperparameter-tune-deploy-with-tensorflow) `mnist-tf.py` 和 `utils.py`
     
    此外，还可以在 GitHub 示例页上找到本指南的完整 [Jupyter Notebook 版本](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/ml-frameworks/tensorflow/deployment/train-hyperparameter-tune-deploy-with-tensorflow/train-hyperparameter-tune-deploy-with-tensorflow.ipynb)。 该笔记本包含扩展部分，其中涵盖智能超参数优化、模型部署和笔记本小组件。

## <a name="set-up-the-experiment"></a>设置试验

本部分将准备训练实验，包括加载所需 Python 包、初始化工作区、创建实验以及上传训练数据和训练脚本。

### <a name="import-packages"></a>导入程序包

首先，导入必需的 Python 库。

```Python
import os
import urllib
import shutil
import azureml

from azureml.core import Experiment
from azureml.core import Workspace, Run

from azureml.core.compute import ComputeTarget, AmlCompute
from azureml.core.compute_target import ComputeTargetException
from azureml.train.dnn import TensorFlow
```

### <a name="initialize-a-workspace"></a>初始化工作区

[Azure 机器学习工作区](concept-workspace.md)是服务的顶级资源。 它提供了一个集中的位置来处理创建的所有项目。 在 Python SDK 中，可以通过创建 [`workspace`](https://docs.microsoft.com/python/api/azureml-core/azureml.core.workspace.workspace?view=azure-ml-py&preserve-view=true) 对象来访问工作区项目。

根据在[先决条件部分](#prerequisites)中创建的 `config.json` 文件创建工作区对象。

```Python
ws = Workspace.from_config()
```

### <a name="create-a-deep-learning-experiment"></a>创建深度学习试验

创建一个试验和文件夹来容纳训练脚本。 在此示例中，创建一个名为“tf-mnist”的试验。

```Python
script_folder = './tf-mnist'
os.makedirs(script_folder, exist_ok=True)

exp = Experiment(workspace=ws, name='tf-mnist')
```

### <a name="create-a-file-dataset"></a>创建文件数据集

`FileDataset` 对象引用工作区数据存储或公共 URL 中的一个或多个文件。 文件可以是任何格式，该类提供将文件下载或装载到计算机的功能。 通过创建 `FileDataset`，可以创建对数据源位置的引用。 如果将任何转换应用于数据集，则它们也会存储在数据集中。 数据会保留在其现有位置，因此不会产生额外的存储成本。 有关详细信息，请参阅 `Dataset` 包中的[操作](https://docs.microsoft.com/azure/machine-learning/how-to-create-register-datasets)指南。

```python
from azureml.core.dataset import Dataset

web_paths = [
            'http://yann.lecun.com/exdb/mnist/train-images-idx3-ubyte.gz',
            'http://yann.lecun.com/exdb/mnist/train-labels-idx1-ubyte.gz',
            'http://yann.lecun.com/exdb/mnist/t10k-images-idx3-ubyte.gz',
            'http://yann.lecun.com/exdb/mnist/t10k-labels-idx1-ubyte.gz'
            ]
dataset = Dataset.File.from_files(path=web_paths)
```

使用 `register()` 方法将数据集注册到工作区，以便将其与其他人共享，在各种试验中重复使用，以及在训练脚本中按名称引用。

```python
dataset = dataset.register(workspace=ws,
                           name='mnist dataset',
                           description='training and test dataset',
                           create_new_version=True)

# list the files referenced by dataset
dataset.to_path()
```

## <a name="create-a-compute-target"></a>创建计算目标

创建用于运行 TensorFlow 作业的计算目标。 在此示例中，创建启用了 GPU 的 Azure 机器学习计算群集。

```Python
cluster_name = "gpucluster"

try:
    compute_target = ComputeTarget(workspace=ws, name=cluster_name)
    print('Found existing compute target')
except ComputeTargetException:
    print('Creating a new compute target...')
    compute_config = AmlCompute.provisioning_configuration(vm_size='STANDARD_NC6', 
                                                           max_nodes=4)

    compute_target = ComputeTarget.create(ws, cluster_name, compute_config)

    compute_target.wait_for_completion(show_output=True, min_node_count=None, timeout_in_minutes=20)
```

[!INCLUDE [low-pri-note](../../includes/machine-learning-low-pri-vm.md)]

有关计算目标的详细信息，请参阅[什么是计算目标](concept-compute-target.md)一文。

## <a name="create-a-tensorflow-estimator"></a>创建 TensorFlow 估算器

[TensorFlow 估算器](https://docs.microsoft.com/python/api/azureml-train-core/azureml.train.dnn.tensorflow?view=azure-ml-py&preserve-view=true)提供了一种在计算目标上启动 TensorFlow 训练作业的简单方法。

TensorFlow 估算器是通过泛型 [`estimator`](https://docs.microsoft.com//python/api/azureml-train-core/azureml.train.estimator.estimator?view=azure-ml-py&preserve-view=true) 类实现的，该类可用于支持任何框架。 有关使用泛型估算器训练模型的详细信息，请参阅[通过估算器使用 Azure 机器学习训练模型](how-to-train-ml-models.md)

如果你的训练脚本需要额外的 pip 或 conda 包才能运行，则可以通过使用 `pip_packages` 和 `conda_packages` 参数传递其名称在生成的 Docker 映像中安装这些包。


> [!WARNING]
> Azure 机器学习通过复制整个源目录来运行训练脚本。 如果你有不想上传的敏感数据，请使用 [.ignore 文件](how-to-save-write-experiment-files.md#storage-limits-of-experiment-snapshots)或不将其包含在源目录中。 改为使用[数据存储](https://docs.microsoft.com/python/api/azureml-core/azureml.data?view=azure-ml-py&preserve-view=true)来访问数据。


```python
script_params = {
    '--data-folder': dataset.as_named_input('mnist').as_mount(),
    '--batch-size': 50,
    '--first-layer-neurons': 300,
    '--second-layer-neurons': 100,
    '--learning-rate': 0.01
}

est = TensorFlow(source_directory=script_folder,
                 entry_script='tf_mnist.py',
                 script_params=script_params,
                 compute_target=compute_target,
                 use_gpu=True,
                 pip_packages=['azureml-dataprep[pandas,fuse]'])
```

> [!TIP]
> Tensorflow 估算器类中添加了对 **Tensorflow 2.0** 的支持。 有关详细信息，请参阅此[博客文章](https://azure.microsoft.com/blog/tensorflow-2-0-on-azure-fine-tuning-bert-for-question-tagging/)。

有关自定义 Python 环境的详细信息，请参阅[创建和管理用于训练和部署的环境](how-to-use-environments.md)。 

## <a name="submit-a-run"></a>提交运行

[运行对象](https://docs.microsoft.com/python/api/azureml-core/azureml.core.run%28class%29?view=azure-ml-py&preserve-view=true)在作业运行时和运行后提供运行历史记录的接口。

```Python
run = exp.submit(est)
run.wait_for_completion(show_output=True)
```

执行运行时，会经历以下阶段：

- **准备**：根据 TensorFlow 估计器创建 Docker 映像。 将映像上传到工作区的容器注册表，缓存以用于后续运行。 还会将日志流式传输到运行历史记录，可以查看日志以监视进度。

- **缩放**：如果 Batch AI 群集执行运行所需的节点多于当前可用节点，则群集将尝试纵向扩展。

- **正在运行**：将脚本文件夹中的所有脚本上传到计算目标，装载或复制数据存储，然后执行 entry_script。 将 stdout 和 ./logs 文件夹中的输出流式传输到运行历史记录，可将其用于监视运行。

- **后期处理**：将运行的 ./outputs 文件夹复制到运行历史记录。

## <a name="register-or-download-a-model"></a>注册或下载模型

训练模型后，可以将其注册到工作区。 凭借模型注册，可以在工作区中存储模型并对其进行版本控制，从而简化[模型管理和部署](concept-model-management-and-deployment.md)。 通过指定参数 `model_framework`、`model_framework_version` 和 `resource_configuration`，无代码模型部署将可供使用。 这允许你通过已注册模型直接将模型部署为 Web服务，`ResourceConfiguration` 对象定义 Web 服务的计算资源。

```Python
from azureml.core import Model
from azureml.core.resource_configuration import ResourceConfiguration

model = run.register_model(model_name='tf-dnn-mnist', 
                           model_path='outputs/model',
                           model_framework=Model.Framework.TENSORFLOW,
                           model_framework_version='1.13.0',
                           resource_configuration=ResourceConfiguration(cpu=1, memory_in_gb=0.5))
```

此外，还可以使用“运行”对象下载模型的本地副本。 在训练脚本 `mnist-tf.py` 中，TensorFlow 保护程序对象将模型保存到本地文件夹（计算目标本地）。 可以使用“运行”对象下载副本。

```Python
# Create a model folder in the current directory
os.makedirs('./model', exist_ok=True)

for f in run.get_file_names():
    if f.startswith('outputs/model'):
        output_file_path = os.path.join('./model', f.split('/')[-1])
        print('Downloading from {} to {} ...'.format(f, output_file_path))
        run.download_file(name=f, output_file_path=output_file_path)
```

## <a name="distributed-training"></a>分布式训练

[`TensorFlow`](https://docs.microsoft.com/python/api/azureml-train-core/azureml.train.dnn.tensorflow?view=azure-ml-py&preserve-view=true) 估算器还支持跨 CPU 和 GPU 群集的分布式训练。 你可以轻松运行分布式 TensorFlow 作业，Azure 机器学习将为你管理业务流程。

Azure 机器学习支持 TensorFlow 中的两种分布式训练方法：

- 使用 [Horovod](https://github.com/uber/horovod) 框架进行[基于 MPI](https://www.open-mpi.org/) 的分布式训练
- 使用参数服务器方法进行本机[分布式 TensorFlow](https://www.tensorflow.org/deploy/distributed)

### <a name="horovod"></a>Horovod

[Horovod](https://github.com/uber/horovod) 是 Uber 开发的用于分布式训练的开放源代码框架。 它提供了通向分布式 GPU TensorFlow 作业的简单路径。

若要使用 Horovod，请在 TensorFlow 构造函数中为 `distributed_training` 参数指定一个 [`MpiConfiguration`](https://docs.microsoft.com/python/api/azureml-core/azureml.core.runconfig.mpiconfiguration?view=azure-ml-py&preserve-view=true) 对象。 此参数可确保为你安装 Horovod 库，以便在训练脚本中使用。

```Python
from azureml.core.runconfig import MpiConfiguration
from azureml.train.dnn import TensorFlow

# Tensorflow constructor
estimator= TensorFlow(source_directory=project_folder,
                      compute_target=compute_target,
                      script_params=script_params,
                      entry_script='script.py',
                      node_count=2,
                      process_count_per_node=1,
                      distributed_training=MpiConfiguration(),
                      framework_version='1.13',
                      use_gpu=True,
                      pip_packages=['azureml-dataprep[pandas,fuse]'])
```

### <a name="parameter-server"></a>参数服务器

此外，你还可以运行[本机分布式 TensorFlow](https://www.tensorflow.org/deploy/distributed)，它使用参数服务器模型。 在此方法中，你将在一组参数服务器和工作线程中进行训练。 工作线程在训练期间计算梯度，而参数服务器聚合梯度。

若要使用参数服务器方法，请在 TensorFlow 构造函数中为 `distributed_training` 参数指定一个 [`TensorflowConfiguration`](https://docs.microsoft.com/python/api/azureml-core/azureml.core.runconfig.tensorflowconfiguration?view=azure-ml-py&preserve-view=true) 对象。

```Python
from azureml.train.dnn import TensorFlow

distributed_training = TensorflowConfiguration()
distributed_training.worker_count = 2

# Tensorflow constructor
tf_est= TensorFlow(source_directory=project_folder,
                      compute_target=compute_target,
                      script_params=script_params,
                      entry_script='script.py',
                      node_count=2,
                      process_count_per_node=1,
                      distributed_training=distributed_training,
                      use_gpu=True,
                      pip_packages=['azureml-dataprep[pandas,fuse]'])

# submit the TensorFlow job
run = exp.submit(tf_est)
```

#### <a name="define-cluster-specifications-in-tf_config"></a>在“TF_CONFIG”中定义群集规范

另外，你还需要 [`tf.train.ClusterSpec`](https://www.tensorflow.org/api_docs/python/tf/train/ClusterSpec) 的群集网络地址和端口，因此，Azure 机器学习会为你设置 `TF_CONFIG` 环境变量。

`TF_CONFIG` 环境变量是一个 JSON 字符串。 下面是介绍参数服务器变量的示例：

```JSON
TF_CONFIG='{
    "cluster": {
        "ps": ["host0:2222", "host1:2222"],
        "worker": ["host2:2222", "host3:2222", "host4:2222"],
    },
    "task": {"type": "ps", "index": 0},
    "environment": "cloud"
}'
```

对于 TensorFlow 的高级别 [`tf.estimator`](https://www.tensorflow.org/api_docs/python/tf/estimator) API，TensorFlow 将分析 `TF_CONFIG` 变量并为你生成群集规范。

如果使用 TensorFlow 的低级别核心 API 进行训练，则需要分析 `TF_CONFIG` 变量并在训练代码中生成 `tf.train.ClusterSpec`。

```Python
import os, json
import tensorflow as tf

tf_config = os.environ.get('TF_CONFIG')
if not tf_config or tf_config == "":
    raise ValueError("TF_CONFIG not found.")
tf_config_json = json.loads(tf_config)
cluster_spec = tf.train.ClusterSpec(cluster)

```

## <a name="deploy-a-tensorflow-model"></a>部署 TensorFlow 模型

无论使用哪种估算器进行训练，都可以采用与 Azure 机器学习中任何其他已注册模型完全相同的方式部署你刚才注册的模型。 部署指南包含有关模型注册的部分，但由于你已有一个已注册的模型，因而可以直接跳到[创建计算目标](how-to-deploy-and-where.md#choose-a-compute-target)进行部署。

## <a name="preview-no-code-model-deployment"></a>（预览版）无代码模型部署

除了传统的部署路线之外，还可以为 Tensorflow 使用无代码部署功能（预览版）。 通过如上所示使用 `model_framework`、`model_framework_version` 和 `resource_configuration` 参数注册你的模型，可以简单地使用 `deploy()` 静态函数来部署模型。

```python
service = Model.deploy(ws, "tensorflow-web-service", [model])
```

完整的[操作指南](how-to-deploy-and-where.md)更深入地介绍了 Azure 机器学习。

## <a name="next-steps"></a>后续步骤

在本文中，你训练并注册了一个 TensorFlow 模型，并了解了部署选项。 有关 Azure 机器学习的详细信息，请参阅以下其他文章。

* [在训练期间跟踪运行指标](how-to-track-experiments.md)
* [优化超参数](how-to-tune-hyperparameters.md)
* [Azure 中分布式深度学习训练的参考体系结构](https://docs.microsoft.com/azure/architecture/reference-architectures/ai/training-deep-learning)
