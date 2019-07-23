## RabbitMQ SSL

### what is SSL?
모든 정보 전송이 https://로 암호화 프로토콜을 통해 안전하게 전송
### what is advatages of SSL
1) 스니핑 방지
2) 피싱 방지
3) 데이터 변조 방지
4) 기업 신회도 향상
----------

### TLS vs SSL
#### TLS
- 인터넷에서 정보를 암호화해서 송수신하는 프로토콜
- 표준
#### SSL
- 표준 아님

##### 즉, TLS 프로토콜은 SSL 프로토콜에서 발전한 것

### TLS 통신 과정
- 디지털 인증서 교환 및 유효성 검증에 의한 상호 인증
- 공유 비밀 키 생성을 위한 키 분배 문제점을 막아주는 비대칭 암호화 기술 사용
- SSL 또는 TLS는 메세지의 대칭 암호화에 공유 키를 사용함
![tls](https://user-images.githubusercontent.com/37536415/61677565-0c25f280-ad3b-11e9-8de3-a38b73d33e42.png)

출처 : boansecurity.blogspot.com

----------

### what is rabbitmq?
- AMQP(Advanced Message Queing Protocol)을 구현한 메시지 브로커
- mq : message queue – 신뢰성있는 메시지 전달
- 현업 분업을 위해서 필요
#### rabbitmq workflow
![rabbit](https://user-images.githubusercontent.com/37536415/61677823-e0573c80-ad3b-11e9-9945-fdd22d55a403.png)
![ra](https://user-images.githubusercontent.com/37536415/61677828-e2210000-ad3b-11e9-856d-333aa0ee3c25.png)

#### 현대의 server architecture
- micro service : 서버 하나씩 작은 단위 기능으로 줄이기
- 왜 서버로 쪼개냐 : 서로 독립된 실행을 위해, 분업을 할 수도 있음
- 정해진 규약에 맞춰서 개발만 하면 됨. 
###### 위의 하나씩 개발된 것을 합쳐주고 싶어서 rabbit MQ를 사용

----------
# start!

###### 개발 환경 : ubuntu 18.04
###### sudo를 붙여서 사용하지 않고, 관리자 권한으로 접근 하는 법 : su
![unnamed](https://user-images.githubusercontent.com/37536415/61678218-2c56b100-ad3d-11e9-8c12-f4244ceac7f3.png)

만약, root password가 설정되어 있지 않다면 아래 명령어 입력
~~~
sudo passwd
~~~

-------------

## download rabbitMQ

1) 서버 설치
~~~
apt-get install rabbitmq-server
~~~

2) 서버 실행
~~~
service rabbitmq-server start
~~~

3) rabbitmq 플러그인 설치
3-1) 플러그인의 설치상태 확인
~~~
rabbitmq-plugins list
~~~
3-2) 플러그인의 설치
~~~
rabbitmq-plugins enable rabbitmq_management
~~~

4) 계정 생성
4-1) user와 userpassword에 원하는 Id 값, passwd 값 넣기
~~~
rabbitmqctl add_user user userpassword
~~~
4-2) 위의 ID 값으로 관리자 계정 등록(user 위치에 id값 넣기)
~~~
rabbitmqctl set_user_tags user administrator
~~~
4-3) 위의 ID에게 관리자 권한 주기
~~~
$ rabbitmqctl set_permissions -p / user ".*" ".*" ".*"
~~~

5) 사이트 들어가기
~~~
service rabbitmq-server restart
~~~
ubuntu에서 http://localhost:15672 로 접속

![7](https://user-images.githubusercontent.com/37536415/61681614-28309080-ad49-11e9-8a0e-5c1908045f89.png)

-------------

## create key(CA, server, client)
###### what is CA? 인증 기관(certificate authority)

##### 시작은 root directory
1) 디렉토리 만들기
~~~
cd etc
~~~
~~~
cd rabbitmq
~~~
rabbitmq directory가 없다면, mkdir rabbitmq

~~~
mkdir testca
cd testca
mkdir certs private
chmod 700 private
echo 01 > serial
touch index.txt
~~~

