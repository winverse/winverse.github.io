---
author: winverse
title:  "[Docker] 3. Docker compose를 이용한 다중 컨테이너 구현"
tags: 
  - docker
categories: 
  - infra
is_published: true
show_date: true
date: "2023-04-12"
description: "Docker-compose를 이용한 다중 컨테이너 애플리케이션을 어떻게 구축하는지 알아봅시다."
---

# 지난 이야기
[지난 시간](/infra/docker-handle-multi-container)에 Docker를 통해서 여러개의 컨테이너 환경을 구축하는 방법을 배웠는데, 이렇게 하다보면 컨테이너를 작동시키기 위함 Command가 너무 복잡해진다는 단점이 있다.

이런 문제를 해결하기 위해서 Docker-compose를 사용하도록 하자.

# Docker-compose란?
Docker compose는 여러 개의 Docker 컨테이너로 구성된 애플리케이션의 서비스를 정의하고 공유하는데 도움이 되는 도구다. Compose를 이용하면 yml 파일에서 서비스를 정의하고, 단일 명령으로 모든 서비스를 생성하고 시작할 수 있다. Compose의 주요 기능으로는 Docker의 확장 된 개념으로 단일 호스트에서 여러 개의 격리된 환경을 가질 수 있다.

# Code
이전 사례에서 front, backend, mongo를 컨테이너로 케이스를 예시로 docker-compose의 항목들을 이해해보자.
```yml
version: "3.8"
services: 
  backend:
    build: ./backend 
    ports:
      - '80:80'
    volumes:
      - logs:/app/logs # Named Volume
      - ./backend:/app # Bind mounts Volume 
      - /app/node_modules # anonymouse Volume
    env_file:
      - ./.env
    depends_on: 
      - mongodb
  frontend:
    build: ./frontend
    ports:
      - '3000:3000'
    volumes:
      - ./frontend/src:/app/src
    stdin_open: true 
    tty: true
    depends_on:
      - backend
  mongodb:
    container_name: "mongo-container"
    image: 'mongo'
    volumes:
      - data:/data/db
    # environment:
      # - MONGO_INITDB_ROOT_USERNAME=max 도 가능하다
      # MONGO_INITDB_ROOT_USERNAME: max
      # MONGO_INITDB_ROOT_PASSWORD: secret
    env_file:
      - ./.env
volumes:
  data: # mongodb 안에서 data라는 volume이 사용되었기 때문에 작성해줘야 함
  logs: 
  # Anonymouse Volume 및 bind Mounts volume은 적을 필요 없음
```

- `version`: docker-compose 파일의 버전을 의미한다.
- `service`: 이 부분에서는 이번에 사용할 3개의 컨테이너 front, back, mongo가 정의 되어 있는 것을 확인 할 수 있다. 여기서는 3개의 서비스가 등록 되었으며 계층 구조로 각 서비스에 대한 설정이 가능하다.
  - `build`:빌드하는데 사용하는 Dockerfile의 경로를 지정한다. 복합적인 상황이라면 `context`와 `dockerfile`키를 이용하면 된다.
  - `ports`: 컨테이너 내부에서 expose하는 포트를 의미한다. 이것을 통해서 컨테이너와 호스트 간의 연결을 설정하는데 사용된다. `호스트 포트:컨테이너 포트`의 형태로 구성 되어 있다.
  - `volumes`: `service`가 사용할 volume들을 의미하는데 여기에 bind Mounts, Anonymouse Volume, Named Volume 모두 가능하지만 일반적으로는 Named Volume을 사용한다.
  - `env_file`: `service`에 대한 환경 파일의 경로를 정의한다.
  - `depends_on`: 백엔드 서버가 의존하는 서비스를 정의한다.
  - `stdin_open`: 표준 입력 사용 여부. docker에서 -i 옵션과 동일하다.
  - `tty`: 컨테이너에 대한 터미널을 제공할지 여부. docker에서 -t 옵션과 동일하다.
- `volumes`: `service`에서 사용한 volume들을 정의해준다. 여기서는 backend에서는 logs가 사용되었고, mongo에서는 data가 사용되었기에 정의한다.
  
# Network 설정은?
docker-compose에서는 network옵션도 제공하고 있다. 그렇지만 docker-compose를 사용하게 되면 세개의 서비스 backend, front, mongo가 이미 동일한 network로 바인딩 되기 때문에 network 옵션을 더 사용할 것이 아니라면 굳이 추가할 필요는 없다.

# 결론
이렇게 docker와 docker-compose에 대해서 알아보는 시간을 가졌는데, 사용해보니 왜 사용해야하는지 잘 알게 되었고 무엇보다 서버 프로그래머가 인프라로 개념을 확장하는데 가장 기본이 되는 길이라고 생각하고 있다.
일반적으로 찾아보면 많은 서적들이 docker -> docker-compose -> Kubernates -> Terraform의 구성을 하고 있는 것은 우연이 아니라고 생각하고 있다. 이번 기회를 통해서 더욱 인프라에 대해서 잘 알게 되는 계기가 되었으면 좋겠다. 아 물론! Terraform을 하려면 적어도 하나의 Cloud 서비스에 대해서 알고 있다고 가정 해야할 거 같다.

# 시리즈
[1. [Docker] 기본 개념 정리 및 자주 사용하는 Command](/infra/docker-basic-command)  
[2. [Docker] 다중 컨테이너 애플리케이션 구축하기](/infra/docker-handle-multi-container)  
[3. [Docker] Docker compose를 이용한 다중 컨테이너 구현](/infra/docker-compose)  