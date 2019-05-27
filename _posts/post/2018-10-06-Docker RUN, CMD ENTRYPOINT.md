---
title: 파이썬에서 파일 다루기
tag: Doker
excerpt: >-
 Dockerfile 작성시 RUN, CMD, ENTRYPOINT를 잘 몰라 헤맸던 기억이 있어 관련 내용을 간단하게 정리해 보고자 한다.
category: post
---

RUN, CMD, ENTRYPOINT 의 차이점에 대해서 알아보자.

검색을 하면 한글로도 너무나 잘 설명을 해 놓은 곳이 많기에 관련 링크를 남기고 여기서는 간단한 차이점만 알아보자

(물론 더 자세하게 개념을 이해하기 위해서는 Docker의 cache와 layer의 개념을 알아야 하지만 몰라도 어느정도는 이해하고 활용할 수 있다.)

### RUN

이미지를 빌드 할 때 원하는 명령어를 실행 할 수 있고, 실행한 명령은 새로운 레이어를 형성한다.

### CMD

만들어진 이미지를 실행 시킬 때(docker run  할 때) default로 실행되는 명령어다. 이미지를 작동시켰을 때 default로 실행되는 명령어를 지정할 때 사용된다.

### ENTRYPOINT

ENTRYPOINT는 CMD와 같이 만들어진 이미지를 실행 시킬 때 default로 실행되는 명령어다. CMD와 ENTRYPOINT를 함께 작성 할 경우 CMD 명령은 무시되고 CMD에 해당되는 항목은 ENTRYPOINT 명령의 매개변수로 전달된다.

예를 들어 dockerfile에 아래와 같이 작성하면 

```dockerfile
ENTRYPOINT ["echo"]
CMD ["hello"]
```

빌드한 이미지를 실행할 경우 컨테테이너 안에서 아래와 같이 실행 된다.

```bash
$ sudo docker build --tag example .
$ sudo docker run example
hello
```

또 한가지 CMD와의 차이점은 CMD의 경우 docker run 을 실행할 때 커맨드라인에서 전달 된 인자를 실행 시킬 수 있는데 ENTRYPOINT의 경우 Dockerfile에 작성한 ENTRYPOINT의 내용이 반드시 실행되고 docker run 을 실행할 때 커맨드라인에 전달된 인자는 파라미터값으로만 전달된다는 점이다.



참고 (여기에 관련 내용이 자세히 설명되어 있다)

* https://aboullaite.me/dockerfile-run-vs-cmd-vs-entrypoint/

