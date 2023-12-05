팀에서 프로젝트를 구성할 때 처음에는 각자 repository를 분리했었다. 그렇게 사용하다가 테크 리더가 불편함을 말했다. 리더 한 명이 코드 리뷰를 하는데 페이지를 바꿔가면서 리뷰를 하는 게 피곤하다고 했다. 나는 하나의 코드에서 에러가 나도 전체가 동작을 멈추게 되는 것 아니냐고 질문을 했고 거기에 대한 해결책으로 npm의 워크스페이스를 사용하게 되었다
## NPM Workspace
npm workspace를 사용하면 workspace로 지정된 디렉터리가 하나의 패키지로 인식된다. 패키지로 인식된다는 의미는 package.json이 존재한다는 말이 되고 거기에 기록된 의존성들은 root 디렉터리에 node_modules에 전부 설치된다.

결국 root > node_modules에는 모든 모듈들이 들어가게 되고 추가로 워크스페이스로 지정된 패키지도 link로 포함되게 된다

![](https://i.imgur.com/OuPgSkI.png)

## 더 생각해 볼만한 점
서로 다른 워크스페이스에서 같은 모듈을 다른 버전으로 사용하고 있을 경우에 문제가 생기지 않을까? 라는 생각을 해봤는데 nodejs에서는 버전이 다르면 다른 모듈로 인식해서 불러온다고 한다.
[Package manager tips](https://nodejs.org/api/modules.html#package-manager-tips)
