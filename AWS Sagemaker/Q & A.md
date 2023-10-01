1. Studio JupyterServer Instance를 변경하는 방법? --> 각 사용자는 주어진 유형(ex: ml.t3.medium)의 instance를 하나만 실행할 수 있고, 각 instance는 최대 4개의 app을 할당할 수 있다. 그리고 각 app을 사용해 여러 개의 notebook과 terminal을 생성할 수 있다. 
만약 동일한 instance에서 4개 이상의 app을 실행해야 한다면, 다른 유형의 기본 instance에서 실행하도록 선택할 수 있다. 

2. Studio notebook 또는 파일을 공유하는 일반적인 방법? 지금은 notebook을 공유하고 싶을 때 share를 클릭하면 나오는 메세지 --> Sharing is not enabled. Please contact your Amazon SageMaker Studio administrator.

3. Studio에서는 training을 돌리기위한 Estimator를 사용할 때 local mode로 전환해서 현재 notebook을 실행 중인 instance에서 돌아가게끔 하는 방법이 없는 것으로 알고 있다. 그렇다면 Studio에서 local mode 같이 돌려볼 수 있는 방법이 따로 존재하는지?

4. estimator의 entrypoint에 지정된 스크립트에서 접근 가능한 컨테이너 환경 변수들 각각의 사용법?




