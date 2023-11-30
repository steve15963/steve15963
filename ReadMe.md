# 1. 팀 프로젝트

## 1 - 1 사용자 맞춤형 동영상 블러 서비스

- 프로젝트명 : Orialz
- 장소 : **SSAFY** 특화프로젝트 A603팀
- 역할 : **팀장**, 백엔드, 인프라, 프론트
- 목적 : 최근 유튜브나 OTT에 사용자가 싫어하거나 특정 대상이 보면 안되는 영상이 자주 올라오고 있습니다 이러한 **동영상**을 **인공지능**이 분석하여 **사용자 맞춤형 블러** 처리해주는 서비스입니다.

- 기획

  ![블러샘플.gif](/Images/블러샘플.gif)

- 결과

  ![블러계획.gif](/Images/main.gif)

- ERD
    
    ![ERD.png](/Images/erd.png)
    
- 인프라 설계
    
    ![인프라.png](/Images/인프라.png)
    

협업툴 -JIRA, MatterMost, Git, Notion

- 분산처리 샘플 실행 모습
    
    3 노드 CPU실행 테스트 모습
    
    [![Video Label](http://img.youtube.com/vi/j4_h7WlUmzw/0.jpg)](https://youtu.be/j4_h7WlUmzw)
    
    2 노드 CPU + GPU 실행 모습( 나머지 2대는 HDFS 대역폭용 )
    
    [![Video Label](http://img.youtube.com/vi/jTCtRnw_VPM/0.jpg)](https://youtu.be/jTCtRnw_VPM)
    
- 블러 시연 영상

    [![Video Label](http://img.youtube.com/vi/QbmakE5nIag/0.jpg)](https://youtu.be/QbmakE5nIag)
    
- 기술 스택
    - Front
        - Node 18 LTS
        - React
            - 전체화면시 Z-index를 무시하고 동영상이 최상단에 오는 이슈
                - 동영상의 부모 객체를 전체화면하여 Z-index를 살리는 방법.
        - Figma
        - Axios
    - Back
        - Spring Boot 2.7.13
        - Lombok
        - Spring Security
        - FFMPEG
            - 동영상을 처리하기 위한 라이브러리
            - 분산 서비스를 위한 HLS
                
                → TS 파일 요청시 빠르게 인코딩하여 프론트에게 제공
                
                → 성능 이슈 실시간 블러 스트리밍 서비스가 불가능을 위한 방식.
                
                → HTML객체 요소를 통하여 프론트에서 블러 처리하는 방식.
                
            - 분산 처리를 위한 FFMPEG Frame별 이미지 추출
                
                → FFProbe를 통하여 동영상의 프레임을 추출
                
                → 1 / 프레임을 통하여 각 사진의 동영상 시간을 파악
                
        - MYSQL
        - JPA
            - 샘플 동영상 3분에 약 16만 ~ 30만건의 쿼리가 발생
            - 일반적인 JOIN을 통하여 데이터 추출시 약 10초의 딜레이 발생
            - Query Cost를 고려하여 인덱스와 JPQL을 작성하여 3초 이내의 응답 완성.
        - Hadoop
            - 단일 컴퓨터로 4C CPU로 샘플 처리 시간 약 20시간이 소요되었습니다.
            - 해결하기 위하여 Redis등 MQ시스템을 이용한 비동기 처리 생각하였습니다.
                - 비동기 처리에서 작업에 필요한 데이터 Missing으로 인한 작업 지연
                - 모든 서버에 전송하기에는 부족한 네트워크 대역폭 및 스토리지 용량
                - 작업에 대한 추후 디버그를 위한 로그 문제.
                - 이러한 문제를 해결하기 위한 솔루션이 바로 Hadoop
                    - 네트워크 대역폭과 스토리용량 데이터 Missing으로 인한 문제
                        
                        → HDFS로 해결.(약 6Gbps)
                        
                    - 작업에 대한 디버깅으로 인한 로그 문제
                        
                        → Hadoop Job 로그 및 출력 데이터 별도로 보관 가능
                        
                        → 출력 로그를 이용하여 인공지능 출력에 대하여 추후 피드백 가능.
                        
                - AWS, Oracle Cloud, 집 3분할 클러스터링
                    
                    → DR, HA에 대하여 대비가 되었습니다.
                    
            - 하둡으로 처리 결과 2000% 빠른 50분대에 처리 할 수 있었습니다.
                - NLineInputFormater를 이용하여 5줄이 최적의 속도를 낼 수 있었습니다.
            - Split 페이즈 Line별 작업 분할.(5라인)
            - Map 페이즈 key = Time, Value = Image HDFS URL
            - Reduce 페이즈 key= Time, Value = AI Answer JSON
        - Pytorch - Socket Server
            - 처음 Hadoop Job내부에 Process Builder 객체를 이용하여 설계
            - Process Builder 사용시 인공지능 모델 로딩 시간으로 인한 작업 지연 발생
                - 인공지능 모델을 한번만 미리 로드하여 응답 속도 향상
                - 하둡 컨테이너 모든 Resource 노드에서 Pytorch소켓 서버를 운영
            - YOLOv8 학습(팀원)과 MMDetection의 2트랙 개발 진행.
    - Infra
        - EC2 - 2대
        - Oracle OCI - 3대
        - SSAFY 노트북 - 1대
        - Docker
        - Docker-compose
        - Nginx - 접근 도메인에 따른 다른 webRoot
        - Jenkins - CI/CD
        - GitLab
- 넘어온 기술적 한계
    
    Hadoop 공부
    
    FFMPEG의 출력물을 Hadoop에서 처리하는 방법
    
    파일 분할 업로드를 통하여 대용량 파일 업로드 방식 구현.
    
    → 동시 세션 및 버퍼 메모리 부족 문제 해결.
    
    원격에서 Hadoop Job을 제출하는 방법
    
    Docker 오피셜 이미지로 서비스 구현하기
    
    FFMPEG 소스 코드로 설치하기
    
    Jenkins - FFMPEG 빌드 옵션에 따른 세부 옵션 차이로 인하여 다양한 시도.
    
    **CPU 아키텍처** 차이로 인한 컴파일 및 **호환성 이슈**.
    
    동영상 블러 처리 **컴퓨팅 파워 이슈**
    
    컴퓨팅 파워 이슈 해결을 위하여 **hadoop 클러스터링**
    
    클러스터링을 구성하며 hdfs-site.xml, yarn-site.xml등의 **Ip Bind 범위 설정**을 통하여
    **다양한 리존**에 있는 서버를 **하나의 Hadoop 시스템**으로 작동하게 하였습니다.
    
    **WAN을 통한 클러스터링 구축** 으로 인한 **해킹 및 방화벽 설정** 등
    (I Am Doctor Who 라는 해커에게 GetShell 이라는 Job을 처음에 제출 받았습니다.)
    

## 1 - 2 실시간 실내 위치 측위 및 키오스크 서비스

- 프로젝트 명 : Orialz
- 장소 : **SSAFY** 특화프로젝트 A105 팀
- 역할 : 팀장, 백엔드, 임베디드, 인프라
- 주제 : IOT와 Web의 결합
- 목적 : 반려동물의 시장은 성장하고 있으나 이에 대한 **적절한 홍보** 및 **소통 시스템**이 적당한 부분이 없고 **동물의 상태를 체크**가 가능한 형태가 없기에 이를 서비스하고 기획및 개발한 서비스입니다

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6c47ea87-203d-40c8-8d89-668c0bf656d7/Untitled.png)

- [페이지 - 링크](http://i9a105.p.ssafy.io)
- ERD
    
    ![Copy of Pet.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5de92b21-2528-4b48-a101-e26b4afbd293/Copy_of_Pet.png)
    
- 협업툴
    - JIRA - 에자일, 스프린트, 스토리포인트, 이슈 트래킹
        
        ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0f5b23e1-c8a1-4bcd-86bf-6f41bad2f590/Untitled.png)
        
    - MatterMost - JIRA Hook, 팀원 소통
        
        ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/fddf8347-0a66-413c-926d-68e146c2d7c1/a05a5767-a0b8-4594-a133-a036db8d55a4/Untitled.png)
        
        ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c218f29c-78cc-443c-b66a-6aa2e737c82f/Untitled.png)
        
    - Email - JIRA Hook
        
        ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f5ea1349-91df-4e87-81ca-e5727683bd23/Untitled.png)
        
    - Git - 소스 관리, 자동 배포, JIRA연동
        
        ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/df59dd6c-d82f-488d-9284-b16af5a6acb3/Untitled.png)
        
- 기술 스택
    - Front
        - Node 18 LTS
        - React
        - Figma
        - Axios
    - Back
        - Spring Boot 2.7.13
        - Lombok
        - JPA
        - javaxMail
        - HttpRequest - 외부 연동 API (사업자 확인)
        - Spring Filter
        - Spring ExceptionHandler
    - Infra
        - EC2
        - Docker
        - Nginx
        - Jenkins
        - GitLab
- 전략
    - MVC 패턴 적용 - 유지보수 용의
    - JAVA의 상속을 이용한 CreateAt, UpdateAt과 같은 공통 속성 관리
    - 데이터 베이스에서 Soft-Delete 전략
    - SPA으로 CSR방식을 이용하여 API DOC로 백엔드와 프론트가 Rest통신으로 구현
    - 블루투스로 위치 추적 및 체온 측정.
- 시연 영상
    
    https://youtu.be/lnQ2JS36cBA
    
- 넘어온 기술적 한계
    - 메모리
        1. 아두이노 우노 R3의 경우
            
            ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/040483f9-bcad-4b07-b95a-3215a5db6e5e/Untitled.png)
            
            메모리가 약 2KB의 작은 컴퓨터입니다.
            
            JSON빌더, ESP-01, WIFI를 위한 라이브러리가 모두 들어 갈 수 없었습니다.
            
            그렇기에 JSON빌더는 문자열 데이터로써 조립을 하였고..
            
            ESP-01제어, esp01과 HM-10의 AT커맨드에 대한 제어를 수행하였습니다.
            
    - 동시제어 및 버퍼제어
        
        아두이노 우노 R3의 경우는 HW 시리얼 포트가 하나밖에 없습니다.
        HW 시리얼을 디버깅 포트로 고정 하고 사용하고.
        
        SW시리얼 포트를 통하여 BT, WIFI를 순차 동시제어 하였습니다.
        
        문제점 : 
        
        1. 각 모듈을 제어중 문제는 없었습니다.
        2. 두 모듈 제어 코드를 하나의 코드로 합치면 데이터 오염이 발생하였습니다.
        
        해결 :
        
        1. Delay를 통하여 데이터 오염이 사라진 부분을 확인하였습니다.
        2. 고민을 해본 결과 프린터의 스풀링이 기억나게 되었습니다.
        3. 모듈 변경시 버퍼의 데이터 유무를 체크하도록 하여 해결하였습니다.
            
            flush(), close(), available()등
            
        
    - 소켓 전송
        
        위 메모리 문제처럼 라이브러리를 사용할 수 없는 상태였습니다.
        
        그렇기에 저는 Socket통신을 이용하여 TomCat서버에 Http Post 요청을
        
        보내기 위하여 **RFC7230**규약을 준수하면서 Tomcat과 Post 통신하였습니다.
        
    - 3각 측량 - 거리 정확도 25배 향상.
        
        수학적인 문제가 있었습니다.
        
        아직까지도 잘 이해가 되지 않았지만..
        
        선행 연구를 하신 석,박사님과 인터넷 고수님들의 힘을 빌려
        
        행렬이라는 수학과 오타와의 싸움에서 승리하였습니다.
        
    - 칼만 필터
        
        계산된 값으로 3각 측량을 요청한다면 정확한 출력이 되었습니다.
        
        다만 블루투스 비콘의 신호 세기에는 노이즈가 많이 존재하였고..
        
        이를 해결하기 위하여 칼만 필터를 적용하여 아래와 같은 표를 만들어 냈습니다.
        
        ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b5f256c9-68f8-42a0-a96f-0f907667d73b/Untitled.png)
        
        ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/112e6c12-f8c0-4b6f-b661-fac7c4cca4f0/Untitled.png)
        
