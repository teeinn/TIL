```python
import sagemaker
from sagemaker.tensorflow import TensorFlow
from sagemaker import get_execution_role

sess = sagemaker.Session()
role = get_execution_role()
output_path = "s3://" + sess.default_bucket() + "/DEMO-tensorflow/mnist"

local_mode = False
if local_mode:
    instance_type = "local"
else:
    instance_type = "ml.c4.xlarge"

est = TensorFlow(
    entry_point="train.py",
    source_dir="code",  # directory of your training script
    role=role,
    framework_version="2.3.1",
    model_dir=False,  # don't pass --model_dir to your training script
    py_version="py37",
    instance_type=instance_type,
    instance_count=1,
    volume_size=250,
    output_path=output_path,
    hyperparameters={
        "batch-size": 512,
        "epochs": 1,
        "learning-rate": 1e-3,
        "beta_1": 0.9,
        "beta_2": 0.999,
    },
)
```

<br/><br/>
Tensorflow Estimator는 사용자가 정의한 컨테이너 환경에서 training script를 동작시키는 class이다. 위에서 Tensorflow 내에 정의한 파라미터 값들을 하나하나 살펴보면 아래와 같다.
<br/><br/>

```python
* entry_point: A user-defined Python file used by the training container as the instructions for training. 
* role: An IAM role to make AWS service requests
* instance_type: The type of SageMaker instance to run your training script. Set it to local if you want to run the training job on the SageMaker instance you are using to run this notebook.
* model_dir: S3 bucket URI where the checkpoint data and models can be exported to during training (default: None).
* instance_count: The number of instances to run your training job on. Multiple instances are needed for distributed training.
* output_path: the S3 bucket URI to save training output (model artifacts and output files).
* framework_version: The TensorFlow version to use.
* py_version: The Python version to use.
```

<br/><br/>
Tensorflow 훈련을 돌리기 위한 entrypoint로 train script를 넣어준다. Estimator는 parameter에 정의된 runtime environment에 맞는 docker image를 다운로드 하고 entry point에 지정된 training script를 도커 이미지에 주입한다. 이 training script는 estimator로부터 정의된 training image가 제공하는 유용한 환경 변수에 접근할 수 있다. 다음은 그러한 컨테이너 환경 변수의 목록을 보여준다. 
<br/><br/>

```python
SM_MODEL_DIR
SM_CHANNELS
SM_CHANNEL_{channel_name}
SM_HPS
SM_HP_{hyperparameter_name}
SM_CURRENT_HOST
SM_HOSTS
SM_NUM_GPUS
SM_NUM_NEURONS
SM_NUM_CPUS
SM_LOG_LEVEL
SM_NETWORK_INTERFACE_NAME
SM_USER_ARGS
SM_INPUT_DIR
SM_INPUT_CONFIG_DIR
SM_RESOURCE_CONFIG
SM_INPUT_DATA_CONFIG
SM_TRAINING_ENV

다음의 페이지를 참고하면 각각의 환경 변수가 어떤 역할을 하는지
알 수 있다.
https://github.com/aws/sagemaker-training-toolkit/blob/master/ENVIRONMENT_VARIABLES.md
```

<br/><br/>
SM_CHANNELS_{channel_name}을 예로 들어보자면,
<br/>
```python
prefix = "DEMO-mnist"
bucket = sess.default_bucket()
loc = sess.upload_data(path="./data", bucket=bucket, key_prefix=prefix)

channels = {"training": loc, "testing": loc}
```
이렇게 channels를 정의해두면, train.py 안에서 접근이 가능하다.