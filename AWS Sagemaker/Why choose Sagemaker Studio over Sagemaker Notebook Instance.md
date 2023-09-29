reference: https://dataintegration.info/dive-deep-into-amazon-sagemaker-studio-notebooks-architecture
<br/>
<br/>
<br/>
<br/>


**Jupyter notebook in Sagemaker Notebook Instance 이 실행되는 방식**

- jupyter 노트북은 기본적으로 단일 컴퓨터에 호스팅되고 웹 브라우저를 통해 access 할 수 있다. 하지만 팀이 성장하고 머신 러닝 워크로드가 production으로 이동하게 된다면 여러 가지 **문제점이** 발생 할 수 있다.
    1. 각 Data Scientist는 자신만의 ML 문제를 해결하기 위해 사용자 정의 종속성 및 패키지를 설치해야 하며 다른 사람들의 작업에 영향을 미치지 않아야 한다.
    2. 각각의 ML 워크 프로세스는 다른 컴퓨팅 리소스가 필요 할 수 있다. 예를 들어, 데이터 처리에는 High memory가 요구되고, 훈련에는 GPU가 요구된다. 따라서 리소스를 빠르게 확장하거나 축소할 수 있는 쉬운 방법이 필요하다. 
        
        이를 극복하기 위해 DS는 종종 Jupyter 환경의 인스턴스 유형을 변경해야 하며 이로 인해 작업 영역을 다른 인스턴스로 이동해야 하므로 중단과 생산성 감소를 초래할 수 있다. 
        
    3. 대규모 팀에서는 팀이 사용하는 모든 DS environment를 정기적으로 패치, 보안 및 유지 관리하는 것이 부담이 될 수 있다. 
    4. 서로의 작업을 쉽게 공유하는 환경의 부재 
    5. ML 워크로드가 자동화된 방식으로 배포, 모니터링 및 재훈련이 되어야 하는데 이것을 다른 도구나 서비스를 사용해야만 가능하다. 즉 실험에서 프로덕션까지 이동하는 것이 원활하지 않다.
<br/>
<br/>
<br/>
<br/>

**Sagemaker Studio**

- Machine Learning을 위한 Fully Integrated Development Environment (IDE)라고 할 수 있다.
- Data scientist가 IDE를 떠날 필요 없이 ml model을 build, train, tune, debug, deploy, monitor까지 할 수 있다.
<br/>
<br/>

    
    [Studio domain]
    
    - Amazon Elastic File System(EFS) Volume, 해당 도메인에 액세스 권한을 부여받은 사용자 목록 및 보안, 응용 프로그램, 네트워킹 등과 관련된 구성과 논리적으로 연결된 것이 도메인이다. 도메인 내의 사용자들은 notebooks 및 기타 자원을 공유할 수 있다.
    - 이 domain에 등록된 사용자는 User profile에 나타나고, 각각의 profile은 execution role이라던가 EFS volume 등의 유니크한 정보를 가질 수 있다.
<br/>
<br/>

    [Sagemaker image]
    
    - Aws ECR에 저장된 Docker Container Image를 가리키는 메타데이터로, 주로 머신러닝 및 딥러닝 framework library 및 다른 종속성을 실행하는데 필요한 정보를 포함한다.
    - Sagemaker studio는 pre-built image를 제공하고, 사용자가 정의한 이미지를 가져와 studio domain에 연결하는 옵션도 제공한다. 이 사용자 정의 이미지는 aws ecr 저장소에 저장되어야 한다. domain 전체에 사용자 정의 이미지를 연결하거나 아니면 특정 사용자 프로필에 연결하는 옵션을 선택할 수도 있다.
<br/>
<br/>

    [Studio notebook]
    
    - Studio에서 생성한 notebook은 Sagemaker notebook 보다 빠르게 생성되고, 기반이 되는 computing resource는 탄력적으로 변경 가능하며 이는 백그라운드에서 자동으로 이루어진다.
    - 각 사용자가 본인의 home directory를 가질 수 있어 notebook이나 다른 파일들을 저장할 수 있고, 파일을 쉽게 다른 사용자와 공유할 수도 있다.