- 느낀점
    
    잘 하는 것 보다.
    
    팀원 믿을 수 있는 팀원과 있다는 사실이 가장 중요하게 느껴졌습니다.
    
    그리고 팀장의 텐션과 말, 막히는 구간에 대한 조언을 통해서
    
    팀원의 능력이 200% 그 이상으로 이끌어 낼 수 있다는 사실을 알게되었고..
    
    리더로 팀원들 앞에서 이끌고 나가지만..
    
    결정 사항이 필요할 때는 가끔 보스처럼 강제성이 필요하다는 사실도 알게 되었습니다.
    

## 1 - 3여행지 추천 및 관리 서비스

- 프로젝트 명 : 공유해
- 장소 : SSAFY 1학기 관통 프로젝트
- 역할 :  팀장, 백엔드, 프론트
- 주제 : 여행 일정을 관리해주는 웹 어플리케이션
- 목적 : Spring Boot를 이용하여 CSR방식으로 처음 개발해본 서비스입니다.
API Doc, Flow Chart등 다양한 문서를 먼저 작업하고 개발을 해본 첫 팀 프로젝트입니다.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/fddf8347-0a66-413c-926d-68e146c2d7c1/fece1fd8-c785-4b6b-a153-ba024651f971/Untitled.png)

https://youtu.be/xbGlbdtn-FA

