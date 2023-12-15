현재 프로젝트는 MSA를 지향하는 프로젝트이고 각각의 서비스를 독립적으로 다루기 위해서 기능 별로 코드 저장소를 분리했다. 독립적인 것은 좋았으나 관리하기는 불편해서 하나의 공간에서 소스 코드를 관리하기로 하였다
## NPM Workspace
npm workspace를 사용하면 workspace로 지정된 디렉터리가 하나의 패키지로 인식된다. 패키지로 인식된다는 의미는 package.json이 존재한다는 말이 되고 거기에 기록된 의존성들은 root 디렉터리에 node_modules에 전부 설치된다.

결국 root > node_modules에는 모든 모듈들이 들어가게 되고 추가로 워크스페이스로 지정된 패키지도 link로 포함되게 된다

![](https://i.imgur.com/4V6anAL.png)

