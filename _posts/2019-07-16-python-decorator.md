---
title: "[Python] Decorator 이해하기"
categories:
  - Python
tags:
  - python
---


- ### Refer
     - ##### [파이썬 - 데코레이터](https://niceman.tistory.com/168)

     - ##### [python decorator 어렵지 않아요](https://bluese05.tistory.com/30?category=559959)


- ### Decorators
    wrapper 로써 동작하며 원본 함수를 수정할 필요없이 원본함수의 실행 전/후의 동작을 변경할 수 있다.
    부가적이면서 반복적인 로직을 데코레이터로 선언하면 사용이 매우 자유로워진다.
    
    예를들면 아래와 같은 간단한 함수가 있다.
    ```python
    def function_1():
        print("Called function")
    ```

    이 함수에 동작시간이 얼마나 걸렸는지 확인하기 위한 타이머 기능을 넣는다고 하면
    ```python
    import time

    def function_1():
        start = time.time()
        print("Called function")
        print("{}초 걸림".format(time.time() - start))
    ```

    위처럼 코드를 추가하게 된다. 그런데 추가해야될 로직이나 함수가 많아지면 코드가 지저분해지며 가독성도 떨어지게 된다.
    ```python
    import time
    
    def function_1():
        start = time.time()
        print("Called function1")
        print("{}초 걸림".format(time.time() - start))
    
    def function_2():
        start = time.time()
        print("Called function2")
        print("{}초 걸림".format(time.time() - start))
    
    def function_3():
    	start = time.time()
        print("Called function3")
        print("{}초 걸림".format(time.time() - start))
    
    ....
    ```

- ### Decorator 사용

    위 코드에 데코레이터를 사용해보자.
    ```python
    import time

    def timer(func):
        def wrapper():
            start = time.time()
            func()
            print("{}초 걸림".format(time.time() - start))
        return wrapper


    @timer
    def function_1():
        print("Called function1")

    @timer
    def function_2():
        print("Called function2")

    @timer
    def function_3():
        print("Called function3")
    ```
    데코레이터 함수로 정의해 재사용 함으로써 가독성과 직관성이 훨씬 좋아진 것을 볼 수 있고, 사용도 매우 간편하다.

- ### 언제 사용하는가?
  보통 로깅, 유효성 검사, 권한처리, 성능테스트 등으로 사용하기 유용하다.