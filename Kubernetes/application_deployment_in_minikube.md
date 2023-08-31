fastapi framework로 작성한 application과 db로 쓰인 postgreSQL을 쿠버네티스에 올려보는 작업을 해보려고 한다. cluster 구성을 위해 minikube를 사용 할 것이다. minikube 설치법은 공식 홈페이지에 잘 나와있으니 참고하면 된다. 
- - - 

1. **minikube 설치 후**
    
    ```bash
    $ minikube start
    ```
    
2. **postgreSQL pod를 위한 yaml 파일 작성법**
    
    CatchTable/k8s 폴더에 가보면 필요한 기본적인 yaml 파일들이 정리되어 있다. 
    
    secret.yaml, storage.yaml (pv & pvc), deployment.yaml, service.yaml이 필요하다.
    
    secret.yaml에는 postgreSQL 접속에 필요한 환경 변수를 base64 encoding을 통해 작성해놨고,
    
    ```bash
    apiVersion: v1
    kind: Secret
    metadata: 
      name: postgres-secret
      namespace: development
    type: Opaque
    data:
      POSTGRES_USER: bXl1c2Vy
      POSTGRES_PASSWORD: bXlwYXNzd29yZA==
      POSTGRES_DB: bXlkYg==
    ```
    
    storage.yaml에는 일단 현재는 minikube를 사용 중이기 때문에 local storage를 사용한다. 후에 postgreSQL pod를 deployment가 아닌 statefulset으로 변경하게 되면 클라우드 스토리지를 사용해서 다이나믹하게 저장소를 생성 할 것이다. 
    
    ```bash
    kind: PersistentVolume
    apiVersion: v1
    metadata:
      name: postgres-pv-volume
      labels:
        type: local
        app: postgres
    spec:
      storageClassName: manual
      capacity:
        storage: 1Gi
      accessModes:
        - ReadWriteMany
      hostPath:
        path: "/mnt/data"
    
        
    ---
    kind: PersistentVolumeClaim
    apiVersion: v1
    metadata:
      name: postgres-pv-claim
      namespace: development
      labels:
        app: postgres
    spec:
      storageClassName: manual
      accessModes:
        - ReadWriteMany
      resources:
        requests:
          storage: 1Gi
    ```
    
    deployment와 service를 한 파일에 모아놓았다. pod를 생성할 때 secret.yaml 파일에 기재된 환경 변수를 불러와야 제대로 배포될 수 있다. service는 Cluster IP로 클러스터 내부에서만 접근 가능하게 하였고, 같은 클러스터에 있는 fastapi application에서 접속할 수 있다.
    
    ```bash
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: postgres
      namespace: development
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: postgres
      template:
        metadata:
          labels:
            app: postgres
        spec:
          containers:
            - name: postgres
              image: postgres
              resources:
                limits:
                  memory: "128Mi"
                  cpu: "500m"
              ports:
                - containerPort: 5432
              env:
                - name: POSTGRES_USER
                  valueFrom:
                    secretKeyRef:
                      name: postgres-secret
                      key: POSTGRES_USER
                - name: POSTGRES_PASSWORD
                  valueFrom:
                    secretKeyRef:
                      name: postgres-secret
                      key: POSTGRES_PASSWORD
                - name: POSTGRES_DB
                  valueFrom:
                    secretKeyRef:
                      name: postgres-secret
                      key: POSTGRES_DB
    
              volumeMounts:
                - mountPath: /var/lib/postgresql/data
                  name: postgredb
          volumes:
            - name: postgredb
              persistentVolumeClaim:
                claimName: postgres-pv-claim
    
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: postgres-service
      namespace: development
    spec:
      selector:
        app: postgres
      ports:
        - protocol: TCP
          port: 5432
          targetPort: 5432
    ```
    
