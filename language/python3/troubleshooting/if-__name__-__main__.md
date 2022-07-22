# `if __name__ == '__main__'`

Tags: [#troubleshooting](), [#\_\_main\_\_](), [#\_\_name\_\_](), [#scripting]()  
**Solves [Issue](https://github.com/ybyo/automation-scripts/issues/12)**

## Description

최근 Python3를 활용하여 스크립트를 작성하고있다. 여기엔 2가지 이유가 있는데 먼저 Python3 사용에 익숙해지기 위해서, 그리고 평소에 자주 사용하는 커맨드 등을 스크립트로 자동화하여 효율을 높이기
위해서다. 특히 공부하는 입장에서 Docker 컨테이너를 삭제하는 상황은 굉장히 많이 발생한다. 과거에 Docker 실습, 혹은 프로젝트 수행 과정에서 실행된 컨테이너 중 현재에는 사용하지 않는 것들도 많기
때문이다.

이 경우 자주 인터넷 블로그에서 '모든 도커 컨테이너 삭제' 혹은 '모든 도커 컨테이너 이미지 삭제' 등을 검색하거나 윈도우 스티커 메모 등에 커맨드 등을 따로 기록해두기도 한다.

이젠 더 이상 블로그를 기웃거리지 않고, 스크립트 작성 능력도 기를 겸 Python3로 반복해서 사용하는 명령어 혹은 작업을 스크립트로 작성하고 Github에 공개하기로
했다([레포지토리](https://github.com/ybyo/automation-scripts)). 이번에 작성한
스크립트는 [모든 도커 컨테이너 삭제 스크립트](https://github.com/ybyo/automation-scripts/blob/main/scripts/docker/remove_all_docker_containers.py)
이다. 필요한 메소드들을 검색하며 차근차근 진행하던
중 [테스트](https://github.com/ybyo/automation-scripts/blob/main/scripts/docker/remove_all_docker_containers_test.py) 과정에서 한
가지 문제가 발생했다.  `pytest -v` 명령어를 통해 테스트 수행 중 다음과 같은 오류 메시지가 발생했다.

```
========================================================== ERRORS ==========================================================
__________________________________ ERROR collecting remove_all_docker_containers_test.py ___________________________________
remove_all_docker_containers_test.py:5: in <module>
    from remove_all_docker_containers import remove_containers
remove_all_docker_containers.py:46: in <module>
    ask_user()
remove_all_docker_containers.py:36: in ask_user
    answer = input("Continue? (y/N)")
/usr/local/lib/python3.10/dist-packages/_pytest/capture.py:192: in read
    raise OSError(
E   OSError: pytest: reading from stdin while output is captured!  Consider using `-s`.
----------------------------------------------------- Captured stdout ------------------------------------------------------
All of your docker containers on your host will be removed.
Continue? (y/N)
================================================= short test summary info ==================================================
ERROR remove_all_docker_containers_test.py - OSError: pytest: reading from stdin while output is captured!  Consider usin...
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!! Interrupted: 1 error during collection !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
===================================================== 1 error in 0.08s =====================================================

```

여기에 사용한 테스트 코드의 실행 순서는 다음과 같다.

1. 임시 컨테이너 busybox 실행(테스트 대상 컨테이너가 없으면 안되므로)
2. '모든 도커 컨테이너 삭제' 함수 실행
3. 이후 해당 호스트에서 현재 실행 중인 컨테이너의 갯수가 0이면 테스트 통과, 0이 아니면 테스트 실패

오류 내용을 요약해보면 다음 오류가 핵심 문구임을 알 수 있다.

```
raise OSError(
E   OSError: pytest: reading from stdin while output is captured!  Consider using `-s`.
```

테스트를 위해 출력이 캡쳐되는 와중에 `stdin`에서 입력을 받고있다는 것이다.

## Troubleshooting

처음 이 오류의 발생 원인을 생각해보았을 땐 pytest 자체에 테스트 대상 함수 혹은 테스트 대상 함수의 작동 순서 등을 정하는 명령어가 있으며, 이를 필히 지정해줘야 정상적인 테스트가 진행되는 줄 알았다.
왜냐햐면 테스트코드 자체에서 컨테이너를 삭제하는 함수만을 불러왔는데 사용자에게 모든 컨테이너를 정말 삭제할 것인지 묻는 `ask_user` 함수가 실행되고 있었으며, 그로 인해 문제가 발생했기 때문이다.

이를 해결하기위해 관련 내용을 검색해보고, [pytest tutorial](https://www.tutorialspoint.com/pytest/index.htm)에서 예전에 pytest 사용법을 공부할 때 놓친
부분이 있나 살펴보았다. 살펴보는 과정에서 `Fixture`나 `Grouping the Tests` 등의 내용은 유용하게 배웠지만, 이러한 내용들이 발생한 문제의 해결책은 아니었다.

그러던 중 예전에 Python3로 작성된 코드를 살펴볼 때 `if __name__ == '__main__'`이 공통적으로 들어가있는 것을 확인하, 이것의 역할이 궁금해 검색해봤던 기억이 났다. 위 코드와 관련된
내용을 설명하고 있는 [공식 문서](https://docs.python.org/3/library/__main__.html)를 찾아보니 핵심 역할은 다음과 같았다.

> 1. the name of the top-level environment of the program, which can be checked using the `__name__ == '__main__'`
     expression; and
>2. the `__main__.py` file in Python packages.

이 두 가지 설명만으론 실제로 어떤 역할을 수행하는지 감이 잘 오지않는다. 이후 스크롤을
내려보니 [Idiomatic Usage](https://docs.python.org/3/library/__main__.html#idiomatic-usage) 부분부터 필요한 내용을 설명하고 있다. 설명에 사용한 예시
상황마저도 지금 상황처럼 테스트 코드를 작성하는 과정에서 발생하는 의도치않은 코드 실행을 예로 들고있다. 문서가 설명하기를 외부 파일로부터 호출되는 경우 실행을 원치 않는
코드는   `if __name__ == '__main__'` 코드 뒤 두어 편리하게 실행을 방지할 수 있다고 말한다. 처음 어떤 개념을 공부할 땐 공식 문서가 너무 많은 내용들을 자세히 다루고있어 읽기 꺼려지는
순간이 있다. 그러나 현재 상황처럼 Troubleshooting이 필요할 때 가장 정확한 답을 제공하는 것 또한 공식문서이다.

이번 Troubleshooting 과정에서 또 한 가지 알게된 것이 Python3의 경우 단순히 외부 함수를 호출하면 호출 대상 함수 외에도 **해당 파일을 직접 실행**하는 것 처럼 동작한다는 점이다. 이를 사전에
알고있었으면 수고를 덜었을 텐데, 테스트 코드를 실행하는 과정에서 알게됐다. 이런 상황을 마주하지 못해 처음엔 왜 오류가 발생하는 지 이해하지 못했다. 또한, 이러한 상황을 마주한적이 없으니  `'__main__'`
를 처음 배울 때도 그 의미나 용도를 제대로 파악하지 못했다.

그러나, 테스트 코드 작성과정에서 이렇게 직접 Troubleshooting을 수행하고나니 `if __name__ == '__main__'`의 필요성과 용도를 제대로 깨우칠 수 있었다.

이후에 코드를 수정하고 `pytest -v`를 통해 테스트를 다시 수행하 다음 처럼 정상적인 테스트 결과를 확인할 수 있었다.

```shell
====================================================================================== test session starts ======================================================================================
platform linux -- Python 3.10.4, pytest-7.1.2, pluggy-1.0.0 -- /usr/bin/python3
cachedir: .pytest_cache
rootdir: /media/sf_src/automation-scripts/scripts/docker
collected 1 item

remove_all_docker_containers_test.py::test_remove_all_docker_containers PASSED                                                                                                            [100%]

====================================================================================== 1 passed in 27.12s =======================================================================================
```

**요약 및 배운점**

> 1. Python3는 외부 함수를 호출할 경우, 해당 함수가 정의된 파일 자체를 실행하는 것과 같은 동작을 수행한다.
> 2. `if __name__ == '__main__'`은 직접 실행될 때와 외부 파일로 호출될 때의 작동 방식을 구분짓는 역할을 한다.
> 3. Troubleshooting이 필요할 땐 공식 문서를 살펴보자.