<br/>
<br/>

    [APP]
    
    앱은 도메인 내의 user를 위해 **docker container로 구현된** application이다. Studio에서는 두 가지 유형의 앱을 지원한다. 
    
    - JupyterServer - JupyterServer app은 **Jupyter 서버를 실행**한다. 도메인 내의 각 사용자는 고유한 전용 jupyterserver app을 실행한다.
    - KernelGateway - KernelGateway app은 Sagemaker Image Container를 실행한 것이다. 각각의 User는 여러 개의 KernelGateway App을 동시에 실행 시킬 수 있다. 위의 Sagemaker Image 설명과 같이 built-in image도 있고, 개인이 Custom image를 만들어 제공할 수도 있다.

    일단 사용자가 웹 브라우저를 사용해서 Studio UI에 접속하면, JupyterServer Container 내에서 돌고 있던 notebook server와 HTTPS 연결이 설정된다. 이 JupyterServer Container는 service에서 관리하는 EC2 Instance에서 작동된다. 
    
    각 사용자는 주어진 유형(ex: ml.t3.medium)의 instance를 하나만 실행할 수 있고, 각 instance는 최대 4개의 app을 할당할 수 있다. 그리고 각 app을 사용해 여러 개의 notebook과 terminal을 생성할 수 있다. 
    만약 동일한 instance에서 4개 이상의 app을 실행해야 한다면, 다른 유형의 기본 instance에서 실행하도록 선택할 수 있다. 
    Shared Amazon EFS volume은 모든 KernerlGateway & JupyterServer app에 마운트된다. 
<br/>
<br/>

    [Terminal Access]
    
    notebook 이용 뿐만아니라 JupyterServer app (*system terminal*)과 KernelGateway app (*image terminal*)에 terminal을 붙일 수 있다. 전자의 경우는 notebook server를 확장하기 위한 설치나 file system operation을 수행할 때 사용될 수 있고, 후자의 경우는 컨테이너에 특정 library 설치나 command line에서 스크립트를 돌리고 싶을 때 사용 할 수 있다. 
<br/>
<br/>

    [Storage]
    
    각 사용자는 domain 하에 amazon EFS(Elastic File System) Volume에서 자체 개인 홈 디렉토리를 생성합니다. 이 File System은 모든 notebook server container와 모든 kernel gateway container에 자동으로 마운트 된다.
    
    또한, studio는 notebook snapshots과 metadata를 저장하기 위해 S3를 사용해 노트북 공유를 가능하게 한다. 그리고 studio를 열 때, 노트북이 실행되고 있는 instance에 amazon EBS volume이 붙고, 그 instance에서 작동되는 노트북이 모두 삭제되면 volume도 삭제된다. 
<br/>
<br/>

    [Network]
    
    studio는 기본적으로 2개의 vpc를 사용하는데, 하나는 studio 자체에서 컨트롤 되고 public internet traffic에 열려있는 vpc가 있다. 다른 하나는 user에 의해 컨트롤 되고, studio domain과 efs volume 간의 encrypted traffic을 사용하는 vpc가 존재한다. 
<br/>
<br/>

    [Pricing]
    
    Studio 그 자체로는 추가 비용이 발생 하지 않는다. 비용은 사용자가 studio notebooks을 생성했을 때 compute와 storage에 대해 청구된다. compute instance type에 따른 비용 청구 요금이 다르고, efs에 저장된 data file or scripts에 따라 다르다. 
    
    처음에 user를 생성해서 studio를 실행 시켰을 때, browser에서 studio UI를 렌더링하는 JupyterServer app에 대한 비용은 청구되지 않는다. 대신에 그 밑에 붙은 EFS storage에 대한 청구는 된다. 
    
    sagemaker-auto-shutdown 을 이용하면 특정 기간 동안 동작하지 않는 kernelGateway app, kernels, image terminals을 종료시킬 수 있다. 종료하지 않으면 비용은 계속 청구된다. 
    https://github.com/aws-samples/sagemaker-studio-auto-shutdown-extension
    