---
title: >-
 if __name__ == "__main__": 의미
tag: python
excerpt: >-
 해당내용을 패스트캠퍼스 수업시간에 한번 들은적이 있는데 다시보면 헷갈리고 명확하게 개념을 이해하고 싶어서 구글검색을 감행했다.
category: post
---


# `if __name__ == "__main__"`: 의미

해당내용을 패스트캠퍼스 수업시간에 한번 들은적이 있는데 다시보면 헷갈리고 명확하게 개념을 이해하고 싶어서 구글검색을 감행했다.

원래 처음의 생각은 검색으로 나온 정보를 내 언어로 다시 정리하려고 했다.(그래야 이해도 잘 되고, 내가 궁금한 부분을 명확하게 정리할 수 있기 때문이다)

그러나 구글신은 이러한 나의 마음을 이미 알기라고 한듯 이미 미친듯이 잘 정리된 블로그를 소개해 주었다.

그래도 내가 다시 정리하려고 했는데 다시보니 그 블로그 글을 똑같이 타이핑하는 것과 다름 없었다. 그래서 해당 블로그 글을 그대로 가져오고 출처를 남기기로 했다.

그냥 링크만 남겨도 될 것 같지만 혹시나 그 글이 없어질 것을 염려했고, 내 저장소에 직접 남기고 싶었기 때문에 글을 복사해 왔다.

아래는 해당 글과 링크

