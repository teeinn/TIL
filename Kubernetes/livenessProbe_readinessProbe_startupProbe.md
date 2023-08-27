
Pod에서 실행 중인 container application은 다양한 상황에서 작동을 멈출 수 있다. 예를 들어, 일시적인 커넥션 오류, configuration 오류, application 에러, Deadlocks, CPU 사용, 메모리 누수 등의 문제가 발생할 수 있다. 이때 필요한 것이 Probe이다. 

**Probe**는 Pod의 컨테이너에서 돌고있는 application의 상태를 체크하여, application이 다시 시작할 수 있게 돕는 역할을 한다. Probe를 이용하게 되면 application의 health check를 주기적으로 진행할 수 있고, 개발자는 kubectl 커맨드나 yaml deployment를 통해 probe를 작동시킬 수 있다.

<br/>
<br/>

- - -

이제 3가지 종류의 Probe를 알아보자.  

- **Startup Probe**
- **Readiness Probe**
- **Liveness Probe**
<br/>
<br/>
<br/>

일단 먼저 **각 probe에 공통으로 쓰이는 parameter**를 살펴보자. 

- initalDelaySeconds: container가 작동된 후 얼마만의 시간 후에 probe를 시작할지
- timeoutSeconds: probe 시작 후에 얼마동안 기다릴지
- periodSeconds: 얼마나 자주 probe를 실행할지.
- successThreshold: probe가 실패한 후, 최소한 몇 번의 probe가 성공해야 성공했다고 할지
- failureThreshold: probe가 몇 번을 실패해야 완전히 실패했다고 할지
<br/>
<br/>
<br/>

이제 **application health check를 위해 공통으로 사용되는 방법**을 알아보자. 

- HTTP Probes
    
    REST API와 같은 application의 health check에 적합한 방법이다. HTTP Probe가 GET 요청을 통해 application의 상태를 검사한다. 200-399의 응답 코드를 보내줄 경우 성공으로 간주한다.
    
- Container Execution Probes
    
    컨테이너 내에서 실행되는 프로세스나 쉘 스크립트의 종료 코드를 기반으로 컨테이너의 상태를 판단해야 하는 시나리오에서 이상적인 체크방법. 컨테이너 내부에서 명령을 실행하고 상태 코드가 0으로 종료된다면 성공으로 간주한다.
    
- TCP Probes
    
    TCP 소켓 검사는 데몬으로 실행되고, 데이터베이스 서버, 파일 서버, 웹 서버 및 애플리케이션 서버와 같이 TCP 포트를 열어두는 애플리케이션에 이상적이다. 이 경우에는 probe 연결을 노드에서 수행하며 파드 내에서 수행하지 않는다. 따라서 호스트 매개변수에 서비스 이름을 사용할 수 없다. 

<br/>
<br/>
<br/>
<br/>
<br/>
<br/>

**Startup Probe**

startup probe는 컨테이너에서 실행 중인 application이 시작됐는지 검사한다. startup probe는 다른 probe들이 실행 되기 전에 제일 먼저 실행되고, 초기에 딱 한번 실행된다. 만약에 startup probe가 실패하면 컨테이너는 종료되고 pod의 restartPolicy에 따라 재시작 된다. 

```python
startupProbe:
    httpGet:
    path: /healthz
    port: 8080
    initialDelaySeconds: 3
    periodSeconds: 3
```

startup probe가 한번 성공하면 그 후에는 liveness probe가 뒤를 이어 받는다. 
<br/>
<br/>
<br/>

**Liveness Probe**

liveness probe는 컨테이너에 돌고있는 application이 정상적으로 작동하고 있는지 확인한다. 만약 비정상으로 작동되고 있다면 container를 죽이고 다시 redeploy한다. liveness probe로 application이 deadlock 상태인지 발견할 수 있다. 또한, readiness probe가 정상적으로 작동하는지에 상관없이 동작한다. 

```python
apiVersion: v1
kind: Pod
metadata:
    labels:
    test: liveness
    name: liveness-exec
spec:
    containers:
    - name: liveness
    image: registry.k8s.io/busybox
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -f /tmp/healthy; sleep 600
    livenessProbe:
        exec:
        command:
        - cat
        - /tmp/healthy
        initialDelaySeconds: 5
        periodSeconds: 5
```

<br/>
<br/>

**Readiness Probe**

readiness probe는 컨테이너가 요청을 처리할 준비가 됐는지를 검사한다. 만약 readiness probe의 결과가 실패라면, 쿠버네티스는 모든 서비스의 endpoints에서 그 컨테이너의 IP 주소를 제거한다 (ex. service load balancer에서 IP가 제거됨). 개발자들은 작동하는 컨테이너가 아직 트래픽 요청을 받지 말아야 할 때( 데이터베이스 connection, 다른 서비스가 작동 상태가 될 때까지 기다려야 할 때 등) readiness probe를 정의한다. 

쿠버네티스는 rolling update 시에 readiness probe에 의존한다. 새로운 서비스가 요청을 처리할 준비가 됐다는 신호를 받기 전까지 이전 컨테이너를 작동시킨다. 

```python
apiVersion: v1
kind: Pod
metadata:
name: goproxy
labels:
app: goproxy
spec:
containers:
- name: goproxy
image: registry.k8s.io/goproxy:0.1
ports:
- containerPort: 8080
readinessProbe:
    tcpSocket:
    port: 8080
    initialDelaySeconds: 5
    periodSeconds: 10
```

<br/>
<br/>

reference: https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/