---
title: CGI,WSGI,uwsgi,uWSGI
tag: [python, webserver, CGI, WSGI]
category: post
---

flask나 django로 웹 애플리케이션을 배포하다 보면 uWSGI, WSGI 등의 용어를 찾아볼 수 있다. 용어가 비슷비슷해 헷갈리는 경우가 많아 용어 정리 및 동작 방식에 대해 정리해 보았다.

## CGI

유저가 웹사이트를 들어올 때 브라우저는 해당 사이트를 제공하는 웹서버와 연결된다. 서버는 요청된 파일을 파일 시스템 안에서 찾고 찾은 내용을 유저에게 보내준다. 이러한 방식은 HTTP protocol의 대략적인 모습이다. 하지만 동적인 웹사이트들은 파일시스템 안의 파일을 기봔으로 만들어져 있기 보다는, 요청이 들어왔을 때 웹서버에 의해서 가동되어 유저에게 제공될 콘텐츠를 만들어 내는 프로그램에 기반한다. 이러한 프로그램은 웹서버가 지원 할 수 있는 어떠한 프로그래밍 언어로든지 만들어 질 수 있다. 대부분의 HTTP서버는 C 혹은 C++로 만들어져 있기 때문에 바로 파이썬 코드를 호출 할 수 없었고 그 사이에는 파이썬과 웹서버를 연결해줄 인터페이스가 필요했다. 이러한 인터페이스는 파이썬과 같은 언어로 만들어진 프로그램이 어떻게 server와 상호작용 해야 할 지를 결정했다. 일반적으로 CGI라고 하는 이 인터페이스는 가장 오래된 것으로 거의 모든 웹 서버에서 지원된다. 웹서버와 상호작용 하기 위해 CGI를 사용하는 프로그램은 모든 요청마다 서버에 의해 새롭게 가동 될 필요가 있다. 즉, 모든 request는 새로운 python interpreter를 가동시키기 때문에 많은 부하를 받아내는데에는 한계가 있다.

## mod_python

mod_python은 Apache process에 인터프리터를 내장시키는 방식으로서, 주로 php에서 넘어온 개발자들이 python을 웹에서 사용하기 위해 만들게 되었다(mod_php). 기존의 CGI가 각각의 request마다 process를 만들고 process가 생길 때 마다 interpreter를 가동시키는 방법이라면, mod_python은 python interpreter를 아파치 서버의 process에 내장 함으로써 각각의 request별로 python interpreter를 가동시키지 않아도 되게끔 만들어 성능 향상을 이뤄냈다. 하지만 문제점도 있는데 다음과 같다.

    * PHP interpreter와는 다르게 python interpreter는 캐시를 사용하는데 파일이 변경 될 경우 웹서버가 재가동 되어야만 변경사항이 적용된다.
    * 아파치 서버는 request를 처리하기 위해 자식 프로세스를 사용하는데 모든 자식 프로세스들은 python interpreter를 사용하지 않더라도 python interpreter가 가동된 상태에서 동작해야만 했다.
    * 특정 버전의 libpython 의존성이 있다.
    * 아파치 웹서버와 엮여 있었기 때문에 다른 웹 서버와 쉽사리 연결되지 않았다.

위와 같은 문제들 때문에 새로운 프로그램을 만 들 때 mod_python을 사용하는데에 어려움이 있지만, 특정 상황에서는 아직도 mod_python을 사용하는 것이 좋을 수 있다. 하지만 WSGI가 생겨남으로써 WSGI를 통해 mod_python 환경에서 python application을 구동 할 수도 있게 되었다.

## FastCGI

FastCGI와 SCGI는 mod_python과는 방식으로 CGI의 성능 문제를 해결하려고 시도했다. mod_python과 같이 인터프리터를 웹서버에 내장하는 방식 대신에 오랬동안 동작하는 백그라운드 프로세스를 만들어서 사용하는 방식으로 성능 향상을 이뤄냈다. 웹서버에서 백그라운드 프로세스로 내용전달을 가능하게끔 하는 모듈은 여전히 웹서버 안에 있다. 백그라운드 프로세스는 서버와는 독립적으로 동작했기 때문에 파이썬을 포함해 어떤 언어로든지 만들어 질 수 있었다. 다만 사용 되는 언어에서 웹서버와 커뮤니케이션을 조작하기 위한 라이브러리를 가지고 있어야 한다. SCGI는 단지 "simpler FastCGI" 이기 때문에 FastCGI와 SCGI의 차이점은 거의 없다. 다만 웹서버가 SCGI를 지원하는 것에는 제한이 있기 때문에 대부분의 사람들은 똑같은 방식으로 동작하는 FastCGI를 사용한다.
오늘 날에는 FastCGI를 직접적으로 사용하는 일은 없다. mod_python처럼 단지 WSGI application을 배포하는 용도로 사용된다.

## WSGI

