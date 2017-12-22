---
title: "이미지 생성을 위한 Dockerfile Cheat Sheet"
date: 2017-12-22T22:50:42+09:00
draft: true
---
Docker 이미지를 생성하기위해 기본적으로 알아야할 문법과 사용예를 기술하고 있습니다. 하지만 상세한 설명이 없거나 설명되지 않은 명령어 부족한 부분이 있을 수 있으며 이부분은 당연히 [Docker 공식 문서](https://docs.docker.com/engine/reference/builder/#dockerignore-file) 를 참고하시면 좋습니다.

## .dockerignore 파일
docker 이미지에 포함하지 말아야할 혹은 불필요한 데이터를 정의합니다. 즉 git을 사용할때 .gitignore 와 같은 역할을 수행합니다.

|규칙 |설명|
|---|---|
|`*/temp*` | 루트의 모든 하위 폴더에서 이름이 temp로 시작하는 파일 및 폴더를 제외합니다. ex) /somedir/temporary.txt, /somedir/temp|
|`*/*/temp*`| 루트보다 두 단계 아래에있는 서브 폴더에서 temp로 시작하는 파일 및 디렉토리를 제외합니다. ex) /somedir/subdir/temporary.txt
|`temp?`|루트 디렉토리에서 temp 이름의 한 문자가 더 붙은 파일 및 폴더를 제외합니다 .ex) /tempa, /tempb, /tempc .....

