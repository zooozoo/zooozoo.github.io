---
category: record
---
# 나의 여행기 기록 SNS EXPLOG를 개발하며 기록해 놓은 자료입니다.  



## 11-21 화요일

**서버 사전세팅관련**

* 도커만들기
* postgresql rds 연결
* boto3 s3연결




## 11-22 수요일

* installed apps에 corsheaders 추가하고 사용하는 것 확인해보기

  다른 도메인에서 우리쪽 자원을 요청할 때 그걸 허용하기 위해서 추가해 놓은 것

* settings.py 수정

  * base.py
    1. base.py facebook 관련설정 일단 주석처리
    2. base.py  database관련 설정 삭제
  * local.py
    1. local.py debug true로 바꿔놓기
    2. random, string 삭제
    3. allowed hosts 에 elasticbeanstalk 삭제
  * dev.py
    1. S3_USE_SIG4 확인해 보기 (강사님 자료에는 없음)
  * deploy.py
    1. allowed hosts에 url 추가



* s3 설정 관련 강의자료는 10.30 자에 있음




## 11-23 목요일

* 환경변수 설정관련 명령어 기록
  * `export DJANGO_SETTINGS_MODULE=config.settings.dev`



## 11-24 금요일

**기획 확정되기 전이라도 할 수 있는 것들**

* member앱 만들기
  * signup, login, logout, 기능
* post앱 만들기
  * post create, post list, post detail



## 11-27 월요일

**이메일 인증 구글에서 허용하는 설전 전까지의 기록**

apis.py

```python
class Signup(APIView):
    def post(self, request, *args, **kwargs):
        serializer = SignupSerializer(data = request.data)
        if serializer.is_valid():
            # user = serializer.validated_data
            # user.is_active = False
            serializer.save()
            user_pk = serializer.data['pk']
            user = User.objects.get(pk=user_pk)
            user.is_active = False
            current_site = get_current_site(request)
            message = render_to_string('acc_active_email.html', {
                'user': user,
                'domain': current_site.domain,
                'uid': urlsafe_base64_encode(force_bytes(user_pk)),
                'token': account_activation_token.make_token(user),
            })
            mail_subject = 'Activate your blog account.'
            to_email = user.email
            email = EmailMessage(mail_subject, message, to=[to_email])
            email.send()
            return Response(user.is_active)
            # return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

def activate(request, uidb64, token):
    try:
        uid = force_text(urlsafe_base64_decode(uidb64))
        user = User.objects.get(pk=uid)
    except(TypeError, ValueError, OverflowError, User.DoesNotExist):
        user = None
    if user is not None and account_activation_token.check_token(user, token):
        user.is_active = True
        user.save()
        # login(request, user)
        # return redirect('home')
        return Response('Thank you for your email confirmation. Now you can login your account.')
    else:
        return Response('Activation link is invalid!')
```

tokens.py

```python
from django.contrib.auth.tokens import PasswordResetTokenGenerator
from django.utils import six

class AccountActivationTokenGenerator(PasswordResetTokenGenerator):
    def _make_hash_value(self, user, timestamp):
        return (six.text_type(user.pk) + six.text_type(timestamp)) +  six.text_type(user.is_active)


account_activation_token = AccountActivationTokenGenerator()
```

urls.py

```python
from django.conf.urls import url

from .apis import LoginView, Signup, activate

urlpatterns = [
    url(r'^login/$', LoginView.as_view(), name='login'),
    url(r'^signup/$', Signup.as_view(), name='signup'),
    url(r'^activate/(?P<uidb64>[0-9A-Za-z_\-]+)/(?P<token>[0-9A-Za-z]{1,13}-[0-9A-Za-z]{1,20})/$', activate, name='activate'),
]
```

acc_active_email.html

```
{% raw %}
{% autoescape off %}
Hi {{ user.username }},
Please click on the link to confirm your registration,

http://{{ domain }}{% url 'member:activate' uidb64=uid token=token %}
{% endautoescape %}
{% endraw %}
```



**회의**

회원가입시 사용 필드

```
email, nickname, imgprofile
```



## 11-28 화요일

**token auth duration  관련조사**

