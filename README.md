# 모바일 게임 성능 최적화 방법 보고서
##### 🔎 목차
> [Profiling](#profiling)<br/>
> [Memory](#memory)<br/>
> [Adaptive Performance](#adaptive-performance)<br/>
> [Programming and code architecture](#programming-and-code-architecture)<br/>
> [Project configuration](#project-configuration)<br/>
> [Assets](#assets)<br/>
> [Graphics and GPU optimization](#graphics-and-gpu-optimization)<br/>
> [User interface(UI)](#user-interface)<br/>
> [Audio](#audio)<br/>
> [Animation](#animation)<br/>
> [Physics](#physics)<br/>



<br/><br/><br/>
# Profiling

**프로파일러** : 애플리케이션의 성능 정보를 알려주는 툴이다.
<br/><br/>

### 1. 맹목적으로 하지 말 것.
프로파일러를 켜서 성능측정을 하고, 어디서 병목현상이 나오는지 정확하게 확인을 해야하고 최적화에 들어가야한다.<br/>
성능이 하락한다고 무작정 리소스 변경을 요청하지말고, 어떤 부분에서 성능 저하가 있었는지 확인해라.<br/>
프로파일링의 주 책임자는 프로그래머라는점을 잊으면 안된다.<br/>
<br/><br/>

### 2. 출시 직전에 프로파일링하면 일이 크게된다.
마지막 다 되어서 빌드할 때가 되었을 때 프로파일링하면 문제가 생긴다.<br/>
예를 들어 모든 개발을 마치고 프로파일링을 해봤는데 쉐이더에서 성능 이슈가 났다고 가정하자.<br/>
이미 거의 다 한 프로젝트에 리소스들을 갈아엎어야하는 상황이 발생한다. 일정이 다 미뤄지는 문제가 발생한다.<br/>
<br/><br/>
수시로 프로파일링을 해야한다.<br/>
그래야 출시 막판에 가서 불상사가 생기지 않는다.<br/>
<br/><br/>

### 3. 프로파일링할때 타겟 디바이스에서 돌려야한다.
PC성능, 휴대폰의 성능이 다르기때문에 각 디바이스에서의 병목지점이 달라진다. <br/>
에디터에서 측정한 성능은 러프한 용도로 볼 수는 있지만, 실제 기기에서의 상황은 다르다는 것을 알아야한다.<br/>
<br/>
여러 디바이스에서 돌아가야하기 때문에 최저 디바이스를 정하여 돌아가는지 테스트를 진행하라.<br/>
<br/>
iOS는 Xcode / 안드로이드는 안드로이드 스튜디오에서 제공하는 프로파일러를 추천한다. <br/>
<br/><br/>

### 4. 측정 단위
성능 측정을 할 때 보통 FPS로 많이 따진다. 그러나 FPS 측정치는, 러프하게 볼수있는 탐지수치긴 한데, 병목을 탐지하고 병목을 수정하고 병목에 대한 해결책을 세우기에는 좋은 수치는 아니다.<br/>
프로파일러를 보면 ms 로 많이 나와있다. 성능 측청시에 ms로 성능 측정을 하는것이 좋다.<br/>
```
FPS = 1000/ms (1초에 몇프레임을 그리냐)
ms = 1000/FPS (1프레임을 그리는데, 얼마만큼의 시간이 필요하나?)
```

예를들어서 <br/>
30ms가 측정되었고 <br/>
렌더링에 5ms / 업데이트 5ms / 물리처리 20ms 로 측정되었으면. 하면 딱 봤을 때 물리연산부분에서 최적화하는것이 좋겠다라고 판달할 수 있다.<br/>
<br/>
<br/>
### 5. CPU/GPU 병목 구분하여 봐라
측정을 할 때 CPU 바운더리의 문제인지, GPU 바운더리인지 체크를 해야한다.<br/>
CPU GPU 병렬적으로 돌아가기 때문에, CPU 최적화해도 GPU 성능 변화가 없다. 그래서 CPU 병목인지 GPU 병목인지 봐야한다. <br/>
<br/>
<br/>
### 6. Device 온도
PC에서는 디스크 대빵큰거 웽~ 하며 돌아간다.<br/>
근데 모바일은 쿨러가 없다. 냉각판정도로 아주 작은게 있을뿐이다. 발열에 취약할수밖에 없다.<br/>
<br/>
폰이 뜨거워지면 프로세서는 성능을 낮춰버린다.(열을 낮추고 기기를 보호하기위해) -> 스로틀링<br/>
<br/>

<br/><br/>
# Memory

메모리 적인 측면에서 모바일이 PC 보다 취약하다.<br/>
```
PC에는 CPU, RAM, Storege가 구성되어있다. 
RAM 이 8G 짜리라고 가정하고, 게임 16기가짜리를 돌렸다고 가정하자. 안돌아갈 것 같지만 돌아간다. 왜냐면 Storege의 메모리를 활용하기 때문이다. 웬만해서 프로그램이 죽지 않는다.

모바일은 칩 하나에 다 꾸겨 넣다보니 일단 RAM이 아주 작다. 
그리고 PC에서 있었던 저장장치의 일부를 저장공간으로 활용하는 그런 기능이 없다. 
즉, 8기가 램을 끼웠는데 게임 8기가짜리 돌리려하면 프로그램이 죽어버린다.
```


### 가비지컬렉터를 염두할 것.
C#은 가비지컬렉터로 자동으로 메모리를 관리한다.<br/>
C++은 delete로 프로그래머가 메모리 해제를 직접 해줘야하는 것과 다르다.<br/>
가비지 컬렉터는 제너레이션 단계를 거쳐 쭉 돌아가며 ref 카운트를 돌리면서 카운트가 0인 요소를 날려버린다.<br/>
주기적으로 이렇게 돌아가니 성능에 영향이 있을 수 밖에 없다. <br/>
<br/>
또한 메모리의 할당과 해제가 반복되다보면 메모리가 이빨빠진 모양이 된다.(메모리 단편화문제)<br/>
가비지컬렉터는 이빨빠진 부분을 재배치해서 빈칸들을 없애버린다. (메모리 압축기법) <br/>
이 또한 연산이 많다.<br/>
<br/>
결론적으로 C#을 사용하며 메모리 할당을 신나게 하면 안된다. <br/>
메모리 할당이 일어나면 일어날수록 GC의 부담이 많아지므로 할당을 최대한 줄여야한다.<br/>
<br/><br/>
### string 사용을 최소화<br/>
예를 들어서 Json, XML 파싱하는 프로젝트를 하게되면 문자열 많이써서 계속 GC의 타겟이 된다.<br/>
이 대안으로 사전 빌드단계나 사전 단계를 거쳐 데이터를 읽어서 스크립터블 오브젝트나 에셋 데이터로 변환해서 데이터를 사용해야한다.<br/>
<br/>
또 다른 예로 다음과 같은 코드가 있다고 하자.
```cs
m_Animator.SerBool("Jump",true);
```
애니메이터 jump를 false로 한다. 이 부분에서 string 매개변수로 넘겨주기때문에 가비지 컬렉터 대상이다 <br/>
근데 만약 이 스크립트를 update문에 써버리면 성능저하가 매우 많이될 것이다.<br/>
이 경우에는 id를 들고와서 사용하도록 하자.<br/>
<br/>
코루틴 같은 경우에도. yeild return new.. 이렇게 반복적으로 사용해서, 가비지컬렉터 작동된다. 그래서 캐싱을 해서 쓴다면 가비지가 발생하지 않을것이다.<br/>

<br/><br/><br/>
# Adaptive Performance
안드로이드폰같은 경우 쓸 수 있다.<br/>
가변적인 퍼포먼스 조절할 수 있다.<br/>
<br/>
(1) 명시적으로 휴대폰 뜨거운지 알수있고<br/>
(2) 스로틀링이 빠지는 상황인지, 직전인지, 아닌 상황인지 알 수 있다.<br/>
<br/>
결론적으로 퍼포먼스를 컨트롤 할 수 있다.<br/>



<br/><br/><br/>
# Programming and code architecture
프로그래밍 코딩할때 신경써야하는 부분이다.

### 1. 유니티의 생명주기를 확실히 알 것.
순서를 알고 있어야 성능을 좀 더 신경쓰면서 코드를 작성할 수 있다.<br/>



### 2. 빈 함수를 쓰지 말자.
MonoBehavior 스크립트를 상속받아 썼을 때, Update함수가 코드블럭이 비어있어도 선언만 되면 성능을 먹고있다. <br/>
```cs
void Update()
{

}
```
빈 함수는 제거하는게 좋다. 에디터에서만 돌아가는 로직이 있으면, <br/>
```cs
#if UNITY_EDITOR
void Update()
{
  
}
#endif
```
이렇게 define으로 막아버려야한다. 이렇게 해야 최종 릴리즈때는 포함이 안된다.<br/>

<br/>

### 3. Debug.Log의 함정
Debug.Log를 릴리즈모드로 배포를하면 호출안되는줄 많이들 알고있는데, 아니다. 빌드파일에 다 들어간다.<br/>
<br/>

### 4. Get/Add Component 주의점
컴포넌트를 실시간으로 붙였다 떼었다 하는것이 성능을 많이먹는다. AddComponent 동적으로 하면 안좋다.<br/>
게임 오브젝트 컴포넌트를 GetComponent로 얻어왔고 자주쓰인다면 캐싱해놓고 쓰는게 좋다. 매 프레임마다 GetComponent 하면 성능을 먹는다.<br/>

```cs
private Renderer myRenderer;
void Start()
{
  myRenderer = GetComponent<Renderer>();
}

void Update()
{
  Func(myRenderer);
}
```

<br/>

### 5. 메모리풀
모두가 알고있듯이 오브젝트 풀링을 해야한다.<br/>
예를 들어 슈팅게임 같은 경우 총알을 매번 new 하는게 아니고, 한꺼번에 만들어놓고 재활용해서 써야한다.<br/>
<br/>
주의점 : activePool 껍데기 root에 오브젝트를 만들고, 또 DisabledPool 비활성화된 오브젝트들 옮기고 이런식으로 root를 왔다갔다하면 이쁘게 하이어라이키 창이 정렬되긴하겠지만 setParent 라는것을 통해서 여기저기 리페어런팅된다. 즉, 성능을 먹는다. <br/>
웬만하면 안하는게 낫다. <br/>

<br/>

### 6. 스크립터블 오브젝트
적극권장한다.<br/>
공통적인 속성을을 묶어 쓴다.<br/>

상속 구조로하면 코딩할떄는 구조적으로 예쁘겠지만, 성능적으로 안좋을수있다. 파라미터가 많을수록 직렬화가 일일이 다 된다. -> 성능이슈 <br/>
스크립터블 오브젝트는 에셋이고, 직렬화가 이미 되어있는 아이이기때문에 그걸 가져다쓰면, 성능적으로 이득이다.<br/>


<br/><br/>
# Project configuration

### 1. 프로젝트 설정창
끄거나 줄여라. -> 내가 안쓰는것같으면 끄거나 줄이면 된다.<br/>
<br/>
플레이어 퀄리티세팅, Auto Graphics API(OpenGL 버전별로켜져있음) 이런거도 안쓰는건 꺼버려라.<br/>
물리쪽도 필요없는건 꺼야한다.<br/>
<br/>
유니티 공간과 물리공간이 따로있어서, 게임 오브젝트와 물리공간이랑 동기화를 시켜주는 과정이 있다.<br/>
보통은 그 동기화시켜주는게 FixedUpdate에서 보통 진행을 한다. <br/>
<br/>
Auto Simulation, Auto SyncTransforms 이게 강제로 동기를 해주는거여서, 물리가 중요하지 않는 게임이면 꺼주는게 좋다.<br/>

<br/>

### 2. 타겟 프레임레이트
발열이 심해져도 성능이 떨어진다는 의미이다.<br/>
액션게임이면 타켓 프레임 40-50 이런거<br/>
퍼즐게임이면 타겟 프레임 30<br/>

프레임 레이트를 조절하여 발열과 효율성 퍼포먼스를 좋게해라.<br/>

![image](https://github.com/user-attachments/assets/b441ffe5-ba01-4f7a-83d1-8b2d57dd4269)
<br/>

### 3. 하이어라이키가 점점 커지는거 멈춰라
씬 안에 막<br/>
![image](https://github.com/user-attachments/assets/f8195184-67f3-4ff7-a2d0-d4faab557318)
<br/>
이런식으로 구조가 있다고 하면 A 오브젝트를 움직이면 아래 깔려있는 애들도 transform 재연산이 되어야한다.<br/>
그럼 한번 A 오브젝트를 움직였을 뿐인데 딸려있는 트랜스폼 연산이 엄청 많아지는 것이다.<br/>
그래서 하이어라이키를 복잡하게 하면 할수록 성능이슈가 생긴다는 것이다.<br/>
<br/>
하이어라이키가 단순하면 단순할수록 효율적으로 돌릴 수 있다.<br/>


<br/>

### 4. Transform 한번만
만약 한 캐릭터 안에 move.cs, Jump.cs, Dash.cs, Dodge.cs 있다고 가정하자. <br/>
이 스크립트를 까봤더니 모두 트랜스폼을 건들이고 있다고 하면, 한 플레이어를 처리하기 위해서 트랜스폼 연산이 엄청 많이 매번 일어나게 되는것이다.<br/>
<br/>
또한 코드 작성할때<br/>
```cs
transform.position- = ...
transform.rotation- = ...
```

이러면 한줄한줄 다 재연산일어난다. (두번 일어난다.)<br/>
<br/>
![image](https://github.com/user-attachments/assets/ffbfcfa9-302d-4243-b037-8d74c698fcf0)
<br/>
한번에 처리되는 함수를 잘 활용하자.<br/>

<br/><br/>

# Assets
이미지나 사운드들이 포맷이 다양한데, 유니티 입장에서는 포맷을 다시 변환하게 되므로 원본 포맷 어떤것인지는 성능에 상관없다.<br/>
<br/>

### 1. Texture 세팅
Texture 설정을 플랫폼별로 오버라이드를할수있다. (세팅을 해줄 수 있다)<br/>
<br/>

![image](https://github.com/user-attachments/assets/8d8a8346-37a8-4d75-8778-efa9ec9bb42e)
<br/>

*) 텍스쳐 사이드는 2의 n승으로 해야한다. (POT)<br/>

<br/>

![image](https://github.com/user-attachments/assets/692c7774-5043-4dd8-a392-268b9e33516e)

<br/>
이건 웬만하면 끄는게 좋다.<br/>
데이터를 읽을때, SSD, HDD 가 있으면 RAM에 올리고, VRAM(비디오 메모리,GPU메모리)에 올리게 된다. <br/>
GPU메모리에 올린것은 CPU 메모리에서 날려버린다. <br/>
하지만 이 Read/Write Enable을 체크하면 CPU 메모리에 없애지않고 그대로 남아있게 한다. <br/>
결론적으로는 메모리가 두번 올라가있기때문에 앵간하면 끄는게 좋다.<br/>
<br/>

![image](https://github.com/user-attachments/assets/2e3369b0-16bb-485a-948f-9d315486e510)

<br/>
MipMap은 512 텍스쳐가 있으면, 245, 129... 복사본들을 만드는것을 말한다.<br/>
그럼 메모리영역이 33% 뻥튀기가 된다.<br/>
<br/>
하지만 3D 게임에서는 멀리있는 오브젝트 가까히 있는 오브젝트가 많은 연산없이 처리되기 때문에 런타임 효율, 메모리 효율적인 측면에서 필요하다.<br/>
그런데, 2D게임에서는 일반적으로 꺼야한다!! 필요가 없기 때문이다.<br/>

<br/>

### 2. Mesh 세팅
매시도 임포트 세팅할때 파라미터 많다.<br/>
**Compress the mesh** -> 압축하는 것. 데이터 정밀도를 떨어뜨리는거긴하다. 빡세게하되, 시각적 품질차이를 보면서 해야해야한다.<br/>
**Disable Read/Write** 메시를 동적으로 조작하는게 없으면, 기능 끄는것이 좋다. 텍스쳐랑 똑같은 관점이다.<br/>
<br/>
안쓰는것같다 하면 꺼야한다.<br/>
스키닝 연산이 없으면 rig를 끄면 된다. 블랜드 쉐입(캐릭터 표정 등)안사용하는 매시면 꺼버린다. (배경건물, 미사일 오브젝트면 끄면된다.)<br/>
<br/>
Tangent -> 노멀맵을 사용하지 않으면 탄젠트 끄면된다. 
<br/>
<br/>
매번 이렇게 껏다 켰다 할수는 없으니(실수할수도) **AssetPostprocessor** 라는게 있다. 이걸쓰면 자동화할 수 있다.<br/>



<br/><br/>
# Graphics and GPU optimization
<br/>
최신버전부터는 URP가 기본이다. 기존 파이프라인보다는 URP가 성능이 좋다.
<br/>

### 1. Batch your draw call
URP에서는 드로우콜 절약할 수 있다.<br/>
배칭은 드로우콜을 절약할 수 있는 기법이다.<br/>
- 스태틱 배칭 : 예를 들어 움직이지 않는 배경 오브젝트들은 스태틱배칭으로 처리<br/>
<br/>

### 2. 프레임 디버거를 사용해라.
프레임 디버거 : 드로우콜을 디버깅 할 수 있는 기능이다.<br/>

<br/>

### 3. 동적 라이트를 많이사용하는것을 피해라
동적 라이트는 성능영향을 많이 끼친다는것을 염두하자.<br/>
스태틱 라이트, 라이트맵,라이트프로브를 권장한다.<br/>
<br/>
**라이트맵** -> 라이트의 결과물을 텍스쳐로 저장해놓는것이다.<br/>
실시간 연산이 들어가는것이 아니어서 효율적이다. -> 정적인 오브젝트에게만 사용할수있음.<br/>
<br/>
**라이트프루브** -> 동적인 오브젝트에 라이트를 표현할 때 사용한다.<br/>
어떤 공간에 라이트 결과물을 저장해놓고 오브젝트가 지나가면 오브젝트가 라이트프루브의 영향을 받는다. 섬세한 표현은 불가능하겠지만, 빛의 변화를 표현할 수 있다.<br/>
<br/>

### 4. 그림자는 꺼라.
웬만해서 끄는게좋다 드로우콜과 GPU 성능을 많이 먹는다.<br/>

<br/>


### 5. LOD 쓰면 좋다
LOD : 가까히 있는것은 자세히, 멀리있는 것은 뭉그러져 표현하는 기법<br/>
<br/>

### 6. 오브젝트 컬링
오클루젼 컬링 : 안보이는 오브젝트는 그리지 않는 것이다. (랜더링을 거치지 않는것)<br/>
<br/>
문제는, 광활한 야외에서는 이런 경우는 많지 않다. 그럼, 비효율적일수도있다. 가려진건없는데 괜히 연산한다고 안좋을수도있으니 상황에 따라 잘 쓰자.<br/>

<br/>

### 7. 카메라 사용 제한을 두자.
실제 카메라가 컬링연산을 하는데, 카메라를 여러개 두면 컴링연산이 중복으로 일어난다.(시네머신 카메라는 해당하지 않는다.)<br/>
<br/>


### 8. Renderer.material 조심하자.
이걸 호출하는 순간 복사본이 만들어진다.<br/>
비행기가 2개가 있고 같은 mat을 사용하고 있다고 가정하자. <br/>
근데 만약 비행기 1에서 Render.mat-을 호출하면 복사가 일어나고 두 비행기가 같은 amt를 사용하지 않게된다. 즉 배칭이 깨져버린다.<br/>
<br/>
만약 복사본 안만들어지게 하고싶으면 Renderer.sharedMaterial 를 사용하면된다.<br/>

<br/>

### 9. 리플렉션 프루브는 최소화하자.
라이트 프루브는 비용이 싸다 얘는 숫자만 저장되기 때문이다. 숫자 9개만 저장되어서 가볍다.<br/>
하지만 리플렉션 프루브는 6면의 텍스쳐가 저장된다. <br/>
컨셉은 비슷하지만 저장되는 결과물이 다르기때문에 성능 차이가 많이난다.<br/>
<br/>


<br/><br/>
# User interface

### 1. 캔버스를 신경 쓰자
드로우콜 배칭은 캔버스 단위로 이루어진다.<br/>
캔버스 안에서 하나의 버튼이 변하게되면, 캔버스 안에 있는 다른애들도 폴리곤을 구성하기위해서 다 바꿔줘야 한다.<br/>
그래서 캔버스 안에 동적인 아이들 정적인 아이들 섞으면 비효율적이고, 동적인 부분 모아놓은 캔버스/정적인 부분 모아놓은 캔버스 나눠야 성능적인 부분에서 좋다.<br/>
여러개의 캔버스를 나누는게 효율적이다.<br/>

<br/>

### 2. 보이지 않는 UI 요소는 꺼라
보이지않는 UI 요소는 꺼주는게 좋다.<br/>
캔버스 영역 밖에 버튼이 있다가 화면안으로 들어갔다 나왔다 하는거면 캔버스 밖에 있어도 연산이 되어서 안쓰면 꺼줘야좋다.<br/>

<br/>

### 3. 인터렉션이 일어나지 않는 요소의 레이캐스트는 끄자
인터렉션이 안일어나는 요소들의 레이캐스트는 꺼줘라. <br/>

캔버스에서 많은 요소가 있는 상태에서 하나의 버튼만 레이를 쓰다고 가정하면 캔버스 레이를 꺼주고 버튼에만 데이캐스터를 붙여줘야 성능적으로 효율적이다. <br/>

레이캐스트 타갯도 안쓰면 꺼줘야한다.<br/>
<br/>

### 4. 레이아웃 그룹같은 경우는 피하는게좋다.
얘는 동적으로 연산하고있어서 안쓰는게 좋다.<br/>
<br/>


<br/><br/>

# Audio

### 1. 모바일은 웬만하면 mono로 세팅해라.
스트레오를 괜히 둬서 메모리 뻥튀기 할 필요가 없다.<br/>
<br/>

### 2. 무슨 포맷이어도 상관없는데, Wav 포맷 권장한다.
원본이 뭐든 상관없이 압축 다시하기때문에 그냥 wav 파일 쓰는게 제일 손실이 적을테니까 이거 쓰는게 좋다<br/>
<br/>

### 3. 압축 포맷 설정
Compression FOrmat(압축포맷) 은 디폴트로 Vorbis를 쓰면 된다. <br/>
짧은사운드같은 경우는 ADPCM 쓰면된다.<br/>
<br/>


### 4. 로드타입 설정
로드타입<br/>
Decom- Load : 압축을 다 풀어서 메모리에 올린다. (비효율적) 작은사운드만 적용 하자.<br/>
Compre -Me : 압축한 상태로 메모리를 올린다. 중간크기의 사운드를 얘로 적용하자<br/>
Streaming : 배경음(용량이 큰 아이들)을 Streaming으로 하면된다. <br/>
 <br/>


### 5. 볼륨 0인 요소는 메모리에서 날려라
볼륨 0 이어도 연산처리된다. 그냥 메모리에서 날려버리자 <br/>
 <br/>

<br/><br/>

# Animation

### 제네릭이냐, 휴머노이드냐
휴머노이드는 제네릭보다 30%~50% 더 메모리를 사용한다. <br/>
메모리 효율적으로는 제네릭이 좋긴하다. <br/>


<br/><br/>

# Physics
### 1. Prebake Collision Meshes 체크하는게 좋다.
충돌처리하기위한 내용들을 바닥이나 벽에 로딩타임에 미리 구워놓는 기능이다. 효율적이다.  <br/>

### 2. Collision Matric 충돌처리 사용해서 행렬 체크하는것이 좋다.
![image](https://github.com/user-attachments/assets/163b7db0-63e8-48db-a856-9b748f94ea53)

<br/>
-> 연산비용을 줄일 수 있다.<br/>

<br/>

### 3. 콜라이더는 기본도형 콜라이더를 쓰는게 좋다. 
매시콜라이더는 최적화할수있는 여지들이 사라진다.<br/>

