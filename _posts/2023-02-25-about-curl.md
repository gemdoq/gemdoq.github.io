---
layout: single
title: "curl에 대하여"
date: 2023-02-25 09:38:17 +0900
categories: knowledge
tags: [curl]
typora-root-url: ../
---

## curl에 대하여
> - curl이란
> - 설치 및 주요 옵션
> - HTTP/HTTPS 다운로드(GET Method)
> - POST/File 업로드(POST, PUT)
> - PATCH, DELETE 메서드
> - HTTP 인증
> - Cookie
> - HTTP Header 설정
> - SSL/TLS 옵션

<br>

## curl이란

curl은 Linux/Unix계열 및 Windows 등 주요 OS에서 구동되는 command line용 data transfer tool

Download/Upload 모두 가능하며, HTTP/HTTPS/FTP/LDAP/SCP/TELNET/SMTP/POP3 등 주요한 프로토콜을 지원

libcurl이라는 PHP, ruby, PERL 및 여러 언어에 바인딩되어 있는 C기반 library가 제공되므로 C/C++ 프로그램 개발 시 위의 protocol과 연계가 필요하다면 libcurl을 사용하여 손쉽게 연계 가능

<br>

## 설치 및 주요 옵션

### 설치

Linux나 Mac OS X에는 기본 탑재

Windows는 build된 바이너리를 설치해도 되고, compiler가 있다면 소스를 받아서 직접 빌드

Windows버전은 cygwin나 MinGW로 빌드한 것보다 VIsual Studio로 빌드한 버전을 다운받는 것이 좋음

