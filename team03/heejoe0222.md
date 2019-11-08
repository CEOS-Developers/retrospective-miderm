# Veeto 서버 1차 회고
10기 개발자 최희조의 [Veeto](https://github.com/heejoe0222/veeto) 웹서비스 mvp 구현 후 개인 회고

## 구현한 MVP: django API 서버+ React SPA
USER 관련 기능을 제외한 상태에서 액티비티 방 목록을 보고 방에 참여하거나 방을 생성할 수 있음
* 5개의 API 설계 및 구현
* API 명세서 작성
* AWS EC2와 RDS에 ~~일단은~~ 배포

## 배운 점
* __django-Rest-Framework__  
스터디에서 실습을 해봤지만 실제로 프로젝트에서 적용해보는 것은 매우 달랐다. 내가 구현하고 싶은 기능에 맞춰 적절한 APIView를 선택하고, 이를 그대로 또는 변형해서 우리 서비스에 맞게 작성해보면서 이전의 대충 이해하고 넘어갔던 부분(이를테면 seriealizer)을 확실하게 알게 되었다. 몇 개의 api를 작성하는데 매우 많은 시간이 걸렸었는데 앞으로는 더 빨라지지 않을까? :satisfied:하는 기대가 있다...ㅎㅎ 아 그리고 앞으로 작성 시에도 [DRF 공식 도큐먼트](https://www.django-rest-framework.org/api-guide/generic-views/)를 자꾸보고 익숙해지자!
* __PostgreSQL__  
mysql을 주로 사용했어서 postgresql에 익숙해지기까지는 좀 시간이 걸릴 것 같다. DB 접속과 유저 및 테이블 조회 같은 간단한 sql문부터 일일히 찾아보면서 했었지만 점차 익숙해지고 있다. 데이터 관리나 조회 시 pgAdmin4 툴을 같이 쓰고 있다. 아직은 크게 불편함을 못 느껴서 한동안은 이걸로 계속 쓰는걸로.
* __AWS EC2와 RDS 연동__  
직접 쓰기 전까지는 RDS를 EC2를 붙여서 사용하는 느낌이라고 생각했는데 RDS 인스턴스를 따로 생성해서 하나의 엔드포인트로 기능한다는 사실을 알게 되었다. EC2에서 psql 명령어로 RDS 호스트 주소로 접속 가능하고, 본 프로젝트에서는 Django의 배포파일 설정에서 db settings HOST를 RDS로 연결함으로써 EC2에서 RDS를 연결하였다.
* __Cross-Origin Resource Sharing(CORS)__  
기존의 HTTP요청에서는 img나 link태그 등으로 다른 호스트의 css나 이미지파일 등의 리소스를 가져오는 것이 가능한데 script태그로 쌓여진 코드에서의 다른 도메인에 대한 요청은 Same-origin policy에 대한 규약으로 인해 접근이 불가하다고 한다. 따라서 django로 돌아가고 있는 api서버와 페이지를 그려주는 react서버는 명목상 아예 다른 서버로 구분되기 때문에 script태그 안에서의 api를 통한 데이터의 접근제어를 위해서는 HTTP 접근제어 규약을 추가해줘야 하는 것! 장고에서 `django-cors-headers` 패키지를 추가해줘서 이를 해결할 수 있다. `settings.py`에서 `INSTALLED_APPS`과 `MIDDLEWARE`에 위 패키지를 추가하고, `CORS_ORIGIN_WHITELIST`항목을 추가하여 script안에서의 리소스 요청을 허용할 도메인 추가해주면 된다.
([참고](https://this-programmer.com/entry/%EA%B0%84%EB%8B%A8%ED%95%9C-react-JS-Django-%EC%96%B4%ED%94%8C%EB%A6%AC%EC%BC%80%EC%9D%B4%EC%85%98-%EB%A7%8C%EB%93%A4%EA%B8%B0))


## 아쉬웠던 점
* api 설계 시 프론트 측면에서 먼저 고려하지 않아 나중에 다시 api를 수정해야되는 일이 발생하였다. 앞으로는 프론트 단계에서의 구조를 먼저 고려한 후 프론트 개발자와 의논을 선행하는 방식으로 설계할 것이다.
* api 명세서의 필요성을 느껴서 스스로 만들었었는데 (찾아볼 생각을 못해서 있는지 몰랐다....:joy:) 은근 보여주는 형식과 구조를 정하는데 시간이 걸려서 앞으로는 drf 라이브러리나 좋은 문서화 도구를 쓰면 좋을 꺼 같다. 만들어보는 과정 자체는 의미있었다.
* mvp 발표 때 배포 후 aws 도메인으로 http API 가져오는 것까지 성공하고 싶었는데 못했던 게 아쉽다.
  * 백 이슈: AWS와 RDS 연동하는 것까지는 비교적 쉽게 한 것 같은데([규주 개발자님의 도움으로](https://www.notion.so/3-AWS-RDS-Django-8ed40066a1004d45a6f651dc610f956c)) 연동 후에 `runserver`가 안되서 계속 삽질의 연속이었다. 원인은 개발 환경과 배포 환경 분리 과정에서 코드를 잘못 입력해서 aws에서 실행 시 `prod.py`가 아닌 `dev.py`로 `manage.py`를 실행하고 있던 것이었다. 급해서 대충 이해하고 코드 바꿔서 적용하기 급급했던 것과 오류의 원인을 매우 늦게 꺠달은 것을 반성하며 manage.py와 settings 폴더 내부의 개발과 배포 환경 분리 및 작동 방식을 완전히 이해하게 되었다.
  * 프론트 이슈: mvp 구현까지 기간이 얼마 안 남은 상태에서 와이어프레임이 확정되었기 때문에 레이아웃을 구현하기 급급했었다. 내가 이 부분에서 리액트나 js 코드를 알아서 api 요청하는 코드에 도움을 줄 수 있었으면 좋았을텐데 그러지 못해서 아쉬웠다. (여러모로 리액트를 조금이라도 공부하자..:cry:)


## 앞으로 무엇을 할지?
__필수적인 기능__
* User 관련 모델 설계 및 API 구현
* 바뀐 UI에 따라 API 수정
* [List of Useful URL Patterns](https://simpleisbetterthancomplex.com/references/2016/10/10/url-patterns.html) 참고해서 api url 정리하자
* AWS S3 연동 for 프로필 사진 관리
* 학생증 인증 기능 (또는 학교 이메일 인증으로..)
* sns 공유를 통해 추천 및 가입 시 포인트 부여하는 기능 (아직 기획 단계에서 확정은 아님)

__추가적인 기능__
* 소셜 로그인
* 채팅방 기능
* 등등