2) testca 디렉토리에 openssl.cnf 파일 생성
~~~
vi openssl.cnf
~~~
아래 내용 복사 붙여넣기
~~~
[ ca ]
default_ca = testca

[ testca ]
dir = .
certificate = $dir/cacert.pem
database = $dir/index.txt
new_certs_dir = $dir/certs
private_key = $dir/private/cakey.pem
serial = $dir/serial

default_crl_days = 7
default_days = 365
default_md = sha256

policy = testca_policy
x509_extensions = certificate_extensions

[ testca_policy ]
commonName = supplied
stateOrProvinceName = optional
countryName = optional
emailAddress = optional
organizationName = optional
organizationalUnitName = optional

[ certificate_extensions ]
basicConstraints = CA:false

[ req ]
default_bits = 2048
default_keyfile = ./private/cakey.pem
default_md = sha256
prompt = yes
distinguished_name = root_ca_distinguished_name
x509_extensions = root_ca_extensions

[ root_ca_distinguished_name ]
commonName = hostname

[ root_ca_extensions ]
basicConstraints = CA:true
keyUsage = keyCertSign, cRLSign

[ client_ca_extensions ]
basicConstraints = CA:false
keyUsage = digitalSignature
extendedKeyUsage = 1.3.6.1.5.5.7.3.2

[ server_ca_extensions ]
basicConstraints = CA:false
keyUsage = keyEncipherment
extendedKeyUsage = 1.3.6.1.5.5.7.3.1
~~~

3) openssl 명령어 .pem과 .cer 생성
3-1) pem 생성 : cakey.pem 생성
~~~
openssl req -x509 -config openssl.cnf -newkey rsa:2048 -days 365 \
    -out cacert.pem -outform PEM -subj /CN=MyTestCA/ -nodes
~~~
결과 : 
<img width="575" alt="1" src="https://user-images.githubusercontent.com/37536415/61680961-81e38b80-ad46-11e9-8a64-61c2bfb4105c.png">
3-2) cem 생성
~~~
openssl x509 -in cacert.pem -out cacert.cer -outform DER
~~~
결과 : <img width="618" alt="2" src="https://user-images.githubusercontent.com/37536415/61681007-b48d8400-ad46-11e9-9894-fab2e8355988.png">

4) server용 key 생성
- testca 디렉토리 공간(/etc/rabbitmq)에서 server 디렉토리 만들기
~~~
cd /etc/rabbitmq
mkdir server
~~~
- server 디렉토리에서 pem 만들기
~~~
cd server
openssl genrsa -out key.pem 2048
~~~
결과 : <img width="617" alt="3" src="https://user-images.githubusercontent.com/37536415/61681138-38e00700-ad47-11e9-8500-c40604334fa6.png">
~~~
openssl req -new -key key.pem -out req.pem -outform PEM \
  -subj /CN=$(hostname)/O=server/ -nodes
~~~
결과 : <img width="621" alt="4" src="https://user-images.githubusercontent.com/37536415/61681203-8492b080-ad47-11e9-82c6-8e5d94d508e8.png">

~~~
cd ../testca

openssl ca -config openssl.cnf -in ../server/req.pem -out \
  ../server/cert.pem -notext -batch -extensions server_ca_extensions
~~~
결과 : <img width="623" alt="5" src="https://user-images.githubusercontent.com/37536415/61681205-85c3dd80-ad47-11e9-9947-bf512cde389f.png">

~~~
cd ../server

openssl pkcs12 -export -out keycert.p12 -in cert.pem -inkey key.pem -passout pass:MySecretPassword
~~~
결과 : <img width="619" alt="6" src="https://user-images.githubusercontent.com/37536415/61681228-a724c980-ad47-11e9-93ad-f46d8b7a9edd.png">


5) client 용 key 생성
- rabbitmq 디렉토리로 이동(/etc/rabbitmq)
~~~
cd /etc/rabbitmq
~~~
- client 디렉토리 생성
~~~
mkdir client
cd client
~~~
- key 생성
~~~
openssl genrsa -out key.pem 2048

