
**Coroutines**이란 비동기 프로그래밍을 지원하는 기능으로, 하나의 thread 내에서 함수의 실행을 중단하고 나중에 다시 재개할 수 있는 유용한 기능이다. (reference: https://realpython.com/introduction-to-python-generators/#understanding-the-python-yield-statement, https://lerner.co.il/2020/05/08/making-sense-of-generators-coroutines-and-yield-from-in-python/, https://peps.python.org/pep-0342/, https://www.educative.io/blog/python-concurrency-making-sense-of-asyncio)

**python에서 3가지 유형의 Coroutines을 사용할 수 있다.**

1. **Simple coroutines:** traditional generator coroutine (no async io)
2. **Generator based coroutines:** async io using legacy asyncio implementation
3. **Native coroutines:** async io using latest async/await implementation

각 유형을 하나씩 살펴보자. 
- - - 

### **Simple coroutines**
    
    python의 generator 함수는 값을 얻기 위한 실행을 멈췄다가 필요할 때 다시 재개할 수 있다는 점에서 coroutines과 비슷하지만 역으로 인자를 받을 수 있는 기능을 제공하지 않아 완전하지 않았다. 하지만, 다음의 기능들이 추가됨으로써 generator가 비동기 프로그래밍을 지원할 수 있게 되었다. 
    
    1. 선언문으로 동작되었던 yield가 표현식으로도 가능하게 되었다. 다음의 예시를 보고 이해해보자.
        
```python
def myfunc():
    x = ''
    while True:
        print(f'Yielding x ({x}) and waiting…')
        x = yield x
        if x is None: 
            break
        print(f'Got x {x}. Doubling.')
        x = x * 2
```
    
    위의 코드에서 보면 ‘x = yield x’ 에서 yield를 x에 assign하고 있는 것을 볼 수 있다. 이 뜻은 yield가 x에 값을 제공할 수 있어야 한다는 것인데 어떻게 할 수 있을까? 
        
    2. send()
        
        send() method를 사용하게 되면 위의 yield로 생성된 generator에 원하는 값을 보내줄 수 있다. 위의 코드를 send()와 함께 사용한 예시이다. 
        
```python
>>> g = myfunc()
>>> next(g)
Yielding x () and waiting...
''
# yield x가 값을 받을 때까지 멈춰있다
>>> g.send(10)
Got x 10 Doubling.
Yielding x (20) and waiting...
20
# yield x가 값을 받을 때까지 멈춰있다.
>>> g.send(123)
Got x 123 Doubling.
Yielding x (246) and waiting...
>>> g.send(None)
StopIteration
```
        
    3. throw()
        
    generator가 중단됐던 지점에 exception을 발생시킨다. 다음은 그 예시이다. 
        
```python
def infinite_palindromes():
    num = 0
    while True:
        if is_palindrome(num):
            i = (yield num)
            if i is not None:
                num = i
        num += 1

pal_gen = infinite_palindromes()
for i in pal_gen:
    print(i)
    digits = len(str(i))
    if digits == 5:
        pal_gen.throw(ValueError("We don't like large palindromes"))
    pal_gen.send(10 ** (digits))

##### output
11
111
1111
10101
Traceback (most recent call last):
    File "advanced_gen.py", line 47, in <module>
    main()
    File "advanced_gen.py", line 41, in main
    pal_gen.throw(ValueError("We don't like large palindromes"))
    File "advanced_gen.py", line 26, in infinite_palindromes
    i = (yield num)
ValueError: We don't like large palindromes
```
        
    4. close()
        
    generator를 멈추게 하는 기능이다. 특별히 무한한 sequence generator를 제어하는데 유용하다. 다음은 위의 코드를 close()로 변경해서 실행해 보았다. 

```python
pal_gen = infinite_palindromes()
for i in pal_gen:
    print(i)
    digits = len(str(i))
    if digits == 5:
        pal_gen.close()
    pal_gen.send(10 ** (digits))

#### output
11
111
1111
10101
Traceback (most recent call last):
    File "advanced_gen.py", line 46, in <module>
    main()
    File "advanced_gen.py", line 42, in main
    pal_gen.send(10 ** (digits))
StopIteration
```
        

### **Generator based coroutines**
    
    python에서는 generators와 coroutines으로 사용되기 위한 generators를 구별하기 시작했다. 이러한 coroutines을 generator based coroutines 이라고 하고 함수를 정의할 때 @asynio.coroutine을 사용하도록 장려하고 있다. 중요한 점은 ‘yield from’의 사용이다. 
    
- **yield from**
        
    기존의 generator는 yield를 사용해 호출자에게 값을 반환했고, 이를 가지고 수동으로 반복하고 모든 결과를 다시 yield 해야 하는 비효율적인 방식을 가졌다. 
    
    이를 해결하기 위해 도입된 것이 yield from이고, 다른 제너레이터에게 제어권을 완전히 위임해서 이 서브제너레이터가 완료될 때까지 실행이 된다. 그 결과는 호출자에게 직접 전달되며 호출하는 제너레이터의 개입 없이 이루어진다. 또한, 보낸 값과 exception도 현재 실행 중인 서브제너레이터로 직접 전달된다. 이를 위한 예시를 자세히 살펴보자. 
    
    다음은 yield from을 사용하지 않은 코드이다.
    
    ```python
    def wrapper(data):
        for one_item in data:
            yield one_item
    
    >>> g = wrapper('abcd')
    >>> list(g)
    ['a', 'b', 'c', 'd']
    
    # g인 generator가 반본적으로 for loop안에서
    # 다음 element를 계속 부르고있다.
    ```
    
    이 코드를 yield from을 사용해서 편하게 만들 수 있다. yield from은 호출된 generator를 for loop로 돌리듯 반복적으로 호출해주는 기능이라고 보면 될 것 같다.
    
    ```python
    def wrapper(data):
        yield from data
    
    >>> g = wrapper('abcd')
    >>> list(g)
    ['a', 'b', 'c', 'd']
    
    # yield from은 호출하는 g generator의 개입 없이
    # subgenerator인 data가 실행을 완료할 때까지 호출한다.
    
    ```