- ERD
    
    ![1.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/fddf8347-0a66-413c-926d-68e146c2d7c1/0e0366a3-fc42-44c0-9554-7c446b98f9a2/1.png)
    
- API DOC
    
    ![API DOC.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/fddf8347-0a66-413c-926d-68e146c2d7c1/df0f6ca1-5a48-4518-803d-b72e3f9cb508/API_DOC.png)
    
- 느낀점
    
    Vue Bootrap을 이용하여 개발하며 웹 프론트에 대한 전반적인 지식을 습득할 수 있었습니다.
    
    작은 어플리케이션에도 수 많은 기획과 서류가 필요하다는 사실을 알게 되었습니다.
    

## 1 - 4 여행지와 일정 생성 및 추천 서비스

- 프로젝트 명 : PIJJA
- 장소 : SSAFY 공통프로젝트 A605팀 팀원
- 역할 : 안드로이드 앱, 퍼블리셔, API검증, 등
- 목적 : MBTI의 성향에서 P라는 즉흥적인 성향을 가진 사람에게 J성향처럼 계획을 제시해주는 자동 일정 생성 앱입니다.

 - 시간 상 정리를 못하였습니다 정리중입니다. -

# 2. 개인 프로젝트

---

## 2 - 1 음성 인식 및 답변 장치

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/fddf8347-0a66-413c-926d-68e146c2d7c1/03ff8f44-3d3f-4241-9e21-9461d26c7a45/Untitled.png)

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/fddf8347-0a66-413c-926d-68e146c2d7c1/05259bb9-e0dd-4318-b1ec-3b2b4b19bb45/Untitled.png)