위 사용방법 외에도 모든 명명 규칙은 [Go언어의 match 규칙](https://golang.org/pkg/path/filepath/#Match)을 그대로 사용합니다  와 같은 형식을 사용합니다. (이는 Docker가 Go로 짜여졌기 때문입니다.)

## Dockerfile 작성법
Dockerfile은 Docker 이미지를 작성할때 기본적으로 필요한 파일로 해당 이미지가 컨테이너에 적재되어 정상적으로 동작하기 위한 사용할 파일, 라이브러리, 의존성 등을 모두 입력합니다.

### 주석
주석은 파이썬과 같이 # 을 사용합니다. `#이렇게 주석을 쓰면 됩니다.`

### FROM (필수)
 - `FROM <이미지>`
 - `FROM <이미지>:<태그>`
 - `FROM <이미지>@<digest>`

FROM명령은 새로운 빌드 단계를 초기화하고 기반이될 이미지를 선택합니다.
로컬에 이미지가 없을 경우에는 [Docker Hub](https://hub.docker.com/) 에서 이미지를 다운로드합니다. 예를들어 파이썬3 어플리케이션을 Docker 이미지로 만들고 싶으면 파이썬이 미리 설치되어 있는 이미지를 기반으로 하면 되는데 (OOP에서 상속을 받는것 처럼) 이 경우 `FROM python:3` 를 사용하면 됩니다.

현재 만들어져 있는 공개 이미지를 보고 싶으면 https://hub.docker.com/explore/ 를 참고하면 됩니다. 또한 사설 이미지를 위한 저장소도 클라우드에 구축하여 사용하는것 또한 가능합니다.

### LABEL (옵션)
- `LABEL <키>=<값> <키>=<값> <키>=<값> ...`

KeyValue 형태로 Docker 이미지에 각종 메타데이터를 저장할수 있습니다. 예를 들면 다음과 같이 사용할 수 있습니다.
``` Dockerfile
LABEL maintainer="SvenDowideit@home.org.au"
LABEL version="1.0"
LABEL description="이것은 예제입니다"
```
하지만 Docker 문서에서는 LABEL 명령어가 이미지에 새 레이어를 생성하기 때문에 한번에 여러개의 값을 입력하는것이 효율적이라고 권고 하고있습니다. 위 예시를 아래와 같이
``` Dockerfile
LABEL maintainer="SvenDowideit@home.org.au" \
      version="1.0" \
      description="이것은 예제입니다"
```
물론 LABEL 은 이미지를 생성함에 있어서 당연히 생략가능합니다.

### RUN
 - `RUN <명령어>`
 - `RUN ["<실행 파일>", "<매개 변수1>", "<매개 변수2>"]`

이미지에서 필요한 스크립트 / 명령어를 입력합니다. 예를들어 내가 배포할 서비스가 특정 종속성으로 인해 실행하기전 미리 설치되어야 하는 라이브러리, 프레임워크, 등이 있는 경우 apt-get 과 같은 명령어를 사용하여 설치할 수 있습니다.

즉 Docker 가 이미지를 생성할때 이미지 위에서 RUN 으로 지정된 명령들을 실행하여 얻은 결과 (패키지,라이브러리 설치,등등) 를 포함하여 커밋 됩니다.

> 이때 주의할점은 `RUN [ "echo", "$HOME" ]` 이러한 방식으로 사용했을때 $HOME 은 시스템의 환경변수로 치환될 것 으로 보이지만 쉘로 실행 되는 것이 아니라 직접 실행파일을 호출하는 것 이기 때문에 치환되지 않고 에러를 일으킬 것 입니다. 만약 쉘환경을 이용하고 싶다면 `RUN [ "sh", "-c", "echo $HOME" ]` 이러한 방식으로 사용하는 것이 올바른 사용법 입니다.

### CMD
 - `CMD ["<실행파일>","<매개 변수1>","<매개 변수2>"]`
 - `CMD <명령어> <매개 변수1> <매개 변수2>`

컨테이너가 실행될때 (start | run) 실행할 명령을 입력하면 됩니다. 예를들어 파이썬 어플리케이션을 실행할때는 `CMD ["python3","app.py"]` 와 같이 사용하면 됩니다.

> `RUN`과 동일한 주의사상이 있습니다.

### EXPOSE
 - `EXPOSE <port>`

호스트와 연결될 포트번호를 설정합니다.
### ENV
 - `ENV <key> <value>`
 - `ENV <key>=<value> ...`

환경변수를 설정합니다.
### COPY
 - `COPY <src>... <dest>`
 - `COPY ["<src>",... "<dest>"]`

 로컬의 파일 혹은 폴더를 이미지의 파일시스템에 추가합니다.

### ADD
 - `ADD <src>... <dest>`
 - `ADD ["<src>",... "<dest>"]`

파일 또는 폴더를 원격 파일의 URL 경로 혹은 로컬에서 이미지의 파일 시스템으로 추가합니다.

> COPY 와 ADD 는 기능적으로 매우 유사합니다만. ADD 의 경우 외부 파일을 다운받거나 압축파일은 자동으로 압축을 풀어주는 등의 부가기능이 있습니다. 다만 이러한 부가기능들은 명령어만 보았을때 명확하지 않기 때문에 대부분의 경우 COPY 를 사용하는 것을 권장하고 있습니다.

### 예시
아래는 파이썬 앱/서비스를 Docker 이미지로 만드는 Dockerfile의 간단한 예시입니다.

``` Dockerfile
#python:3 를 Base 이미지로 지정
FROM python:3

#작업 경로 설정
WORKDIR /usr/src/app

# 현재 경로의 requirements.txt 파일을 이미지의 /usr/src/app 경로로 복사
COPY requirements.txt /usr/src/app
# pip명령어를 이용하며 requirements.txt 파일에 기술된 패키지 설치
RUN pip install --no-cache-dir -r requirements.txt

# 현재 경로의 로컬파일을 이미지의 /usr/src/app 경로로 복사
COPY . /usr/src/app
# 호스트와 연결할 포트 번호
EXPOSE 50050
# ENTRYPOINT는 컨테이너가 시작되었을 때 스크립트 혹은 명령을 실행합니다.
# ENTRYPOINT가 있으면 CMD는 ENTRYPOINT에 매개 변수만 전달하는 역할을 합니다.
ENTRYPOINT ["python3"]

# example.py 실행
CMD ["example.py"]
```


## 참고자료
 - https://docs.docker.com/engine/reference/builder/
 - https://github.com/wsargent/docker-cheat-sheet
 - https://subicura.com/2017/01/19/docker-guide-for-beginners-1.html
 - https://sangwook.github.io/2015/01/16/docker-container-network.html
 - https://stackoverflow.com/questions/24958140/what-is-the-difference-between-the-copy-and-add-commands-in-a-dockerfile
