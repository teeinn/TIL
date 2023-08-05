python의 datatype에는 Numeric / Sequence Type / Set / Dictionary / Boolean / Binary Types (memoryview, bytearray, bytes) 가 있다. 하나씩 파악해보고 비교해보자. (reference: https://www.geeksforgeeks.org/python-data-types/) ![Alt text](python_datatypes.jpg)


- **Numeric Data**: numeric data type은 숫자 값이다. 값에는 Integer, Float, Complex number가 있다.

---


- **Sequence Data**: *순서가 있는 collection 데이터* 타입이다. String, List, Tuple이 이에 해당한다.
    - **String**: unicode characters를 나타내는 bytes arrays를 string이라고 한다. string은 하나 이상의 character들의 collection이라고 볼 수 있다. python에는 char 데이터 타입은 없고, string이 길이 1이면 character라고 보면 된다.
    - **List**: 리스트는 순서가 있는 collection 데이터 타입이다. python의 변수는 메모리에 저장된 값에 대한 정보를 담고있는 object를 가리키기 때문에, 리스트에 다양한 데이터 타입이 한번에 담길 수 있다.
        - python의 list는 dynamic array이다. 따라서, append시에 메모리 공간이 있으면 O(1)의 시간복잡도가 소요되지만, 공간이 없다면 현재 리스트 크기의 doubling을 해주어 이전 값들을 새롭게 할당한 공간에 O(n)으로 옮겨준다. 따라서 amortized O(1)이라고 할 수 있다.
        - **시간복잡도**:
            
            Append O(1)
            
            Pop last O(1)
            
            Insert O(n)
            
            Pop or Remove intermediate O(n)
            
            Get Item O(1) 
            
    - **Tuple:** 튜플은 순서가 있는 collection이고 중요한 점은 immutable하다. 즉, 한번 생성되고 나면 값을 변경할 수 없다는 말이다.
        - **시간복잡도**: list의 method 중에 mutable 하지 않은 method는 모두 사용 가능하고 시간복잡도도 동일하다.
    - **List vs Tuple**: 단 한가지의 차이점이라면 List는 생성된 후 element의 값 변경이 가능해 mutable하고, Tuple은 값 변경이 불가능한 immutable한 데이터 타입이다.  

---


- **Set**: set은 순서가 존재하지 않는 collection 데이터 타입이다. mutable하고 값의 중복을 허용하지 않는다.
    - **Set vs Tuple**: 둘다 collection type이다. set은 mutable하고 tuple은 immutable하다. set은 unordered collection 이지만, tuple은 ordered collection 이다. set은 중복을 허용하지 않지만, tuple은 중복을 허용한다.
    - **시간복잡도**:
        
        add O(1)
        
        check for item in set O(1)
        
        remove O(1)
        
        pop O(1)
        
        Union, Intersection, Difference O(len(A)+len(B))

---
    
        
- **Dictionary**: 순서가 없는 collection data type이고, key : value 쌍으로 저장된다. value에는 어떠한 데이터 타입의 값도 저장이 될 수 있고, 중복될 수 있으며 mutable하다. 그러나 key는 유일무의한 값이어야 하기 때문에 중복될 수 없고, immutable하다.
    - **시간복잡도**:
        
        Get Item O(1)
        
        Set Item O(1)
        
        Delete Item O(1)
        
        Iterate Over Dictionary O(n)
        
    - **dictionary의 insert 시에 메모리가 부족할 경우는 어떻게 될까?** dictionary 또한 list 와 마찬가지로 메모리가 부족할 경우 이전보다 2배 이상의 메모리를 할당하여 이전 메모리에서 새롭게 할당한 메모리로 값을 옮겨온다. 그 후 이전 메모리를 release하고 metadata에 정보를 업데이트 하는 과정을 거친다. 즉, worst case의 시간복잡도는 O(n)이다. 이런 경우를 최대한 줄이기 위해, python에서는 dictionary에 메모리를 할당할 때 애초에 널널하게 공간을 할당해준다. 그래서 방대한 데이터를 dictionary에 저장해야 한다면 메모리를 많이 잡아 먹을 수 있다.
    - **key 값에 들어갈 수 없는 자료형은?** 일단 key는 immutable하고 중복이 없어야 하기 때문에, int, float, string, boolean, tuple이 들어갈 수 있고, list, set, dictionary 처럼 mutable한 자료형은 사용될 수 없다.
        
        ```python
        #tuple이 hashmap의 key에 쓰인 예시
        coordinates = { (0,0) : 100, (1,1) : 200}
        coordinates[(1,0)] = 150
        coordinates[(0,1)] = 125
        
        print(coordinates)
        # {(0, 0): 100, (1, 1): 200, (1, 0): 150, (0, 1): 125}
        ```
        
    - **Hashmap (Dictionary)의 동작 원리**
        
        Direct-address table 방식은 불필요한 공간이 낭비 된다는 점과, key가 다양한 자료형을 담을 수 없다는 단점을 가지고 있었다. hashmap을 이를 보완하는데, hash function을 정의해 k를 대입하여 고유한 index 값을 뽑아냈다. 
        
    - **Hashmap (Dictionary)의 Collision과 해결방법**
        
        hash_function(k)를 거쳐 나온 해시값이 똑같은 경우를 collision이 발생했다고 한다. 따라서, collision 상황이 적도록 hash_function을 잘 설계해야 한다. 다음은 collision이 발생했을 때 해결하는 방법들을 설명한다. 
        
        1. **separate chaining**
            
            Linked list 또는 tree를 이용하는 방식이다. collision이 발생하면 linked list에 노드를 추가하여 데이터를 저장한다. 
            
            **Insert**: 서로 다른 두 key가 같은 해시값을 얻게 된다면 linked list에 node를 추가하여 데이터를 저장한다. 시간복잡도는 O(1)이다. 
            
            **Search**: 기본적으로 O(1)의 시간복잡도를 갖지만 worst case에는 O(n)의 시간복잡도를 갖는다. 
            
            **Deletion**: 삭제도 조회와 마찬가지로 O(1)의 기본적인 시간복잡도를 갖고 worst case의 경우 O(n)이 된다.
            
            기본적으로 Linked list를 이용하지만 위의 worst case 처럼 길이가 길어지면 Binary Search Tree를 이용해 O(log n)의 시간복잡도로 구성할 수 있다.
            
        2. **open addressing**
            
            비어있는 slot을 정해진 규칙에 따라 찾는 방법이다. separate chaining 방식은 새로 메모리를 할당하지만, open addressing은 추가적인 메모리를 사용하지 않으므로 상대적으로 메모리를 적게 사용한다. 크게 3가지 방법으로 나뉜다.
            
            - **Linear Probing(선형 조사법)**
                
                충돌이 발생한 해시 값으로부터 +1, +2, +3…만큼 일정하게 건너 뛰면서 빈 slot에 저장하는 방법
                
            - **Quadratic Probing(이차 조사법)**
                
                제곱수 +1^2, +2^2, +3^2…만큼 일정하게 건너 뛰면서 빈 slot에 저장하는 방법
                
                위의 2가지 방법은 충돌이 발생하지 않을 때까지만 일정하게 이동하기 때문에 특정 영역에 데이터가 몰리는 **clustering 현상**이 발생할 수 있다는 단점이 있다. 이 현상이 발생하면 평균적인 탐색 시간이 증가하게 된다. 
                
            - **Double Hashing(이중 해시)**
                
                clustering 문제가 발생하지 않도록 hash function을 2개를 정의해 하나는 처음 해시값을 얻을 때 사용하고, 또 다른 하나는 충돌이 발생할 때 일정하지 않는 이동폭을 얻기 위해 사용한다.