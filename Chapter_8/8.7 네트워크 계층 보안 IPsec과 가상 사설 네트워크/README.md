# 8.7 네트워크 계층 보안: IPsec과 가상 사설 네트워크

IPsec이라고 알려진 IP 보안 프로토콜은 네트워크 계층의 보안을 제공한다.

<br/>

### IPsec 특징

- 호스트나 라우터 같은 네트워크 계층의 어떤 두 개체 사이에서 IP 데이터그램을 보호한다.
    - **데이터그램의 페이로드 부분을 암호화**하여 TCP 세그먼트 등이 암호화된다.
- 위의 기밀성을 제공하는 것처럼 출발지 인증, 데이터 무결성, 재생 공격 방지 같은 보안 서비스를 제공할 수 있다.

<br/>

## 8.7.1 IPsec과 가상 사설 네트워크

흩어져있는 기관들은 그들의 호스트와 서버가 기밀성을 유지하며 안전하게 서로에게 데이터를 전송할 수 있도록  종종 자신만의 IP 네트워크를 갖기를 원한다.

이를 위해 공공 인터넷과 완전히 분리된 라우터, 링크, DNS 시스템을 포함하는 물리적으로 독립된 네트워크를 실제로 설치할 수 있고, 이러한 네트워크를 `사설 네트워크(private network)`라고 한다.

<br/>

큰 유지 비용이 드는 사설 네트워크 대신 오늘날에는 기존 공공 인터넷 상에 `VPN`을 설치한다.

`VPN` 을 이용하면 기관의 사무실간 트래픽은 공공 인터넷을 통해 전송된다.

그러나 기밀성을 제공하기 위해 이 트래픽들은 공공 인터넷에 들어가기 전에 암호화된다.

<br/>

<p align="center"><img width="500" src="https://user-images.githubusercontent.com/76640167/217747985-e18a7a43-82de-41fe-831c-cb5fee6a16d1.png" alt="VPN"></p>

공공 인터넷을 통과하지 않을 때는 평범한 IPv4 데이터그램이 사용되고, 통과해야할 때는 IPsec을 지원하는 라우터가 IPv4 데이터그램을 IPsec 데이터그램으로 바꾼 후 인터넷으로 전송한다.

IPsec 데이터그램은 전형적인 IPv4 헤더를 가지고 있어 공공 인터넷의 라우터는 IPv4 데이터그램과 똑같이 이를 전달한다.

실질적으로는 IPsec 데이터그램의 페이로드는 IPsec헤더를 포함하고, 이는 IPsec 처리를 위해 사용된다.

<br/>

## 8.7.2 AH와 ESP 프로토콜

출발지 IPsec 개체가 보안 데이터그램을 목적지 개체에 보낼 때 AH 프로토콜이나 ESP 프로토콜을 사용한다.

- AH프로토콜
    - 출발지 인증과 데이터 무결성을 제공하지만, 기밀성을 제공하지 않는다.
- ESP 프로토콜
    - 출발지 인증과 데이터 무결성, 기밀성을 제공한다.
    - 대부분 기밀성을 필요로 하여 널리 사용된다.

<br/>

## 8.7.3 SA

IPsec 데이터그램을 전송하기전 출발지 개체와 목적지 개체는 네트워크 계층에서 논리적 연결을 설립하는데 이것이 바로 `SA(security association)`이다.

SA는 단방향이어서 서로에게 데이터를 보내기 위해서는 두개의 SA가 필요하다.

기관 내에 n개의 호스트가 있다면, SA는 2n개 그리고 지점의 라우터가 m개가 있다면 추가로 2m개 즉, `2n + 2m`개 필요하다.

<br/>

게이트웨이 라우터나 랩톱이 인터넷으로 보내는 모든 트래픽이 IPsec 방식으로 보호되지는 않는다.

예를 들어, 보안이 필요한 기관 내의 호스트가 아닌 구글 등의 공공 인터넷의 웹서버에 접속할 수도 있기 때문에 라우터나 랩톱은 IPv4 데이터그램과 IPsec 모두를 전송한다.

