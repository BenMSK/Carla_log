# CARLA [0.9.X] 실행 방법

본 글에서는 칼라 [0.9.x]의 실행 방법에 대해 설명한다.  
칼라를 Source로 Linux 빌드하는 방법을 초점으로 한다.  
빌드는 다음 링크를 참고한다.
https://carla.readthedocs.io/en/latest/build_linux/

칼라가 빌드가 되었다고 가정하고 설명을 진행한다.

***
##### [1] 서버 실행: 

CARLA는 기본적으로 Unreal이라는 Engine을 통해 CARLA 서버를 구축하고 이를 Python API와 연동하여 사용한다.
실행 코드는 다음과 같다.
```
$ ~/carla-0.9.x && ./CarlaUE4.sh
```
추가적으로 서버의 fps, visualiation quality, 화면의 크기 및 화면 on/off 조절등 다양한 것들이 인자로 조절이 가능하다. 그 중 몇가지를 보여준다면,
- Carla 0.9.x display off mode commands --> ```DISPLAY= ./CarlaUE4.sh -opengl```
- Carla 0.9.x display on mode commands --> ```./CarlaUE4.sh -windowed -ResX=640 -Resy=480 -quality-level=Low```
***
##### [2] 서버 접속:
\
CARLA 서버가 실행이 된다면, 기본적으로 Town03이 소환된다. (물론, 서버 default map 또한, [1] 서버 실행에서 인자로 조절 가능)  
실행된 서버에서 ```carla-0.9.x/PythonAPI/examples```로 들어가게 되면, 많은 python script를 보게 될 것이다. 이 중, 직접적으로 내 차량을 돌리는 핵심 코드는 '**manual_control.py**'이다. 이를 실행하면 현재 실행된 서버에 접속하게 된다.  
추가적으로 서버에 다른 NPC를 소환하고 싶다면, '**spawn_npc.py**' 코드를 보자.
***
##### [3] Tip:
위 두가지 과정만 진행하면 기본적인 CARLA 실행 및 내 차량을 돌리는 것이 가능해진다.  
CARLA시뮬레이터에서 하고 싶은 대부분은 [Carla GitHub](https://github.com/carla-simulator/carla "칼라 깃허브") 의 이슈를 참고하자. 생각하는 대부분의 것들이 있다.
이 중, 괜찮다고 생각하는 Tip을 몇가지 적어보려고 한다.  
- CARLA 빌드 시, python api 또한 빌드하게 된다 (```make PythonAPI```). 이때, 기본 CARLA 함수들은 C++로 구성되어 있으므로 이를 **.so** 로 묶어 python에서 **.egg**  파일을 import하여 사용하게 된다. 특히, **.egg** 을 import할 때, import가 안되는 문제가 생길 때가 있다. **.so**를 못찾았다고 불러올 수 없다고 한다. 그래서 필자는 CARLA의 가장 최근 버전을 빌드한 package([nightly build](https://github.com/carla-simulator/carla/blob/master/Docs/download.md "칼라 최신"))를 다운받아서 그 안에 있는 **.so**를 복사하여 사용하였다. 에러는 없어졌지만, 서버 창에서는 버전이 안맞는다는 말이 나온다. 하지만, 사용하는데는 문제가 없다.
  - (**이 방법은 근본적인 해결방법이 아니다. 즉, 0.9.9 (현재 CARLA의 최신 버전)를 빌드하고 위 같은 에러가 나온다면 nightly build 버전과는 차이가 없어, 이 방법이 통한다. 하지만, 0.9.9 이하의 버전 (0.9.0-.8)을 빌드하고 nightly build 버전을 사용한다면 안될 가능성이 크다. 필자의 후배는 위 에러가 없이 잘 사용하므로, 조금 더 공부가 필요한 부분.**)
- CARLA에서 laser sensor같은 것 또한 사용 가능하다. 이는 *manual_control.py*의 없는 class다. (왜 없는지는 모르겠다...) class의 이름은 LineOfSightSenser. 코드는 다음과 같고, *manual_control.py*에 새로운 class로 추가하면 사용이 가능하다.
- [**중요**] 기존 맵을 수정하는 방법은 간단하다. 우선, ```carla-0.9.x/ && make launch```를 이용하여 Unreal Editor를 실행한다. 실행된 map에서 여러 object들을 옮기며 수정한다 (단축키는 Unreal Editor Documentation을 참고.). 실행된 map을 저장하고 (Ctrl+S) Unreal Editor를 종료(Ctrl+C)한다. ```carla-0.9.x/ && make package```를 이용하면, 방금 수정한 그 맵을 기반으로 새로운 ```./CarlaUE4.sh```을 만들게 된다. 만들어진 맵은 ```carla-0.9.x/Dist/CARLA_ShippingXXX/LinuxNoEditor```에 존재한다. **LinuxNoEditor**안에 있는 ```./CarlaUE4.sh```을 실행한다.
- 강화학습과 같은 환경을 구축하고 훈련을 할 때는 agent를 파괴하고 다시 생성하는 방식으로 구축하면 안된다. 왜냐하면, 칼라 서버의 입장에서는 agent를 파괴하고 이와 다른 새로운 agent를 생성하는 것이기에 컴퓨터 메모리가 증가하게 된다. 이때는 차량의 state를 초기화한 후, 위치를 조정하는 방향으로 구축하자.
- 다른 NPC, 즉, 동적 장애물은 어떻게 제어할까? 이는 생각보다 쉽다. 코드 상에서 NPC를 소환하고 이를 내 agent를 제어하듯이 하면 된다. 기본적인 방법은 모두 동일하며, 내 agent가 가진 센서를 그대로 사용하게 할 수 있다. 
  - ex) 동적 장애물이 충돌이 일어나면, 리셋을 시킨다던가....

> This repository has **non-profit purposes**.\
> It is going to be updated, futher.\
> If you have any question, \
> please contact to msk930512@gmail.com.
