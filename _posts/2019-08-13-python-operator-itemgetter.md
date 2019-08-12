---
title: "[Python] Dictionary 와 Tuple 정렬하기"
categories:
  - Python
tags:
  - python
  - itemgetter
---

- ### Refer
     
    - ##### [튜플을 정렬하는 여러가지 방법](https://andrew0409.tistory.com/66?category=839769)

    - ##### [딕셔너리 정렬하기](https://kkamikoon.tistory.com/138)

---

- ### itemgetter

    ```python
    from operator import itemgetter

    test_tuple = [('choi',15), ('park',20), ('kim', 40), ('lee',10), ('cho', 70)]
    test_tuple.sort(key=itemgetter(1))
    print(test_tuple)

    #  [('lee', 10), ('choi', 15), ('park', 20), ('kim', 40), ('cho', 70)]
    ```
    위 코드처럼 튜플안에 있는 원소로 정렬할때 itemgetter()에 정렬 기준이될 인덱스를 넣어주면 쉽게 정렬할수 있다.
    itemgetter(1)은 튜플의 두번째 원소로 정렬시키겠다는 말이 된다. 0은 첫번째 원소
    itemgetter(1,0)을 넣게 되면 두번째원소로 정렬하고 그다음 첫번째 원소로 2차 정렬을 한다는 뜻이다.
    같은 이름이 있을때 나이 순으로 정렬한다고 생각할 수 있다.

    ```python
    test_dict1 = {10 : "B", 50 : "A", 30 : "D", 80 : "C"}
    test_dict2 = sorted(test_dict1.items(), key=itemgetter(1))
    print(test_dict2)

    # [(50, 'A'), (10, 'B'), (80, 'C'), (30, 'D')]
    ```
    딕셔너리에 입력된 key 와 value값들을 정렬해야 할 때 사용하면 유용하다.
    위 코드는 value로 정렬한 것이다. itemgetter에 0 을 넣으면 key 로 정렬하고 1은 value로 정렬한다.
---