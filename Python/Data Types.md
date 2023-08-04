python의 datatype에는 Numeric / Sequence Type / Set / Dictionary / Boolean / Binary Types (memoryview, bytearray, bytes) 가 있다. 하나씩 파악해보고 비교해보자. (reference: https://www.geeksforgeeks.org/python-data-types/) ![Alt text](python_datatypes.jpg)

- - - 
- **Numeric Data**: numeric data type은 숫자 값이다. 값에는 Integer, Float, Complex number가 있다.
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
- **Set**: set은 순서가 존재하지 않는 collection 데이터 타입이다. mutable하고 값의 중복을 허용하지 않는다.
    - **Set vs Tuple**: 둘다 collection type이다. set은 mutable하고 tuple은 immutable하다. set은 unordered collection 이지만, tuple은 ordered collection 이다. set은 중복을 허용하지 않지만, tuple은 중복을 허용한다.
    - **시간복잡도**:
        
        add O(1)
        
        check for item in set O(1)
        
        remove O(1)
        
        pop O(1)
        
        Union, Intersection, Difference O(len(A)+len(B))
        
- **Dictionary**: 순서가 없는 collection data type이고, key : value 쌍으로 저장된다. value에는 어떠한 데이터 타입의 값도 저장이 될 수 있고, 중복될 수 있으며 mutable하다. 그러나 key는 유일무의한 값이어야 하기 때문에 중복될 수 없고, immutable하다.
    - **시간복잡도**:
        
        Get Item O(1)
        
        Set Item O(1)
        
        Delete Item O(1)
        
        Iterate Over Dictionary O(n)
        
    - **key 값에 들어갈 수 없는 자료형은?** 일단 key는 immutable하고 중복이 없어야 하기 때문에, int, float, string, boolean, tuple이 들어갈 수 있고, list, set, dictionary 처럼 mutable한 자료형은 사용될 수 없다.
    
    ```python
    #tuple이 hashmap의 key에 쓰인 예시
    coordinates = { (0,0) : 100, (1,1) : 200}
    coordinates[(1,0)] = 150
    coordinates[(0,1)] = 125
    
    print(coordinates)
    # {(0, 0): 100, (1, 1): 200, (1, 0): 150, (0, 1): 125}
    ```