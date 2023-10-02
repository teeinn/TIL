1. **Use pre-built sagemaker docker image**
    
    sagemaker에서 제공하는 머신 러닝 프레임워크 (Apache MXNet, Tensorflow, PyTorch, and Chainer)의 built-in algorithms이나 pre-built docker images를 사용 할 수 있다. 또한 scikit-learn이나 SparkML 같은 라이브러리도 지원한다. 또한, 이러한 pre-built sagemaker image를 확장시켜 library나 필요한 기능을 추가할 수 있다. 
    
    - sagemaker에서 제공하는 pre-built image와 algorithms 목록: h[ttps://docs.aws.amazon.com/sagemaker/latest/dg-ecr-paths/sagemaker-algo-docker-registry-paths.html](https://docs.aws.amazon.com/sagemaker/latest/dg-ecr-paths/sagemaker-algo-docker-registry-paths.html)
    
    - 다음의 페이지는 tensorflow 프레임워크의 image classification algorithm을 어떻게 가져와서 훈련을 돌릴 수 있는지 예시를 보여준다 :
        
        https://docs.aws.amazon.com/sagemaker/latest/dg/image-classification-tensorflow.html#IC-TF-inputoutput 
        
        https://github.com/aws/amazon-sagemaker-examples/blob/main/introduction_to_amazon_algorithms/image_classification_tensorflow/Amazon_TensorFlow_Image_Classification.ipynb
        
    - pre-built docker image를 자신에게 맞게 확장시키는 방법:
        
        https://docs.aws.amazon.com/sagemaker/latest/dg/prebuilt-containers-extend.html#prebuilt-containers-extend-tutorial
        
2. **Adapt your own container**
    
    이미 존재하는 본인의 docker image를 ecr에 올려서 훈련을 돌리는 방법이다. 다음의 페이지를 참고하자.
    
    https://docs.aws.amazon.com/sagemaker/latest/dg/adapt-training-container.html
    
3. **Create a container with own algorithms and models**
    
    아예 처음부터 container를 생성해서 본인의 algorithms과 models를 적용하고 싶을 때 사용하는 방법이다.
    
    https://docs.aws.amazon.com/sagemaker/latest/dg/your-algorithms-training-algo-dockerfile.html