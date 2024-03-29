---
author: winverse
title:  "[Docker] 1.기본 개념 정리 및 자주 사용하는 Command"
categories: 
    - infra
tags: 
    - docker
is_published: true
show_date: true
date: "2023-03-30"
description: "왜 Docker를 사용해야하는지, Image와 Container의 개념을 이해하고 자주 사용하는 Command를 정리합니다."
---

# 여기서 배우게 될 것들
1. Docker를 사용하는 이유와 성능상의 이점
2. Image
3. Container
4. Volume
5. Network

# 1. Why use the Docker?
## 1-1. 좋아지는 개발자의 경험
도커는 개발 및 배포를 단순화하는데 그 이유가 컨테이너를 사용하기 때문입니다. 컨테이너는 애플리케이션과 그 종속성을 함께 묶어서 실행 가능한 패키지로 만들어줍니다. 이렇게 하면 애플리케이션을 다양한 환경에서 일관되게 실행할 수 있습니다. 

예를 들어, 개발자가 로컬 컴퓨터에서 작업한 애플리케이션을 테스트 서버 또는 프로덕션 서버로 배포할 때 종종 문제가 발생합니다. 이러한 문제는 서버의 운영 체제, 라이브러리 버전 등이 개발자의 로컬 컴퓨터와 다르기 때문입니다. 이 경우 도커를 사용하면 애플리케이션과 종속성을 함께 묶어서 컨테이너로 만들 수 있습니다. 그런 다음 이 컨테이너를 테스트 서버 또는 프로덕션 서버에서 실행할 수 있습니다. 이렇게 하면 애플리케이션이 일관되게 실행되므로 개발 및 배포가 단순화됩니다. 이렇게 하면 개발자의 경험이 좋아집니다.

## 1-2. VM 대비 성능
도커 컨테이너는 가상 머신보다 성능이 뛰어납니다. 가상 머신은 전체 OS를 로드해야 하기 때문에 도커 컨테이너보다 리소스가 더 많이 필요합니다. 도커 컨테이너의 경량 아키텍처는 가상 머신보다 리소스가 적게 필요합니다. 이로 인해 도커 컨테이너는 거의 기본 성능을 제공합니다. 경량화되어 있기 때문에 몇 밀리초 안에 시작할 수 있습니다. 가상 머신을 시작하는 것은 컴퓨터 내부에 독립적인 기계를 설정하는 것과 같습니다. VM 인스턴스를 시작하는 데 몇 분 정도 걸릴 수 있습니다.

## 1-3. Docker를 사용하므로 달성하려는 목표
도커를 사용하면 개발자들이 애플리케이션을 더 쉽게 개발하고 배포할 수 있습니다. 이는 도커가 개발 및 배포를 단순화하기 때문입니다. 또한 도커는 팀원 간의 협업을 개선하는 데 도움이 됩니다. 모든 팀원이 동일한 소프트웨어 종속성 및 실행 환경으로 작업할 수 있도록 합니다.


# 2. Image
도커의 두 가지 핵심 구성 요소 중 하나인 이미지는 컨테이너의 청사진/템플릿입니다.    
이미지는 읽기 전용이며 애플리케이션과 필요한 애플리케이션 환경(운영 체제, 런타임, 도구 등)을 포함합니다. 이미지 자체는 실행되지 않고 대신 컨테이너로 실행될 수 있습니다. 이미지는 사전 빌드(예: DockerHub에서 찾을 수 있는 공식 이미지)되거나 Dockerfile을 정의하여 직접 빌드할 수 있습니다. Dockerfile에는 이미지가 빌드될 때는 실행되는 Command마다 이미지의 계층(Layer)을 생성합니다. 여러개의 Layer는 이미지를 효율적으로 재구축하고 공유하는 데 사용됩니다. CMD Command는 특별합니다. 이미지가 빌드될 때가 아니라 해당 이미지를 기반으로 컨테이너가 생성되고 시작될 때 실행됩니다.