[스택오버플로우 토큰인증 기간](https://stackoverflow.com/questions/22943050/how-long-is-token-valid-django-rest-framework)

[스택오버플로우 토큰인증기간 설정관련 답변](https://stackoverflow.com/questions/14567586/token-authentication-for-restful-api-should-the-token-be-periodically-changed)

[drf 공식문서 throttling 관련](http://www.django-rest-framework.org/api-guide/throttling/)

* 강사님 조언 : throtting은 token auth duration customizing 관련해서 사용하는 기능이 아니다. 스택오버플로우에 나와있는 것 처럼 해야하는데 앱을 하나 새로 만들어서 토큰 데이터베이스에 로직을 통하여 특정순간의 시간을 기록하고 해당 시간을 기준으로 해서 인증 만료 기한을 설정할 수 있다. 설정하기 까다로운 점들이 있기 때문에 extra 기능으로 생각해서 진행하는게 좋다.

**알아봐야 할 것들**

* corsheaders : 강의 때 했던 프로젝트에 보면 installed apps에 corsheaders가 추가 되어있는데 어떨 때 사용하는거고 우리 프로젝트에 필요한건지 확인해보기

**팀원들의 의견을 들어봐야 할 것들**

* post list의 pagination

**강사님 동영상에 settings.py 에 LOGIN_URL = 'member:login' 기록한 이유**

The URL where requests are redirected for login, especially when using the [`login_required()`](https://docs.djangoproject.com/en/1.11/topics/auth/default/#django.contrib.auth.decorators.login_required)decorator.

**Post 관련 기록**

* author를 username으로 바꾸는 방법 관련 링크
  * http://www.django-rest-framework.org/api-guide/relations/#reverse-relations
  * http://www.django-rest-framework.org/api-guide/relations/#slugrelatedfield

  > SerializerMethod 로 해결

* 클래스명과 api를 좀더 명시적으로 refactoring 할 필요 있음

* user 정보와 연동하는 부분 확인하기

* post-reply에서 post 요청시에 foreign key로 연결된 post의 pk와, author의 pk를 각각 직업 입력해 줘야 한다.
  * 언뜻 생각한 솔루션으로는 serializer에서 request의 author 정보를 받아와서 저장 하면 될 것 같다는 생각...
  * 리플을 작성할 때 작성하는 곳의 포스트를 default로 지정해 놓을 수만 있다면 쉽게 해결 가능한데, 지금으로선 backend에서 해결 할 수 있는 문제가 아닌 것 같다.

* post-text-create 에서 post요청시 authorization 인증 요구를 하지 않는 문제, 또한 위와 같이 해당 post의 pk값을 직접 입력해 줘야 한다.

* post-path-create 에서 post요청시 authorization 인증 요구를 하지 않는 문제,  해당 post의 pk값을 직접 입력해 줘야 한다.

* 사진 크기 한도 적어줘야함

**Post api body입력정보**

* post-create: 입력정보 / authorization 필요
  * author(user의 pk값), title, start_date(반드시 `2017-11-28T02:01:00+09:00`형식이어야 함)
* post-reply: 입력정보 / post 요청시 authorization 필요 / get 요청시 전체 리스트 반환
  * post: post의pk값, author: author의pk값, content
* post-reply-update:
  * 포스트에 상관없이 리플의 pk값을 기준으로 해당 리플의 내용을 보여준다.
  * delete 요청을 통해 해당 리플 삭제
* post-text-create / post 요청시 authorization 요구안함
  * title, content, post(해당 포스트의 pk 값을 입력해줘야 함)
* post-text-detail
  * text의 pk값을 기준으로 해당 text 불러옴
  * delete 요청을 통해 해당 text 삭제
* post-path-create / post 요청시 authorization 요구안함
  * lat(위도, float), lng(경도, float), post
  * post 에 pk값 입력해 줘야함




## 11-29 수요일

**팀원들의 의견을 들어봐야 할 것들(어제꺼에 이어서)**

- post list의 pagination
- 사진 한번에 올릴 수 있는 갯수(나는 최대 10장정도 생각하고 있음)
- 올리사진을 한번에 묶음으로 보여주는 걸 앱쪽에서 해줄 수는 없는지


**오늘 작업한 내용**

* post list

  *  author를 pk값이 아닌 username으로 변경

* post create

  * author에 값을 입력하지 않아도 자동으로 usename이 추가될 수 있도록 변경

* post detail

  * post의 author 값 -> username으로 보이게끔 변경

* post reply (create reply)

  * permission classes 추가
  * post 요청시 author pk를 입력하지 않아도 인증된 username이 나오게끔 변경

  > 리플 post 요청시에는 어떤 post와 연결되어 있는 정보인지 앱쪽에서 정보를 넘겨줘야 한다.

* post-text-create

  * author정보를 확인하는 로직은 없다.


**기록**

post text에 author permission 추가하는거 시도할 때 썼던 코드.

```python
class PostTextSerializer(serializers.ModelSerializer):
    author = serializers.SerializerMethodField()

    class Meta:
        model = PostText
        fields = (
            'pk',
            'title',
            'content',
            'created_at',
            'post',
            'author'
        )

    def get_author(self, obj):
        post_pk = obj.post.pk
        post_qs = Post.objects.get(pk=post_pk).author
        author = UserSerializer(post_qs,).data
        return author

```

validate(self, data) 메서드에서 date를 print 해보면`OrderedDict([('title', 'text test1'), ('content', 'test11111'), ('post', <Post: Post object>)])`이렇게 나오고 author 필드가 나오지 않아서, post의 author와 text의 author를 비교할 수 없다.

```python
    def validate(self, data):
        print(data)
        post_author = data['post']
        text_author = self.context['request.user']
        # text_author = data['author']
        if post_author == text_author:
            return data
        raise serializers.ValidationError('해당 post의 작성자만 글을 작성할 수 있습니다.')
        print(data)
```

위의 내용을 아래의 방법으로 실마리를 찾아가는중, 현재로선 if 논리가 전혀 먹히지 않음

```python
        def validate(self, data):
        print(data)
        post = data['post']
        post_author = post.author
        request_user = self.context['request']

        print(type(post_author))
        print(type(request_user.user))
        print(post_author)
        print(request_user.user)

        if str(post_author) == str(request_user):
            return data
        else:
            raise serializers.ValidationError()
```

위의 오류 발생시 프린트 되는 내용

```python
OrderedDict([('title', 'text test1'), ('content', 'test11111'), ('post', <Post: Post object>)])
<class 'member.models.User'>
<class 'member.models.User'>
shz@gmail.com
shz@gmail.com

```



**이슈**

* request user가 포스트의 author가 아니면 해당 포스트가 가지고 있는 text를 작성하지 못하도록 막고 싶은데 잘 안된다.
* 모델링에 문제가 있음을 발견 -> 해결방법으로는 사진 올리기 버튼을 누르면 photo item 테이블을 자동으로 생성하게 하고 다른 url에서 해당 photo item필드와 foreignkey로 연결해서 사진을 upload한다.


크리에잇 테스트부터 시작하면 된다.



## 11-30 목요일

**작업내용 (model 리모델링)**

* post create
* post list
* subpost create



**회의**

* password - 하나만 받기
* 구성을  User 안에 토큰값 넣기



## 12-3 일요일

**user 가 작성한 post list 가져오기 성공 apis.py 기록**

```python
class GetUserProfile(APIView):
    def get(self, request, *args, **kwargs):
        user = request.user
        posts = user.post_set.all()
        return_data = []
        print(posts)
        for post in posts:
            serializer = PostSerializer(post)
            return_data.append(serializer.data)
        return Response(return_data)

```

**Tocken auth logout stackoverflow answer**

https://stackoverflow.com/questions/30739352/django-rest-framework-token-authentication-logout

**날씨정보를 가져와서 보여주려고 한다면 다음의 블로그의 3번 을 다시 읽어보자**

http://swchoi06.tistory.com/entry/%EC%93%B8%EB%A7%8C%ED%95%9C-%EB%82%A0%EC%94%A8-API



## 12-4 월요일

**팔로잉/팔로우 기능 관련 인스타그램 예시**

```python
In [3]: u1 = User.objects.create(username='석헌주', age=30)

In [4]: u2 = User.objects.create(username='이한영', age=30)

In [5]: u3 = User.objects.create(username='이경', age=30)

In [6]: u4 = User.objects.create(username='한재중', age=30)


In [7]: Relation.objects.create(from_user=u1, to_user=u2)
Out[7]: <Relation: Relation (from: 석헌주, to: 이한영)>


In [17]: u1.followers.all()
Out[17]: <QuerySet []>

In [19]: u2.followers.all()
Out[19]: <QuerySet [<User: 석헌주>]>


In [23]: Relation.objects.create(from_user=u2, to_user=u1)
Out[23]: <Relation: Relation (from: 이한영, to: 석헌주)>


In [24]: u1.following_user_relations.all()
Out[24]: <QuerySet [<Relation: Relation (from: 석헌주, to: 이한영)>, <Relation: Relation (from: 석헌주, to: 이경)>]>

In [25]: u1.follower_relations.all()
Out[25]: <QuerySet [<Relation: Relation (from: 이한영, to: 석헌주)>]>


In [26]: u1.following_user.all()
Out[26]: <QuerySet [<User: 이한영>, <User: 이경>]>

In [27]: u1.followers.all()
Out[27]: <QuerySet [<User: 이한영>]>



```



## 12-5 화요일

**오늘 작업할 것**

* 팔로우/팔로잉 기능 완성하기
  * 모델 만들고 기능확인
  * api로 어떻게 보여줄 것인지 확인
* 이슈사항
  * 자기 자신을 follow 할 수 있는 부분을 막아야함
  * to_user의 pk가 존재하지 않는 user instance 일경우 나오는 에러를 잡아야 한다
    * 받은 pk값(integer)를 활용해 User instance인지 확인할 수 있는 방법에 관한 구글 서치 내용




**새로 알게 된점**

**serializer context**

serializer 클래스에서 request 정보를 가져 오려면 아래 와 같이 context를 추가해 준 후에

````
serializer = FollwingSerializer(
            data=data,
            context={'request': request}
        )
````

serializer 클래스 에서 다음고 같이 사용한다.

```
self.context['request'] -> <rest_framework.request.Request object at 0x7fcb9ceb34a8> 반환
self.context['request'].user -> request한 user를 반환
```

**field 타입에서 발생하는 에러메시지 커스터마이징**

[관련답변 stackoverflow](https://stackoverflow.com/questions/26943985/custom-error-messages-in-django-rest-framework-serializer) -> 실패함



## 12-6 수요일

**deploy 할 때 internal server error 해결한 기록**

uwsgi 에러 로그

```
*** Starting uWSGI 2.0.15 (64bit) on [Wed Dec  6 06:03:43 2017] ***
compiled with version: 5.4.0 20160609 on 31 October 2017 07:17:38
os: Linux-4.10.0-38-generic #42~16.04.1-Ubuntu SMP Tue Oct 10 16:32:20 UTC 2017
nodename: b24083e148e0
machine: x86_64
clock source: unix
detected number of CPU cores: 4
current working directory: /srv/app/explog
writing pidfile to /tmp/app.pid
detected binary path: /root/.pyenv/versions/3.6.2/envs/app/bin/uwsgi
!!! no internal routing support, rebuild with pcre support !!!
uWSGI running as root, you can use --uid/--gid/--chroot options
*** WARNING: you are running uWSGI as root !!! (use the --uid flag) ***
chdir() to /srv/app/explog
your memory page size is 4096 bytes
detected max file descriptor number: 1048576
lock engine: pthread robust mutexes
thunder lock: disabled (you can enable it with --thunder-lock)
uwsgi socket 0 bound to UNIX address /tmp/app.sock fd 3
Python version: 3.6.2 (default, Oct 31 2017, 07:04:27)  [GCC 5.4.0 20160609]
PEP 405 virtualenv detected: /root/.pyenv/versions/app
Set PythonHome to /root/.pyenv/versions/app
Python main interpreter initialized at 0x22efbe0
python threads support enabled
your server socket listen backlog is limited to 100 connections
your mercy for graceful operations on workers is 60 seconds
mapped 145536 bytes (142 KB) for 1 cores
*** Operational MODE: single process ***
Traceback (most recent call last):
  File "/root/.pyenv/versions/app/lib/python3.6/site-packages/django/apps/config.py", line 111, in create
    entry = module.default_app_config
AttributeError: module 'post' has no attribute 'default_app_config'

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "./config/wsgi/__init__.py", line 1, in <module>
    from .local import *
  File "./config/wsgi/local.py", line 16, in <module>
    application = get_wsgi_application()
  File "/root/.pyenv/versions/app/lib/python3.6/site-packages/django/core/wsgi.py", line 13, in get_wsgi_application
    django.setup(set_prefix=False)
  File "/root/.pyenv/versions/app/lib/python3.6/site-packages/django/__init__.py", line 27, in setup
    apps.populate(settings.INSTALLED_APPS)
  File "/root/.pyenv/versions/app/lib/python3.6/site-packages/django/apps/registry.py", line 85, in populate
    app_config = AppConfig.create(entry)
  File "/root/.pyenv/versions/app/lib/python3.6/site-packages/django/apps/config.py", line 114, in create
    return cls(entry, module)
  File "/root/.pyenv/versions/app/lib/python3.6/site-packages/django/apps/config.py", line 44, in __init__
    self.path = self._path_from_module(app_module)
  File "/root/.pyenv/versions/app/lib/python3.6/site-packages/django/apps/config.py", line 77, in _path_from_module
    "with a 'path' class attribute." % (module, paths))
django.core.exceptions.ImproperlyConfigured: The app module <module 'post' (namespace)> has multiple filesystem locations (['/srv/app/explog/post', './post']); you must configure this app with an AppConfig subclass with a 'path' class attribute.
unable to load app 0 (mountpoint='') (callable not found or import error)
*** no app loaded. going in full dynamic mode ***
spawned uWSGI master process (pid: 14)
spawned uWSGI worker 1 (pid: 22, cores: 1)
subprocess 15 exited with code 0
--- no python application found, check your startup logs for errors ---
[pid: 22|app: -1|req: -1/1] 172.17.0.1 () {44 vars in 749 bytes} [Wed Dec  6 15:03:58 2017] POST /member/login/ => generated 21 bytes in 0 msecs (HTTP/1.1 500) 2 headers in 83 bytes (0 switches on core 0)
--- no python application found, check your startup logs for errors ---

```

해결방법

```
강복이꺼를 merge시킨 이후에 추가된 각 모듈에 __init__.py 가 없어서 생긴 문제
__init.py__를 추가한 이후에 작동함
```

참고한 stackoverflow 링크

https://stackoverflow.com/questions/28205264/django-app-improperly-configured-the-app-module-has-multiple-filesystem-locati

더 알게된 점


> uwsgi 에러 로그를 확인할 때 가장 아랫쪽에 나온 요청처리관련 에러로그를 포함하여 윗쪽에 상세하게 적혀있는 에러로그를 확인하면 좀더 수월하게 에러내용을 확인할 수 있다.




**프로필 페이지에 해야할 작업 목록**

1. 사진
2. 팔로워 팔로잉 수
3. 세팅
4. 포스트, 라이크, 드래프트
5. 4번에 따른 화면

위의 내용을 바탕으로 내가 해야할 것들

1. 사진 -> user 프로필사진 보여주는 로직
2. 팔로워 팔로잉 수 -> 한번에 정보 가져올 수 있게 처리
3. 포스트, 라이크, 드래프트 -> 해당유져가 좋아요 누른 포스트리스트, 본인이 작성한 포스트리스트



1. user 정보를 받아오는 url (사진, username, email, 팔로잉숫자/팔로워숫자 포함)
2. 본인이 작성한 포스트리스트를 보여주는 url
3. 해당 user가 좋아요 누른 포스트 리스트 보여주는 url


위의 세가지를 한번에 보여주는 url 로직 구현해봐야 함

```
user = {
유저 정보
}
post = {
내가 작성한 post 정보
}
like_post = {
좋아요 누른 post 정보
}
```



1. 내가 팔로우한 사람 리스트 보여주는 url
2. 나를 팔로우한 사람 리스트 보여주는 url




## 12-7 목요일

**오늘 할 일**

1. 프로필 url 구조 확정되면 프로필 기능 만들기
   * url 구조 확정되면 바로 만들기
2. 언팔기능 만들기
   * 만들어진 url로 다시 요청을 하면 언팔을 하도록 할지 아니면 url 하나를 더 만들어서 언팔기능을 구현할지 상의 후 진행
3. user 정보 수정 기능 만들기

**언팔 관련해서 알아둘 점**

`user.following_users.first().delete()` 이런식으로 delete를 하게 되면 `user.following_users.first()`명령이 User object를 불러오기 때문에 user를 삭제 하게 된다. 왜냐하면 `following_users`가 User테이블에 정의된 field이기 때문이다.

관계를 삭제하기 위해서는 m2m으로 연결괸 intermediate테이블의 필드를 불러와야 한다. 따라서 `user.following_user_relations.first().delete()`와 같은 방식으로 언팔로우 기능을 만들어 낼 수 있다.



**User password 변경과 관련된 참고 자료**

https://stackoverflow.com/questions/38845051/how-to-update-user-password-in-django-rest-framework



## 12-8 금요일

**이미지파일 변경시 기존에 있던 사진 삭제하는 방법 관련 유용한 답변**

https://stackoverflow.com/questions/16041232/django-delete-filefield

**이미지파일 용량제한 걸어둘 수 있는 방법에 관한 유용한 답변**

https://stackoverflow.com/questions/38842053/how-to-check-the-file-size-limit-in-django

https://www.google.co.kr/search?newwindow=1&ei=s5UqWsq6D8r_8QXpoIzgAw&q=django+size+image&oq=django+_size&gs_l=psy-ab.3.4.0i19k1l2j0i30i19k1l6j0i8i30i19k1l2.285284.296298.0.302801.15.13.2.0.0.0.261.1619.0j8j1.9.0....0...1c.1j4.64.psy-ab..5.8.1281...0j0i67k1j0i13i30k1j0i13k1.0.3YSiz1jgxvU

**오늘 할 일**

1. password 변경하는거까지 만들고 api 작성, 배포 -> ** 테스트를 해봐야 하지만 거의 완성 **
2. 팔로잉 팔로워 리스트 만들기
3. 언팔기능을 기존 팔로잉 기능에 추가



1. 프로필 url 구조 확정되면 프로필 기능 만들기
   - url 구조 확정되면 바로 만들기
2. 언팔기능 만들기
   - 만들어진 url로 다시 요청을 하면 언팔을 하도록 할지 아니면 url 하나를 더 만들어서 언팔기능을 구현할지 상의 후 진행
3. user 정보 수정 기능 만들기



**하면서 새로 알게된 점**

serializer에서 field validation을 사용했는데 validation을 통과하지 못했을 경우 return으로 serializer.VAlidationErrors를 돌려주면 필드에 오류내용이 들어간 체로 is_valid를 통과하여 해당 데이터가 아래와 같이 serializer.data에 담겨진다. return 으로 어떤 값을 돌려주게 되면 해당 필드에 그 내용이 그대로 전달된다.

(아래는 return으로 했을 때의 response값)

```
{
    "old_password": "['잘못된 비밀번호 입니다.']",
    "new_password": "1111"
}
```

is_valid 를 통과하지 못하게 하고 관련 에러를 serializer.errors에 담아서 보내주기 위해서는 raise를 사용해야 한다. raise는 그야말로 에러를 띄워주는 그런 역할을 하는 것 같다.

(아래는 raise로 했을 때의 response값)

```
{
    "old_password": [
        "잘못된 비밀번호 입니다."
    ]
}
```



## 12-11 월요일

**오늘 할일**

1. following, user profile, password, api 새로 적은거 올리기
2. following/follower list api 문서 작성 -> user 정보를 몽땅 주는걸로 수정 한 내용 적용해서 작성
3. api 문서에 token 값 header에 넣는 정보 같이 적어주기
4. user가 작성한 post list
5. 현재 접속한 user가 해당 포스트에 좋아요를 눌렀는지 아닌지 확인하기



## 12-12 화요일

**psot 테스트 중 기록**

1. post delete update 에서
   * 해당 post pk값이 없을 경우 500 서버에러
2. post reply create 에서
   * 해당 post pk 값이 없을 경우 500 서버에러
3. post text api에서
   * 해당 text pk값이 없을 경우 500 서버에러
4. post path api에서
   * 해당 path pk값이 없을 경우 500 서버에러
5. post photo api에서
   - 해당 photo pk값이 없을 경우 500 서버에러
6. post like api에서
   - 해당 post pk값이 없을 경우 500 서버에러




**오늘 할일**

1. user image profile default image 적용시키기
2. 로그인 할 때 server error 나는 부분확인해서 처리하기 -> 수정완료


**이슈사항**

1. default image를 설정하려고 했는데 s3를 함께 사용하는 상테에서 어떻게 해야하는지 잘 모르겠음
   * 도움이 될 법한 url
     * 모델에 메서드를 작성해서 파일사이즈와, url 설정을 건드리는 것
       https://djangotricks.blogspot.kr/2013/12/how-to-store-your-media-files-in-amazon.html
     * access denied 에 대한 stackoverflow 의 답변
       https://stackoverflow.com/questions/13167493/access-denied-error-with-amazon-s3
     * 기본적으로 default 이미지 설정에 관련한 stackoverflow 의 답변
       https://stackoverflow.com/questions/6351636/provide-a-default-profile-picture-if-ever-empty



## 12-13 수요일

**오늘 할일**

1. user image profile default image 적용시키기
   * 트러블 슈팅
     filefield, fieldfile, storage 를 확인해 보고 강사님 코드 이해하고 사용해보기
     ​
2. image file 용량 제한 -> 완성


**이슈사항**

1. 에러코드를 추가하는 부분에 있어서 exception을 custom 하는 부분이 많아서 어려움이 있음



**알게 된것**

1. 장고 문서에서 확인한 것은 아니지만 stackoverflow 에서의 답변에서 유추해 보면

   ```python
   megabyte_limit = 2.0
               if filesize > megabyte_limit*1024*1024:
                   raise ValidationError("Max file size is %sMB" % str(megabyte_limit))
   ```

   1024를 2번 곱한 값이 MB 단위가 되는것으로 보아 기본 debug에서 int로 표시된 값을 1바이트로 유추해 볼 수 있다.

   (bit < byte < KB < MB <GB) 8bit = 1byte, 1024byte = 1KB, 나머지도 1024를 곱하면 단위가 올라간다.




## 12-19 화요일

**금일 이슈사항**

1. user profile 관련
   * user가 작성한 post 목록 보여줄 때 post list 구조랑 똑같이 보여달라는 요청이 있었음 -> 행아웃 미팅 후 처리
     author 정보만 빼고 나머지 정보는 post list 정보와 똑같이 보여주는 걸로 작업
   * 다른사람이 자신의 user profile을 볼 수 있도록 설정 -> pk값으로 다른사람의 profile을 볼 수 있는 url을 하나더 만드는 방법으로 진행
2. nginx upload 용량 설정 관련 `.ebextentions` 적용
   잘 적용이 안되었는데 수정한 내용으로 다음 배포 때 잘 되나 다시 확인


**강사님께 물어볼 사항**

* 혹시 git merge시에 indent가 흩으러져 있는 경우가 있었는지
* 사진파일 용량 제한 관련해서 nginx 설정을 ebextentsions에


**금일 작업내용**

1. nginx 용량제한 관련 수정 및 배포
2. token 확인하는 url 기능 api 문서 작성
3. userprofile 에서 author를 제외한 나머지 정보를 post list와 같은 구조로 보여주기
4. user profile update 오류 해결, 및 profile update, password update, following api 에러 response refactoring





## 12-20 수요일

**금일 작업내용**

1. 다른 사람의 userprofile 정보도 볼 수 있도록 url 추가 및 api 문서 작성 -> userprofile api 문서 update 요청 해야함
   -> 완료
2. user profile 에서 response정보중 posts 부분을 post list와 완전하게 같게 만들어서 적용, api 문서 업데이트 -> response 정보를 post list랑 완전히 똑같이 받을 수 있도록 수정함
   -> 완료
3. like psot api의 response 관련 수정요청 -> 내가 수정하고 변경내용 강복이한테 메시지 남겨놓음
   -> 완료

**이슈사항**

1. user following 정보를 어떤식으로 보여줘야 할 지 앞쪽과 협의후에 진행해야 함


## 12-21 목요일

금일 작업내용

1. 배포작업
   * 바뀐 내용
     1. text, photo, path 에서 response시에 보여주는 데이터가 좀 달라짐
     2. post like에서 좋아요 누른 user도 다 보여주는 형태로 바뀜
     3. 다른 사람의 userprofile 정보도 볼 수 있도록 url 추가
     4. user profile 에서 response정보중 posts 부분을 post list와 완전하게 같게 만들어서 적용




## 12-27 수요일

**회의**

* user profile 쪽 post 의 순서를 최신작성 글이 가장 위에 올라갈 수 있도록 -> order
* user profile 쪽 post pagination 알아보기 -> 보류하고 순서만 수정
* user profile 쪽 img_profile 에 default image가 나올 수 있도록
* 좋아요에 푸시알람 주기


**꿀팁**

`mindnode` 라는 마인드맵을 그려주는 프로그램이 있다.

대체제로 `FreeMind`라는 프로그램이 있다.

**금일 작업내용**

1. user profile post의 순서를 최신작성 들이 위에 올라갈 수 있도록 ordering




## 12- 28 목요일

**배포전 변경사항 정리**

1. post 에서 title max_length 수정 (30 -> 60)
2. created_at `2017-12-12T13:15:32.909751+09:00`에서 ->  `17-12-12-24:00`
3. post/postcontent/postreply api 에서 delete시 respone 오는 걸로 추가
4. post text 에 type 추가
5. user profile 쪽에 post list랑 liked posts를 최신순으로 정렬



**이슈사항**

배포시 ebextensions에 nginx 관련 설정을 해줬는데 강사님께 받은 소스와 , 스택오버플로우에서 알아낸 소스 둘다 502 bad gateway 에러를 뿜었다. 관련 error log 는 다음과 같다

```
2017/12/28 11:25:53 [emerg] 28017#0: "user" directive is not allowed here in /etc/nginx/sites-enabled/elasticbeanstalk-nginx-docker-proxy.conf:1
```

해당 폴더에 들어가서 elasticbeanstalk-nginx-docker-proxy.conf를 직접 삭제하는 조치를 취해봤지만 같은 에러 반복 및 nginx가 잘 돌아가는지도 확인이 안되었다.



강사님께 받은 소스

```
user                    nginx;
error_log               /var/log/nginx/error.log warn;
pid                     /var/run/nginx.pid;
worker_processes        auto;
worker_rlimit_nofile    33193;

events {
   worker_connections  1024;
}

http {
   include       /etc/nginx/mime.types;
   default_type  application/octet-stream;

   log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                     '$status $body_bytes_sent "$http_referer" '
                     '"$http_user_agent" "$http_x_forwarded_for"';

   include       conf.d/*.conf;

   map $http_upgrade $connection_upgrade {
       default     "upgrade";
   }

   server {
       listen        80 default_server;
       access_log    /var/log/nginx/access.log main;

       client_header_timeout 60;
       client_body_timeout   60;
       keepalive_timeout     60;
       gzip                  off;
       gzip_comp_level       4;
       gzip_types text/plain text/css application/json application/javascript application/x-javascript text/xml application/xml application/xml+rss text/javascript;

       # Include the Elastic Beanstalk generated locations
       include conf.d/elasticbeanstalk/*.conf;
   }

   client_max_body_size 50M;
}
```

(스택오버플로우에서 찾은 소스)[https://stackoverflow.com/questions/18908426/increasing-client-max-body-size-in-nginx-conf-on-aws-elastic-beanstalk]

```
files:
    "/etc/nginx/conf.d/proxy.conf" :
        mode: "000755"
        owner: root
        group: root
        content: |
           client_max_body_size 5M;
```



## 12-20 토요일

**작업내용**

1. following 에서 response data가 int 형식으로 되도록 수정



## 1-2 화요일

**해야할 일**

1. 다른 사람 user profile 확인하는 url 에서 요청한 사람의 follow following 숫자를 가져오는 문제 확인하고 수정
   -> 수정완료
2. user profile에 img_profile 부분 url로 전달할 수 있도록 수정
   -> 수정완료
3. 변경사항 확인하고 배포후 공지
   -> dhksfy

**배포내용 정리**

1. 다른 사람 user profile 확인하는 url 에서 요청한 사람의 follow following 숫자를 가져오는 문제
   -> 요청한 url의 pk값을 기준으로 follow following 숫자를 가져오도록 수정

2.  user profile에 img_profile 부분 url로 전달할 수 있도록 수정
   -> 수정완료

3. following의 url 에서 response 의 value 값이 integer로 올 수 있도록 수정

   ```json
   {
       "from_user": 1,
       "to_user": 2
   }
   ```

4. post reply create 요청시 응답 다음과 같이 올 수 있또록 수정

   ```json
   {
       "pk": 1,
       "post": 1,
       "author": {
           "pk": 1,
           "username": "zoozoo",
           "email": "shz@gmail.com",
           "img_profile": "/media/user/5.jpg",
           "token": "e9f446816d19ba6a356024ed2c83eba1ebc49ed6"
       },
       "content": "testestes",
       "created_at": "2018-01-02 15:06"
   }
   ```

5. post detail에서 path는 0번째 순서를 가지고 text, photo 는 순서대로 정렬되게끔 수정(다음은 post detail의 예시)

   ```json
   {
       "post_content": [
           {
               "post": 2,
               "order": 3,
               "content_type": "path",
               "content": {
                   "pk": 1,
                   "lat": 111,
                   "lng": 222
               }
           }
       ]
   }
   ```




**행아웃**

이미지를 보내고 두번째 이미지를 보내면 첫번째 이미지가 두번째 이미지로 변경됨



# 01-03 수요일

**elb get 요청 관련 (health check)**

nginx access.log 에 아래와 같이 health check 요청이 몇초 간격으로 요청됨

```
172.31.18.201 - - [03/Jan/2018:03:36:25 +0000] "GET /post/542/ HTTP/1.1" 200 19 "-" "Explog/1.0 (Explog; build:1; iOS 11.2.0) Alamofire/4.6.0"
172.31.12.62 - - [03/Jan/2018:03:36:26 +0000] "GET / HTTP/1.1" 400 37 "-" "ELB-HealthChecker/2.0"
172.31.18.201 - - [03/Jan/2018:03:36:30 +0000] "GET / HTTP/1.1" 400 37 "-" "ELB-HealthChecker/2.0"
172.31.12.62 - - [03/Jan/2018:03:36:41 +0000] "GET / HTTP/1.1" 400 37 "-" "ELB-HealthChecker/2.0"
172.31.18.201 - - [03/Jan/2018:03:36:45 +0000] "GET / HTTP/1.1" 400 37 "-" "ELB-HealthChecker/2.0"
172.31.12.62 - - [03/Jan/2018:03:36:56 +0000] "GET / HTTP/1.1" 400 37 "-" "ELB-HealthChecker/2.0"
172.31.18.201 - - [03/Jan/2018:03:37:00 +0000] "GET / HTTP/1.1" 400 37 "-" "ELB-HealthChecker/2.0"
172.31.12.62 - - [03/Jan/2018:03:37:11 +0000] "GET / HTTP/1.1" 400 37 "-" "ELB-HealthChecker/2.0"
172.31.18.201 - - [03/Jan/2018:03:37:15 +0000] "GET / HTTP/1.1" 400 37 "-" "ELB-HealthChecker/2.0"
172.31.12.62 - - [03/Jan/2018:03:37:26 +0000] "GET / HTTP/1.1" 400 37 "-" "ELB-HealthChecker/2.0"
172.31.18.201 - - [03/Jan/2018:03:37:30 +0000] "GET / HTTP/1.1" 400 37 "-" "ELB-HealthChecker/2.0"
172.31.12.62 - - [03/Jan/2018:03:37:37 +0000] "GET /post/1/list/ HTTP/1.1" 200 985 "-" "Explog/1.0 (Explog; build:1; iOS 11.2.0) Alamofire/4.6.0"
172.31.12.62 - - [03/Jan/2018:03:37:37 +0000] "GET /post/2/list/ HTTP/1.1" 200 1228 "-" "Explog/1.0 (Explog; build:1; iOS 11.2.0) Alamofire/4.6.0"
172.31.12.62 - - [03/Jan/2018:03:37:37 +0000] "GET /post/4/list/ HTTP/1.1" 200 1072 "-" "Explog/1.0 (Explog; build:1; iOS 11.2.0) Alamofire/4.6.0"
172.31.12.62 - - [03/Jan/2018:03:37:37 +0000] "GET /post/3/list/ HTTP/1.1" 200 1054 "-" "Explog/1.0 (Explog; build:1; iOS 11.2.0) Alamofire/4.6.0"
172.31.12.62 - - [03/Jan/2018:03:37:37 +0000] "GET /post/5/list/ HTTP/1.1" 200 966 "-" "Explog/1.0 (Explog; build:1; iOS 11.2.0) Alamofire/4.6.0"
172.31.12.62 - - [03/Jan/2018:03:37:37 +0000] "GET /post/6/list/ HTTP/1.1" 200 1075 "-" "Explog/1.0 (Explog; build:1; iOS 11.2.0) Alamofire/4.6.0"
172.31.12.62 - - [03/Jan/2018:03:37:39 +0000] "GET /member/userprofile/ HTTP/1.1" 200 2841 "-" "Explog/1.0 (Explog; build:1; iOS 11.2.0) Alamofire/4.6.0"
172.31.12.62 - - [03/Jan/2018:03:37:39 +0000] "GET /member/userprofile/ HTTP/1.1" 200 2841 "-" "Explog/1.0 (Explog; build:1; iOS 11.2.0) Alamofire/4.6.0"
172.31.12.62 - - [03/Jan/2018:03:37:41 +0000] "GET / HTTP/1.1" 400 37 "-" "ELB-HealthChecker/2.0"

```

조사후 아래와 같은 검색결과를 얻음

https://github.com/getsentry/sentry/issues/2590 -> nginx 설정

https://serverfault.com/questions/518220/nginx-solution-for-aws-amazon-elb-health-checks-return-200-without-if -> nginx 설정



**배포내용**

1. user profile 에서 response 데이터 중에 user가 작성한 post list, 좋아요 누른 post list 에 각 post img의 default img를 보여줄 수 있게 수정
2. 요청하는 user가 follow 하는 사람들의 post list를 보여주는 url 추가 `/post/follow/`



**push notification**

앱쪽에서 준비해야할 사항들

1. 디바이스에 해당하는 토큰



## 1-5 금요일

**push notification 관련 기록**

패키지의 send_message 메소드를 사용할 때 어떤 key 값을 사용할 수 있는지 내부 함수를 뜯어 봤는데 다음과 같이 확인할 수 있었고 실험 결과 다음 함수의 attribute를 사용하면 된다.

```python
def _apns_prepare(
	token, alert, application_id=None, badge=None, sound=None, category=None,
	content_available=False, action_loc_key=None, loc_key=None, loc_args=[],
	extra={}, mutable_content=False, thread_id=None, url_args=None):
		if action_loc_key or loc_key or loc_args:
			apns2_alert = apns2_payload.PayloadAlert(
				body=alert if alert else {}, body_localized_key=loc_key,
				body_localized_args=loc_args, action_localized_key=action_loc_key)
		else:
			apns2_alert = alert

			if callable(badge):
				badge = badge(token)

		return apns2_payload.Payload(
			apns2_alert, badge, sound, content_available, mutable_content, category,
			url_args, custom=extra, thread_id=thread_id)
```

test로 send_message 사용했을 때 최종적으로 원하던 구조

```
In [46]: test.send_message(message={"title":"Game Request", "body": "Bob wants to play poker"}, sound="chime.aiff", extra={"user" : "user.img_profile"} , badge=1)
```

푸시는 이미지

디바이스토큰 세팅하는 시점 로그인, 회원가입할 때

알람 나가는 시점 좋아요



## 1-6 토요일

**이슈사항**

1. device token을 저장할 경우 잘 저장되었다는 response, 혹은 잘 저장되지 않았다는 response를 돌려줘야 할지... -> 만약에 돌려준다고 하면 기존의 response 에서 하나가 추가되기 때문에 미리 같이 이야기를 해야만 한다.
   여러가지 선택사항중에 가장 간단한 방법중의 하나는 token 정보만 넘겨주고 (device-token 을 key값으로 하고 token 값을 value로 해서 데이터를 넘겨준다) response는 지금과 같이 그대로 받는 것

**기록**

민준님께 받은 토큰 값`19bfe0b5d0363186cbf5482225d341701c7f53f1ef67186537e6e88a1d4e7ea7`



**금일 작업**

1. ios 기기의 device token을 받아서 저장 할 수 있는 기능 추가
   -> login, sign up 시에 `device-token`이라는 key값을 추가하면 해당 key값을 활용해 해당 user와 관련된 APNSdevice 객체를 저장한다. 현재로써는 각 user는 하나의 device token만 가짐
2. 좋아요 버튼 누르면 날아가는 로직까지 만들어 놓음


**남은 작업**

1. pem 파일을 비밀번호 없어도 사용할 수 있게끔 만들어야 함
   (shell 에서 비밀번호 입력해야 할 경우 python에서 어떻게 처리할 수 있는지도 한번 알아보면 좋을 것 같다)


## 1-7 일요일

**기록**

push notification 작업을 하는중 pem 파일의 비밀번호 제거 하는 명령을 한 뒤에  다음과 같은 에러 발생

```
SSLError: [SSL] PEM lib (_ssl.c:3281)
```

에러 내용은 검색해도 잘 안나오고 python 에서 혹시 자동으로 비밀번호를 전달하게 해서 문제를 해결 하려 하다가 다음 stackoverflow 답변 발견

https://stackoverflow.com/questions/10069351/enter-pem-pass-phrase-just-once#

위의 답변에 나온 방법으로 문제 해결함



**이슈사항**

1. push notification에 대한 정보 기록이 필요함 -> 좋아요 누른사람 정보, 어떤포스트 인지
2. badge정보를 저장할 공간이 필요함 badge 를 저장하고 해당정보를 보내줘야함




## 1-8 월요일

**금일 작업**

1. badge 정보를 쌓아나갈 수 있는 model과 model method만들고 api에 적용
   -> 완료
2. badge 정보 0으로 reset 할 수 있는 api 완성
   -> 완료
3. 내 post에 좋아요 누른사람들의 리스트를 받을 수 있는 api
   -> 완료
4. 변경된 내용 배포하고 api문서 작성하기
   update 할 api 문서 : login, sign up, reset-badge, push list, liked post(올바르지 않은 토큰 넣었을 때 error 냄)


**변경사항**

1. login과 sign up 시에 `device-token`이라는 key값을 추가하면 apns device 객체를 생성하도록 함 (response data는 기존과 같다)
2. 올바른 device token이 입력되어 있다면 다른 누군가가 내가 작성한 post에 좋아요 버튼을 눌렀을 경우 push notification이 날라오고, badge값은 좋아요가 눌러질 때마다 1씩 추가되도록 setting 되어 있음
    또한 push notifications에 서버에 저장되어 있는 badge값이 포함되어 날라감
3. user의 badge값을 0으로 초기화 할 수 있는 url 추가. `/push/reset-badge/`라는 url로 로그인한 user가 get 요청을 보내면 자신의 badge값을 0으로 초기화함
4. 자신의 post에 좋아요 누른 목록 보여주는 url 추가. `/push/list/`라는 url로 로그인한 user가 get 요청을 보내면 자신이 작성한 post에 좋아요를 누른 정보가 최신정보 순으로 정렬되어 나타남. pagination은 6

현재로선 push notification은 ios 기준으로만 되어 있습니다.

api 문서는 작성중입니다.



## 1-9 화요일

**작업내용**

1. 배포후 발생한 문제 해결
   device token은 unique true 라서 같은 디바이스 토큰이 들어가지 못함
   -> 중복된 device token값을 입력하게되면 아래와 같은 오류를 발생하게 만듦

   ```
   {
       "device-token": "중복된 토큰값"
   }
   ```

2. ​




**프로젝트 마무리 해야할 것들**

1. elb health check 관련 해결
2. nginx 사진용량 사이즈 조정



**이슈사항**

1. 뱃지 넘버를 세팅할 수 있는 api  `/push/set-badge/`

   ->
   인증된 user가 patch요청,
   key 값으로 `badge` value값으로 `integer` 보내면 요청한 user의 badge정보 세팅되고 아래와 같이 response

   ```json
   {
       "badge": 2
   }
   ```

2. 디바이스토큰 세팅 할 수 있는 api `/push/device-token/`

   ->
   인증된 user가 patch요청,
   key 값으로 `device-token` value값으로 `디바이스 토큰값` 보내면 요청한 user의 device token값이 보낸 요청으로 세팅되고 아래와 같이 response

   ```json
   {
       "registration_id": "qwer"
   }

   ```

   `registration_id` 에 해당되는 `qwer`이 토큰값

3. login, sign up 시에 response 로 device token 값 response로 보여주기
   apnsdevice_set 안에 있는 registrations_id 의 value값이 토큰

   ```json
   {
       "pk": 1,
       "username": "zoozoo",
       "email": "shz@gmail.com",
       "img_profile": "/media/user/5.jpg",
       "token": "e9f446816d19ba6a356024ed2c83eba1ebc49ed6",
       "apnsdevice_set": [
           {
               "registration_id": "1310df61f0d3b1fc94324600ae5ebf75be6a859278b1e3ebe1110111dae86d4f"
           }
       ]
   }
   ```

   ​
