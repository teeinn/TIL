**기본적으로 kubectl은 $HOME/.kube 디렉터리에서 config라는 이름의 파일을 찾는다. kubectl은 이 kubeconfig 파일을 사용하여 클러스터를 선택하고 서버와 통신을 위해 필요한 정보를 찾는다.** 

그렇다면 **kubeconfig** **파일이 하는 역할**이 무엇일까? 
<br/>
<br/>
- - - 

kubeconfig 파일은 Clusters, Users, Namespaces, Authentications에 대한 정보로 구성된다. 이 정보를 이용해 다양한 clusters에 접근할 수 있다. 다음은 일반적인 kubeconfig 파일의 형태를 보여준다. 

```python
apiVersion: v1
kind: Config
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://kubernetes.docker.internal:6443
  name: docker-desktop
users:
- name: docker-desktop
  user:
    client-certificate-data: REDACTED
    client-key-data: REDACTED
contexts:
- context:
    cluster: docker-desktop
    user: docker-desktop
  name: docker-desktop
current-context: docker-desktop
preferences: {}
```

1. Clusters
    
    Clusters는 사용자가 실행할 수 있는 다양한 Clusters에 대한 정보를 담고있는 목록이다. 각각의 cluster는 서버와 가능한 authentication에 대한 정보로 구성된다. 여기서 server는 kubernetes cluster의 주소를 나타낸다.
    
2. Users
    
    Cluster의 다른 사용자와 이러한 user의 인증 상세 정보에 대한 목록을 나타낸다. 사용자는 다음의 방법으로 인증할 수 있다. 
    
    - Certificates
        
        ```python
        users:
          - name: admin
            user:
              client-certificate-data: <base64 encoded client cert data>
              client-key-data: <base64 encoded client key>
        ```
        
    - Authentication tokens
        
        ```python
        users:
          - name: admin
            user:
              token: >_
                dGhpcyBpcyBhIHJhbmRvbSBzZW50ZW5jZSB0aGF0IGlzIGJhc2UgZW5jb2R
        ```
        
    - Basic authentication
        
        ```python
        users:
          - name: admin
            user:
              username: teddy-winters
              password: panda-summers
        ```
        
3. Contexts
    
    각각의 context는 클러스터, 네임스페이스, 사용자, 이 세 가지 매개 변수를 묶는데 사용된다. 기본적으로 kubectl 커맨드라인 툴은 현재 context의 매개 변수를 사용하여 클러스터와 통신한다.
    
    ```python
    contexts:
    - context:
        cluster: production
        namespace: live
        user: admin
      name: production-admin
    ```
    
    위의 예시는 다음과 같다. production-admin 이라는 cluster는 ‘admin’ user의 인증정보를 사용하여 production cluster의 live namespace에 접근 할 수 있다.
    

<br/>
<br/>

- - -
kubeconfig를 위해 사용할 수 있는 command

1. kubectl config view
    
    ```python
    » kubectl config view
    apiVersion: v1
    clusters:
    - cluster:
        certificate-authority-data: DATA+OMITTED
        server: https://kubernetes.docker.internal:6443
      name: docker-desktop
    - cluster:
        insecure-skip-tls-verify: true
        server: https://xxx.xxx.xxx:6443
      name: flask-test-cluster-cluster
    contexts:
    - context:
        cluster: docker-desktop
        user: docker-desktop
      name: docker-desktop
    - context:
        cluster: flask-test-cluster-cluster
        user: dave.rhee_flask-test-cluster
      name: flask-test-cluster-context
    current-context: flask-test-cluster-context
    kind: Config
    preferences: {}
    users:
    - name: dave.rhee_flask-test-cluster
      user:
        token: REDACTED
    - name: docker-desktop
      user:
        client-certificate-data: REDACTED
        client-key-data: REDACTED
    ```
    
2. kubectl config get-contexts
    
    ```python
    » kubectl config get-contexts
    CURRENT   NAME                         CLUSTER                      AUTHINFO                       NAMESPACE
              docker-desktop               docker-desktop               docker-desktop                 
    *         flask-test-cluster-context   flask-test-cluster-cluster   dave.rhee_flask-test-cluster
    ```
    
3. kubectl config use-context
    
    ```python
    » kubectl config use-context docker-desktop  
    Switched to context "docker-desktop".
    ------------------------------------------------------------------------------------------------
    » kubectl config get-contexts
    CURRENT   NAME                         CLUSTER                      AUTHINFO                       NAMESPACE
    *         docker-desktop               docker-desktop               docker-desktop                 
              flask-test-cluster-context   flask-test-cluster-cluster   dave.rhee_flask-test-cluster
    ```
    
<br/>
<br/>

reference: https://kubernetes.io/ko/docs/concepts/configuration/organize-cluster-access-kubeconfig/, https://medium.com/@deyagondsamarth/understanding-the-kubeconfig-3ef43e8716d, https://velog.io/@rhee519/kubernetes-context