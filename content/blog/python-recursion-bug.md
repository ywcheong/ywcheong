---
title: 'sys.setrecursionlimit(10**6)은 만능 해결사가 아니다'
date: '2024-10-31T20:19:49+09:00'

draft: true
---

## 결론부터 말하자면...
알고리즘 문제풀이에 Python을 사용할 경우 [sys.setrecursionlimit](https://docs.python.org/3.13/library/sys.html#sys.setrecursionlimit)의 기본 한계인 1000을 초과하는 로직이 있을 경우
재귀함수가 아닌 일반함수로 구조를 바꾸거나, 또는 C, C++로의 변경을 고려하는 것이 좋다. 그렇지 않을 경우 **Windows에서 탐지가 불가능한 치명적인 버그가 발생할 위험이 있다.**

## 문제의 시작
일의 시작은 [백준 11049(행렬 곱셈 순서)](https://www.acmicpc.net/problem/11049) 문제를 풀던 때였다.
해당 문제는 전형적인 메모이제이션을 활용한 동적 계획법 문제로, 재귀함수로 푸는 것이 그 구조상 깔끔하여 재귀함수를 통해 문제를 해결하고자 하였다.

알고리즘 문제는 C, C++를 활용해 푸는 것이 좋지만 Python으로 진행한 프로젝트도 많았고 평소에도 자주 쓰는 언어였기에 별 생각 없이 풀었는데,
Python에서 시간제한이 빡빡하게 설정되어 있던 문제라 시간 초과 오류를 몇 번 받았다.
(참고로 해당 코드는 ChatGPT에게 C로 변환하라고 지시한 뒤 실행하면 10ms 안에 실행되었다.)
따라서 프로그램의 어느 부분이 느린지 분석하고자 커다란 테스트케이스를 만들어서 입력으로 넣어보았다.

원래의 문제를 간단히 설명하면 n개의 행렬을 곱하는 가장 효율적인 문제였는데, 이때 n이 500 이하였다.
이때 문득 n이 더 크면 어떨지 궁금해져서 테스트케이스를 n = 5000짜리로 생성해서 집어넣게 되었다.

![alt text](/python-recursion-bug-1.png "matrix_size = 5000이다.")

그랬더니 놀랍게도 프로그램은 **즉시** 실행을 종료했고, 어떠한 출력도 내뱉지 않았다.

![alt text](/python-recursion-bug-2.png "당시 상황 재연. 프로그램의 출력이 없다. 화면에는 나타나지 않았지만 ")

Python에서 PS를 하다 보면 흔한 오류인 RecursionError도 아니었다.
오류나 다른 특이 메시지도 없었기 때문이었다.
무엇보다도 n이 적당히 작으면(예: n = 500) 이런 문제 없이 정상적으로 프로그램이 실행되었다.

문제를 해결하려 여러 번 검색했지만 생각보다 국내외 웹에서 관련 원인에 대해 정리된 자료를 찾기 힘들어서 직접 정리하였다.

## 문제의 재현 방법
사실 위 문제는 아주 간단한 Python 코드로 쉽게 재현할 수 있다.

```python
import sys
sys.setrecursionlimit(10000010)

def recursion(n):
    if n == 0:
        return 0
    return n + recursion(n-1)

print("Program Start")
print(recursion(10000000))
print("Program End")
```

위 코드를 **Windows 환경**의 터미널에서 `python3 script.py`와 같이 실행하면 "Program Start" 이후로 아무 출력 없이 명령이 종료되고, Python REPL에서 실행시키면 REPL 자체가 죽어 버린다.
이는 [Python Bugtracker Issue#45645](https://bugs.python.org/issue45645)에도 이미 보고되어 있다.

이슈의 내용을 요약하면 Windows 환경에서 재귀함수가 지나치게 깊어지면 Stack Overflow 에러가 발생하는데, Windows 환경에서는 이 오류를 catch할 수 있는 적절한 방법을 구현할 수 없어 Python이 죽어 버리는 것이다.
Windows에서는 프로그램이 스택 사이즈를 초과하면 프로세스를 즉시 종료하므로 해당 버그는 "수정 없을 것(Won't Fix)"이라는 태그로 종결되었다.
엄밀히 말하자면 [Guard Page](https://learn.microsoft.com/ko-kr/windows/win32/memory/creating-guard-pages) 영역을 생성하는 등 방어는 가능하지만, 깊은 재귀함수를 사용하지 않는 일반적인 Python 프로그램에서
발생하는 오버헤드 등을 고려할 때 도입하지 않은 것으로 보인다.

### Linux에서는 일어나지 않는 버그
![alt text](image.png "Windows에서 작동하지 않는 이 코드는, Linux에서는 0.01초 이내로 실행된다. 사진은 Amazon EC2 Linux 환경.")

Linux에서는 이 버그가 나타나지 않는다.
위의 실행 결과는 `setrecursionlimit`의 값을 21억까지 매우 크게 늘려도 작동했는데, 그 이유는 Linux와 Windows가 기본적으로 프로세스의 스택 크기 제한을 처리하는 방법이 다르기 때문이다.

[Python module - resource 공식 문서](https://docs.python.org/ko/3/library/resource.html)에 따르면 Linux에서는 스택 크기를 직접 통제할 수 있는 `resoruce` 모듈을 제공하지만 Windows에서는 그렇지 않다.
그 이유는 resource 모듈은 linux의 `setrlimit`이라는 시스템 콜을 사용하기 때문이지만, Windows에서는 프로그램 실행 중에 동적으로 스택 크기를 제어하는 기능을 제공하지 않기 때문이다.
Windows에서 스택 영역을 굳이 런타임에 추가로 사용하고 싶다면, 새로운 쓰레드를 생성해 해당 스택 영역을 활용하는 등 간접적인 방법 외에는 존재하지 않는다.
또한 Windows에서는 기본 스택의 크기가 Linux에 비해 작은 편이다.

{{< callout type="warning" >}}
  Linux와 Windows 간의 기본 스택 크기는 컴파일 옵션이나 Linux의 `ulimit -s stack_size` 명령 등으로 그 대소관계가 쉽게 바뀔 수 있다는 점을 유의해야 한다.
  그러나 Windows가 런타임에서 직접적인 방법으로 동적으로 스택 크기를 변화시킬 수 없다는 것은 여전히 유효하기에 상황이 크게 바뀌지는 않는다.
{{< /callout >}}

정리하자면, Linux는 Windows와 달리
* 스택 영역을 실행 중에 동적으로 확장할 수 있고
* (항상 그렇다고는 할 수 없지만) 기본 스택 크기도 큰 편임

이라는 요인이 맞물려져 이러한 차이가 발생한 것으로 보인다.

## 그래서 어떻게 해야 하는데?
근본적인 문제 해결을 위해서는 재귀 함수를 사용하지 않는 것이 좋다.
하지만 PS를 하다 보면 재귀함수가 훨씬 직관적으로 해답을 생성한다는 것도 사실이다.

사실 많은 경우 C, C++ 등의 언어로 PS를 하기 때문에 이런 문제를 겪지 않는다.
C나 C++은 꼬리 재귀 최적화를 지원하지만 Python은 그렇지 않기 때문에 리소스 사용에서도 큰 차이를 보이기 때문이다.
당장 위의 문제를 일으킨 코드도 C로 변환하면 10ms도 걸리지 않는다.

따라서 다음과 같은 방법이 가장 적절해 보인다.

* C, C++을 쓴다면..
    * 꼬리 재귀 최적화를 활용해 재귀함수를 마음껏 쓴다
* Python 또는 다른 언어를 쓴다면...
    * Recursion Depth가 작을 때만 (< 1,000) 재귀함수를 사용한다
    * 가급적이면 Recursion을 사용하지 않는다 (오버헤드가 상당함)
    * `sys.setrecursionlimit` 등으로 임의로 재귀깊이 제한을 해제하는 것은 매우 주의한다

괜히 C, C++가 근본 언어가 아니다.