✔️기사

[부산박물관 특별전 <부산, 관문 그리고 사람> (~12/5)](https://blog.naver.com/with_pen/222583868846)

✔️시연

[부산 박물관_ 부산, 관문 그리고 사람.mp4](https://prod-files-secure.s3.us-west-2.amazonaws.com/fddf8347-0a66-413c-926d-68e146c2d7c1/c7bbfb7a-9d0a-4f19-b5e8-ccec2642319a/%EB%B6%80%EC%82%B0_%EB%B0%95%EB%AC%BC%EA%B4%80__%EB%B6%80%EC%82%B0_%EA%B4%80%EB%AC%B8_%EA%B7%B8%EB%A6%AC%EA%B3%A0_%EC%82%AC%EB%9E%8C.mp4)

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/fddf8347-0a66-413c-926d-68e146c2d7c1/b4c15689-397c-4e09-ba52-39d0f535ab09/Untitled.png)

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/fddf8347-0a66-413c-926d-68e146c2d7c1/5feb02ba-9fe1-4c96-9de8-5d8edb9f4434/Untitled.png)

## 2 - 2 부산농업기술지원센터 - 홍보관

![KakaoTalk_20220919_170942955_01.jpg](https://prod-files-secure.s3.us-west-2.amazonaws.com/fddf8347-0a66-413c-926d-68e146c2d7c1/4ab6bc4e-2d8d-4eff-aea4-6ad3e748256c/KakaoTalk_20220919_170942955_01.jpg)

✔️**구현 사항**

- 사용자 웹 프론트 인터페이스 개발
- LTE Router를 이용하여 네트워크 구축
- 디지털 사이니지 개발
- 1차 스위치 베이스 인터페이스 개발
- 2차 NFC 베이스 인터페이스 개발

✔️**담당 역할**

- 인터페이스 펌웨어 개발
- 디지털 사이니지 웹 프론트 개발
- PM

✔️**1차 프로젝트 문제점 발생.**

![ImagePrint.jpg](https://prod-files-secure.s3.us-west-2.amazonaws.com/fddf8347-0a66-413c-926d-68e146c2d7c1/8f84476f-14c3-42b3-a5be-3f45ebb93590/ImagePrint.jpg)

- 기계식 슬라이드 장치 개발자와 협의를 통하여 스위치를 선정하여 개발
- 기계식 슬라이드 장치가 내려오는 장치의 충격을 흡수하지 못하고 고장 및 사용자 부상을 유발
- 플로팅 및 체터링 현상이 발생하여 이를 해결하기위하여 풀업 및 딜레이를 이용하여 해결

✔️**2차 프로젝트 난제.**

- 5mm이상의 두께의 플라스틱(아크릴)을 뚫고 태깅을 인식할 수 있는가?. → 발품으로 해결
- 국내에 충분한 모듈이 있는가? → 직구로 해결
- 모듈이 정확한가? →
    
    제가 요구한건 Reader 장비였지만 온 장비는 조금 더 저렴한 
    
    NFC Tag 시뮬레이터 장비로 다른 장비가 도착하였습니다.
    
    부산에서 수급가능한 NFC 리더기를 찾아 개발을 진행하였습니다.
    
    그러나 해당 NFC 장비의 라이브러리는 라즈베리파이 전용 장비로 라이브러리가 존재하지 않아
    
    라즈베리파이용 라이브러리를 아두이노용으로 개조하여 사용하였습니다.
    

✔️**프로젝트 리뷰**

- 어떠한 프로젝트든 완벽해 보일 수 있지만.  진행하고 나서 조금은 문제가 생길 수 있다는 점을 알게 되었습니다. 어떠한 프로젝트든 담당자의 소통이 중요하다는 사실도 알게되었습니다.

## 2 - 3 (보류중)노약자를 위한 안구마우스

- 목적
    - 아버지의 오래된 친구분이 뇌출혈로 인한 의사소통 어려움을 해결하기 위하여.
- 사용기술
    - Samsung EyeCan
    - ~~Windows Dos Batch~~ → Cron
    - ~~Windows Event~~
    - Java Swing
    - ~~Arduino Uno~~
        - ~~IR Data Send~~ → 라즈베리 GPIO로 대체
    - ldw931 USB LTE Modem
        - LTE Modem Web SMS System reverse engineering
        - PCI-E의 LTE모뎀을 AT커맨드로 가능하지만 USIM슬롯에 대한 비용으로 차선책
        - 라즈베리파이에 mpci-e있는 모델이 없습니다.
            
            ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/be0142b6-dd61-4b79-9eec-905b6f52a1b3/Untitled.png)
            
        
        ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6aa30e0b-8d5b-4d5a-95d5-547457cbf662/Untitled.png)
        
        ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f750357c-d8d6-4388-adad-5fe294781b94/Untitled.png)
        
    - EyeCan 사용 일부
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d1abdc2d-503e-46ae-bf9d-42cc89529872/Untitled.png)
    

프로젝트 임시 중단.

의사 선생님의 권고로 도구를 제공하지 않고 스스로 불편함을 느끼고

해결을 하도록 노력하게 만들어야 한다는 의견이 있었습니다.

— 추가 작성중 입니다.—

## 아버지 위치 추적기

- 장소 : 집
- 역할 :  1인 개발
- 주제 : GPS 데이터를 파싱하여 위치를 추적하는 서비스.
- 목적 : 컴퓨터를 많이 하는 제가 자주 혼나서 아버지 차량에 몰래 설치한 GPS 위치 추적기 입니다.
    
    ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/fddf8347-0a66-413c-926d-68e146c2d7c1/4898307e-2934-4c73-beb8-60644ef0082c/Untitled.png)