웹서버와 python appication 간의 인터페이스 규격이다. 이전에는 파이썬으로 웹애플리케이션을 만들기 위해서는 웹서버와 웹서버와 동적 프로그래밍간의 인터페이스에 따라서 각각 다르게 개발해야만 했기 때문에 서버 선택이나 프로토콜 선택에 제약이 있었다. WSGI는 웹서버와 애플리케이션 간에 요청을 주고받는 방식을 통일함으로써 이러한 제약사항을 없애주었다. 
참고로 WSGI는 CGI 디자인 패턴을 기반으로 하였으나 파이썬 인터프리터를 어떤 방식으로 동작시켜야 하는지 명시하지 않았고, 앱 객체가 로드되거나 구성되는 방식 또한 마찬가지로 따로 명시하지 않았다. 그렇기 때문에 WSGI를 지원하는 framework 이나 webserver들은 각각 다른 방식으로(CGI/FastCGI/mod_python) webserver와 통신한다. 따라서 WSGI는 CGI/FastCGI/mod_python 과 비교 할 수 있는 개념이 아니다. 이러한 것들은 모두 protocol의 개념이며, WSGI는 이러한 프로토콜 보다 더 상위레벨(high level)의 개념으로 CGI든 FastCGI방식이든 WSGI규격을 준수해 만들어 졌다면 application 단에서는 어떤 웹서버, 어떤 프로토콜을 사용하는지 신경쓰지 않고 WSGI규격에만 맞춰서 웹 애플리케이션이나 프래임웍을 만들면 된다.

## uWSGI

WSGI표준으로 만들어진 웹서버. mod_wsgi, gunicorn, twisted.web, tornado 등이 있다. 다시 말하면 WSGI는 일종의 규격이고 그 규적을 충족시키며 만든 웹서버들이 위와 같은 것들이다.

## uwsgi

웹서버와 WSGI middleware 간에 통신하는 protocol(규격) 
예를 들면 nginx <-> uwsgi <-> uWSGI(python application이 여기서 동작) 이런식의 관계





--------------참고--------------

[Python documentation - PEP3333 python web server gateway interface](https://www.python.org/dev/peps/pep-3333/)

[python web service에 대한 전반적인 내용이 담겨있다.](https://docs.python.org/2/howto/webservers.html#)

[wsgi, uwsgi uWSGI에 대한 용어 구분 stackoverflow 답변](https://stackoverflow.com/a/8691337/8319977)

["Differences and uses between WSGI, CGI, FastCGI, and mod_python in regards to Python?" 질문에 대한stackoverflow 답변](https://stackoverflow.com/questions/3937224/differences-and-uses-between-wsgi-cgi-fastcgi-and-mod-python-in-regards-to-py)

[uWSGI에 대한 stackoverflow의 답변](https://stackoverflow.com/a/49009791/8319977)

[WSGI does not specify how the Python interpreter should be started, nor how the application object should be loaded or configured, and different frameworks and webservers achieve this in different ways.](https://en.wikipedia.org/wiki/Web_Server_Gateway_Interface)

[WSIG middleware에 대한 간단한 설명과 예시](https://rufuspollock.com/2006/09/28/wsgi-middleware/)

[WSGI가 생겨난 배경에 대해서 알 수 있음](https://www.fullstackpython.com/wsgi-servers.html)

[WSGI와 WSGI middleware에 대한 간단하지만 정확하고 쉬운 설명](https://be.groovie.org/2005/10/07/wsgi_and_wsgi_middleware_is_easy.html)

[Middleware에 대해서 잘 설명해 놓음. WSGI로 만들어진 미들웨어가 mod_wsgi, uWSGI, gunicorn, twisted.web, tornado](https://khanrc.tistory.com/entry/WSGI%EB%A1%9C-%EB%B3%B4%EB%8A%94-%EC%9B%B9-%EC%84%9C%EB%B2%84%EC%9D%98-%EA%B0%9C%EB%85%90)

[WSGI를 지원하는 프레임웍, 서버, 어플리케이션, WSGI middleware와 라이브러리를 나열해 놓음](https://wsgi.readthedocs.io/en/latest/)

[wiki에서 cgi](https://ko.wikipedia.org/wiki/%EA%B3%B5%EC%9A%A9_%EA%B2%8C%EC%9D%B4%ED%8A%B8%EC%9B%A8%EC%9D%B4_%EC%9D%B8%ED%84%B0%ED%8E%98%EC%9D%B4%EC%8A%A4)

[cgi와 wsgi의 차이에 대해서](https://brownbears.tistory.com/350)

[cgi가 무엇인지에 대한 포스팅](https://show-me-the-money.tistory.com/50)

[CGI에 대한 상세한 설명](http://snuet.com/CML/C05/C05.html)

[cgi 동작에 관한 설명](https://docs.python.org/2/howto/webservers.html#common-gateway-interface)

[Common problems with CGI scripts](https://docs.python.org/2/howto/webservers.html#common-problems-with-cgi-scripts)
