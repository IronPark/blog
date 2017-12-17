---
title: "Travis Ci 를 이용해서 Hugo 기반 웹사이트 배포 하기"
date: 2017-12-17T18:37:22+09:00
draft: false
categories: ["Development", "Blog"]
tags: ["blog","travis-ci","hugo"]
---
## 정적 웹사이트 생성기의 불편
기본적으로 정적 웹사이트 생성기 ([static site generator](https://www.staticgen.com/)) 를 사용해서 블로그를 운영할때 단점은, 블로그에 대한 소스에 대한 저장소와 최종적으로 생성된 정적 웹사이트의 소스가 저장되는 (github pages) 저장소가 별도로 관리되기 때문에.

`새포스트 작성 > 커밋&푸시(블로그 소스 저장소) > 빌드 > 커밋&푸시(호스팅 저장소)`

위와 같은 워크플로우를 가지게 된다. 간단하지만 매번 하기에는 귀찮은 일이기도 하며 기존의 블로그 서비스(네이버,티스토리,이글루스 등..) 들을 이용할때는 경험하기 힘든? 부분이다.

이번 포스트에서는 [Travis CI](https://travis-ci.org/) 를 이용하여 워크플로우를 다음과 같이 단축 하는 법에 대해서 알아 보자.

`새포스트 작성 > 커밋&푸시(블로그 소스 저장소)`
## Travis CI

[Travis CI](https://travis-ci.org/) 는 지속적인 통합 (Continuous Integration) 위한 서비스 중에 하나로 **오픈소스 프로젝트에 한해서 무료** 로 제공되고 있다.

> 소프트웨어 공학에서, 지속적 통합(continuous integration, CI)은 지속적으로 퀄리티 컨트롤을 적용하는 프로세스를 실행하는 것이다. - 작은 단위의 작업, 빈번한 적용. 지속적인 통합은 모든 개발을 완료한 뒤에 퀄리티 컨트롤을 적용하는 고전적인 방법을 대체하는 방법으로서 소프트웨어의 질적 향상과 소프트웨어를 배포하는데 걸리는 시간을 줄이는데 초점이 맞추어져 있다. 대표적인 CI 툴에는 젠킨스(Jenkins)가 있다. [wikipedia](https://ko.wikipedia.org/wiki/%EC%A7%80%EC%86%8D%EC%A0%81_%ED%86%B5%ED%95%A9)

[Travis CI](https://travis-ci.org/)  로 할 수 있는 일을 간단히 소개하자면. Github에서 개발되고 있는 오픈소스 프로젝트를 해당 프로젝트의 관리자가 [Travis CI](https://travis-ci.org/)  에 연결&세팅을 했을 경우 해당 프로젝트에 커밋이 일어날때마다. 자동으로 빌드, 실행, 테스트를 할 수 있다.

물론 이 기능은 [Travis CI](https://travis-ci.org/) 뿐만 아니라 다양한 CI 툴이 모두 제공하는 것 이지만. [Travis CI](https://travis-ci.org/) 의 경우 별도의 서버에 별도의 툴을 설치하고 세팅하는 과정이 없이 바로 쓸 수 있으며 오픈소스에 한해서는 무료로 쓸 수 있다.

따라서 [Travis CI](https://travis-ci.org/) 를 이용하면 내가 새로운 테마를 적용하거나 새로운 글을 써서 커밋을 했을때 자동으로 빌드 하여 https://github.com/IronPark/ironpark.github.io 로 deploy 를 할 수 있다.

> 사실 정적 웹사이트 생성기중 Jekyll 을 사용하는 경우 별도의 CI툴이 필요 없는데 이는 GitHub Pages 에서 native 로 지원하기 때문이다. Hugo 도 사용자 풀이 굉장히 큰편인데 언젠가 지원해 주었으면 하는 작은 바램이 있다.

## Travis CI + Hugo + Github Pages
### Travis CI 설정
이제 사전 지식을 탑재 했으니 실전으로 들어가 보자! [Travis CI](https://travis-ci.org/) 는 기본적으로 repo 의 root 디렉토리에서 `.travis.yml` 이라는 파일을 설정파일을 읽어 드린다. `.travis.yml` 는 [Travis CI](https://travis-ci.org/) 가 무슨일을 할까? 라는 질문을 했을때 우리가 준비해놓은 답변 이라고 생각 하면 될 듯하다.

``` yaml
language: go
go:
  - master # go 언어의 마지막 버전

install:
  - go get github.com/spf13/hugo #정적 웹 사이트를 빌드하기 위한 hugo 설치

script:
  - hugo # hugo 커맨드를 실행하여 정적 웹사이트를 빌드

deploy:
  local_dir: public # hugo 에서 기본 output폴더 명은 public 임으로 deploy 할 폴더를 public 으로 설정 해 놓는다
  repo: ironpark/ironpark.github.io # 배포될 저장소
  target_branch: master
  provider: pages
  skip_cleanup: true
  github_token: $GITHUB_TOKEN # Travis CI 가 깃허브에 직접 업로드 하기위한 토큰
  email: cjfdhksaos@gmail.com
  name: "ironpark"
  on:
    branch: master
```

설정 파일이 하는 역할은 다음과 같다.

1. github repo를 (여기서는 hugo 기반 블로그 소스 repo) clone 한다. `git clone blog`
2. 작업 영역을 clone 된 레포 안으로 변경 ex) `cd blog`
3. go 언어 설치
4. hugo 설치 ` go get github.com/spf13/hugo`
5. hugo 커맨드 실행 (정적 웹사이트 빌드)
6. 생성된 public 폴더 내부의 파일들을 지정된 repo 로 deploy 한다.

위의 설정 파일은 본 블로그에서 사용하고 있는 설정 파일 임으로 다른 hugo 기반 블로그에서 사용하려면 아래의 변수를 수정할 필요가 있다.

``` yaml
deploy:
  repo: 배포될 github pages 저장소 (###.github.io 이름을 가지는)
  email: 깃허브에서 사용하는 이메일 주소
  name: "커밋될 사용자 이름"
```

> `$GITHUB_TOKEN` 의 경우 `.travis.yml` 에 직접 쓰게 되면 만약 공개 저장소인 경우 보안적인 문제가 생기게 된다 (누군가 자신의 GITHUB_TOKEN 을 취득하여 github 계정에 있는 저장소를 마음대로 보고 쓰고 삭제를 할 수 있다면?)

따라서 `$` 기호를 붙여 환경 변수로 설정 하고 `GITHUB_TOKEN` 을 [Travis CI](https://travis-ci.org/) 의 Dashboard 에서 직접 환경변수로 입력하는 방식을 사용하게 된다.

설정파일을 다 만들었다면. 프로젝트에 포함시켜 자신의 저장소에 포함시켜 커밋&푸시 를 한다.
### Github Token 발급
http://github.com/settings/tokens 에서 `Generate new token` 버튼을 눌러 새로운 토큰을 발급 받는다. 이때 repo 에 대한 권한을 설정해준다.
![github access token scopes](/img/github-token-scopes.png)

### Travis CI 등록

https://travis-ci.org/ 로 접속하여 자신의 github 계정으로 로그인 한뒤
https://travis-ci.org/profile/계정명 으로 이동한다. (혹은 메인 화면에서 왼쪽의 My Repositories 옆의 + 버튼을 누른다.) 이후 `.travis.yml` 을 세팅한 저장소의 스위치를 On 한다.

이는 Travis CI 가 저장소를 지켜보고 커밋이 될때 스크립트를 실행 할 수 있도록 한다.

> 슬프게도 Travis CI 서비스가 나와 함께 살면서 도와주는 AI 비서는 아니기 때문에  시키지도 않은 일을 자동으로 하지는 않는다.

이후 해당 저장소의 settings 페이지에서 Environment Variables (환경변수) 섹션에서 GITHUB_TOKEN 을 입력한다.
![github access token scopes](/img/github-token-travis-ci.png)

## 이제는 기다림뿐!

모든 설정이 끝났으니 이제 Travis CI 가 열심히 일을해서 배포해 주기 때문에, 이제 우리는 귀찮음을 극복하고 열심히 포스팅 하는 일만 남았다!

## 참고 자료들
 - https://www.martinkaptein.com/blog/hugo-with-travis-ci-on-gh-pages/
 - https://blog.outsider.ne.kr/
