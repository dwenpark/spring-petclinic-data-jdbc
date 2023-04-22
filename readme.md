# 실행 방법

## 빌드
> ./gradlew build
> 
> docker build

## 배포
> kubectl apply -f /k8s/mysql.yml
> 
> kubectl apply -f /k8s/petclinic.yml

- 빌드 후, petclinic.yml의 이미지명을 바꾸거나 deployment 배포 후 이미지 업데이트


# 요구사항
### ✅ gradle을 사용하여 어플리케이션과 도커이미지를 빌드한다.
- gradle init으로 maven -> gradle 변환
- Dockerfile 작성

### ✅ 어플리케이션의 log는 host의 /logs 디렉토리에 적재되도록 한다.

```
# application.properties

logging.file.path=/logs
```

```
# petclinic.yml

          volumeMounts:
            - name: logs
              mountPath: /logs
      volumes:
        - name: logs
          emptyDir: {}
```
- 우선 emptyDir로 구성해두었는데 필요하다면 해당 log도 PVC로 관리할 수 있음.

### ✅ 정상 동작 여부를 반환하는 api를 구현하며, 10초에 한번 체크하도록 한다.
### ✅ 3번 연속 체크에 실패하면 어플리케이션은 restart 된다.

```
# petclinic.yml

          livenessProbe:
            httpGet:
              path: /actuator/health/liveness
              port: 8080
            initialDelaySeconds: 10
            periodSeconds: 10
            failureThreshold: 3
          readinessProbe:
            httpGet:
              path: /actuator/health/readiness
              port: 8080
            initialDelaySeconds: 30
            periodSeconds: 10
            failureThreshold: 3

```

### ✅ 종료 시 30초 이내에 프로세스가 종료되지 않으면 SIGKILL로 강제 종료 시킨다.

```
# petclinic.yml

    spec:
      terminationGracePeriodSeconds: 30
```

### ✅ 배포 시와 scale in/out 시에는 유실되는 트래픽이 없어야한다.

- ReadinessProbe로 petclinic이 healthy 하지 않을 경우, 트래픽을 수신하지 않기 때문에 유실되는 트래픽 없음.
- 배포 type이 rollingUpdate이기 때문에 새로 뜨는 pod가 readiness하지 않을 경우, 트래픽을 받지 않음.

### ✅ 어플리케이션 프로세스는 root 계정이 아닌 uid:1000으로 실행한다.

- Dockerfile에서 uid 1000으로 실행되도록 설정
```
# petclinic.yml

    spec:
	  ...
      securityContext:
        runAsUser: 1000
```
- 루트 등의 유저로 변경이 필요할시 오버라이드 가능.

### ✅ DB도 kubernetes에서 실행하며 재 실행 시에도 변경된 데이터는 유실되지 않도록 설정한다.

- 클러스터의 default storageclass를 이용하도록 설정

- 특정 노드에만 실행된다면 hostpath를 이용할 수도 있음.

### ✅ 어플리케이션과 DB는 cluster domain으로 통신한다.

- svc로 jdbc

### ✅ nginx-ingress-controller 통해 어플리케이션에 접속이 가능하다.

- [nginx-ingress-controller 공식 배포 가이드](https://kubernetes.github.io/ingress-nginx/deploy/)

- 참고하여 설치 -> NKS로 테스트할 경우 azure 가이드 참고

### ✅ namespace는 default를 사용한다. 
### ✅ README.md 파일에 실행 방법을 기술한다.
