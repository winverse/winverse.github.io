---
layout: post
author: winverse
title:  "인터럽트(interrupt) 그리고 DMA와 고성능 소켓"
description: 이번 글을 통해서 OS에서 사용되는 인터럽트의 종류를 알아보며 고성능을 내기 위한 Socket 통신에 대해서 알아봅시다.
tags: Interrupt DMA Socket
category: os
is_published: true
---

# 외부 인터럽트
1. 전원 이상 인터럽트
  - 파워 이상 같은

2. 기계 착오 인터럽트
  - CPU 기능적 오류

3. 외부 신호 인터럽트 
  - 타이머에 의한 인터럽트 (자원 할당 시간 초과)
  - 키보드 인터럽트 (ctrl + alt + dn)
  - 외부 장치로부터 요청 (I/O interrupt와는 다르다)

4. 입출력 인터럽트(I/O interrupt)
 - 입출력 장치가 데이터 전송을 요구하건 전송이 끝난 다음
 - 입출력 데이터에 이상이 있는 경우

# 내부 인터럽트
 1. 잘못된 명령
 2. 프로그램 검사 
  - Division by zero (컴퓨터 상에서 나눗셈 작동 원리에 의하면 0으로 계속 숫자를 나누게 되니까 무한 loop가 발생하면서 CPU에 손상을 끼치게 됨)
  - overflow/underflow
  - 기타 Exceptions 

## 인터럽트 동작 순서
  1. 인터럽트 요청
  2. 프로그램 실행 중단
  3. 현재 프로그램 상태 백업 (PCB, PC 등) 
  4. IPR (Interrupt Processing Routine) 동작
  5. ISR (Interrupt Service Routine)이 동작한다.
  6. 상태 복구 (PC)
  7. PC를 이용하여 수행중이던 프로그램을 재개한다.

 > 가만 보면 어느 곳에다가 상태를 저장해두고 다른 업무를 처리한뒤에   
 다시 돌아와서 원래 하던 일을 하는 것이 무언가랑 비슷한거 같다는 생각이...

 > 찾아보니 재미있는 이야기가 있어서 적어본다  
 CPU와 메모리 사이에 입출력 관리자(메모리 매니저, 옛날엔 브릿지라고도 불림)가 존재하였는데, 이 칩의 역할이 CPU와 메모리 사이를 연결 해주다보니 입출력 관리자의 성능과 컴퓨터의 성능에는 아주 밀접한 연관이 있었고, 입출력 관리자의 성능은 메인보드의 가격과 비쌌다고 한다. 예를 들어서 내가 CPU가 성능이 좋은 것을 구매하고 메인보드를 저렴한 것을 구매하면 입출력 관리자의 성능이 부족하여 CPU 성능이 전부 발현되기 어려운 일이 발생한 것이다! 한편, 저렴한 메인보드가 지속적으로 출시되어 자신들의 CPU가 제대로 평가 받지 못하는 일이 생기자 인텔은 입출력 관리자에게 CPU와 메모리의 연결을 맡기지 않고 CPU가 직접 담당 하도록 하였는데 오늘날 메인보드의 성능이 꽤나 높아 졌음에도 불구하고 여전히 인텔 CPU 안에는 메모리를 직접 접근하는 기능들이 부분적으로 남아 있다고 한다. (ex intel Z690, 12gen)

 > printf가 모니터에 출력되기까지
 항상 그런 것은 아니겠지만 프로그램에 의해서 혹은 프로세스에 의해서 IO가 발생되고 모니터에 입출력이 되는 과정을 생각해보면 , 1. 내가 작성한 코드가 PCB에 의해 실행되고 2. printf같은 함수가 커널의 인터페이스(kernel interface api)를 통해 System Call(User layer와 kernel layer를 가로지르는)을 하여 데이터를 저장 해달라고 요청한다. (interrupt 발생) 3.kernel api를 통해 write 함수가 실행되면 전달 받은 데이터를 메모리에 저장한다. 4. 메모리에 저장 된 값을 H/W에서 (주변기기) 읽어서 5. 모니터로 출력하고 난 뒤 6. 결과를 다시 메모리에 알려주는 (interrupt 발생) 이런 과정인데 다른 I/O로 인하여(게임과 같은) 한번에 너무 많은 과정에 들어가게 되면 성능에 이슈가 있을 수 있다. (단 주변기기가 좋은거라면 Driver를 이용한 비동기 처리를 한다고 한다.)

 > Direct X 이야기  
 위에 이야기를에 이어서 window XP에는 kernel에 GDI라 불리는 그래픽 처리 요소가 들어있었는데 일반 그래픽 처리에서는 문제가 되지 않았지만 게임과 같이 고연산이면서 많은 데이터를 다뤄야하는 그런 일에서는 이 GDI를 통하는 것이 안그래도 복잡한 모니터 출력 과정을 더욱 복잡하게 만들었다. 그래서 GDI를 통하지 않고 바로 kernel의 Video 드라이버까지 "직접" 호출하도록 다시 말해서 User Interface에서 System call을 직접할 수 있도록 해준 것이만들어졌는데 이름이 DirectX ("직접")라고 한다.

 > 다시 본론으로 
 과정이 많다고해서 무조건 느려지는 것이 아니라 과정이 길어지면서 "인터럽트" 횟수가 늘어나는 것이 I/O 성능에 이슈가 된다. 그렇기에 DirectX라는 것은 인터럽트 횟수를 줄임으로서 I/O 성능을 올려 최적화한 방법이 되었다.