## 2-1. Image를 만드는 기반인 Dockerfile 예시 (Nodejs)
```dockerfile
FROM node:18

WORKDIR /app

# Build 최적화
COPY package.json .

RUN npm install

# Build 최적화
COPY . .

# Only 문서화 목적
EXPOSE 80

# 컨테니어가 생성되어 시작될때 실행함
CMD ["node", "app.js"]
```

### 2-1-1. COPY를 두번 실행하는 이유
첫 번째 COPY 명령어는 package.json 파일만 /app 디렉토리로 복사합니다. 그런 다음 RUN npm install 명령어를 사용하여 필요한 종속성을 설치합니다.

두 번째 COPY 명령어는 나머지 파일들을 모두 /app 디렉토리로 복사합니다.

이렇게 하는 이유는 도커가 이미지를 빌드할 때 캐시를 사용하기 때문입니다. 각 명령어가 실행될 때마다 도커는 새로운 레이어를 생성하고 캐시합니다. 이전에 빌드된 이미지와 동일한 명령어가 실행되면 도커는 캐시된 레이어를 재사용하여 빌드 시간을 단축합니다.

따라서 package.json 파일만 복사하고 종속성을 설치한 후 나머지 파일들을 복사하면, 소스 코드가 변경되더라도 package.json 파일이 변경되지 않으면 도커는 캐시된 레이어를 재사용하여 종속성 설치 과정을 건너뛸 수 있습니다. 이렇게 하면 이미지 빌드 시간이 크게 단축됩니다.

### 2-1-2. EXPOSE 명령어
`EXPOSE`는 Dockerfile의 명령어 중 하나로 컨테이너가 리스닝할 포트를 지정합니다. 이 명령어는 컨테이너가 실행될 때 호스트와 네트워크 연결을 설정하는 데 사용됩니다.

예를 들어 EXPOSE 80이라고 지정하면 컨테이너가 80번 포트에서 수신 대기하도록 지시합니다. 그러나 이 명령어는 단순히 문서화를 위한 것일 뿐이며 실제로 포트를 게시하지는 않습니다. 포트를 게시하려면 docker run 명령어에서 -p 플래그를 사용하여 호스트의 포트와 컨테이너의 포트를 매핑해야 합니다.

# 3. Container
도커의 다른 핵심 구성 요소는 컨테이너입니다. 컨테이너는 이미지의 실행 인스턴스입니다. 컨테이너를 생성할 때(docker run을 통해) 이미지 위에 얇은 읽기/쓰기 레이어가 추가됩니다. 따라서 하나의 이미지를 기반으로 여러 개의 컨테이너를 시작할 수 있습니다. 모든 컨테이너는 격리되어 실행되므로 애플리케이션 상태나 작성된 데이터를 공유하지 않습니다. 컨테이너 내부의 애플리케이션을 시작하려면 컨테이너를 생성하고 시작해야 합니다. 따라서 개발 및 프로덕션에서 실행되는 것은 최종적으로 컨테이너입니다.

## 3-1. 자주 사용되는 명령어
- `docker build .` : Dockerfile을 빌드하고 파일을 기반으로 자신의 이미지를 만듭니다.
    - `-t NAME:TAG` : 이미지에 이름과 태그를 지정합니다.
- `docker run IMAGE_NAME` : 이미지 IMAGENAME(또는 이미지 ID 사용)을 기반으로 새 컨테이너를 생성하고 시작합니다.
    - `--name NAME` : 컨테이너에 이름을 지정합니다. 이름은 중지 및 제거 등에 사용할 수 있습니다.
    - `-d` : 컨테이너를 분리 모드로 실행합니다. 즉, 컨테이너가 출력하는 출력은 보이지 않으며 명령 프롬프트/터미널은 컨테이너가 중지될 때까지 대기하지 않습니다.
    - `-it` : 컨테이너를 "대화형" 모드로 실행합니다. 그러면 컨테이너/애플리케이션은 명령 프롬프트/터미널을 통해 입력을 받을 준비가 됩니다. -it 플래그를 사용하면 CMD + C로 컨테이너를 중지할 수 있습니다.
    - `--rm` : 컨테이너가 중지될 때 자동으로 제거합니다.
