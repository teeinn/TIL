프로그래밍 할 때, 우리는 함수에 parameter 값들을 넘겨준다. 밑의 예시를 일단 살펴보자. 

```python
def adder(x,y,z):
    print("sum:",x+y+z)

adder(10,12,13)

# output
sum: 35
```

위의 adder() 함수는 parameter 3개를 인자로 받는다. 그렇다면 만약에 5개의 인자를 전달한다면 어떻게 될까?

```python
def adder(x,y,z):
    print("sum:",x+y+z)

adder(5,10,15,20,25)

# output
TypeError: adder() takes 3 positional arguments but 5 were given
```

당연하게도 인자를 초과해서 받았다는 오류가 난다. 
    <br />
    <br />   

---
프로그래밍을 하다보면 우리는 몇 개의 인자를 전달해야 할지 불명확한 상황이 있을 것이다. 이럴 때 사용할 수 있는 것이 바로 *args와 **kwargs이다. 

1. *args - Non Keyword Arguments
2. **kwargs - Keyword Arguments 

하나씩 살펴보도록 하자. 
    <br />
    <br />
    <br />
    <br />


### *args
    
몇 개의 인자를 전달할지 불명확한 상황일 때, 그리고 키워드가 존재하지 않는 인자들을 전달할 때 사용한다. 이 인자들은 **tuple**로 전달이 된다. 다음의 예시를 보자.

```python
    def adder(*num):
    sum = 0
    
    for n in num:
        sum = sum + n

    print("Sum:",sum)

adder(3,5)
adder(4,5,6,7)
adder(1,2,3,5,6)

# output
Sum: 8
Sum: 22
Sum: 17
```
    

### **kwargs

인자의 개수가 정해지지 않았을 때, 그리고 키워드가 존재하는 인자를 보낼 때 쓰인다. 인자는 **dictionary**로 전달이 된다. 다음의 예시를 보면 확실히 감이 올 것이다. 
    
```python
def intro(**data):
    print("\nData type of argument:",type(data))

    for key, value in data.items():
        print("{} is {}".format(key,value))

intro(Firstname="Sita", Lastname="Sharma", Age=22, Phone=1234567890)
intro(Firstname="John", Lastname="Wood", Email="johnwood@nomail.com", Country="Wakanda", Age=25, Phone=9876543210)

# output
Data type of argument: <class 'dict'>
Firstname is Sita
Lastname is Sharma
Age is 22
Phone is 1234567890

Data type of argument: <class 'dict'>
Firstname is John
Lastname is Wood
Email is johnwood@nomail.com
Country is Wakanda
Age is 25
Phone is 9876543210
```