<br/>

<p align="center"><img width="500" src="https://user-images.githubusercontent.com/76640167/217747947-c3029c7f-3030-4b01-87e5-5cafe95c7f89.png" alt="SA"></p>

R1은 SA에 대해 다음과 같은 상태 정보를 포함한다.

- SA에 대한 32 비트 식별자 SPI
- SA 시작점의 인터페이스(200.168.1.100)와 최종점의 인터페이스(193.68.2.23)
- 사용되는 암호화 타입 (AES, DES 등)
- 암호화 키
- 무결성 검사 타입 (MD5, SHA 등)
- 인증키

이들을 사용해 암호화하고, 인증을 하며 R2도 마찬가지로 같은 상태정보를 유지한다.

<br/>

SA가 많을 수 있어 그 상태정보를 유지해야하는데 IPsec 개체는 모든 SA에 대한 상태 정보를 그 개체의 OS 커널에 있는 `SAD(security association Database)`라는 데이터 구조에 저장한다.

<br/>

## 8.7.4 IPsec 데이터그램

<p align="center"><img width="500" src="https://user-images.githubusercontent.com/76640167/217747916-15b647c6-8c1f-4072-ab32-268dd279653b.png" alt="IPsec 데이터그램 포맷"></p>

라우터 R1은 원본 IPv4 데이터그램을 IPsec 데이터그램으로 변환하기 위해 다음 과정을 이용한다.

1. 원래 IPv4 데이터그램의 뒤에 ESP 트레일러를 덧붙인다.
    - 원래 IPv4 데이터그램에 원래 목적지 IP 주소, 출발지 IP 주소가 들어가 있다.
    - **ESP 트레일러**는 패딩, 패딩 길이, 다음 헤더라는 3개의 필드로 이루어져 있다.
        - 패딩 : 원래 데이터그램에 덧붙어 최종 메시지가 정수개의 고정 길이 블록으로 분할될 수 있게 한다.
        - 패딩 길이 : 패딩 비트가 얼마나 삽입되었는지 알려주고, 이를 이용해 패딩을 삭제한다.
        - 다음 : 페이로드에 포함된 데이터의 타입(ex: UDP)을 지시한다.
2. SA에 의해 지정된 알고리즘과 키를 이용하여 위 결과를 암호화한다.
3. 암호화된 결과의 앞에 ESP 헤더 필드를 덧붙인다. 결과로 나온 패키지를 엔칠라다라고 부르자.
    - ESP 헤더에는 SPI와 순서번호를 포함한다.
        - SPI : 수신 개체에게 데이터그램이 어느 SA에 속해 있는지 지시한다. 이 SPI를 가지고 자신의 SAD를 검색하여 알맞은 인증 및 복호화 알고리즘과 키를 결정한다.
        - 순서 번호 : 재생 공격을 막기 위해 새용된다.
4. SA가 지정한 알고리즘과 키를 이용하여 전체 엔칠라다에 대한 인증 MAC을 생성한다.
5. 엔칠라다 뒤에 MAC을 붙여 페이로드를 만든다.
    - 비밀 MAC 키를 엔칠라다에 붙이고 그 결과에 대해 고정 길이의 해시를 계산한다.
6. 전형적인 IPv4 헤더 필드를 가지고 완전히 새로운 IP 헤더를 만들어 위의 페이로드 앞에 붙인다.
    - 새로운 IP 헤더에는 출발지와 목적지 주소는 전달될 라우터 인터페이스의 주소가 들어간다.
    - 헤더 필드의 상위 프로토콜 값으로는 TCP,UDP 등을 나타내지 않고 이것이 ESP 프로토콜을 사용하는 IPsec임을 나타내는 값을 사용한다.

결과로 나온 IPsec 데이터그램은 전형적인 IPv4 헤더를 가지고 페이로드가 따라오는 진짜 IPv4 데이터그램이다.

