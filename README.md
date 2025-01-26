회사 업무에 필요한 Snort Rule 스노트 탐지 룰에 대해 공부한 내용을 정리하였습니다.
```
Snort 는 1998년 Martin Roesch에 의해 개발된 오픈 소스 기반 IDPS ( Intrusion Detection Prevention System ) 입니다.
시그니처 기반 탐지 시스템으로서 패턴과 매칭이 될 경우 탐지되는 방식을 사용하는 시스템입니다.
현재까지도 침입 탐지 시스템 ( IDS ) 중 가장 널리 사용되고 있는 시스템입니다.
```
## Snort 의 기능
Snort 는 아래 작성된 3가지 기능을 수행합니다.    

- Sniffer : 네트워크 트래픽을 캡쳐하는 기능
- Packet Logger : 나중에 분석할 수 있도록 네트워크 트래픽을 파일에 기록하는 기능
- IDS / IPS : 네트워크 트래픽을 분석 후 침입 탐지 및 차단 기능

## Snort 동작 구조

Snort 의 동작 구조는 Sniffer -> Preprocessor -> Detection Engine -> Alert / Log 로 이루어져 있습니다.

- Sniffer : Snort IDS 를 통과한 패킷을 수집합니다.
- Preprocessor : 효율적인 공격 탐지를 위해 Plug-in을 먼저 거쳐 매칭을 확입합니다.
- Detection Engine : Snort Rule과 매칭이 되는지 확인합니다.
- Alert / Log : Snort Rule 매칭 결과에 따라 경고나 로그등 출력을 하게됩니다.

Snort Rule 은 크게 Rule header 와 Rule Option 으로 구분됩니다.     
ex ) alert tcp any any -> any 80 (msg:"SQL injection";content:"'1'='1";nocase;sid:1000001;)

#### Rule Header  
먼저 Rule header에 해당하는 내용을 먼저 살펴보도록 하겠습니다.

- Action : 패턴이 매칭이 되었을 경우 할 행동을 정의
- alert
로그를 남기고 경고를 발생시킵니다.

- log
해당 패킷을 기록합니다.

- pass
해당 패킷을 무시합니다.

- active
alert 를 발생 시킨 후 대응하는 Dynamic 을 유효화합니다.

- reject
패킷을 차단하고 기록 후 TCP Protocol 의 경우 TCP ResetUDP Protocol 의 경우 ICMP Port Unrechable 메세지를 전송합니다.

- drop
상대방에게 응답하지 않고 패킷을 차단하고 기록합니다

- sdrop
기록을 남기지 않고 패킷을 차단합니다.

Protocol : 탐지 할 프로토콜의 종류를 정의합니다. ( 규칙을 작성할 때 소문자로 작성하여야 합니다. )

- TCP
TCP 프로토콜을 탐지합니다.


- UDP
UDP 프로토콜을 탐지합니다.


- ICMP
ICMP 프로토콜을 탐지합니다.


- IP
IP 프로토콜을 탐지합니다.


- Any
모든 프로토콜을 탐지합니다.

#### Direction : 탐지할 방향을 정의

- ->
출발지 -> 목적지로 가는 패킷을 탐지합니다.

- <>
출발지 ~ 목적지 사이 모든 패킷을 탐지합니다.

- Source IP & Port / Destination IP & Port : 출발지, 목적지 IP , Port 정의

- ! ( 부정연산자 )
부정연산자 ( ! ) 를 IP 또는 Port 앞에 기입 할 경우 해당 IP 또는 Port 번호를 제외한 주소만 매칭합니다.

- Any
모든 IP 또는 Port 를 의미합니다.

- 포트번호
지정된 포트번호를 의미합니다.

- 포트번호1:포트번호2
포트번호1 ~ 포트번호2 를 의미합니다

- :포트번호
지정된 포트번호 이하 모든 포트를 의미합니다.

- 포트번호:
지정된 포트번호 이상 모든 포트를 의미합니다.

※ 여러 IP 를 작성할 경우 쉼표 ( , ) 로 IP 주소 목록을 구분하고 구분된 IP 주소 목록을 대괄호를 이용해 표시합니다.     
ex ) alert tcp ![111.111.111.0/24,192.168.1.0/24] any -> [111.111.111.0/24,192.168.1.0/24] 123           
위 예시를 해석 할 경우 111.111.111.0/24 또는 192.168.1.0/24 를 제외하고 tcp 프로토콜을 사용해 목적지 123번       
포트로 패킷이 들어올 경우 경고를 발생이라는 정책이 됩니다.    
 
