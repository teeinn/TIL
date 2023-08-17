python은 모든 것을 object로 취급한다. 예를 들어, x = 2 코드가 존재한다고 하자. x가 가리키는 것은 메모리에 올려진 값 2가 아닌 그 메모리 값에 대한 정보가 저장된 pyobject이다. pyobject에는 데이터의 형, reference count, 메모리 주소 등의 정보가 저장된다. 이 상태에서 또 다른 변수에 값을 어떻게 복사하느냐에 따라 복사되는 것이 달라진다. 복사의 종류인 shallow copy와 deep copy를 알아보자.![Alt text](shallowcopy_deepcopy.jpg)

<br />
<br />

### **Shallow Copy**

python에서 얕은 복사를 하게 되면, 새로운 pyobject를 생성해서 기존 pyobject가 가리키고 있던 메모리 주소 값을 똑같이 가리키게 된다. 즉, 복사한 변수에 변경을 가하면, 기존 메모리에 올려진 값도 바뀐다. 얕은 복사를 하려면 copy() 함수를 사용할 수 있다. 다음의 예시를 보자. 

```python
import copy

original_list = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
shallow_copy = copy.copy(original_list)

# Modify the first sublist in the shallow copy
shallow_copy[0][0] = 0

# Verify that the original list was also modified
print(original_list)  
# Output: [[0, 2, 3], [4, 5, 6], [7, 8, 9]]

```

### **Deep Copy**

깊은 복사를 하게 되면, 새로운 pyobject를 생성하는 것 뿐만 아니라 메모리에 올려져 있던 값도 복사하여 새로운 메모리 주소 값에 저장된다. 따라서, 새로운 변수를 이용해 값을 변경하면, 기존 메모리에 올려진 값은 변화가 없고, 복사된 값만 수정된다. 

```python
import copy

original_list = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
deep_copy = copy.deepcopy(original_list)

# Modify the first sublist in the shallow copy
deep_copy[0][0] = 0

# Verify that the original list wasn't modified
print(original_list)  
# Output: [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
print(deep_copy)  
# Output: [[0, 2, 3], [4, 5, 6], [7, 8, 9]]

```