---
category: record
---

# 수업을 시작하며

* w3s의 자바스크립트를 한번 훑어 본 후 1.5배속으로 강의를 빠르게 들었다.
* 이미 조금씩 맛을 본 상황이어서 빠르게 볼 수 있었던 것 같다.
* 수업 중 기억하고 싶은 것들을 적어 놓았다.



# 2. 수업의 목적

* 실습을 통해 문법을 익혀간다.
* 자바스크립트는 사용자와 상호작용하기 위한 언어
* 자바스크립트는 html을 제어하는 언어
* 웹페이지를 동적으로 만들어줌

# 3. html과 js의 만남: script 태그

* `<script>` ,`</script>`태그를 통해 javascript 를 추가할 수 있다.

# 4. html과 js의 만남: 이벤트

* onclick
  * 속성값으로 반드시 자바스크립트가 와야한다
  * 속성값은 웹브라우저가 기억하고 있다가, 속성이 위치하고 있는 태그에 해당 이벤트가 일어나면 자바스크립트 코드가 실행된다
* 웹브라우저에서 일어날 수 있는 주요 이벤트
  * 10개에서 20개 정도되는 이벤트를 미리 정의해 놓고 있다.
  * 이런 이벤트를 통해서 사용자와 상호작용 할 수 있다.
* 어떤 이벤트와 속성이 있는지는 검색하면 다 나온다

# 5. html과 js의 만남: 콘솔

* 웹브라우저의 콘솔을 통해 자바스크립트를 작동시켜볼 수 있다.
* 콘솔에서 실행시킨 자바스크립트는 해당 웹페이지 안에서 작동하는 자바스크립트처럼 작동합니다

# 6. 데이터타입 - 문자열과 숫자

* 자바스크립트에서의 데이터형태에 대해서 알아봅니다.
* 데이터타입 = 자료형
* 자바스크립트 데이터타입 종류
  * 기본 자료형인 여섯가지 데이터 타입
    * Boolean
    * Null
    * Undefined
    * Number
    * String
    * Symbol (ECMAScript 6에 추가됨)
  * 별도로 Object도 있음
* 산술연산자
  * +, -, * , /
* 문자열
  * '문자열'.length 와 같이 문자열과 관련된 도구들이 많다. 검색을 통해 많은 자료를 찾아볼 수 있다.

# 7. 변수와 대입 연산자

ex) `x = 1;`

* x : 변수 variable
* = : 대입 연산자
* 1 : 상수 constant

# 8. 웹브라우저 제어

# 9. css기초 : style 속성

* css의 핵심적인 것을 알아보자
* 아는 내용이라서 3강 skip

# 12. 제어할 태그 선택하기

* 간단한 자바스크립트 실습

  * ```html
    <input type="button" value="night" onclick="
    document.querySelector('body').style.backgroundColor = 'black';
    document.querySelector('body').style.colopr = 'white';
    ">
    <input type="button" value="day" onclick="
    document.querySelector('body').style.backgroundColor = 'white';
    document.querySelector('body').style.colopr = 'black';
    ">
    ```

# 13. 프로그램, 프로그래밍, 프로그래머

* 순서 -> 프로그램
* 순서를 만드는 것 -> 프로그래밍
* 순서를 만드는 사람 -> 프로그래머
* 순서대로 컴퓨터를 조작하는데에 컴퓨터를 주로 사용한다. 또한 이러한 것을 반복할 수 있는것이 컴퓨터
* 시간의 순서에 따라서, 실행되어야 할 기능을, 프로그래밍 언어의 문법에 맞게, 글로 적어두는 방식 -> 프로그램의 본질적인 목적
* `순서`, `조건`, `방법`, 이러한것의 `정리정돈`
* 앞으로는 본편적으로 프로그래밍 언어가 가지고 있는 기능들을 알아 볼 것이다.

# 14. 조건문 예고