openssl req -new -key key.pem -out req.pem -outform PEM \
    -subj /CN=$(hostname)/O=client/ -nodes
    
cd ../testca

openssl ca -config openssl.cnf -in ../client/req.pem -out \
    ../client/cert.pem -notext -batch -extensions client_ca_extensions
    
cd ../client

openssl pkcs12 -export -out keycert.p12 -in cert.pem -inkey key.pem -passout pass:MySecretPassword
~~~
##### client key 생성은 server와 거의 비슷(차이점 ==> Server : keyUsage = keyEncipherment / Client : keyUsage = digitalSignature)


--------

## SSL 사용하기

config file에 설정
defualt rabbitmq는 config file이 없음(rabbitmq-env.conf 파일은 config 파일이 아님)

![8](https://user-images.githubusercontent.com/37536415/61681615-28c92700-ad49-11e9-9071-42655d56fdb8.png)

위의 사진은 http://localhost:15672 의 일부분이다.
처음에는 rabbitmq.config 파일 옆에 (not found) 라고 적혀있을 것이다.

###### 즉, /etc/rabbitmq 경로에 rabbitmq.config 파일을 생성해주어야 한다.
~~~
cd /etc/rabbitmq
vi rabbitmq.config
~~~
rabbitmq.config 파일에 아래의 내용을 넣어준다.
~~~
[
  {rabbit, [
     {ssl_listeners, [5671]},				
     {ssl_options, [{cacertfile,"/etc/rabbitmq/testca/cacert.pem"},		
                    {certfile,"/etc/rabbitmq/server/cert.pem"},			
                    {keyfile,"/etc/rabbitmq/server/key.pem"},			
                    {verify,verify_peer},				
                    {fail_if_no_peer_cert,false}]}		
   ]}
].
~~~

시스템에서 지원하는 chpher suites나 ssl version 확인하기
~~~
rabbitmqctl eval 'ssl:cipher_suites(openssl).'
~~~
~~~
erl
1> ssl:verions().
~~~
결과 : <img width="519" alt="9" src="https://user-images.githubusercontent.com/37536415/61681812-0552ac00-ad4a-11e9-98b4-6458b545c21a.png">

###### ssl version을 확인하면, 'tlsv1.2', 'tlsv1.1', tlsv1 인 것을 볼 수 있다.
<img width="289" alt="10" src="https://user-images.githubusercontent.com/37536415/61681905-55ca0980-ad4a-11e9-8785-f22b3ff3a5ab.png">

----------
### 인증서 합치기
~~~
cat testca/cacert.pem >> all_cacerts.pem

cat otherca/cacert.pem >> all_cacerts.pem
~~~

-------

### SSL/TLS 버젼 설정

- rabbitmq.config 파일 내용 변경
~~~
[
 {ssl, [{versions, ['tlsv1.2', 'tlsv1.1', tlsv1]}]},
  {rabbit, [
     {ssl_listeners, [5671]},
     {ssl_options, [{cacertfile,"/etc/rabbitmq/testca/cacert.pem"},
                    {certfile,"/etc/rabbitmq/server/cert.pem"},
                    {keyfile,"/etc/rabbitmq/server/key.pem"},
                    {verify,verify_peer},
                    {versions, ['tlsv1.2', 'tlsv1.1', tlsv1]},
                    {fail_if_no_peer_cert,false}]}
   ]}
].
~~~
##### config 파일을 수정했다면, 항상 restart
~~~
service rabbitmq-server restart
~~~

- openssl을 통한 연결 확인
~~~
openssl s_client -connect 127.0.0.1:5671 -tls1

openssl s_client -connect 127.0.0.1:5671 -tls1_2
~~~

## 위의 코드를 실행하고 만약 erro104를 만났을 때(아래 사진), log 파일 확인하기
### openssl s_client error104

<img width="422" alt="1" src="https://user-images.githubusercontent.com/37536415/61682081-3b446000-ad4b-11e9-9f0e-93e18db0eb5d.png">

#### 로그 파일의 위치
<img width="486" alt="2" src="https://user-images.githubusercontent.com/37536415/61682218-bb6ac580-ad4b-11e9-920d-4ef338ba4517.png">

~~~
cd /var/log/rabbitmq

vi 해당 파일
~~~

- error report 확인
![3](https://user-images.githubusercontent.com/37536415/61682452-b3f7ec00-ad4c-11e9-9f57-8ef3445598e3.png)


해당 위치의 .pem의 권한이 잘못된 것
root에서 rabbitmq로 권한을 바꿔주기

- 잘못된 .pem 파일을 가진 곳의 경로로 이동하기
~~~
cd /etc/rabbitmq/server
~~~

- 권한 변경
~~~
chmod rabbitmq:rabbitmq key.pem(error가 나는 pem 파일에 대해 권한 변경)
~~~

- 다시 restart 하고, openssl s_client 명령어 실행해보기
~~~
service rabbitmq-server restart

openssl s_client -connect 127.0.0.1:5671 -tls1
~~~

##### 잘된 경우 Log file
결과 : 
![4](https://user-images.githubusercontent.com/37536415/61682752-dd654780-ad4d-11e9-99d3-ea84437cc51e.png)

----------------
## node js로 ssl connection

- /etc/rabbitmq/client 디렉토리로 이동
~~~
cd /etc/rabbitmq/client
~~~

- npm init
~~~
npm init
~~~

아래와 같이 name, version, description 등을 질문할 때, 그냥 enter key 누르면 됨.
![5](https://user-images.githubusercontent.com/37536415/61682843-32a15900-ad4e-11e9-98e5-771cb9ace1e3.png)

~~~
ls
~~~

디렉토리를 살펴보면, package.json이라는 파일 생성(아래 명령어로 내부 살펴보기)
~~~
vi package.json
~~~

- express, amqplib를 설치해야 함
~~~
npm install express -D
npm install amqplib -D
~~~

- 다시 package.json 내부 살펴보기
~~~
vi package.json
~~~
내부 : ![6](https://user-images.githubusercontent.com/37536415/61683042-00dcc200-ad4f-11e9-9dd5-7dae9440ef41.png)

- index.json 파일 만들기
~~~
vi index.json
~~~

- 아래 내용 붙여넣기(주의할 점 : 생성한 .pem 경로 설정을 잘 해주어야 함. 아래의 코드는 위의 설명한 내용을 기준으로 경로 설정함.)
~~~
var fs = require('fs')
const path = require('path')

var opts = {
  cert: fs.readFileSync(path.resolve(__dirname, '/etc/rabbitmq/server/cert.pem')),
  key: fs.readFileSync(path.resolve(__dirname, '/etc/rabbitmq/server/key.pem')),
  ca: [fs.readFileSync(path.resolve(__dirname, '/etc/rabbitmq/testca/cacert.pem'))],
  rejectUnauthorized: false
}

var q = 'tasks'

function bail (err) {
  console.error(err)
  process.exit(1)
}

// Publisher
function publisher (conn) {
  conn.createChannel(onOpen)
  function onOpen (err, ch) {
    if (err != null) bail(err)
    ch.assertQueue(q)
    const msg = 'something to do'
    ch.sendToQueue(q, Buffer.from(msg))
    console.log('Publisher: ', msg)
  }
}

// Consumer
function consumer (conn) {
  conn.createChannel(onOpen)
  function onOpen (err, ch) {
    if (err != null) bail(err)
    ch.assertQueue(q)
    ch.consume(q, function (msg) {
      if (msg !== null) {
        console.log('Consumer: ', msg.content.toString())
        ch.ack(msg)
      }
    })
  }
}

require('amqplib/callback_api')
  .connect('amqp://guest:guest@localhost:5672', opts, function (err, conn) {
    if (err != null) bail(err)
    consumer(conn)
    publisher(conn)
  })
~~~

- 위의 내용을 저장하고, index.js 파일 실행
~~~
node index.js
~~~
결과 : ![7](https://user-images.githubusercontent.com/37536415/61683315-10103f80-ad50-11e9-94b0-707b9aca5d45.png)



