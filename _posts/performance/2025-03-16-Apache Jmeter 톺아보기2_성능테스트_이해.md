---
layout: single                    
title:  "Apache Jmeter 톺아보기 - 2. 성능테스트 이해"   
categories: Performance          
tag: [Backend]
related_label: " "   #footer
excerpt: "성능 테스트 개념, 유형, 도구, 수행 절차 이해" 
---
<hr>



### **⬛︎ 목차**
<div style="margin-bottom:20px;"></div>
<span style="display: block; margin-bottom: 10px;">&ensp;&ensp;<b>[1️⃣ 성능 테스트란?
](#성능_테스트란) </b></span>
<span style="display: block; margin-bottom: 10px;">&ensp;&ensp;<b>[2️⃣ 성능 테스트 주요 유형](#성능_테스트_주요_유형)</b></span>
<span style="display: block; margin-bottom: 10px;">&ensp;&ensp;<b>[3️⃣ 성능 테스트 도구](#성능_테스트_도구)</b></span>
<span style="display: block; margin-bottom: 10px;">&ensp;&ensp;<b>[4️⃣ 성능 테스트 필수 개념](#성능_테스트_필수_개념)</b></span>
<span style="display: block; margin-bottom: 10px;">&ensp;&ensp;<b>[5️⃣ 성능 테스트 수행 절차](#성능_테스트_수행_절차)</b></span>
<span style="display: block; margin-bottom: 10px;">&ensp;&ensp;<b>[📂 마무리](#마무리)</b></span>


<div style="margin-bottom:100px;"></div>



이 블로그에서는 **웹 성능 테스트**에 집중하여, 핵심 개념과 방법을 알기 쉽게 정리했습니다.
웹 서비스의 성능 테스트가 궁금하신 분들이 **빠르게 이해할 수 있도록** 간결하게 내용을 구성했습니다.


<div style="margin-bottom:100px;"></div>



## 1️⃣성능 테스트란? {#성능_테스트란}
<div style="margin-bottom:20px;"></div>

웹 서비스가 특정 부하 조건에서 **얼마나 안정적으로 작동하는지** 확인하는 과정입니다.  
 
단순히 **응답 속도만을 측정하는 것**이 아니라, 다음과 같은 요소들을 포함해 **서비스의 전반적인 성능을 평가**하는 것이 핵심입니다.

<div style="margin-bottom:20px;"></div>


| 항목 | 설명 |
| --- | --- |
| 확장성 (Scalability) | 사용자 증가에 따라 시스템이 원활하게 확장될 수 있는지 확인하는 과정 |
| 안정성 (Stability) | 장시간 운영 시에도 오류 없이 정상 작동하는지 검증하는 과정 |
| 병목 현상 분석 (Bottleneck Analysis) | CPU, 메모리, 네트워크 등에서 성능 저하를 유발하는 요소가 있는지 식별하는 과정 |
| 최적화 가능성 (Optimization Potential) | 코드, 쿼리, 인프라 등을 개선하여 더 나은 성능을 구현할 수 있는지 평가하는 과정 |


<div style="margin-bottom:100px;"></div>



## 2️⃣ 성능 테스트의 주요 유형 {#성능_테스트_주요_유형}
<div style="margin-bottom:20px;"></div>


각 **테스트 유형마다 목적과 방법이 다르므로**, 아래 표를 참고하여 **적절한 테스트 전략을 선택하는 것이 중요**합니다.


<div style="margin-bottom:20px;"></div>


| **테스트 유형** | **테스트 목적 및 방법** | **사용 도구** |
| --- | --- | --- |
| 부하 테스트 <br> (Load Testing) | - **예상 부하**에서 시스템이 **정상 작동**하는지 **확인** <br> - 동시 사용자 수를 점진적으로 증가시키며 테스트 수행 | JMeter, nGrinder, LoadRunner, Gatling, k6 |
| 스트레스 테스트 <br> (Stress Testing) | - **최대 임계점까지 부하**를 증가시켜 시스템의 최대 용량 한계 테스트 <br> - 예상 사용자 수보다 훨씬 많은 부하를 적용하여 서버 안정성 테스트 | JMeter, nGrinder, LoadRunner, NeoLoad, Tsung |
| 내구성 테스트 <br> (Endurance Test) | - **장시간 부하**를 가하여 시스템의 장기적 안정성 테스트 <br> - 수 시간~ 수 일 동안 지속적인 부하를 가하며 서버 자원 모니터링 | JMeter, nGrinder, k6, Gatling |
| 스파이크 테스트 <br> (Spike Testing) | - **짧은 시간 내 급격한 트래픽 증가에 대한 대응력** 테스트 <br> - 사용자 수를 급격히 증가시킨 후 정상 상태로 복귀 | JMeter, nGrinder LoadRunner, k6 |
| 용량 테스트 <br> (Capacity Testing) | - **시스템이 최대 몇 명의 동시 사용자를 처리**할 수 있는지 평가 <br> - 일정 간격으로 동시 사용자 수를 증가시키며 최대 임계점 찾기 | JMeter, nGrinder, Gatling, k6 |
| 구성 테스트 <br> (Configuration Testing) | - **하드웨어, 네트워크, 소프트웨어 구성별 성능 차이 분석** <br> - CPU, 메모리, 로드 밸런싱 방식 변경 후 성능 비교 | JMeter, nGrinder, LoadRunner, Tsung |


<div style="margin-bottom:100px;"></div>


## 3️⃣ 성능 테스트 도구 {#성능_테스트_도구}
<div style="margin-bottom:20px;"></div>


테스트 도구는 다양한 방식으로 활용할 수 있으며, 테스트 목적에 따라 적절한 도구를 선택하는 것이 중요합니다.


<div style="margin-bottom:20px;"></div>


아래는 **테스트 목적별 추천 도구 및 특징**을 정리한 표입니다.

| 목적 | 도구 | 특징 |
| --- | --- | --- |
| 부하 테스트 & 스트레스 테스트 | JMeter | 가장 널리 사용되는 오픈소스 도구, <br> HTTP, FTP, JDBC 등 다양한 프로토콜과 GUI 지원 |
|  | Gatling | 개발 친화적이며, 코드 기반 테스트 시나리오 작성 가능 |
|  | k6 | JavaScript 기반으로 작성, 클라우드 환경에서 강력한 성능 제공 |
|  | nGrinder |  JVM 기반 분산 부하 테스트 지원 |
| 실제 사용자 기반 테스트 | Selenium | UI 성능 테스트 및 자동화에 최적화 |
| 분산 테스트 | Locust | Python 기반으로, 동시 요청을 쉽게 확장 가능 |
| 네트워크 부하 테스트 | Tsung | Erlang 기반으로, 대용량 트래픽 처리에 적합 |


<div style="margin-bottom:40px;"></div>

<p style="margin-bottom: 5px;">✅ 추천 </p>    
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;● API 부하 테스트 → `JMeter` 또는 `k6`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;● 실제 브라우저 기반 성능 테스트 → `Selenium`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;● 대량 트래픽 분산 테스트 → `Locust` 또는 `Tsung`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;● 엔터프라이즈 환경에서 안정적인 부하 테스트 → `nGrinder`  


<div style="margin-bottom:100px;"></div>


## 4️⃣ 성능 테스트 필수 개념  {#성능_테스트_필수_개념}

<div style="margin-bottom:20px;"></div>


성능 테스트를 효과적으로 수행하기 위해서는 몇 가지 핵심 개념을 이해하는 것이 중요합니다.  

<div style="margin-bottom:40px;"></div>


4-1) 처리량 (Throughput / request per second)  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;● TPS (Transaction Per Second) 라는 단위로 표현되며,    
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;● 서버가 **초당 처리할 수 있는 요청 수**를 의미합니다.    

<div style="margin-bottom:40px;"></div>


4-2) 응답 시간 (Response Time)

<img src="../../assets/images/performance/2025-03-16-Apache Jmeter 톺아보기2_성능테스트_이해/image1.png" alt="" width="80%" height="80%">    

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;● 클라이언트가 서버에 요청 후 **응답을 받을때까지의 걸린 시간**을 의미합니다.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;● 서버의 응답시간은 **대기 시간 + 처리 시간**으로 구성됩니다.    

<div style="margin-bottom:20px;"></div>
    
> **대기 시간**(Latency Time) : <u>클라이언트와 서버간에 데이터를 주고 받는데 걸리는 시간</u> (==서버에 요청을 보낸 후 데이터를 받기 시작할때까지의 시간)  
> **처리 시간** (Processing Time) : <u>실제 서버가 요청을 처리하는데 걸린 시간</u>



<div style="margin-bottom:100px;"></div>




## 5️⃣ 성능 테스트 수행 절차  {#성능_테스트_수행_절차}

<div style="margin-bottom:20px;"></div>


| 단계 | 설명 |  
| --- | --- |  
| ① 테스트 목표 설정 | - **어떤 성능 지표**를 측정할 것인지 설정 <br> - ex1) "로그인 API는 1초 이내에 응답해야 한다.” <br> - ex2) "최대 10,000명의 동시 접속을 처리할 수 있어야 한다.” |
| ② 테스트 환경 구성 | - 실제 운영 환경과 **유사한 테스트 환경 구성 <br> -** ex) 서버 리소스 (CPU, 메모리, 네트워크) 확인 |
| ③ 테스트 스크립트 작성 | - 동시 사용자 수 설정 및 **요청 시나리오 작성** |
| ④ 테스트 실행 및 모니터링 | - **부하 테스트, 스트레스 테스트 등 실행** <br> - 서버 리소스 상태 모니터링 |
| ⑤ 결과 분석 및 최적화 | - **결과 데이터 분석** 후 **성능 개선 조치**  |


<div style="margin-bottom:100px;"></div>


## **📂 마무리**  {#마무리}

<div style="margin-bottom:20px;"></div>


성능 테스트는 단순히 속도를 측정하는 것이 아니라, **서비스의 안정성과 확장성을 보장하기 위한 필수 과정**입니다.  

이 글에서는 **성능 테스트의 주요 유형, 필수 개념, 도구, 수행 절차**까지 살펴보았습니다.   
테스트를 수행할 때는 단순히 한 가지 방법에 의존하기보다, **목적에 맞는 테스트 전략과 적절한 도구를 조합하여 활용하는 것이 중요**합니다.  

<div style="margin-bottom:10px;"></div>


## **📃 참고**
<div style="margin-bottom:20px;"></div>

1. 성능 테스트의 지표와 종류
    - [https://velog.io/@hgo641/성능-테스트의-지표와-종류](https://velog.io/@hgo641/%EC%84%B1%EB%8A%A5-%ED%85%8C%EC%8A%A4%ED%8A%B8%EC%9D%98-%EC%A7%80%ED%91%9C%EC%99%80-%EC%A2%85%EB%A5%98)
2. 성능 테스트 도구 비교
    - [https://baeji-develop.tistory.com/118](https://baeji-develop.tistory.com/118)
