---
layout: post
author: winverse
title:  "solutions architect"
description: "시험에 자주 출제 되는 핵심 내용"
tags: aws solutions-architect
category: aws
is_published: false
---

# 9. RDS, Aurora, ElastiCache
## RDS
  - 읽기 전용 복제본은 Async로 동작하여 사용자들은 최종본만 읽게 된다.
  - 다중 AZ은 동기 복제를 사용한다.
  - 다중 AZ 설정은 DB config 값을 변경하지 않아도 된다.
  - 암호화 되지 않은 인스턴스는 스냅샷을 생성하고 스냅샷에 암호화를 활성 후에 스냅샷을 통해서 복구해야한다.
  - RDS는 최대 5개의 읽기 전용 복제가 가능하다 (전체 6개)
## ElastiCache
  - 서로 다른 EC2 인스턴스 들이 필요 시 사용자들의 상태를 회수 할 수 있게끔 사용됨
## Aurora
  - 최대 15개의 읽기 전용 복제본을 가질 수 있다.(전체 16개)
# 10. Route53
## CNAME vs Alias
옵션 선택에 
### CNAME
  - 비 루트도메인
  - DNS 네임스페이스의 최상위 노드 사용 불가능
### Alias
  - 루트 도메인과 비 루트 도메인 모두 작동함
  - DNS 네임스페이스의 최상위 노드 사용 가능
  - TTL 설정이 불가
  - 레코드는 항상 A/AAAA 레코드로 설정됨
  - ELB, S3에 대해서 별칭을 사용합니다.
  - EC2 DNS 이름에 대해 Alias 레코드를 설정할 수 없습니다.
## Routing Policies
  - simple
    - 단일 리소스 트래픽
    - Health check 기능을 사용할 수 없음
  - Weighted
    - 각 리소스에 전송되는 요청의 비율을 제어합니다.
    - 가중치는 100이 되어야 할 필요는 없습니다.
  - Latency-based
  - Geolocation
    - 지리적 위치 기반으로 latency 기반과 다름
    - 지리 근접 라우팅은 편향을 증가 시켜 한 리전에서 다른 리전으로 트래픽을 보낼 때 유용하다는 것입니다.
  - IP-based Routing
  - Multi-Value(다중 레코드)