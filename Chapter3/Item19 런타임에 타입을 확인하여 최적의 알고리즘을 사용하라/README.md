## 19. 런타임에 타입을 확인하여 최적의 알고리즘을 사용하라

### 제너릭의 한계

* 제네릭은 코드의 재사용성을 높여주지만, 컴파일 타임에 컴파일러가 `T`에 대해 알고 있는 정보(제약 조건) 내에서만 동작한다.

* 제약 조건이 부족할 때, 모든 타입에 적용 가능한 보수적인 알고리즘을 사용할 수밖에 없다.
  → 성능 저하가 발생한다.

###  

### 해결 방법: 런타임 타입 특수화

* 제너릭의 내부 구현에서 런타임의 구체적인 타입을 확인하고, 각 타입에 대해 최적화된 경로로 분기하는 방법이다.
  * 런타임 타입 확인 및 변환을 위해 `is`, `as` 연산자를 사용한다.
  * 예시 - `IEnumerable`과 `IEnumerator`
    * `IEnumerable<T>`
      * 순회 가능한 데이터 집합(Collection)임을 선언하는 인터페이스
      * `IEnumerator<T>` 객체를 반환하는 `GetEnumerator()` 메서드를 구현해야 한다.
    * `IEnumerator<T>`
      * 데이터 집합 내에서 현재 위치를 관리하고 순회하기 위한 인터페이스
      * 다음 멤버들을 구현해야 한다.
        * `bool MoveNext()`: 커서를 다음 요소로 이동 / 성공 시 true, 끝이면 false 반환
        * `T Current`: 현재 커서가 가리키는 요소 반환
       
  * 컬렉션(`IEnumerable<T>`)를 역순으로 순회하는 코드
    ```csharp
      using System.Collections;
      using System.Collections.Generic;
      
      public sealed class ReverseEnumerable&lt;T&gt; : IEnumerable&lt;T&gt;
      {
          private readonly IEnumerable&lt;T&gt; sourceSequence;
      
          public ReverseEnumerable(IEnumerable&lt;T&gt; sequence)
          {
              sourceSequence = sequence;
        }
    
        public IEnumerator&lt;T&gt; GetEnumerator()
        {
    		// IList: 랜덤 액세스 가능 &amp; Count 정보 O
            IList&lt;T&gt; originalList = sourceSequence as IList&lt;T&gt;;
            if (originalList != null)
            {
                return new ReverseListEnumerator(originalList);
            }
            
            // ICollection: Count 정보 O
            ICollection&lt;T&gt; originalCollection = sourceSequence as ICollection&lt;T&gt;;
            if (originalCollection != null)
            {
                return new ReverseCollectionEnumerator(originalCollection);
            }
    				
    		// 일반 IEnumerable&lt;T&gt;
            return new ReverseGeneralEnumerator(sourceSequence);
        }
    
        IEnumerator IEnumerable.GetEnumerator() =&gt; this.GetEnumerator();
    
    
    	// ============================================================================
        //  IList&lt;T&gt;용: 복사 필요 X, Count 정보 활용
        private class ReverseListEnumerator : IEnumerator&lt;T&gt;
        {
            private IList&lt;T&gt; _list;
            private int _index;
    
            public ReverseListEnumerator(IList&lt;T&gt; list)
            {
                _list = list;
                _index = list.Count; 
            }
    
            public bool MoveNext()
            {
                return --_index &gt;= 0;
            }
    
            public T Current =&gt; _list[_index];
    
            object IEnumerator.Current =&gt; Current;
            public void Dispose() { }
            public void Reset() { _index = _list.Count; }
        }
    
        // 일반 IEnumerable&lt;T&gt;용: 크기 모름 &amp; 복사 필요
        private class ReverseGeneralEnumerator : IEnumerator&lt;T&gt;
        {
            private IList&lt;T&gt; _items;
            private int _index;
    
            public ReverseGeneralEnumerator(IEnumerable&lt;T&gt; sequence)
            {
    		    List&lt;T&gt; list = new List&lt;T&gt;(); 
    		    foreach (var item in sequence) 
    		    {
    				list.Add(item);
    			}
                _items = list;
                _index = list.Count;
            }
    
            public bool MoveNext() =&gt; --_index &gt;= 0;
            public T Current =&gt; _items[_index];
    
            object IEnumerator.Current =&gt; Current;
            public void Dispose() { }
            public void Reset() { _index = _items.Count; }
    
      }
    }
    ```

* 장점

  * 타입별로 최적화된 코드를 수행할 수 있다.

  * 외부에서는 내부 구현을 몰라도 되며, 단순히 제네릭 메서드 하나만 호출하면 된다.

* 주의사항

  * 제너릭 내부에서 다양한 타입에 대해 분기를 처리하고 다른 알고리즘을 구현해야 하기 때문에 코드의 복잡성이 증가한다.

  * 런타임 타입 검사 및 형변환으로 인한 오버헤드가 발생한다.

    <figure><p>👨‍🏫 이 오버헤드가 적지는 않다.&nbsp;<br>그러나 내부적으로 로직을 분리했을 때 얻는 구조/성능적 장점이 더 크다면 활용하는 것이 좋다.&nbsp;&nbsp;</p></figure>

  * 유지보수 비용이 증가한다.
    : 새로운 타입이 추가될 때마다 제너릭 내부 구현을 수정해야 할 수도 있다.

 

 
