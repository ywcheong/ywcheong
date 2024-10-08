---
title: 'ACE에 대한 간단한 이야기 + Pydantic을 왜 이제서야 배울까'
date: '2024-10-06T22:43:58+09:00'
---

## Easyplotlib 구조 만들기
[Easyplotlib](../../project/easyplotlib)을 개발하면서 제일 먼저 시작했던 부분은 Frontend와 Backend 사이의 통신 양식을 정하는 것이었다. 이 프로젝트를 간단히 설명하면 (인공지능은 아니지만) 사용자를 대신해 사용자의 요구대로 자동으로 코드를 생성해 주고, 이를 실행한 뒤 그 결과를 보여주는 게 주 기능이다. 코드를 동적으로 생성한 뒤 실행한다는 특성상 잘못된 방식으로 구조를 설계할 경우 [ACE (Arbitrary Code Execution; 임의 코드 실행)](https://www.okta.com/identity-101/arbitrary-code-execution/) 취약점이 발생할 우려가 컸기에, 상당히 신경써서 만들어야겠다는 생각이었다.


## 임의 코드 실행 취약점
브라우저상에서 직접 코드를 생성해서 서버로 보낸 뒤 실행하는 방법은 너무 간단하지만, 보안에 대해 조금이라도 알고 있다면 절대적으로 피해야 할 개발 방법이라는 것을 알 수 있다. 서버에서 기대한 코드는 `matplotlib.pyplot.plot()` 같은 순한 함수였겠지만 실제로는 이런 코드가 날아올 수 있기 때문이다.

```python
import subprocess
subprocess.run(['rm', '-rf', '/', '--no-preserve-root'])
```

### Python 2의 input()은 위험하다
Python 2의 예시를 들자면, 사용자의 터미널 입력을 받는 함수는 `raw_input()`과 `input()`으로 두 가지가 있었다. `raw_input()`은 Python 3의 `input()`과 같이 항상 str 타입을 반환하지만 `input()`은 가능한 경우 자동으로 캐스팅을 해 주는 차이가 있어서 초보 시절에 자주 애용했던 기억이 난다.

```python
# Python 2
>>> raw_input()
3    # 입력
'3'  # 출력
>>> input()
3    # 입력
3    # 출력
```

그런데 사실 충격적인 점은 [Python 2의 input() 구현](https://python.readthedocs.io/en/v2.7.2/library/functions.html#input)은 `eval(raw_input())`이었다! 이로부터 야기되는 보안 이슈도 어마어마했을 것이다.

ACE의 다른 예시를 들자면 C나 C++와 같이 메모리 영역에 직접 접근 가능한 언어에서 발생하는 버퍼 오버플로를 활용한 공격도 크게 보면 ACE 취약점의 일부로 볼 수 있겠다. 정상적인 방법으로는 실행되지 않는 사용자 입력 영역을 실행시킨다는 개념은 같기 때문이다.

물론 위의 ACE 취약점을 예방하는 방법은 여러 가지가 있다. Python에서 파일 및 네트워킹 기능을 제거한 채 컴파일한 커스텀 언어를 사용할 수도 있꼬고, AWS를 활용해 가상화 레이어를 추가하거나, 특정 함수의 동작을 패턴 매칭이나 [RestrictedPython](https://github.com/zopefoundation/RestrictedPython) 같은 외부 패키지로 제한할 수도 있다. 하지만 가장 좋은 것은 [백준](https://www.acmicpc.net/)과 같은 온라인 저지처럼 ACE가 반드시 필요한 경우가 아니라면 이를 처음부터 예방하는 것이다.

## Easyplotlib request 설계
사실 Easyplotlib은 저런 이상한 설계를 애초부터 고려할 생각이 없었다. 데이터를 구조화할 수 있기 때문이다. 다만 문제가 있었다면 구조화된 데이터가 너무 복잡해 이를 검증할 코드를 처음부터 짜기가 상당히 피곤했다.

아래 카드를 클릭하면 데이터의 '대충 버전' 정의를 볼 수 있다. 다만 아래 버전은 최신화되지 않으며 [이 링크](https://github.com/ywcheong/easyplotlib/blob/main/docs/interface.md)에서 최신 커밋의 '대충 버전'을 볼 수 있다.

{{% details title="**구조화된 데이터 보기 (JSON)**" closed="true" %}}

* request_id : *Is uuid4*
* figure
    * size
        * row : *Is numeric, plt.subplot(row, _)*
        * column : *Is numeric, plt.subplot(_, column)*
    * axes [List] : *length of `figure.size.row`*
        * [List] : *length of `figure.size.column`*
            * *Is one of `axes[].name` or null. Null axes will not be rendered, as it never exists*
    * style
        * *Every possible key-value pairs are defined at __figure-style__*
* axes [List]
    * name : *Is string*
    * plot [List]
        * *Is one of `plot[].name`*
    * style
        * *Every possible key-value pairs are defined at __axes-style__*
* plot [List]
    * name : *Is string*
    * format : *Every possible values are defined at __plot-format-list__*
    * data
        * key: *Depending on `plot[].format`, there are different required and optional keys. Check __plot-format-list__.*
        * value: *Is one of `data[].name`*
    * style
        * *Every possible key-value pairs are defined at __plot-style__*
* data [List]
    * name : *Is string*
    * value [List]
        * *Is numeric*

{{% /details %}}

이 긴 JSON을 하나하나 검증하는 것은 정신이 나갈 것 같았고, 곧 FastAPI 개발 도중 데이터를 깔끔하게 받을 수 있다는 이유로 사용을 추천한다고 어디선가 읽은 Pydantic 패키지에 대해 조사해보게 되었다.

## Pydantic 진작 쓸걸
[Pydantic](https://docs.pydantic.dev/latest/)은 여러 기능이 있지만 데이터 검증 기능을 메인으로 내세우는 패키지이다. 기능 설명을 하면 너무 복잡해지지만, 아래와 같은 상황을 생각해 보자.

```python
class User:
    name    : str               # 이름
    age     : int               # 나이
    spouce  : Optional[User]    # 배우자
    friends : List[User]        # 친구
```

Python에서는 Type hint를 통해 Linter의 작동을 보조할 수는 있지만 그것이 실제 데이터의 타입 일치를 보증해 주지는 않는다. 극단적으로는 위 코드에서는 다음 코드도 실제 에러를 일으키기 전까지는 (예: `User(spouce=None).spouce.name`) 어떠한 검증도 '직접' 진행하지 않으면 오류가 숨게 된다. Pydantic의 유스케이스를 몇 개 나열하자면

* 타입 강제
    * 특정 데이터가 Type Hint를 따르도록 강제할 수 있음
    * Type을 따르지는 않지만 캐스팅이 가능한 경우 (예: `'23' -> 23`) 자동 캐스팅을 허용할 수 있음
    * 자동 캐스팅을 비활성화하거나 캐스팅이 불가한 경우 `ValidationError`를 발생
    * 명시되지 않은 entry를 오류발생/무시/별도처리 등 조건지정 가능
* 복잡한 유효성 검사
    * Type만을 검사하는 것이 아닌, 복잡한 조건을 argument만으로 쉽게 부여할 수 있음
        * 정수의 경우 최대-최소, 문자열의 경우 길이 조건 및 정규식 등 설정 가능
    * 지나치게 복잡한 조건의 경우 검사함수를 직접 작성할 수 있음
* 모델 유효성 검사
    * 각 entry만 검사하는 것이 아닌, 여러 entry 간의 상호관계를 기반으로 모델 유효성도 검사할 수 있음 (커스텀 함수)
        * 예를 들어 `if(self.spouce) assert(self.spouce.name != self.name)`과 같은 복잡한 조건도 함수로 설정가능
* 클래스-JSON 상호변환 지원
    * 클래스 인스턴스를 JSON으로, 또는 그 반대로 변환 가능
    * 클래스 인스턴스 선언을 비롯한 모든 과정에서 별도의 함수호출 없이 자동으로 위의 유효성 검사가 수행됨

사실상 User Input Sanitization이 필요한 모든 곳에 사용할 수 있다. 이걸 진작 배웠으면 지금까지 한 프로젝트에서 이 고생은 안 해도 됐었을 것 같아서 안타까웠다.

## Logfire도 배우고 싶은데
Pydantic 공식 사이트에 Logfire라고 Pydantic과 호환되는 로깅 툴이 있다. 이것도 배우고 싶은 마음은 드는 까닭이, 과거 내가 작성했던 프로젝트의 코드를 보면 그런 생각이 들 수밖에 없다.

```python
# 구 프로젝트 코드 (일부 변경됨)
def putLog(name, action=None, data=None):
    if action is None:
        logging.info("{:<29} | {:<8} |",
            get_current_time_ISO(), name
        )
    elif data is None:
        logging.info("{:<29} | {:<8} | action = {:<20}".format(
            get_current_time_ISO(), name, action
        ))
    else:
        logging.info("{:<29} | {:<8} | action = {:<20} | data = {}".format(
            get_current_time_ISO(), name, action, data
        ))

putLog("SystemComponentA", "auto-sync", "[sync-ratio = {:.3f}]".format(
    theory_max
))
```

이렇게 누더기 같은 코드로 로깅을 했는데, 사실 모든 줄에 `print`를 하면서 테스팅하는거랑 별반 다르지않은 주먹구구식이다.

## 개발자 마인드셋
사실 지금까지 Pydantic을 안 배운 이유는 간단하다. 지금까지 Pydantic을 필요로 할 만큼 복잡한 데이터 Validation 로직이 필요 없었기 때문이다. 항상 하는 생각이지만 비즈니스 로직이 복잡해지면 주먹구구식에는 한계가 생기고, 이로 인해 새로운 기술을 배워야 하는 시점이 오는 것 같다. 그리고 새 기술을 배우고 나면 '이걸 진작 배울걸 왜 이제서야 활용할까'와 같은 생각이 든다.

하지만 결국 필요를 느껴야 배우게 되는 것 같다. 그리고 그 필요를 가장 빠르게 느낄 수 있는 곳은 결국 프로젝트 아닐까. 단일 책임 원칙도, 테스팅도, 커버리지도, AWS도, Python도, Websocket도, Agile도, OS도 결국 필요하니까 배울 수밖에 없었다. 귀납적으로 생각하면 Logfire도 필요를 느낀 미래의 내가 배우지 않을까, 아마도? ■