


## 컨테이너 자동확장 - HPA

```shell
# 테스트용 pod 생성
$ kubectl create deploy order --image=jinyoung/monolith-order:v20210504
$ kubectl expose deploy order --port=8080
```

```shell
# 스케일 테스트
$ kubectl scale deploy order --replicas=3
$ kubectl scale deploy order --replicas=1
```

```shell
# HPA 적용
$ kubectl autoscale deployment order --cpu-percent=50 --min=1 --max=3
```

```shell
# 적용된 HPA 확인
$ root@k3s:/home/ubuntu/k3s# kubectl get hpa
NAME    REFERENCE          TARGETS         MINPODS   MAXPODS   REPLICAS   AGE
order   Deployment/order   <unknown>/50%   1         3         0          3s
```

신규 야믈 파일 생성 후 order-deploy.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: order
  labels:
    app: order
spec:
  replicas: 1
  selector:
    matchLabels:
      app: order
  template:
    metadata:
      labels:
        app: order
    spec:
      containers:
        - name: order
          image: jinyoung/monolith-order:v20210602
          ports:
            - containerPort: 8080
          resources:
            requests:
              cpu: "200m"            
          readinessProbe:
            httpGet:
              path: '/actuator/health'
              port: 8080
            initialDelaySeconds: 10
            timeoutSeconds: 2
            periodSeconds: 5
            failureThreshold: 10
          livenessProbe:
            httpGet:
              path: '/actuator/health'
              port: 8080
            initialDelaySeconds: 120
            timeoutSeconds: 2
            periodSeconds: 5
            failureThreshold: 5
```

```bash
# 현재, 배포된 주문서비스를 삭제하고 재배포한다.
$ kubectl delete -f order-deploy.yaml
$ kubectl apply -f order-deploy.yaml
```

## 공통(부하테스트)

```shell
# siege pod 생성
$ kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: siege
spec:
  containers:
  - name: siege
    image: apexacme/siege-nginx
EOF
```

```
$ kubectl exec -it siege -- /bin/bash
$ siege -c1 -t2S -v http://order:8080/orders
$ exit
``