3. **fastapi application을 위한 yaml**
    
    마찬가지로 CatchTable/k8s 폴더로 가면 파일들이 정리되어 있는데, deployment, service, ingress, secret yaml 파일들이 필요하다. 위의 postgreSQL과 다른 점은 ingress를 정의함으로써 나중에 설치할 ingress controller로 부터 트래픽을 받을 수 있다. 
    
    먼저, secret.yaml 을 살펴보자. fastapi application에서 db에 접속할 수 있으려면 필요한 환경 변수 값들을 저장해 놓았다. 이것 역시도 base64로 인코딩하였다. 
    
    ```bash
    apiVersion: v1
    kind: Secret
    metadata:
      name: catchtable-secret
      namespace: development
      labels:
        app: catchtable
    data:
      DATABASE_USERNAME: bXl1c2Vy
      DATABASE_PASSWORD: bXlwYXNzd29yZA==
      DATABASE_HOST: cG9zdGdyZXMtc2VydmljZQ==   # postgres-service
      DATABASE_NAME: bXlkYg==
    ```
    
    그리고 deployment와 service를 정의하였는데, deployment에는 secret에서 불러오는 환경 변수 값들이 들어가고, private docker registry에서 이미지를 pull 해온다. service를 clusterIP로 정의함으로써 클러스터 내에서만 접근할 수 있게 하였고 따로 ingress를 정의하여 외부에서 접근할 수 있게 할 것이다.
    
    ```bash
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: catchtable
      namespace: development
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: catchtable
      template:
        metadata:
          labels:
            app: catchtable
        spec:
          containers:
            - name: catchtable
              image: teeinn/catchtable:latest
              ports:
                - containerPort: 8000
              envFrom:
                - secretRef:
                    name: catchtable-secret
              readinessProbe:
                httpGet:
                  port: 8000
                  path: /docs
                initialDelaySeconds: 15
              livenessProbe:
                httpGet:
                  port: 8000
                  path: /docs
                initialDelaySeconds: 15
                periodSeconds: 15
    
    ---
    # apiVersion: v1
    # kind: Service
    # metadata:
    #   name: catchtable
    # spec:
    #   type: LoadBalancer
    #   selector:
    #     app: catchtable
    #   ports:
    #     - port: 8000
    #       targetPort: 8000
    #       nodePort: 30000
    #       protocol: TCP
    
    apiVersion: v1
    kind: Service
    metadata:
      name: catchtable
      namespace: development
    spec:
      type: ClusterIP
      selector:
        app: catchtable
      ports:
        - port: 8000
          targetPort: 8000
          protocol: TCP
    ```
    
    이제, ingress를 정의해보자. ingress에는 외부에서 접근하기 위한 url 규칙을 선언해 주었고, 포트도 정해주었다. 이때, ingress.class 설정이 중요한데 이것이 설정되어 있어야 nginx ingress controller에서 찾을 수 있다. 
    
    ```bash
     apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
      name: my-ingress
      namespace: development
      annotations:
        # nginx.ingress.k8s.io/rewrite-target: /
        kubernetes.io/ingress.class: nginx
    spec:
      rules:
      - host: myapp.catchtable.com
        http:
          paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: catchtable
                port:
                  number: 8000
    ```
    
4. **namespace 폴더도 구성하여 namespace를 생성하는 yaml 파일을 작성해 두었다.** 
5. **이제, 본격적으로 minikube에 배포를 해보자.** 
    
    ```bash
    $ kubectl apply -f namespace
    $ kubectl apply -f postgres/
    $ kubectl apply -f fastapi/
    $ kubectl get all -n development  # 잘 배포되었는지 확인해보자
    
    $ minikube service list
    |-------------|------------------|--------------|-----|
    |  NAMESPACE  |       NAME       | TARGET PORT  | URL |
    |-------------|------------------|--------------|-----|
    | default     | kubernetes       | No node port |     |
    | development | catchtable       | No node port |     |
    | development | postgres-service | No node port |     |
    | kube-system | kube-dns         | No node port |     |
    |-------------|------------------|--------------|-----|
    ```
    
6. **모든 것이 완료 되었으면, 이제 minikube에 nginx ingress controller를 설치해보자. minikube 자체적으로 제공하는 방법을 사용하자.**
    
    ```bash
    $ minikube addons enable ingress
    $ minikube service list
    |---------------|------------------------------------|--------------|---------------------------|
    |   NAMESPACE   |                NAME                | TARGET PORT  |            URL            |
    |---------------|------------------------------------|--------------|---------------------------|
    | default       | kubernetes                         | No node port |                           |
    | development   | catchtable                         | No node port |                           |
    | development   | postgres-service                   | No node port |                           |
    | ingress-nginx | ingress-nginx-controller           | http/80      | http://192.168.49.2:32197 |
    |               |                                    | https/443    | http://192.168.49.2:31306 |
    | ingress-nginx | ingress-nginx-controller-admission | No node port |                           |
    | kube-system   | kube-dns                           | No node port |                           |
    |---------------|------------------------------------|--------------|---------------------------|
    ```
    
7. **이제 외부에서 ingress-ngix의 ip를 fastapi application ingress에 정의했던 url 주소와 mapping 시켜보자**.
    
    ```bash
    $ sudo nano /etc/hosts
    
    127.0.0.1       localhost
    192.168.49.2    myapp.catchtable.com
    ```
    
8. **Address Bar에 이제 [myapp.catchtable.com](http://myapp.catchtable.com) 또는 [myapp.catchtable.com/docs](http://myapp.catchtable.com/docs) 를 타이핑하면 application 접속이 가능한 것을 볼 수 있다. DB도 문제없이 잘 돌아간다.**