출처: [http://pinocc.tistory.com/175 [땅뚱 창고]](http://pinocc.tistory.com/175)

------

참고 : [http://stackoverflow.com/questions/419163/what-does-if-name-main-do](http://stackoverflow.com/questions/419163/what-does-if-name-main-do)

​         [http://bytebaker.com/2008/07/30/python-namespaces/](http://bytebaker.com/2008/07/30/python-namespaces/)

파이썬 프로그래밍을 보다보면 아래와 같은 문장을 만나곤 한다. 정확하게 어떤 의미인지 알고 싶어 여기저기 검색한 내용을 정리했다.

`if __name__ == "__main__"`

이 문장을 이해하기 위해서는 파이썬의 namespace 라는 개념을 이해해야 한다. namespace 를 얘기하기 이전에 파이썬에서 name(변수명)이 의미하는 것을 생각해보자.

아래와 같이 파이썬에서는 name 에 값을 줄 수 있다. 그리고 값 뿐 아니라 function 과 같은 형태도 name 을 줄 수 있다. 또한 동일한 name을 재사용할 수 있다.

```python
i = 12

s = "Hello World"

l = [1, 2, 3]

def foo():

print("This is a function")

f = foo

var = 12

var = "Hello World"

var = [1, 2, 3]
```

파이썬에서 names는 파이썬 객체 시스템과 함께 간다고 생각하면 된다. 즉 integer, string, list 및 function 도 모두 파이썬에서는 객체형태로 표현되고, name은 그 객체에 접근하기 위해 사용한다.

**namespace 와 module**

namespace 는 names 를 담을 수 있는 공간이라고 생각하면 된다. 파이썬에서 namespace 를 이해하기 위해서 파이썬 모듈에 대한 약간의 이해가 필요하다. 파이썬에서 module 은 파이썬 코드를 담고 있는 파일이다. 해당 파일에는 파이썬 클래스, 함수 또는 단순하게 names 의 리스트가 들어있을 수 있다.

각 모듈은 자신만의 유일한 namespace 를 갖는다.(모듈의 namespace 이름은 보통 모듈의 파일이름과 같다.) 그래서 동일한 모듈내에서 동일한 이름을 가지는 클래스나 함수를 정의할 수 없다. 또한 모듈은 각각 완벽하게 독립적(isolated)이기 때문에(namespace 가 다르기 때문에), 두 모듈은 동일한 이름을 갖는 클래스나 함수를 정의할 수 있다.

**import 와 namespace**

import 명령을 가지고 namespace 에 대해서 조금 더 알아보도록 하자. module 을 import 하는 방법은 여러가지가 있다. 방법에 따라 namespace 가 달라 질 수가 있다.

1. import <module_name>

   모듈을 import 하는 가장 간단한 방법이고, 일반적으로 추천되는 방법이다. 이렇게 import 를 하게 되면 module 의 name 을 prefix 로 사용함으로써 모듈의 namespace 에 접근할 수 있다.

   아래 예제에서 sys 는 모듈 이름이고, path 는 sys 모듈의 namespace 에 담겨있는 name 이다. 따라서 path 에 접근을 하기 위해서는 모듈 이름인 sys 를 prefix 로 붙여서 sys 모듈의 namespace 에 접근한 후에 사용해야 한다.

   ```python
   import sys

   sys.path

   ['', 'C:\Python34\Lib\idlelib', 'C:\Windows\system32\python34.zip', 'C:\Python34\DLLs', 'C:\Python34\lib', 'C:\Python34', 'C:\Python34\lib\site-packages']
   ```

2. from <module_name> import <name,>

   모듈의 namespace 에서 import 에서 지정된 name 들을 직접 가져오도록 한다. 이렇게 하게 되면 import 이후에 지정한 name 들은 module 의 name을 prefix 로 지정하지 않고도 접근이 가능하다. 하지만, 이 경우에 module 에서 import 된 이름과 main script 에서 지정된 이름이 동일한 경우, 나중에 정의되는 이름으로 대체되어서 이전 것에 접근이 불가능하게 된다.

   단지 몇개의 name 만 필요하다고 명확하게 알고 있는 경우에 사용하는 것이 유용하다.

   ```python
   from sys import path

   path

   ['', 'C:\Python34\Lib\idlelib', 'C:\Windows\system32\python34.zip', 'C:\Python34\DLLs', 'C:\Python34\lib', 'C:\Python34', 'C:\Python34\lib\site-packages']
   ```

3. from <module_name> import *

   2 와 동일하지만, module 에 있는 모든 name 을 직접 현재 namespace 로 가져오게 된다. 이렇게 되면 namespace 가 섞이게 되어서 일반적으로 사용을 권장하지 않는다. 차라리 첫번째 타입(1번)의 import 를 사용하는 것이 좋다.

**__main__ namespace**

import 의 경우에 namespace 가 처리되는 것을 알아보았는데, import 가 아니고 파이썬 인터프리터가 최초로 파일을 읽어서 실행하는 경우를 살펴보자. 파이썬 인터프리터는 소스파일을 읽고, 그 안의 모든 코드를 실행하게 되는데, 코드를 실행하기 전에 특정한 변수값을 정의한다. 그중 하나가 __name__ 이라는 변수를 __main__ 으로 세팅을 한다.

즉 python script.py 와 같이 직접 쉘에서 실행하는 경우에는 파이썬 인터프리터가 해당 script.py 모듈을 script 라는 namespace 가 아닌__main__ 이라는 namespace 로 간주하여 다루게 된다.

따라서 처음에 궁금했던 아래 문장은 '만일 이 파일이 인터프리터에 의해서 실행되는 경우라면' 이라는 의미를 갖는다.

`if __name__ == "__main__"`

즉 본인이 구현한 코드가 다른 파이썬 코드에 의해서 모듈로 import 될 경우도 있을 수 있고, 파이썬 인터프리터에 의해서 직접 실행될 경우도 있을 수 있는데, 위 코드는 인터프리터에 의해서 직접 실행될 경우에만 실행하도록 하고 싶은 코드 블럭이 있는 경우에 사용한다.

아래 예제 코드와 결과를 보면 이해하기 쉽다.

(참고: [http://ibiblio.org/g2swap/byteofpython/read/module-name.html](http://ibiblio.org/g2swap/byteofpython/read/module-name.html))

<code>

```
#!/usr/bin/python
# Filename: using_name.py

if __name__ == '__main__':
	print 'This program is being run by itself'
else:
	print 'I am being imported from another module'
```

<output>

```
This program is being run by itself
$ python using_name.py

$ python
>>> import using_name
I am being imported from another module
>>>

```
