# 나만의 뉴스 스탠드 NEWS-COLLECTER 프로젝트를 진행하며 기록해 놓은 자료입니다.


**Custom Homepage Project [www.news-collecter.com](https://www.news-collecter.com/)**
홈페이지로 쓰고자 직접 제작한 웹사이트 입니다.
주기적으로 포탈과 언론사의 메인 기사 제목을 10개정도씩 가져와 한 화면에 보여주는 사이트입니다.



## 참고한 사이트 모음
* [html 레이아웃 관련](https://www.codeproject.com/Articles/546960/HTML-Quick-Start-Web-Application)


* [프로젝트 Github : https://github.com/zooozoo/homepage-project](https://github.com/zooozoo/homepage-project)
* if statement에서 else를 쓰는게 좋은지 아니면 그냥 return 하는게 좋은지에 관한 이야기, 속도나 기능적인 부분에서 차이는 없고
얼마나 읽기 편한지와같은 문제가 고려되는데 else를 쓰기보다는 바로 return하는 방식이 더 좋다는 의견이 많다. 관련 논쟁 스택오버플로우 링크
[https://stackoverflow.com/questions/14844531/is-it-preferable-to-use-an-else-in-python-when-its-not-necessary](https://stackoverflow.com/questions/14844531/is-it-preferable-to-use-an-else-in-python-when-its-not-necessary)
[https://stackoverflow.com/questions/9191388/it-is-more-efficient-to-use-if-return-return-or-if-else-return](https://stackoverflow.com/questions/9191388/it-is-more-efficient-to-use-if-return-return-or-if-else-return)


## 다음 단계

* user과 화면에서 동적으로 언론사 배치순서를 조정할 수 있도록 만들기

* task.py 에서 버전 삭제로직 변경해야함 -> 전체를 삭제하는 것이 아닌 특정 버전 이하는 모두 삭제할 수 있게끔



## 나중에 해야할 일

* header 파트 레이아웃 리팩터링 필요
  -> 부트스트랩 레이아웃 사용해서 해결

* 크롤링한 데이터에서 '문답'과 같은 요소들 없애기
  -> 글자수에 조건을 걸어서 해결

* nav에 news_select_section에서 한글로 언론사이름 보여줄 수 있게 수정
  -> filed에 verbose name 설정하고, index.html의 card header 부분에 field.name을 field.value로 변경해서 해결

* main에 checkbox form에 initial data 전달하는거 더 효율적으로 할 수 있는지 살펴보기
  -> model form 이기 때문에 데이터베이스에서 가져온 모델 인스턴스를 전달함으로써 initiated된 폼을 사용할 수 있음

* 에러가 나면 데이터베이스에 있는 내용을 끌어올 수 있도록 만들기

* news-select-section에 있는 checkbox를 1열 래이아웃으로 바꾸기

* 산발적으로 발생하는 에러 잡기
  에러
  ```
    File "/home/zoozoo/projects/project-hp/my_homepage/main/utils.py", line 230, in khan_news_title
      title=soup.find('div', class_='topNews').a.text,
    AttributeError: 'NoneType' object has no attribute 'a'
  ```



## 개발과정에 대한 기록

### 배포를 위한 기본 세팅
  1. aws계정 생성
  2. IAM 유저와, 보안그룹 생성
  3. aws cli를 활용해 기본정보 등록
  4. local 에서 postgres 연결 확인
  5. settings 를 패키지로 만들고 deploy 와 dev용 setting 모듈 분리
  6. RDS, S3와 연결되고 로컬에서 80번port로 접속 가능한 docker만들기

### docker를 만드는 과정에서 있었던 일
    docker를 만드는 과정에서 basedocker를 새로 구성할 필요가 있어서 간단하게 새 명령어를 집어넣고 새로 buil를 하는데 pyenv를 의존성문제를 해결하기 위한 구성을 다울로드 하는 과정에 에러가 발생했다.

    ```
    E: Failed to fetch http://security.ubuntu.com/ubuntu/pool/main/t/tzdata/tzdata_2016j-0ubuntu0.16.04_all.deb  404  Not Found [IP: 91.189.88.152 80]

    E: Failed to fetch http://archive.ubuntu.com/ubuntu/pool/main/libd/libdrm/libdrm2_2.4.76-1~ubuntu16.04.1_amd64.deb  404  Not Found [IP: 91.189.88.152 80]

    E: Failed to fetch http://security.ubuntu.com/ubuntu/pool/main/i/icu/libicu55_55.1-7ubuntu0.3_amd64.deb  404  Not Found [IP: 91.189.88.152 80]

    E: Failed to fetch http://security.ubuntu.com/ubuntu/pool/main/libx/libxml2/libxml2_2.9.3+dfsg1-1ubuntu0.3_amd64.deb  404  Not Found [IP: 91.189.88.152 80]

    E: Failed to fetch http://security.ubuntu.com/ubuntu/pool/main/c/curl/curl_7.47.0-1ubuntu2.4_amd64.deb  404  Not Found [IP: 91.189.88.152 80]

    E: Failed to fetch http://archive.ubuntu.com/ubuntu/pool/main/libd/libdrm/libdrm-amdgpu1_2.4.76-1~ubuntu16.04.1_amd64.deb  404  Not Found [IP: 91.189.88.152 80]

    E: Failed to fetch http://archive.ubuntu.com/ubuntu/pool/main/libd/libdrm/libdrm-intel1_2.4.76-1~ubuntu16.04.1_amd64.deb  404  Not Found [IP: 91.189.88.152 80]

    E: Failed to fetch http://archive.ubuntu.com/ubuntu/pool/main/libd/libdrm/libdrm-nouveau2_2.4.76-1~ubuntu16.04.1_amd64.deb  404  Not Found [IP: 91.189.88.152 80]

    E: Failed to fetch http://archive.ubuntu.com/ubuntu/pool/main/libd/libdrm/libdrm-radeon1_2.4.76-1~ubuntu16.04.1_amd64.deb  404  Not Found [IP: 91.189.88.152 80]

    E: Failed to fetch http://archive.ubuntu.com/ubuntu/pool/main/m/mesa/libglapi-mesa_17.0.7-0ubuntu0.16.04.2_amd64.deb  404  Not Found [IP: 91.189.88.152 80]

    E: Failed to fetch http://archive.ubuntu.com/ubuntu/pool/main/m/mesa/libgl1-mesa-dri_17.0.7-0ubuntu0.16.04.2_amd64.deb  404  Not Found [IP: 91.189.88.152 80]

    E: Failed to fetch http://archive.ubuntu.com/ubuntu/pool/main/m/mesa/libgl1-mesa-glx_17.0.7-0ubuntu0.16.04.2_amd64.deb  404  Not Found [IP: 91.189.88.152 80]

    E: Failed to fetch http://archive.ubuntu.com/ubuntu/pool/main/o/openssl/libssl-dev_1.0.2g-1ubuntu4.8_amd64.deb  404  Not Found [IP: 91.189.88.152 80]

    E: Failed to fetch http://archive.ubuntu.com/ubuntu/pool/main/o/openssl/libssl-doc_1.0.2g-1ubuntu4.8_all.deb  404  Not Found [IP: 91.189.88.152 80]

    E: Unable to fetch some archives, maybe run apt-get update or try with --fix-missing?
    The command '/bin/sh -c apt-get install -y make build-essential             libssl-dev zlib1g-dev libbz2-dev             libreadline-dev libsqlite3-dev wget             curl llvm libncurses5-dev libncursesw5-dev             xz-utils tk-dev' returned a non-zero code: 100
    ```

    대략 이런 에러였는데 `apt-get update`부분을 살펴보니 이전에 같은 태그명으로 생성했던 적이 있어서 캐시를 사용해 build 하는 것을 확인했다. 캐쉬를 쓰지 않고 생으로 docker를 빌드하기 위한 명령어는 `--no-cache`명령어를 붙여주면 된다.
    [관련 답변내용 link](https://stackoverflow.com/questions/35594987/how-to-force-docker-for-clean-build-of-an-image?utm_medium=organic&utm_source=google_rich_qa&utm_campaign=google_rich_qa)

    설치중에 `debconf: delaying package configuration, since apt-utils is not installed`이런 에러가 있었는데 해당 내용이 [링크](https://github.com/phusion/baseimage-docker/issues/319)에서 논의된 적이 있었고 일단 빌드가 완성되는 것으로 보아 별 문제가 없다고 판단 계속 진행하도록 하겠다. (추후 무슨 문제가 생겼을 경우 해결하기 위한 기록)

### elasticbeanstalk 구축하기

  * 키페어 profile 설정문제

    awscli에서 사용하는 기본 프로파일은 환경변수로 설정되어 있다.
    `export AWS_PROFILE=user2`명령을 통해 환경변수를 설정함으로써 매번 profile을 지정하지 않아도 된다.
    [관련링크](https://docs.aws.amazon.com/ko_kr/cli/latest/userguide/cli-multiple-profiles.html)
  * eb cli 환경에서 키페어 profile 설정문제

    eb cli를 사용 할 경우 추가적으로 `export AWS_EB_PROFILE=user2` 명령을 통해 eb default 환경변수를 설정해 주어야만 한다
    [관련링크](https://docs.aws.amazon.com/ko_kr/elasticbeanstalk/latest/dg/eb-cli3-configuration.html#eb-cli3-profile)

  * elasticbeanstalk에서 최신버전의 docker를 지원하지 않아 발생하는 문제라고 생각 -> docker를 삭제하고 다시 설치하기로 결정

    * 삭제하려고 하는데 docker를 찾지 못함 -> `dpkg -l | grep -i docker` 명령어로 설치된 `docker-ce`확인후에 삭제
      [관련링크](https://askubuntu.com/questions/935569/how-to-completely-uninstall-docker?utm_medium=organic&utm_source=google_rich_qa&utm_campaign=google_rich_qa)
    * 특정버전의 docker를 설치 [관련링크](https://askubuntu.com/questions/935569/how-to-completely-uninstall-docker?utm_medium=organic&utm_source=google_rich_qa&utm_campaign=google_rich_qa)

  * docker를 재설치 해도 같은문제가 발생했고, 문제는 `.gitignore`에 `Dockerfile`이 포함되어 있기 때문이었다. -> `.ebignore`추가

  * 결론적으로는 docker의 버전이 다른게 문제가 되는지는 정확하기 확인하진 못했지만 그래도 일단 버전을 맞추는게 좋을 것으로 판단된다.

### request 한글 깨짐현상
 * Beautifulsoup 에서 깨지는걸로 생각했는데, request모듈에서의 encoding문제였다. 참고한 블로그와 사이트
  [http://sfixer.tistory.com/entry/python3-beautifulsoup-%ED%95%9C%EA%B8%80-%EA%B9%A8%EC%A7%90](http://sfixer.tistory.com/entry/python3-beautifulsoup-%ED%95%9C%EA%B8%80-%EA%B9%A8%EC%A7%90)
  [http://pythonstudy.xyz/python/article/403-%ED%8C%8C%EC%9D%B4%EC%8D%AC-Web-Scraping](http://pythonstudy.xyz/python/article/403-%ED%8C%8C%EC%9D%B4%EC%8D%AC-Web-Scraping)
 * 더 간단한 해결책 발견
  `request.text` -> 값을 `unicode`로 가져옴
  `request.content` -> 값을 `str`형으로 가져옴
  `str`형으로 가져오면 간단하게 문자를 사용할 수 있다.
  참고한 블로그
  [http://pwnbit.kr/85](http://pwnbit.kr/85)

### `requests.exceptions.TooManyRedirects: Exceeded 30 redirects.` 에러 관련

### 문자열에 탭이나, 줄바꿈 문자 제거하는 것
  [stackoverflow 답변](https://stackoverflow.com/questions/10711116/strip-spaces-tabs-newlines-python/10711166#comment25234743_10711116)
  위의 답변들 중에 나는 아래의 방법으로 해결
  ```python
  >>> import re
  >>> myString="I want to Remove all white \t spaces, new lines \n and tabs \t"
  >>> re.sub('\s+',' ',myString)
  'I want to Remove all white spaces, new lines and tabs '
  ```

### `input`태그를 disabled 처럼 보이게 만들고 readonly 처럼 사용하기
  [stackoverflow 답변](https://stackoverflow.com/questions/7743208/making-a-text-input-field-look-disabled-but-act-readonly)처럼 해결 할 수도 있을 것 같다.
  다만 난 답변을 찾다가 django forms의 widget설정에서 readonly 옵션을 넣어봤더니 disabled와 같은 모습으로 템플릿에 나와서 그대로 사용했다.

### crawler_test.py python script에서 django의 model 을 import하여 실행하기 위한 세팅

  django 자체를 import해서 어떤 환경변수를 설정해줘야만 사용할 수 있다.

  아래는 내가 적용했던 세팅

  ```
  import os
  import django

  os.environ.setdefault("DJANGO_SETTINGS_MODULE", "my_homepage.settings")
  django.setup()

  from main.models import NewsTitle
  ```
  [참고한 stackoverflow 답변](https://stackoverflow.com/questions/15048963/alternative-to-the-deprecated-setup-environ-for-one-off-django-scripts?utm_medium=organic&utm_source=google_rich_qa&utm_campaign=google_rich_qa)

### celery 를 활용하여 크롤링한 데이터 저장하기
  1. celery beat를 활용 -> 폐기
    주기적으로 크롤링을 작동시켜서 크롤링한 데이터를 저장하려 했으나 elasticbeanstalk와 docker를 활용하여 배포할 경우 auto scaling시에 문제가 발생할 수 있다. (같은 ec2가 여러개 생길경우 인스턴스 갯수 마다 beat가 따로 동작하기 때문에 크롤링이 중복되는 문제 발생)
    -> 데이터 생성시간을 체크해서 일정시간이 지나지 않았으면 작동하지않게 할 수도 있다. -> que요청과 시간을 확인하는 로직은 항상 들어가야 하기 때문에 delay보다 자원이 더 낭비될 가능성이 크다.
  2. celery delay 활용
    created time field를 활용해 request마다 데이터베이스의 크롤링 데이터가 stale 한 경우 delay로 크롤링작업 실행. concurrency 문제가 발생할 수 있기 때문에 celery task 작동시에 database lock을 사용한다.

### celery docker에서 작동해보기 -> 폐기
  1. default user는 locallhost환경으로만 접속가능하기 때문에 새로운 user를 만들어 줘야 한다.
    `rabbitmqctl add_user username password`

### aws sqs를 활용하여 celery를 사용하기로 함
  celery 실행시 `pycurl`을 설치하라고 함, `pycurl` 을 설치하기 위해선 `sudo apt-get install libcurl4-openssl-dev` 명령어를 통해 `libcurl4-openssl-dev`라는걸 설치해야만 함 설치후 로컬환경에서 잘 작동
  [pycurl 설치관련 stackoverflow 답변](https://stackoverflow.com/questions/23937933/could-not-run-curl-config-errno-2-no-such-file-or-directory-when-installing)

### elasticbeanstalk에서 내 도메인 이름 사용하기
  1. route53 등록전 name서버를 aws로 교체
  [참고 블로그](http://wingsnote.com/57)
  2. route53에서 내 도메인과 elb 도메인 연결하기
  [route 53 설명서](https://docs.aws.amazon.com/ko_kr/elasticbeanstalk/latest/dg/customdomains.html)

### request 수정하기
  경향신문 크롤러에서 요청하면 산발적으로 `page not found`를 response 하는 경우가 발생했다. 이럴경우 기존에 beautifulsoup으로 크롤링 하는 구조에서
  아래와 같은 에러를 뿜게된다.
  에러
  ```
    File "/home/zoozoo/projects/project-hp/my_homepage/main/utils.py", line 230, in khan_news_title
      title=soup.find('div', class_='topNews').a.text,
    AttributeError: 'NoneType' object has no attribute 'a'
  ```
  try, except문으로 해결하려 했으나 조금 더 검색해 보니 request할 때 브라우저에서 확인하는 것과 같은 page를 받기 위해서는 header에 관련 내용을 함께
  요청해야 한다는 것을 알게 되었다.
  [stackoverflow 답변](https://stackoverflow.com/questions/27652543/how-to-use-python-requests-to-fake-a-browser-visit?utm_medium=organic&utm_source=google_rich_qa&utm_campaign=google_rich_qa)
  아래와 같이 request의 header에 관련내용을 추가해 줌으로써 브라우져에서 확인하는 것과 같은 response를 받게 되었다.
  ```python
    user_agent = 'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/41.0.2272.101 Safari/537.36'
    headers = {'User-Agent': user_agent}
    req = requests.get('http://www.khan.co.kr/', headers=headers)
  ```

### elasticbeanstalk single docker 플랫폼에서 cloudwatch logs 사용하여 로그 남기기

  * elasticbeanstalk에서 ec2 인스턴스의 로그를 확인하기 위한 기본 방법은 설명서 나와있다.
    [Elastic Beanstalk 환경의 Amazon EC2 인스턴스의 로그 보기](https://docs.aws.amazon.com/ko_kr/elasticbeanstalk/latest/dg/using-features.logging.html)
  * cloudwatch log를 사용하기 위해서는 2가지 방법중 하나를 선택해야만 한다.
