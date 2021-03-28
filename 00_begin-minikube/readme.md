# 쿠버네티스 안내서
[reference](https://subicura.com/k8s/guide/#%E1%84%80%E1%85%A1%E1%84%8B%E1%85%B5%E1%84%83%E1%85%B3)

---
# 준비
## 쿠버네티스(minikube) 설치
- 쿠버네티스 운영환경 설치를 위해선 최소 3대의 마스터 서버와, 컨테이너 배포를 위한 n개의 노드 서버 필요
- 처음부터 하다가는 멘탈 터져서 구름혐오증 생김 ㅇㅇ
- 지금은 마스터와 노드를 하나의 서버에 설치하여 관리할거임
- 대표적으로 **minikube**, **k3s**, **docker for desktop**, **kind**가 있음

## minikube
- 쿠버네티스 클러스터 실행요건 (많음!)
    - (필수) scheduler, controller, api-server, etcd, kubelet, kube-proxy
    - (선택) dns, ingress controller, storage class 등
- minikube를 사용하면 이러한 설치를 단일화 해줌
- 무료임!!

## kubectl
- 쿠버네티스 CLI도구 (쿠버네티스 클러스터에 명령어를 전달하는 가장 흔한 방법)
- 그럼 다른 방법도 있음??

## windows 10 - minikube, kubectl 설치
- **virtual box**를 사용하는 방법을 볼거임 (내 기기가 전부 window10 home이라...)
1. **virtual box** 설치 
    - [virtual.org](https://www.virtualbox.org/)에 접속해서 window host 다운로드
    - 자동으로 path등록 됨
2. **minikube** 설치
    - [파일 링크](https://github.com/kubernetes/minikube/releases/latest/download/minikube-installer.exe)를 클릭하면 `minikube-installer.exe`파일이 설치됨
    - 실행 후 설치 진행
    - 자동으로 path등록 됨
    - 기본 명령어  (기억하자!)
    ~~~console
    # 버전확인
    minikube version

    # 가상머신 시작 (반드시 관리자권한으로 실행, 근데 난 그냥 되던데...)
    # driver 에러가 발생한다면 virtual box를 사용
    minikube start --driver=virtualbox

    # 특정 k8s 버전 실행
    minikube start --kubernetes-version=v1.20.0

    # 상태확인
    minikube status

    # 정지
    minikube stop

    # 삭제
    minikube delete

    # ssh 접속
    minikube ssh

    # ip 확인
    minikube ip
    ~~~
    - docker driver라는 걸 사용할 수도 있지만, 다음기회에!
    - 이 외 linux, macOS에서도 가능 (reference 페이지에 다 있음)
3. kubectl 설치
    - **kubectl v1.20**버전 설치
    - `curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.20.0/bin/windows/amd64/kubectl.exe` 명령행 사용
    - PATH 등록이 안되어 있으니, global하게 사용하려면 등록 필요 [링크](https://stackoverflow.com/questions/4822400/register-an-exe-so-you-can-run-it-from-any-command-line-in-windows)에서 확인 (자주 쓸 듯)
    - **kubectl version**으로 테스트하기

---
# 실습
- 실습은 `minikube start --driver=virtualbox --kubernetes-version=v1.20.0`를 이용한 minikube실행 후 ㄱㄱ
- `C:\workplace\com_study\minikube`에서 진행
- 실습 초기화 방법
    1. `minikube stop`
    2. `minikube delete`
    3. `minikube start --driver=virtualbox --kubernetes-version=v1.20.0`
## 워드프레스 배포
- 워드프레스란? 설치형 블로그 어쩌고 하는거임, PHP와 MySQL로 구성되어 있음
- 이걸 쿠버네티스로 배포해보자 (웹 애플리케이션 배포는 가장 흔한 쿠버네티스 작업 중 하나!!)
- **docker-compose**를 이용한 배포와의 차이점도 알아보자
### (비교를 위한) 도커 컴포즈 배포
![docker-compose deploy](https://subicura.com/k8s/assets/img/wordpress-docker.92b3c859.png)
- **docker-compose.yml**을 다음과 같이 작성하여 배포하면 됨
~~~yml
version: "3"

services:
  wordpress:
    image: wordpress:5.5.3-apache
    environment:
      WORDPRESS_DB_HOST: mysql
      WORDPRESS_DB_PASSWORD: password
    ports:
      - "30000:80"

  mysql:
    image: mysql:5.6
    environment:
      MYSQL_ROOT_PASSWORD: password
~~~
- **워드프레스 컨테이너 + MYSQL컨테이너 + 각각의 포트 및 환경변수설정**이 조합된 컴비네이션!!! 

### 쿠버네티스를 이용한 배포
![kubernetes deploy](https://subicura.com/k8s/assets/img/wordpress-k8s.26407f2b.png)
- Service, Pod, ReplicaSet, Deployment... 뭔지 일단 모르겠고
- **wordpress-k8s.yml**을 작성해보자
~~~yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress-mysql
  labels:
    app: wordpress
spec:
  selector:
    matchLabels:
      app: wordpress
      tier: mysql
  template:
    metadata:
      labels:
        app: wordpress
        tier: mysql
    spec:
      containers:
        - image: mysql:5.6
          name: mysql
          env:
            - name: MYSQL_ROOT_PASSWORD
              value: password
          ports:
            - containerPort: 3306
              name: mysql

---
apiVersion: v1
kind: Service
metadata:
  name: wordpress-mysql
  labels:
    app: wordpress
spec:
  ports:
    - port: 3306
  selector:
    app: wordpress
    tier: mysql

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress
  labels:
    app: wordpress
spec:
  selector:
    matchLabels:
      app: wordpress
      tier: frontend
  template:
    metadata:
      labels:
        app: wordpress
        tier: frontend
    spec:
      containers:
        - image: wordpress:5.5.3-apache
          name: wordpress
          env:
            - name: WORDPRESS_DB_HOST
              value: wordpress-mysql
            - name: WORDPRESS_DB_PASSWORD
              value: password
          ports:
            - containerPort: 80
              name: wordpress

---
apiVersion: v1
kind: Service
metadata:
  name: wordpress
  labels:
    app: wordpress
spec:
  type: NodePort
  ports:
    - port: 80
  selector:
    app: wordpress
    tier: frontend
~~~
- kubectl을 이용한 배포. `kubectl apply -f wordpress-k8s.yml`
- 배포상태 확인. `kubectl get all`
    - **wordpress-blahblah**와 **pod/wordpress-mysql-blahblah**의 *status*가 *Running*인지 확인
    - **service/wordpress**의 *포트번호* 확인 
    ![status](./wordpress-status.png)
- `minikube ip`로 ip알아내기
    ![minikube-ip](./minikube-ip.png)
- [`minikube ip`를 통해 얻은 ip:*포트번호*] 로 웹에 접속해서 확인해보기
    ![wordpress-web](./wordpress-web.png)
  
*성공!!!*

---
# 뒷정리
- 완전한 정리 ㄱㄱ
- 워드프레스 리소스 제거. `kubectl delete -f wordpress-k8s.yml`
- minikube 종료. `minikube stop`
- minikube 제거. `minikube delete`