- `docker ps` : 실행 중인 모든 컨테이너를 나열합니다.
    - `-a` : 중지된 컨테이너를 포함한 모든 컨테이너를 나열합니다.
- `docker images` : 로컬에 저장된 모든 이미지를 나열합니다.
- `docker rm CONTAINER` : 이름이 CONTAINER인 컨테이너를 제거합니다(컨테이너 ID도 사용할 수 있음).
- `docker rmi IMAGE` : 이름/ID로 이미지를 제거합니다.
- `docker container prune` : 모든 중지된 컨테이너를 제거합니다.
- `docker image prune` : 모든 dangling 이미지(태그가 없는 이미지)를 제거합니다.
    - `-a` : 로컬에 저장된 모든 이미지를 제거합니다.
- `docker push IMAGE` : DockerHub(또는 다른 레지스트리)에 이미지를 푸시합니다. 이미지 이름/태그는 저장소 이름/URL을 포함해야 합니다.
- `docker pull IMAGE` : DockerHub(또는 다른 레지스트리)에서 이미지를 가져옵니다(pull). 이 작업은 docker run IMAGE를 실행할 때 이미지가 이전에 가져오지 않은 경우 자동으로 수행됩니다.

# 4. Volume
도커 볼륨은 컨테이너에서 생성된 데이터를 저장하는 데 사용되는 메커니즘입니다. 컨테이너가 삭제되더라도 볼륨에 저장된 데이터는 유지됩니다. 이를 통해 컨테이너 간에 데이터를 공유하거나 컨테이너가 삭제된 후에도 데이터를 유지할 수 있습니다. Volume에 유형에는 `Anonymouse Volumes`, `Named Volumes`, `Bind Mounts` 세가지가 있으며 

`docker volume create my-volume`을 실행하면 my-volume이라는 이름의 새 볼륨을 생성하여 (Named Volumes) 컨테이너와 연결하면 데이터를 유지하여서 사용할 수가 있습니다.

## 4-1. Anonymouse Volumes
익명 볼륨(anonymous volume)은 이름이 지정되지 않은 도커 볼륨입니다. 익명 볼륨은 docker run 명령어에서 -v 플래그를 사용하여 컨테이너의 특정 디렉토리에 마운트할 수 있습니다. 예를 들어 `docker run -v /data`와 같이 실행하면 새로운 익명 볼륨이 생성되고 컨테이너의 /data 디렉토리에 마운트됩니다.

익명 볼륨은 일반 볼륨과 마찬가지로 도커 호스트의 파일 시스템에 저장됩니다. 그러나 익명 볼륨은 이름이 지정되지 않았기 때문에 관리하기 어려울 수 있습니다. 따라서 가능한 경우 이름이 지정된 볼륨을 사용하는 것이 좋습니다.

익명 볼륨도 컨테이너가 중지되더라도 자동으로 삭제되지 않습니다. 익명 볼륨을 삭제하려면 `docker volume prune` 명령어를 사용하여 사용하지 않는 모든 볼륨을 삭제해야 합니다.

## 4-2. Named Volumes
Named Volume은 이름이 지정된 도커 볼륨입니다. Named Volume은 docker volume create 명령어를 사용하여 생성할 수 있습니다. 예를 들어 `docker volume create my-volume`을 실행하면 my-volume이라는 이름의 새 볼륨이 생성됩니다.

Named Volume은 docker run 명령어에서 -v 플래그를 사용하여 컨테이너의 특정 디렉토리에 마운트할 수 있습니다. 예를 들어 `docker run -v my-volume:/data`와 같이 실행하면 my-volume이라는 볼륨이 컨테이너의 /data 디렉토리에 마운트됩니다.

Named Volume은 익명 볼륨과 마찬가지로 도커 호스트의 파일 시스템에 저장됩니다. 그러나 Named Volume은 이름이 지정되어 있기 때문에 관리하기 쉽습니다. Named Volume을 삭제하려면 docker volume rm VOLUME_NAME 명령어를 사용하여 지정된 이름의 볼륨을 삭제할 수 있습니다.