Win32-Generic, Win64-Generic 항목에서 받으면 되며, 추천하는 링크는 [🔗 https://winampplugins.co.uk/curl/](https://winampplugins.co.uk/curl/){:target="_blank"}

### 주요 옵션

```bash
curl [options...] <url>
```

option처리는 GNU getopt를 사용하므로 하이픈 하나를 붙이는 short형식의 옵션과 하이픈 두개로 시작되는 long형식의 options이 있음

| short | long                 | 설명                                                         | 비고                                                         |
| :---- | :------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| -k    | --insecure           | https 사이트를 SSL certicate 검증없이 연결                   | wget의 --no-check-certificate와 비슷한 역할 수행             |
| -l    | --head               | HTTP header만 보여주고 Content는 표시하지 않음               |                                                              |
| -D    | --dump-header <file> | <file>에 HTTP header를 기록                                  |                                                              |
| -L    | --location           | 서버에서 HTTP 301이나 HTTP 302응답이 왔을 경우 redirection URL로 따라감<br />--max-redirs뒤에 숫자로 redirection을 몇 번 따라갈지 지정, 기본 값은 50 |                                                              |
| -d    | --data               | HTTP Post data                                               | FORM을 POST하는 HTTP나 JSON으로 데이타를 주고받는 REST기반의 웹서비스 디버깅 시 유용한 옵션 |
| -v    | --verbose            | 동작하면서 자세한 옵션을 출력                                |                                                              |
| -o    | --output FILE        | curl은 remote에서 받아온 데이터를 기본적으로는 콘솔에 출력<br />-o옵션 뒤에 FILE을 적어주면 해당 FILE로 저장(download 시 유용) |                                                              |
| -O    | --remote-name        | file저장 시 remote의 file명으로 저장<br />-o 옵션보다 편리   |                                                              |
| -s    | --silent             | 정숙 모드<br />진행 내역이나 메시지등을 출력하지 않음<br />-o옵션으로 remote data도 /dev/null로 보내면 결과물도 출력되지 않음 | HTTP response code 만 가져오거나 할 경우 유용                |
| -X    | --request            | Request시 사용할 method종류(GET, POST, PUT, PATCH, DELETE)를 기술 |                                                              |
| -i    | --include            | 응답에 Content만 출력하지 않고 서버의 Reponse도 포함해서 출력(디버깅에 유용) |                                                              |
| -J    | --remote-header-name | 어떤 웹서비스는 파일 다운로드시 [Content-Disposition Header](http://www.w3.org/Protocols/rfc2616/rfc2616-sec19.html){:target="_blank"}를 파싱해야 정확한 파일이름을 알 수 있는 경우가 있음<br />-J 옵션을 주면 헤더에 있는 파일 이름으로 저장 |                                                              |

<br>

## HTTP/HTTPS 다운로드(GET Method)

### 다운로드

다운로드 받은 파일을 콘솔로 출력

```bash
curl http://www.gnu.org/software/bash/manual/html_node/index.html
```

지정한 이름으로 저장

```bash
curl -o index.html http://www.gnu.org/software/bash/manual/html_node/index.html
```

서버의 filename으로 저장

```bash
curl -O  http://www.gnu.org/software/bash/manual/html_node/index.html
```

여러 url에서 동시에 다운로드

```bash
curl -O  http://www.gnu.org/software/bash/manual/html_node/index.html -O http://www.gnu.org/savannah-checkouts/gnu/libiconv/documentation/libiconv-1.13/iconv.1.html
```

### 이어받기

-C/--continue-at <offset>옵션을 주면 이어받기 가능(offset에 -를 주면 전송이후 부분부터 이어받음)

```bash
curl -C - -O  http://www.gnu.org/software/bash/manual/html_node/index.html
```

### 특정일 기준 변경되었으면 받기

-z/--time-cond <time>

HTTP 헤더에 If-Modified-Since: 헤더를 추가하여 특정일 이후에 변경되었으면 다운로드 수행

아래 예제는 2011년 12월 21일 이후에 변경되었으면 다운로드 수행

```bash
curl -z 21-Dec-11 http://www.example.com/yy.html
```

날짜앞에 -를 추가하면 If-Unmodified-Since: 헤더를 추가하여 특정일 이전에 변경되었으면 다운로드 수행

다음 명령어는 2011년 12월 21일 이전에 변경되었으면 다운로드(날짜에 -를 추가)

```bash
curl -z -21-Dec-11 http://www.example.com/yy.html
```

### HTTP 응답코드만 출력

HTTP Header나 contents는 빼고 HTTP Response code만 출력

서버의 정상작동 여부 점검 시 유용

```bash
curl -L -k -s -o /dev/null -w "%{http_code}\n" http://www.example.com/yy.html
```

이를 활용하여 웹 서버의 정상동작 여부를 확인

```bash
#!/bin/bash

if [ "$#" -lt 1 ]; then
    echo "$# is Illegal number of parameters."
    echo "Usage: $0 http://example.org"
    exit 1
fi

SERVER=$1

RESPONSE=$(curl -L -s -o /dev/null -w "%{http_code}" ${SERVER})

if [ ${RESPONSE} -ne 200 ];then
    echo "${SERVER} returned bad status: ${RESPONSE}"
    exit ${RESPONSE}
fi;
```

### 결과값에 HTTP Header 포함

-i ,–include옵션을 사용하면 서버의 응답에 서버가 보낸 HTTP헤더를 추가하여 출력

디버깅에 유용

```bash
curl -i https://api.github.com -u valid_username:valid_password
```

### jq와 연동해서 서버 JSON 파싱

명령행 json처리기인 jq를 연동해서 서버의 JSON응답을 보기 좋게 파싱

```bash
curl 'https://api.github.com/repos/stedolan/jq/commits?per_page=5' | jq .
```

배열로 넘어온 응답 중 첫 번째 원소 추출

```bash
curl 'https://api.github.com/repos/stedolan/jq/commits?per_page=5' | jq '.[1]'
```

#### jq란

jq는 command line용 json processor

curl이나 httpie등 command line용 http 처리기와 연계하여 JSON기반 REST API를 디버깅할 때 유용

<br>

## POST/File 업로드(POST, PUT)

### File Upload(PUT)

-T, --upload-file옵션을 사용하면 서버에 파일을 전송할 수 있음

a.pdf를 서버에 전송

```bash
curl -T a.pdf http://posttestserver.com/post.php
```

-T로 파일을 전송하면 POST가 아닌 PUT Request로 처리

직접 PUT으로 method를 실행할 경우 -X PUT옵션을 추가해서 URL을 호출

```bash
curl -L -X PUT 'https://postman-echo.com/put' \
--data-raw 'This is expected to be sent back as part of response body.'
```

### HTTP FORM POST

-X POST옵션을 추가하거나 -d(--data)옵션을 지정하면 기본값 POST로 설정됨

FORM 파일

```xml
<form method="POST" action="post.php">
    <input type=text name="first_name">
    <input type=text name="last_name">
    <input type=submit name=press value=" OK ">
 </form>
```

POST 데이타는 "param1=value1&param2=value2" 형식으로 전달

```bash
curl -d "first_name=Bruce&last_name=Wayne&press=%20OK%20" http://posttestserver.com/post.php
```

데이타에 공백이나 기타 특수 문자가 있을 경우 URL encoding을 해야 함

공백일 경우 일일이 +로 변환해서 전송해야 하지만 최신 버전의 curl(7.18.0 이후)은 FORM 파라미터를 URL Encoding해주는 --data-urlencode옵션을 사용하면 별도로 인코딩을 해주지 않아도 됌

```bash
curl --data-urlencode "first_name=Bruce" --data-urlencode "last_name=Wayne" --data-urlencode "press= OK " http://posttestserver.com/post.php
```

Hidden field 전송 시 일반 필드처럼 name=value형식으로 전송

### HTTP POST data

-d, --data옵션 뒤에 전송할 데이터를 기술

```bash
curl -L -v -d '{"name": "superman", "age" : 30}' -H "Accept: application/json" -H "Content-Type: application/json" http://posttestserver.com/post.php
```

-d옵션을 추가하면 -X POST는 제외 가능

### HTTP POST File

file을 POST할 경우 file name 앞에 @를 붙여줌 

```bash
curl -d @myPostfile http://posttestserver.com/post.php
```

### HTTP POST Binary File

curl은 POST 시 데이터를 text로 취급하므로 binary데이터는 깨질 수 있음

제대로 전송하려면 --data-binary 옵션을 추가

```bash
curl --data-binary @myBinary.jpg http://posttestserver.com/post.php
```

### HTTP File Upload Form

파일 업로드 FORM

```xml
<form method="POST" enctype='multipart/form-data' action="upload.php">
 <input type=file name=upload>
 <input type=submit name=press value="OK">
</form>
```

localfilename은 업로드 할 파일명, submit은 press=OK

```bash
curl --form upload=@localfilename --form press=OK http://localhost/upload.php
```

<br>

## PATCH, DELETE 메서드

### PATCH

Remote에 있는 Resource의 전체를 교체하는 PUT과 달리 일부 항목만 교체할 때 사용하는 메서드

-X PATCH옵션을 추가해서 URL 호출

```bash
curl -L -X PATCH 'https://postman-echo.com/patch' \
--data-raw 'This is expected to be sent back as part of response body.'
```

### DELETE

Remote에 있는 Resource를 삭제할 때 주로 사용

보안 문제가 있으므로 서버는 권한있는 사용자의 호출인지 여부를 확인하고 처리해야 함

client는 -X DELETE옵션을 추가해서 URL 호출

```bash
curl -L -X DELETE 'https://postman-echo.com/delete' \
--data-raw 'This is expected to be sent back as part of response body.'
```

<br>

## HTTP 인증

### Basic Auth

id/pwd가 필요한 사이트의 경우 -u(–user)옵션 뒤에 userid:password를 지정하여 인증

```bash
curl -v -u userid:password http://www.example.com/user.html
```

### Bearer Token Auth

OAuth나 JWT등에 사용하는 Bearer token을 사용하려면 -H옵션 뒤에 'Authorization: Bearer {TOKEN}'를 추가

{TOKEN}은 실제 토큰으로 변경

```bash
curl -v -L  -X POST -H 'Accept: application/json' -H 'Authorization: Bearer 12345' 'https://www..example.com/api/myresource'
```

<br>

## Cookie

### 쿠키를 파일로 저장

파일로 저장할 경우 -c, --cookie-jar 옵션 뒤 쿠키를 저장할 파일명

```bash
curl -v -I  -c cookiejar.txt https://www.example.com
```

### 파일 또는 문자열에서 쿠키 읽기

-b, --cookie 옵션을 사용하면 쿠키를 저장한 파일에서 읽거나 또는 직접 쿠키 값을 설정하여 HTTP 요청할 수 있음

cookie-jar 에서 쿠키 읽기

```bash
curl -v -I  -b cookiejar.txt https://www.example.com
```

### 쿠키 값 설정해서 전송

tomcat이 사용하는 세션 쿠키인 JSESSIONID를 설정해서 서버에 요청

```bash
curl -v -I  -b "JSESSIONID=8A39E226F2B6D4DC32CE0E8D" https://www.example.com
```

### 새로 세션 쿠키 생성

-b옵션과 함께 -j, --junk-session-cookies옵션을 사용하면 쿠키를 저장한 파일에서 세션 정보만 읽지 않으므로 새로 세션 쿠키를 생성할 수 있음

```bash
curl -v -I -b cookiejar.txt -j https://www.example.com
```

<br>

## HTTP Header 설정

특정한 HTTP Header를 설정해서 보내야 할 경우(Ex: json data등) -H(–header)옵션으로 헤더를 설정

### Content-Type Header 설정

```bash
curl -d @myJson.js -H "Content-Type: application/json" http://localhost:8080/jsonEcho
```

### User-Agent 설정

특정 브라우저인(Browser) 것처럼 동작하기 위해서는 -A(--user-agent)옵션을 사용(http://www.useragentstring.com/)

#### User-Agent Chrome 24.0

Chrome 24.0 으로 User-Agent 설정

```bash
curl -d @myJson.js -H "Content-Type: application/json" --user-agent "Mozilla/5.0 (Windows NT 6.2; WOW64) AppleWebKit/537.14 (KHTML, like Gecko) Chrome/24.0.1292.0 Safari/537.14" http://localhost:8080/jsonEcho
```

#### User-Agent MSIE(Internet Explorer) 10.0

IE 10.0

```bash
curl -d @myJson.js -H "Content-Type: application/json" --user-agent "Mozilla/5.0 (compatible; MSIE 10.0; Windows NT 6.1; WOW64; Trident/6.0)" http://localhost:8080/jsonEcho
```

#### User-Agent Firefox 29.0

Firefox 29.0

```bash
curl -d @myJson.js -H "Content-Type: application/json" --user-agent "Mozilla/5.0 (Windows NT 6.1; WOW64; rv:29.0) Gecko/20120101 Firefox/29.0" http://localhost:8080/jsonEcho
```

### Referer 설정

Referer를 체크하는 사이트일 경우 -e(–referer)옵션으로 Referer URL을 설정

```bash
curl --referer http://www.example.come/from  http://www.example.com/to
```

아니면 -H옵션으로 referer헤더를 지정

```bash
curl -H "Referer: http://www.example.come/from"  http://www.example.com/to
```

### Accept-Encoding으로 컨텐츠 압축 요청

전송속도 향상을 위해 mod_deflate(apache httpd)나 ngx_http_gzip_module을 사용해서 컨텐츠를 압축할 수 있음

웹 서버가 컨텐츠 압축을 사용하는지 확인하기 위해 curl 실행 시 Accept-Encoding헤더를 설정

```bash
curl -I -H 'Accept-Encoding: gzip,deflate' https://www.google.com
```

압축 여부는 Content-Encoding: gzip가 서버 response에 포함되었는지로 확인

<br>

## SSL/TLS 옵션

### SSL/TLS 인증서 검증 설정

curl은 기본적으로 https사이트의 SSL인증서를 검증

인증 기관의 인증서 목록이 없거나 모르는 기관에서 발급한 인증서일 경우 다음과 같은 인증서 검증 에러를 발생시키고 동작을 중지

아래 3가지 해결 방법

#### 인증서 검증 안함

TLS 연결 시 인증서의 유효성을 검증하는 데 요청이 실패하는 경우, -k, --insecure 옵션을 사용하면 인증서 검증을 끄고 TLS 세션 구성

```bash
curl -k -L http://www.example.come
```

일반적으로 -k 는 -L 과 같이 사용하는 것이 편리

#### 인증기관 목록 추가하기

신뢰하는 인증 기관 목록(CA List; Certificate Authority List)에 접속하려는 사이트의 인증서를 발급한 기관을 추가

curl을 -v옵션을 주고 실행해서 CA List파일이 어디에 있는지 위치를 확인

```bash
curl -v https://google.com
```

런타임에 옵션으로 CA List 파일을 지정하려면 curl 실행 시 --cacert 옵션으로 CA List를 지정

```bash
curl -v --cacert myca-bundle.crt https://google.com
```

#### CA 인증서 파일 갱신

새로운 인증 기관이 생겼는데 OS 에 포함된 curl 에는 반영이 안 되서 발생할 수 있음

curl 홈페이지에서는 주기적으로 최신 인증 기관 목록을 갱신, 배포하므로 CA 인증서 목록을 다운받아서 기존 파일에 덮어써도 됨

wget 또는 curl명령을 사용해서 https://curl.haxx.se/ca/cacert.pem에서 인증서를 다운

wget 명령

```bash
wget --no-check-certificate https://curl.haxx.se/ca/cacert.pem
```

curl 명령

```bash
curl -k -L -O https://curl.haxx.se/ca/cacert.pem
```

curl -v 옵션으로 CA 인증서 목록 파일의 위치를 확인한 후에 예전 파일은 백업하고 다운받은 인증서 파일을 덮어씌움

```bash
sudo cp cacert.pem  /etc/ssl/certs/ca-certificates.crt 
```

<br>