# DMA와 고성능 소켓구현  
## 송신측 입장  
컴퓨터는 3층집이지만 네트워크로 계층을 나누면 4층집이 된다(Application Layer, TCP Layer, IP Layer, Network Interface Layer). 먼저 송신측을 보자, 컴퓨터라는 3층집이 User layer가 Kernel layer로 넘어가기 위해서 System call이라는 것을 사용했듯이 네트워크도 Application Layer에서 TCP layer로 가기 위해서 Kernal의 구성 요소를 추상화한 파일을 호출해야하는데. 이때 이 파일을 Socket이라고 한다. 이 Socket은 기본적으로 I/O Buffer를 가지고 있다가 데이터를 받으면 Application Layer든 TCP Layer든 전달한다. 자, 그럼 다시 생각해보면 Application Layer에서 프로그램이 동작하고 난 뒤에 "데이터를 담을 메모리 공간을 확보하고" Socket을 호출하여 send(System call에서는 write지만 Socket에서는 send)
하게 되면 socket은 Buffer에 데이터를 담고 난 뒤에 TCP/IP Layer에 데이터를 전달하고 TCP/IP Layer에서는 받은 데이터를 Segment/Packet을 통해서 데이터를 또 보관하여 NIC에 전달하게 되고 마지막으로 NIC가 frame을 만들고 난 뒤에 목표 지점으로 데이터를 보내게 된다. 

## 수신측 입장
수신측은 마찬가지로 네트워크 계층이 4계층이다. 다만 DMA를 지원하는 NIC를 가지고 있다고 가정한다.
수신측은 데이터를 요청하는 즉시 Application Layer의 "데이터를 받을 공간을 확보하고 이 공간을 (window 기준) IOCP(I/O completion port)가 확보한다" 이후 메모리 공간을 그대로 가지고 있다가 NIC가 데이터를 받게 되면 TCP/IP, Socket을 거치지 않고 바로 NIC가 직접 IOCP가 확보한 공간에 데이터를 집어 넣게 되면 고성능 Socket을 구현할 수 있다.

## 더 깊게 알아보기
그렇다면 IOCP가 일어나는 과정에서 Network에서의 중간 처리 과정은 어떻게 되는 것일까?  
좀 찾아봤는데.. 아직 잘 모르겠다.. 그래도 몇가지 힌트를 얻었는데
일단 NIC에서 원래 처리하던 것들 예를 들어 체크 섬을 같은 것은(체크섬은 NIC에서 처리 - [TCP/IP 네트워크 스택 이해하기](https://d2.naver.com/helloworld/47667) 참고) DMA를 사용한다고 해도 다를 것이 없을테니 원래대로 할것이고, 인도 공과대학교 [강의](https://youtu.be/MpjlWt7fvrw?t=1020)를 보면 OSI 7 layer 기준 L2-L4 packet 처리 과정들이 user spcae에 존재하는 걸로 보이는데 자세한건 좀더 책을 찾아봐야겠다.


# 참고
- [DirectX](https://namu.wiki/w/DirectX#s-2.2)
- [Direct Memory Access](https://ko.wikipedia.org/wiki/%EC%A7%81%EC%A0%91_%EB%A9%94%EB%AA%A8%EB%A6%AC_%EC%A0%91%EA%B7%BC)
- [IOCP](https://jungwoong.tistory.com/43)
- [TCP/IP 네트워크 스택 이해하기](https://d2.naver.com/helloworld/47667)
- [Kernel-bypass techniques for high-speed network packet processing](https://youtu.be/MpjlWt7fvrw)