* 예시 : toggle

  * ```html
    <input type="button" value="black" onclick="
    if(document.querySelector('body').dataset.mode === 'day'){
    document.querySelector('body').style.backgroundColor = 'black';
    document.querySelector('body').style.colopr = 'white';
    document.querySelector('body').dataset.mode = 'night';
    this.value = 'day';
    } else {
    document.querySelector('body').style.backgroundColor = 'white';
    document.querySelector('body').style.colopr = 'black';
    document.querySelector('body').dataset.mode = 'day';
    this.value = 'night';
    }
    ">
    ```

# 15. 비교 연산자와 블리언

* 2배속으로 빠르게 보고 넘어간다.

# 16. 조건문

- 2배속으로 빠르게 보고 넘어간다.

# 17. 조건문의 활용

* ```html
  <input id="night_day" type="button" value="black" onclick="
  if(document.querySelector('#night_day').value === 'night'){
  document.querySelector('body').style.backgroundColor = 'black';
  document.querySelector('body').style.colopr = 'white';
  document.querySelector('#night_day').value === 'day'
  this.value = 'day';
  } else {
  document.querySelector('body').style.backgroundColor = 'white';
  document.querySelector('body').style.colopr = 'black';
  document.querySelector('#night_day').value === 'night'
  this.value = 'night';
  }
  ">
  ```

# 18. 리팩토링 중복의 제거

* 코드 개선작업

* 가독성, 유지보수 편리하게, 중복제거

* ```html
  <input type="button" value="black" onclick="
  var target = document.querySelector('body');
  if(this.value === 'night'){
  target.style.backgroundColor = 'black';
  target.style.colopr = 'white';
  this.value = 'day';
  } else {
  target.style.backgroundColor = 'white';
  target.style.colopr = 'black';
  this.value = 'night';
  }
  ">
  ```

* 코딩을 잘하는 방법 하나 소개 - 중복을 끝까지 찾아가서 제거하라! 많은 기술들이 중복을 제거하기 위해 탄생했다.

# 19. 반복문 예고

* ```javascript
  var links = document.querySelectorAll('a');
  var i = 0;
  while(i<links.length){
      links[i].style.color='powderblue';
      i=i+1;
  }
  ```

# 20. 배열

- 2배속으로 빠르게 보고 넘어간다.

# 21. 반복문

* 2배속으로 빠르게 보고 넘어간다.

# 22. 배열과 반복문

* ```
  var coworkers = ['egoing', 'leezche', 'duru', 'teaho'];
  var i = 0;
  while(i < coworkers.length){
      document.write('<li>'+coworkers[i]+'</li>');
      i = i +1;
  }
  ```

# 23. 배열과 반복문의 활용

* ```javascript
  var alist = document.querySelectorAll('a');
  var i = 0;
  while(i < alist.length){
      alist[i].style.color = 'powderblue';
      i = i + 1;
  }
  ```

# 24. 함수예고

* 2배속으로 빠르게 보고 넘어간다.

# 25. 함수

* 2배속으로 빠르게 보고 넘어간다.
* 함수의 기본사항

# 26.  함수 : 매개변수와 인자

* 2배속으로 빠르게 보고 넘어간다.

# 27. 함수(리턴)

* 2배속으로 빠르게 보고 넘어간다.

# 28. 함수의 활용

* 2배속으로 빠르게 보고 넘어간다.

# 29. 객체 예고

* 객체의 여러가지 특성중에서 한가지만 이야기 할 것입니다. 객체에 대한 느낌을 가지고 확장해 나가시면 됩니다.

* 연관된 함수와 변수를 잘 정리하기 위한 도구

* 함수명이 겹친다면 나중에 쓴 함수가 기존의 함수를 덮어씌운다.