#### Rule Option
Rule Header 가 1차적인 정책을 설정한 것이라면 Rule Option이 실제로 악의적인 내용의 패킷을 탐지하는 영역입니다.     
다르게 말하면 Rule Option은 탐지의 정확도를 향상시켜주는 옵션입니다.        
Rule Option을 사용할 경우 모든 옵션은 세미콜론 ( ; ) 을 사용해 서로를 구분하고, 규칙 옵션 키워드는 콜론 ( : ) 을 사용해 인수와 구분합니다.      
Rule Option은 크게 일반 옵션, Payload, HTTP 옵션, 흐름 옵션으로 구분됩니다.      

#### 일반 옵션 : 규칙에 대한 정보를 제공하는 옵션으로 매칭하는 동안 어떠한 영향도 미치지 않는 옵션   

- msg
Snort 규칙이 탐지될 경우 출력되는 메시지 
msg:“메시지 내용”;


- sid
Snort 규칙을 구별하는 식별자0~1,000,000 번 까지는 예약된 식별자1,000,001이상부터 사용
sid:1000001;


- rev
Snort 규칙의 수정 버전을 정의수정할 경우 1씩 증가
rev:1;


- classtype
Snort 규칙을 분류하는 옵션
clsstype:분류이름;


- priority
우선 순위를 지정하는 옵션1 ~ 10까지 수를 사용( 값이 작을수록 우선순위가 높음 )
priority:10;


- reference
해당 규칙에 참고가 되는 URL을 지정하는 옵션
Reference:래퍼런스명 http://…;

- Payload
Payload는 Snort Rule에서 실질적으로 악성 패킷을 탐지하는 옵션

- content
패킷 데이터에서 매칭할 문자열을 지정하는 옵션
content:”abc”;=> abc 문자열을 탐지


- nocase
Content 옵션 뒤에 작성하여 content 문자열을 대소문자 구분없이 탐지하도록 함
content:”abc”;nocase;=> Abc,ABC,aBc,abC 모두 탐지


- offset
패킷의 페이로드에서 매칭을 시작할 문자열의 위치를 지정하는 옵션( 지정한 바이트 만큼 떨어진 위치부터 탐색시작, 시작은 0Byte )
offset:3;=> 4Byte부터 탐색을 시작


- distance
content 옵션 값 매칭 후를 기준으로 지정된 패턴 탐색을 시작하기 전 무시해야 하는 패킷의 거리를 지정하는 옵션
content:”abc”;content:”test”;distance:5;=> abc 문자열 매칭 후 지점을 기준으로 5byte 이후 test 문자열 탐색


- within
content 옵션 매칭 후 다음 지정된 탐색을 진행할 범위를 지정하는 옵션
content:”abc”;content:”test”;within:5;=> abc 문자열 매칭 후 지점을 기준으로 5Byte 이내에 test 문자열 탐색




#### HTTP 옵션
content 옵션 값이 탐색할 범위를 HTTP 영역으로 한정할 때 사용하는 옵션

- http_method
패킷 페이로드 중 HTTP 메소드 영역에서 content 옵션 값을 매칭하는 옵션


- http_uri / http_raw_uri
패킷 페이로드 중 URI 영역을 탐색하는 옵션raw가 포함된 옵션은 디코딩되지 않은 페이로드를 대상으로 탐색하는 옵션


- http_cookie
http 쿠키 값을 탐색하는 옵션


- http_header / http_raw_header
HTTP 헤더 영역을 요청과 응답에 관계없이 탐색하는 옵션


- http_client_body
HTTP 바디 영역을 탐색하는 옵션


- http_stat_code
HTTP 응답 메시지에서 상태 코드 영역을 탐색하는 옵션


- http_stat_msg
HTTP 응답 메시지에서 상태 메세지 영역을 탐색하는 옵션

#### 흐름 옵션 : 패턴을 매칭할 때 적용해야 하는 트래픽의 방향을 지정하는 옵션

- flow
흐름 옵션을 사용할 때 반드시 사용해야 하는 옵션


- to_client / from_server
Server -> Client 방향으로 오는 패킷을 매칭하는 옵션


- to_server / from_client
Client -> Server 방향으로 오는 패킷을 매칭하는 옵션


- Established
세션이 성립된 상태의 패킷을 매칭하는 옵션


- statless
세션 성립 여부와 상관 없이 매칭하는 옵션

