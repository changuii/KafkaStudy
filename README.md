# KafkaStudy


도서 : `실전 카프카 개발부터 운영까지`
- [링크](https://www.yes24.com/Product/Goods/104410708)



## 목차

`chapter 1`: 카프카 개요  
`chapter 3`: 카프카 기본 개념과 구조  
`chapter 4`: 카프카의 내부 동작 원리  
`chapter 5`: 프로듀서 내부 동작 원리와 구현



## CHAPTER 1. 카프카 개요

[1. 카프카의 특징](https://github.com/changuii/KafkaStudy/blob/main/chapter1/KafkaFeature.md)
- 개요

## CHAPTER 3. 카프카 기본 개념과 구조

[3.1 카프카 기초](https://github.com/changuii/KafkaStudy/blob/main/chapter3/KafkaBasic.md)
- 카프카 기본 용어, 리플리케이션, 파티션, 세그먼트 기초 개념

[3.2 카프카 핵심 개념](https://github.com/changuii/KafkaStudy/blob/main/chapter3/KafkaCore.md)
- 카프카가 어떻게 안정적이고 높은 처리량을 갖는지에 대한 구체적인 내용
- 분산 시스템, 페이지 캐시, 배치 전송 처리, 압축 전송, 토픽 구조, 고가용성 보장, 주키퍼 의존성

[3.3 프로듀서와 컨슈머의 기본 동작](https://github.com/changuii/KafkaStudy/blob/main/chapter3/KafKaProCon.md)
- 카프카의 프로듀서와 컨슈머의 기본 동작들에 대한 내용
- 컨슈머 그룹에 대한 간단한 내용


## CHAPTER 4. 카프카의 내부 동작 원리

[4.1 카프카 리플리케이션](https://github.com/changuii/KafkaStudy/blob/main/chapter4/Replication.md)
- 카프카 리플리케이션이 어떻게 동작과정 및 전통적인 메시징 큐와의 차이점
- 카프카가 어떻게 안정성과 속도를 가지면서 리플리케이션을 하는가

[4.2 카프카 컨트롤러](https://github.com/changuii/KafkaStudy/blob/main/chapter4/Controller.md)
- 카프카의 리더가 예기치 않게 또는 제어된 종료에서의 리더 선출과정
- 두 가지 경우에서의 리더 선출과정 및 다운타임의 차이

[4.3 로그 세그먼트](https://github.com/changuii/KafkaStudy/blob/main/chapter4/Log(LogSegment).md)
- 카프카의 로그의 저장 및 로그 저장 파일인 세그먼트 관리 방법


## CHAPTER 5. 프로듀서의 내부 동작 원리와 구현

[5.1 파티셔너 ](https://github.com/changuii/KafkaStudy/blob/main/chapter5/Partitioner.md)
- 카프카에서 파티셔너가 하는 일
- 파티셔너 전략 (라운드 로빈 전략, 스티키 파티셔닝 전략)

[5.2 프로듀서 배치](https://github.com/changuii/KafkaStudy/blob/main/chapter5/Batch.md)
- 프로듀서의 배치 전송 시 고려해야 할 사항

[5.3 중복없는 전송](https://github.com/changuii/KafkaStudy/blob/main/chapter5/TransWithoutDupli.md)
- 프로듀서의 중복없는 전송 과정
- 이전의 메시지 큐에서 사용하는 전략들 

[5.4 정확히 한 번 전송](https://github.com/changuii/KafkaStudy/blob/main/chapter5/ExactlyOnceTransfer.md)
- 정확히 한 번 전송하는 과정
- transaction API, 트랜잭션 과정