* ```javascript
  function LinksSetColor(color){
      var alist = document.querySelectorAll('a');
      var i = 0;
      while(i < alist.length){
          alist[i].style.color = color;
          i = i +1;
      }
  }
  function BodySetColor(color){
      document.querySelector('body').style.color = collor;
  }
  function BodySetBackgroundColor(color){
      document.querySelector('body').style.backgroundColor = collor;
  }
  function nightDayHandler(self){
      var target = document.querySelector('body');
      if(self.value === 'night'){
          BodySetBackgroundColor('black');
          BodySetColor('white');
          self.value = 'day';
          LinksSetColor('powderblue');
      } else {
          BodySetBackgroundColor('white');
          BodySetColor('black');
          self.value = 'night';
      }
  }
  ```

# 30. 객체 쓰기와 읽기

* ```javascript
  var coworkers = {
      "programmer":"egoing",
      "designer":"leezche"
  };
  document.write("programmer : "+coworkers.programmer+"<br>");
  document.write("designer : "+coworkers.designer+"<br>");
  coworkers.bookkeeper = "duru";
  document.write("bookkeeper : "+coworkers.bookkeeper+"<br>");
  coworkers["data scientist"] = "teaho";
  document.write("data scientist : "+coworkers["data scientist"]+"<br>");

  ```

# 31. 객체와 반복문

* ```javascript
  for(var key in coworkers) {
      document.write(key+' : '+coworkers[key]+'<br>');
  }
  ```

# 32. 객체프로퍼티와 메소드

* ```javascript
  coworkers.showAlll = function() {
      for (var key in this) {
          document.write(key+ ' : '+this[key]+'<br>');
      }
  }
  coworkers.showAll();
  ```

# 33. 객체의 활용

* ```javascript
  var Body = {
  	setColor:function (color){
          document.querySelector('body').style.color = color;
  	},
      setBackgroundColor:function (color){
          document.querySelector('body').style.backgroundColor = color;
  	}
  }
  var Links = {
      setcolor:function (color){
    	  var alist = document.querySelectorAll('a');
   	   var i = 0;
   	   while(i < alist.length){
    	      alist[i].style.color = color;
     	      i = i +1;
      	}
  	}
  }
  ```

# 34. 파일로 쪼개서 정리 정돈하기

* 2배속으로 빠르게 보고 넘어간다.

# 35. 라이브러리와 프래임워크

* 비슷하긴 하지만 약간 뉘앙스가 다르다

* library

  * 부품이 되는 소프트웨어를 재사용하기 쉽도록 정리되어있는 느낌
  * 라이브러리를 땡겨와서 쓰는 느낌
  * ex - jQuery

* framework

  * 공통적인 부분을 프래임웍을 통해서 만들고 달라지는 부분을 조금씩 다르게 함으로서 우리가 만들고자 하는 것을 첨음부터 만들지 않아도 되게 해주는 것, 반재품과 같은 느낌
  * 프래임워크에 들어가서 사용하는 느낌

* jQuery

  ```javascript
  var Links = {
      setcolor:function (color){
    		$('a').css('color', color);
  	}
  }
  ```

# 36. UI vs API

* UI
  * 사용자가 시스템을 제어하기 위해서 사용하는 조작장치
* API
  * 애플리케이션을 만들기 위해서 프로그래밍을 할 때 사용하는 조작장치
  * 모든 프로그래밍 언어에 해당되는 것

# 37. 종강

* 프로젝트를 시작하세요!
* 시작이 늦어질수록, 공부가 길어질 수록 자신이 짠 코드를 긍정하기 어려워 집니다.
* 프로젝트를 시작하기 전 조언
  1. 모든 기능을 총동원 하려 하지 마세요, 필수 불가결한 최소한의 도구를 가지고 문제를 해결해 보세요
  2. 최소한의 도구로 사용하다가 한계에 부딧치는 순간 다른 개념들을 사용해보며 각각의 관계에 익숙해지다 보면 -> 유창하게 다른 개념들을 사용할 수 있게 되는데
  3. 이렇게 하다가 다시 한계에 부딧치게 된다. 그럼 그때가 다시 공부를 시작할 때
