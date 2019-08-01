---
title: "[Python] 알아두면 좋은 itertools 함수들 Part 1"
categories:
  - Python
tags:
  - python
  - itertools
---

- ### Refer
     - ##### [데이터 분석에 피가 되는 itertools 익히기](https://hamait.tistory.com/803)

     - ##### [Python 201: An Intro to itertools](https://www.blog.pythonlibrary.org/2016/04/20/python-201-an-intro-to-itertools/)

     - ##### [Python 공식문서](https://docs.python.org/3/library/itertools.html)

---

1. ### count(start, step)
    count 함수는 반복을 멈추기 위한 조건이 없다. 

    ```python
    for i in count():
        print(i)
    ```
    위 코드는 무한히 루프를 돈다. 인자를 넣지 않으면 default로 인자 각각 1로 정해진다. 즉 1부터 1씩 증가하는 무한루프를 실행하게된다.
    ```python
    for (num, i) in zip(count(5, 10), ['a','b','c']):
        print(num, i)
    ```
    결과: 5 a 15 b 25 c 

---

2. ### cycle()
    cycle함수는 순환가능한 객체의 요소를 무한히 반복한다.
    ```python
    arr = ['hello']
    for i in cycle(arr):
        print(i)
    ```
    결과는 hello가 무한히 콘솔에 찍히게 된다.

    ```python
    for i in cycle('hello')
        print(i)
    ```
    결과는 h e l l o h e l l o .....

----

3. ### repeat(obj, time)
    인자로 object와 반복할 숫자를 넣어주면 된다. time인자를 넣지 않으면 무한 반복되도록 구현되어있다.
    ```python
    list(repeat('hi', 5))
    ```
    결과 : ['hi', 'hi', 'hi', 'hi', 'hi']

    ```python
    >> list(map(pow, range(10), repeat(2)))
    ```
    결과 : [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
