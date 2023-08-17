

python에서는 모든 것을 객체로 취급하기 때문에 함수 또한 다른 함수의 인자로 전달이 될 수 있다. 다음의 예시를 보자. 

```python
def inner_function():
	print("inner_function is called")
    
def outer_function(func):
	print("outer_function is called")
 	func()
   
outer_function(inner_function)
# outer_function is called
# inner_function is called
```

함수를 객체로 취급하다 보니 함수 객체를 return 해 줄 수 있다. 

```python
def outer_function():
	print("outer_function is called")
    
	def inner_function():
    		print("inner_function is called")
      
	return inner_function

returned_function = outer_function()
# outer_funciton is called
# inner_function 객체의 참조 값을 받았지만, 아직 호출은 하지 않았다. 

returned_function()
# inner_function is called
# 여기서 함수 객체의 호출이 이루어진다.
```
<br  />
<br  />
<br  />
함수를 인자로 취급할 수 있다는 점, 함수가 다른 함수를 정의할 수 있다는 점, 그리고 그 함수 객체를 return 할 수 있다는 점. 이 3가지가 decorator를 만들기 위한 조건이 된다. 데코레이터는 함수의 소스 코드를 수정하지 않고 함수의 동작을 변경할 수 있는 기능을 제공하여 기능 확장하는 방법을 제공한다. 

<br />
<br />
만약에 어떤 함수의 return 값에 1을 더하는 데코레이터 함수를 만든다고 가정해보자. 그럼 다음과 같이 코드를 짤 수 있다. 

```python
def add_one_decorator(func):
	def add_one():
    	value = func()
        return value + 1
	return add_one

def example_function():
	return 1
    
final_value = add_one_decorator(example_function)
# 여기서 add_one 함수의 객체를 돌려주고
print(final_value()) # 2
# 여기서 받은 함수 객체를 호출한다. 
```

위의 코드에서 볼 수 있듯이, example_function 자체를 수정하지 않았고, 다른 함수를 정의해서 기능을 확장시켰다. 이것이 데코레이터의 역할이다. 
<br />
python에서는 데코레이터 함수를 정의하기 위해 **@ syntax**를 사용한다. 이를 위의 코드에 사용해보면 다음과 같다. 

```python
@add_one_decorator
def example_function():
	return 1

# 위의 코드는 아래의 코드와 동일하다.
final_value = add_one_decorator(example_function)
```

**데코레이터가 붙은 example_function을 호출하게 되면 add_one 함수 객체가 반환이 된다.** 
<br />

이제 데코레이터 함수가 인자를 받는 방법을 알아보자. 예를 들어, 다음의 코드를 보면 add 함수에 인자 a, b를 전달하고 있다. 그러나 반환된 add_one 함수 객체는 아무런 인자도 받고 있지 않기 때문에 에러가 난다.

```python
def add_one_decorator(func):
	def add_one():
    	value = func()
        return value + 1
        
    return add_one
    
@add_one_decorator
def add(a,b):
	return a + b
    
add(1,2)
# TypeError: add_one_decorator.<locals>.add_one() takes 0 positional arguments but 2 were given
```
<br />

이제 인자를 전달할 수 있는 방법을 살펴보자. 다음의 코드에서 add_one 함수에 *args, **kwargs를 넘겨줌으로써 원하는 수의 non keyword 인자와 keyword 인자를 받을 수 있게 해주었다. 이 예시에서는 1, 2가 전달되기 때문에 (1,2)가 *args에 전달 될 것이다. *kwargs는 empty dictionary가 전달 될 것이다. 이런 방식으로 기존 함수에서 받았던 인자들을 데코레이터 함수의 인자로 똑같이 전달할 수 있다.

```python
def add_one_decorator(func):
	def add_one(*args, **kwargs):
		value = func(*args, **kwargs)
		return value + 1
	return add_one
     
@add_one_decorator
def add(a,b):
	return a+b
  
print(add(1,2)) # 4
```
<br  />
<br  />
<br  />
<br  />
<br />


**그렇다면, 데코레이터 함수는 실제 어디에 쓰이게 될까?** 

- ### **Logging**
    
    규모가 큰 애플리케이션을 구축할 때, 어떤 함수가 실행되었고, 인자가 무엇이 전달되었고, return 값은 무엇이었는지 기록을 하면, 나중에 troubleshooting 시에 큰 도움이 될 수 있다. 또한 프로그램의 상태를 보는 것에도 도움이 된다. 따라서, 데코레이터 함수를 정의해서 어떤 함수의 로그 파일을 저장하는 방식으로 사용한다. 다음의 예시를 보자. 
    
    ```python
    import logging
    
    def function_logger(func):
        logging.basicConfig(level = logging.INFO, filename="main.log")
        def wrapper(*args, **kwargs):
            result = func(*args, **kwargs)
            logging.info(f"{func.__name__} ran with positional arguments: {args} and keyword arguments: {kwargs}. Return value: {result}")
            return result
        return wrapper
    
    @function_logger
    def add_one(value):
        return value + 1
    
    print(add_one(1))
    
    # 다음은 main.log에 저장될 정보이다.
    INFO:root:add_one ran with positional arguments: (1,) and keyword arguments: {}. Return value: 2
    ```
    
- ### **Caching**
    
    애플리케이션에서 특정 함수를 반복적으로 호출하는 일이 생긴다면 이를 캐시화 해서 값을 재사용 하는 것이 훨씬 효율적일 것이다. 같은 함수가 호출되고, 같은 인자가 전달되며, 같은 값이 반환된다면 이를 캐시화해서 다음에 똑같이 호출됐을 때, 같은 값을 돌려주는 것이 좋다. 이를 위해 사용하는 것이 데코레이터 **@lru_cache** (from functools module)이다.
    
    ```python
    from functools import lru_cache
    
    @lru_cache
    def fibonacci(n):
        if n <= 1:
            return n
        return fibonacci(n - 1) + fibonacci(n - 2)
    ```
    
    위의 코드에서 피보나치 함수는 n을 인자로 받는데, 이미 한번 호출되었던 인자라면 cache화해서 다시 계산할 필요가 없게 한다. 

<br  />
<br  />

    

reference: https://www.freecodecamp.org/news/python-decorators-explained/