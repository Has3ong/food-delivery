
## 클라우드 아키텍처 구성, MSA 아키텍처 구성도
## 도메인 분석 - 이벤트 스토밍
## 분산트랜잭션 - Saga
## 보상처리 - Compensation
## 단일진입점 - Gateway
## 분산 데이터 프로젝션 - CQRS

## 컨테이너 자동확장 - HPA

- 참조 : 마이크로서비스 확장과 오토 스케일아웃
- URL : https://www.msaez.io/#/courses/cna-full/093c4d10-b34b-11ee-94ee-83b5b47c8792/ops-autoscale

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


## 컨테이너로부터 환경분리 - ConfigMap/Secret
- 참조 : PDF 392 PAGE

## 클라우드스토리지 활용 - PVC
- 참조 : PDF 378 PAGE

## 셀프힐링/무정지배포 - Liveness/Rediness Probe
- 참조 : 마이크로서비스 확장과 오토 스케일아웃
- URL : https://www.msaez.io/#/courses/cna-full/093c4d10-b34b-11ee-94ee-83b5b47c8792/ops-autoscale
## 서비스 메쉬 응용 - Mesh
- 참조 : 마이크로서비스 확장과 오토 스케일아웃
- URL : https://www.msaez.io/#/courses/cna-full/093c4d10-b34b-11ee-94ee-83b5b47c8792/ops-autoscale
## 통합 모너티렁 - Loggregation/Monitoring
- 참조 : 마이크로서비스 확장과 오토 스케일아웃
- URL : https://www.msaez.io/#/courses/cna-full/093c4d10-b34b-11ee-94ee-83b5b47c8792/ops-autoscale

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