전달하는 라우터들은 IPsec으로 암호화된 정보라는 것을 모른체 목적지 라우터 인터페이스로 이를 전달한다.

<br/>

라우터 R2는 IPsec 데이터 그램을 받으면 다음과 같은 과정을 수행한다.

1. IP 헤더를 보고 IPsec ESP 프로토콜을 적용해야함을 알게된다.
2. 엔칠라다를 들여다보고 R2는 SPI를 이용하여 데이터그램이 어느 SA에 속한것인지 알아낸다.
3. 엔칠라다의 MAC을 계산하여 ESP MAC 필드의 값과 일치하는지 확인한다.
    - 일치하여야 조작되지 않은 것이다.
4. 데이터그램이 재생된 것이 아닌지 순서 번호 필드를 검사한다.
5. SA와 관련된 복호화 알고리즘과 키를 이용하여 암호화된 부분을 복호화한다.
6. 원래의 IP 데이터그램을 최종 목적지로 전송하기 위해 지점 네트워크로 전달한다.

IPsec 개체는 SAD와 함께 `SPD(security policy database)`라 불리는 자료구조를 유지한다.

SPD는 어떤 형태의 데이터그램(출발지 IP 주소, 목적지 IP 주소, 프로토콜 타입으로 결정)이 IPsec으로 처리되어야 하는지와 그때 사용될 SA를 지시한다.

즉, SPD의 정보는 도착한 데이터그램으로 무엇을 할지 알려주고, SAD의 정보는 어떻게 할 것인지를 알려준다.

<br/>

### 8.7.5 IKE : IPsec에서의 키 관리

수백 수천 개의 IPsec 라우터와 호스트로 이루어진 큰 VPN에서는 종단점의 SAD에 직접 SA 정보를 입력 하는 방법은 불가능하다.

즉, SA를 생성하는 자동화된 방법이 필요한데 이 방법으로 `IKE 프로토콜`을 이용한다.

<br/>

각 IPsec 개체는 그 개체의 공개키가 포함된 인증서를 갖는다.

SSL 에서와 마찬가지로 IKE 프로토콜은 두 개체가 인증서를 교환하고 인증과 암호화 알고리즘에 대한 협상을 하게 하며 IPsec SA의 세션키를 생성하기 위한 중요한 자료를 안전하게 교환할 수 있게 한다.

IKE는 이러한 작업을 수행하기 위해 두단계를 거친다.

<br/>

### 첫번째 단계

R1과 R2는 메시지 쌍을 두 번 교환한다.

- 첫 번째 메시지 교환시에는 양측이 라우터 사이의 IKE SA를 설립하기 위해 디피-헬만 알고리즘을 사용한다.
    - IKE SA는 IPsec SA와는 완전히 다르다.
    - IKE SA는 두 라우터 사이에 인증되고 암호화된 채널을 제공한다.
    - 이 첫 번째 메시지를 교환하는 동안 IKE SA의 암호화와 인증을 위한 키가 설립된다.
    - 또한, 두번째 단계에서 IPsec SA 키를 계산하기 위해 사용될 주 비밀키도 만들어진다.
    - 첫번째에서는 누구도 비밀키로 서명함으로써 자신의 신분을 드러내지 않는다.
- 두번째 교환에서는 양측이 메시지에 서명하여 서로에게 신분을 드러낸다.
    - 그러나 메시지가 보안 처리된 IKE SA 채널을 통해 전송되므로 단순한 도청자에게는 신분이 누설되지 않는다.
    - 양측이 IPsec SA에 의해 사용될 IPsec 암호화 및 인증 알고리즘에 대해 협의한다.

<br/>

### 두번째 단계

양측이 각 방향으로 하나씩 SA를 설립한다.

이 단계를 마칠 때는 이 2개의 SA에 대한 암호화 및 인증 세션키가 양측에 만들어져있다.

이후 보안 처리된 데이터그램을 만들기 위해 SA를 사용할 수 있다.