## 4-3. Bind Mounts
Bind Mounts는 도커 호스트의 파일이나 디렉토리를 컨테이너의 파일 시스템에 마운트하는 방법입니다. Bind Mounts를 사용하면 호스트의 파일 시스템에서 데이터를 직접 읽고 쓸 수 있습니다.

Bind Mounts는 docker run 명령어에서 -v 플래그를 사용하여 설정할 수 있습니다. 예를 들어 `docker run -v /path/on/host:/path/in/container`와 같이 실행하면 호스트의 /path/on/host 디렉토리가 컨테이너의 /path/in/container 디렉토리에 마운트됩니다.

Bind Mounts를 이용하여 개발 중인 애플리케이션에 대한 코드가 컨테이너의 재시작 없이 즉각적으로 반영하도록 할 수 있습니다.

```sh
# /path/on/host를 바꿀 수 있음, 단 shell 환경에서만 가능합니다
# in mac
docker run -v ${pwd}:/path/in/container 

# in windows with powershell
docker run -v ${pwd}:/path/in/container 

# in windows with CMD
docker run -v %cd%:/path/in/container
```

Bind Mounts는 개발 중에 소스 코드와 같은 데이터를 실시간으로 공유하는 데 유용합니다. 그래서 Docker를 이용하여 개발중에 사용하기에 좋습니다. 그러나 Bind Mounts는 호스트의 파일 시스템에 직접 접근하기 때문에 사용할 때 주의해야 합니다. 가능한 경우 볼륨을 사용하는 것이 좋습니다.

# 5. Network
Docker 네트워크는 컨테이너들이 서로 연결되거나 비-Docker 작업과 연결될 수 있도록 해줍니다. Docker 컨테이너와 서비스는 Docker에서 배포되었는지 여부와 상관없이 서로 통신할 수 있습니다.

Docker의 네트워킹 시스템은 드라이버를 사용하여 플러그 가능합니다. 기본적으로 제공되는 네트워크 드라이버에는 bridge, host, overlay, macvlan 등이 있습니다

- bridge: 드라이버는 기본 네트워크 드라이버로, 독립 실행형 컨테이너가 통신해야 할 때 주로 사용됩니다. - host: 드라이버는 스탠드얼론(독립 실행형) 컨테이너의 경우 컨테이너와 Docker 호스트 사이의 네트워크 격리를 제거하고 호스트의 네트워킹을 직접 사용합니다. 
- overlay: 드라이버는 여러 Docker 데몬을 연결하고 스웜 서비스가 서로 통신할 수 있도록 합니다.
- macvlan: 컨테이너에 커스텀 MAC 주소를 설정할 수 있습니다. 그러면 이 주소를 해당 컨테이너와 통신하는데 사용할 수 있습니다.
- none: 모든 네트워킹이 비활성화 됩니다.

Docker를 설치하면 docker0 라는 가상 인터페이스가 생성됩니다. docker0 는 일반적인 가상 인터페이스가 아니며 도커가 자체적으로 제공하는 네트워크 드라이버 중 브리지 (Bridge)에 해당합니다.

## 5-1. 사용 예시
먼저, docker network create 명령어를 사용하여 사용자 정의 네트워크를 만듭니다.
```sh
docker network create my-network
```
그런 다음 docker run 명령어에 --network 옵션을 추가하여 컨테이너를 실행하고 해당 네트워크에 연결합니다.

```sh
docker run -d --name my-container --network my-network my-image
```
이제 my-container 컨테이너는 my-network 네트워크에 연결되어 있습니다.

기존에 실행 중인 컨테이너를 네트워크에 연결하려면 docker network connect 명령어를 사용합니다.

```sh
docker network connect my-network my-container
```
이제 my-container 컨테이너는 my-network 네트워크에 연결되어 있습니다.


# 시리즈
[2. [Docker] 다중 컨테이너 애플리케이션 구축하기](/infra/docker-handle-multi-container)  
[3. [Docker] Docker compose를 이용한 다중 컨테이너 구현](/infra/docker-compose)