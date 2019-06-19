---
title: CGI, WSGI, uwsgi
tag: [python, webserver, CGI, WSGI]
category: post
---

[wiki에서 cgi](https://ko.wikipedia.org/wiki/%EA%B3%B5%EC%9A%A9_%EA%B2%8C%EC%9D%B4%ED%8A%B8%EC%9B%A8%EC%9D%B4_%EC%9D%B8%ED%84%B0%ED%8E%98%EC%9D%B4%EC%8A%A4)

> 공용 게이트웨이 인터페이스(영어: Common Gateway Interface; CGI)는 웹 서버 상에서 사용자 프로그램을 동작시키기 위한 조합이다. 존재하는 많은 웹 서버 프로그램은 CGI의 기능을 이용할 수 있다.

웹 서버 프로그램의 기능의 주체는 미리 준비된 정보를 이용자(클라이언트)의 요구에 응답해 보내는 것이다. 그 때문에 서버 프로그램 그룹에서는 정보를 그 장소에서 동적으로 생성하고 클라이언트에 송신하려하는 조합을 작성하는 것이 불가능했다. 서버 프로그램에서 다른 프로그램을 불러내고, 그 처리 결과를 클라이언트에 송신하는 방법이 고안되었다. 이를 실현하기 위한 서버 프로그램과 외부 프로그램과의 연계법을 정한 것이 CGI이다.

CGI는 환경변수나 표준입출력을 다룰 수 있는 프로그램 언어에서라면 언어의 구별을 묻지 않고 확장하여 이용하는 것이 가능하나, 실행속도나 텍스트 처리의 용이함 등의 균형에 의해 펄이 사용되는 경우가 많았다. 최근[언제?]에는 펄뿐 아니라 파이썬, 루비 등도 널리 쓰이고 있다.

대표적인 애플리케이션에는 전자게시판, 접속 카운터, 위키나 블로그 시스템 등이 있다.

근래[언제?]에는 웹 서버의 프로세스로서 인터프리터를 상주시킴으로써, CGI로부터 프로그램을 호출해 부하를 줄임으로써 성능을 개선한 자바 서블릿이나 mod perl, mod php, FastCGI 등도 공개되었다.

[cgi와 wsgi의 차이에 대해서](https://brownbears.tistory.com/350)

mod-perl, mod-php, mod-python : 웹서버 프로세스로 인터프리터를 상주시키는 방법으로 속도 개선한다. 웹 서버에서는 따로 프로세스를 fork헤서 처리한다.

FastCGI : web server와 소켓이나 tcp연결을 통해 동적인 요청에 응답해 준다. web server와 독립적인 process로 관리된다.

출처: https://show-me-the-money.tistory.com/50 [개발자, Trend를 파헤치다.]

[cgi가 무엇인지에 대한 포스팅()https://show-me-the-money.tistory.com/50)

**차이점**
* cgi
    * 호출(여러번의 요청)이 있을 때 마다 process를 생성하여 처리
* FastCGI
    * 하나의 프로세스로 여러 요청을 처리

웹서버가 프로그램과 상호작용 하는 여러가지 방식이 있고 다음과 같다. CGI, Fast CGI, mod_python
하지만 이들의 통신 방식은 python의 여러 프레임워크마다 그리고 흔히 Proxy로 사용하는 웹서버마다 다르게 정의되어 있음

이러한 웹서버와 python application간의 통신 방식에 존재하는 종속성을 없애기 위해 표준을 정의한 것이 바로 WSGI.

그리고 표준대로 웹서버와 python application간의 다리 역할을 해줄 수 있는 구현체가 WSGI 서버

Python은 테스팅용으로 내부적으로 wsgiref라는 WSGI 서버를 가지고 있음.

대표적인 WSGI 서버는 flup, gunicorn, uwsgi등이 존재합니다.


[wsgi, uwsgi uWSGI에 대한 용어 구분 stackoverflow 답변](https://stackoverflow.com/a/8691337/8319977)

WSGI, uwsgi : 둘다 protocol이다


WSGI는 파이썬에 종속된 개념이고, FCGI는 언어와 상관없는 socket wire 프로토콜이다.
WSGI가 더 놓은 레이어에서 동작하며, 따라서 WSGI-FCGI를 동시에 사용 할 수도 있다.
https://brownbears.tistory.com/350


[CGI에 대한 상세한 설명](http://snuet.com/CML/C05/C05.html)

[관련내용에 대한 답변에 여러 링크가 있는 stackoverflow 답변](https://stackoverflow.com/questions/3937224/differences-and-uses-between-wsgi-cgi-fastcgi-and-mod-python-in-regards-to-py)


[cgi 동작에 관한 설명](https://docs.python.org/2/howto/webservers.html#common-gateway-interface)
Programs using CGI to communicate with their web server need to be started by the server for every request. So, every request starts a new Python interpreter – which takes some time to start up – thus making the whole interface only usable for low load situations.

cgi는 request가 올 때 마다 새로운 python interpreter를 실행시킨다. (시작하는데 시간이 걸림)


[Common problems with CGI scripts](https://docs.python.org/2/howto/webservers.html#common-problems-with-cgi-scripts)
* python cgi script를 실행시킬 수 있도록 권한지정을 해줘야 한다.
* 시스템에 맞게 script 파일의 시작과 끝 지점을 수정해 줘야 한다. 
* web server가 CGI 스크립트를 접근 할 수 있도록 따로 설정해줘야 한다.


[mod_python](https://docs.python.org/2/howto/webservers.html#mod-python)
* mod_python은 주로 php에서 넘어온 개발자들이 python을 웹에서 사용하기 위해 만들게 되었다. (mod_php는게 있음)
* mod_python은 Apache process에 인터프리터를 내장시키는 방식
* mod_python이 기존 CGI보다 향상된 이유 -> 기존의 CGI는 request마다 interpreter를 시작시켜야만 했지만 mod_python은 process에 python interpreter를 내장 함으로써 성는 개선을 시킴
* 하지만 문제도 있음
    * PHP interpreter와는 다르게 python interpreter는 캐시를 사용하는데 파일이 변경 될 경우 웹서버가 재가동 되어야만 변경사항이 적용된다.
    * 아파치 서버는 request를 처리하기 위해 자식 프로세스를 사용하는데 모든 자식 프로세스들은 사용되지 않더라도 python interpreter를 가동시켜야만 한다.
    * 특정 버전의 libpython 의존성이 있다.
    * 아파치 웹서버와 엮여 있었기 때문에 다른 웹 서버와 쉽사리 연됭되지 않았다.
* 위의 문재점들로 인해 새로운 프로그램을 만 들 때 mod_python을 사용하지 말아야 겠지만 특정 상황에서는 아직도 mod_python을 사용하는 것이 좋을 수도 있다. 하지만 WSGI가 생겨남으로써 WSGI 프로그램을 mod_python 환경에서 구동될 수 있게 되었다.


[FastCGI and SCGI](https://docs.python.org/2/howto/webservers.html#fastcgi-and-scgi)
* FastCGI와 SCGI는 다른 방식으로 CGI의 성능 문제를 해결하려고 시도했다. 인터프리터를 웹서버에 내장하는 방식 대신에 오랬동안 동작하는 백그라운드 프로세스를 만들어서 사용했다.
* 하지만 백그라운드 프로세스로 내용전달을 위한 모듈은 여전히 웹서버안에 있다.
* 백그라운드 프로세스는 서버와는 독립적으로 동작했기 때문에 파이썬을 포함해 어떤 언어로든지 만들어 질 수 있었다. 다만 사용 되는 언어에서 웹서버와 커뮤니케이션 하기 위한 라이브러리를 가지고 있어야만 했다.
* 오늘날에는 FastCGI 단독으로 사용되는 경우는 없다. mod_python 처럼 WSGI 어플리케이션을 배포해서 사용하는 용도로 사용 될 뿐이다.

[mod_wsgi](https://docs.python.org/2/howto/webservers.html#mod-wsgi)
* mod_wsgi는 로우레벨의 게이트웨이를 없애려고 시도에 의해 만들어 졌다. FastCGI나 SCGI, mod_python과 같은 것들은 WSGI어플리케이션을 배포하기 위해 사용된다.
* mode_Wsgi는 2가지 모드가 있는데 임베디드 모드와, 데몬모드가 있다. 데몬 모드는 FastCGI와 유사하다. 하지만 FastCGI와는 다르게 mod_wsgi는 자기 자신의 worker-process를 가지며 이러한 것은 관리를 쉽게 만든다.

[WSGI](https://docs.python.org/2/howto/webservers.html#wsgi-servers)
CGI 또는 mod_python과 같은 다양한 저수준 게이트웨이에 연결하는 데 사용되는 코드를 WSGI 서버라고합니다. 이것을 통해서 이미 다양한 웹서버를 사용할 수 있기 때문에 파이썬 웹 어플리케이션은 거의 어느 환경이든 배포될 수 있습니다.



[uWSGI, WSGI에 대해서 잘 설명해 놓은 stackoverflow답변](https://stackoverflow.com/a/49009791/8319977)

[uWSGI와 uwsgi에 대해 잘 설명해 놓음](https://stackoverflow.com/a/38685758/8319977)
Note that uWSGI is a full fledged http server that can and does work well on its own. I've used it in this capacity several times and it works great. If you need super high throughput for static content, then you have the option of sticking nginx in front of your uWSGI server. When you do, they will communicate over a low level protocol known as uwsgi.

[WSGI로 만들어진 미들웨어가 mod_wsgi, uWSGI, gunicorn, twisted.web, tornado](https://khanrc.tistory.com/entry/WSGI%EB%A1%9C-%EB%B3%B4%EB%8A%94-%EC%9B%B9-%EC%84%9C%EB%B2%84%EC%9D%98-%EA%B0%9C%EB%85%90)
[WSGI는 파이썬에 종속된 개념이고, FCGI는 언어와 상관없는 socket wire 프로토콜이다. WSGI가 더 높은 레이어에서 동작하며, 따라서 WSGI-FCGI를 동시에 사용할 수 도 있다.](https://khanrc.tistory.com/entry/WSGI%EC%99%80-CGI%EC%9D%98-%EC%B0%A8%EC%9D%B4)
[결국 WSGI는 서버,게이트웨이 와 애플리케이션,프레임워크의 주고 받는 protocol 방식](https://ko.wikipedia.org/wiki/%EC%9B%B9_%EC%84%9C%EB%B2%84_%EA%B2%8C%EC%9D%B4%ED%8A%B8%EC%9B%A8%EC%9D%B4_%EC%9D%B8%ED%84%B0%ED%8E%98%EC%9D%B4%EC%8A%A4)


# 정리하자면

**CGI**
웹 서버 프로그램의 기능의 주체는 미리 준비된 정보를 이용자(클라이언트)의 요구에 응답해 보내는 것이다. 그 때문에 서버 프로그램 그룹에서는 정보를 그 장소에서 동적으로 생성하고 클라이언트에 송신하려하는 조합을 작성하는 것이 불가능했다. 서버 프로그램에서 다른 프로그램을 불러내고, 그 처리 결과를 클라이언트에 송신하는 방법이 고안되었다. 이를 실현하기 위한 서버 프로그램과 외부 프로그램과의 연계법을 정한 것이 CGI이다.

**WSGI**
웹서버와 python appication 간의 인터페이스 규격이다. WSGI는 CGI/FastCGI/mod_python 과 비교 할 수 있는 개념이 아니다. 이러한 것들은 모두 protocol의 개념이며 WSGI middleware는 이런 protocol을 통해서 webserver와 통신한다. WSGI는 CGI 디자인 패턴을 기반으로 하였으나 꼭 CGI처럼 서브 프로세스를 띄워서 request를 처리할 필요는 없다. CGI가 될 수도 있지만, 안될 수도 있다.(사용하는 방식 나름이라는 이야기) CGI방식이 아니라면 mod_python처럼 webserver에 인터프리터를 내장하거나 FastCGI처럼 daemon process를 띄우는 방식으로 동적인 요청을 처리 할 수 있다. 참고로 이렇게 될 경우 파이썬 애플리케이션은 WSGI middleware위에 얹혀서 돌아가게 된다. WSGI프로토콜 서버의 종류로는 uWSGI, mod_wsgi, gunicorn, twisted.web, tornado 등이 있다. 

**uWSGI**
WSGI표준으로 만들어진 웹서버. WSGI middleware라고 할 수 있으며, 같은 역할을 하는 다른 웹서버로는 mod_wsgi, gunicorn, twisted.web, tornado 등이 있다. 다시 말하면 WSGI는 일종의 규격이고 그 규적을 충족시키며 만든 웹서버들이 위와 같은 것들이다.

**uwsgi**
웹서버와 WSGI middleware 간에 통신하는 protocol(규격) 
예를 들면 nginx <-> uwsgi <-> uWSGI(python application이 여기서 동작) 이런식의 관계
