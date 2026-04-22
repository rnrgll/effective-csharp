## 3. 캐스트보다는 `is`, `as`가 좋다

### 일반 캐스트 `(T)obj`


```csharp
MyType t = (MyType)obj;
```


* 컴파일러가 컴파일 타임에 객체의 정적 타입 기준으로 형변환 가능 여부를 판단한다.

  <table><tbody><tr><th><p>단계</p></th><th><p>수행 주체</p></th><th><p>역할</p></th></tr><tr><td><p>컴파일 타임</p></td><td><p>컴파일러</p></td><td><p>“이 형변환 문법적으로 가능한가?”</p></td></tr><tr><td><p>런타임</p></td><td><p>CLR</p></td><td><p>“이 객체가 실제로 그 타입인가?”</p></td></tr></tbody></table>

* 결과

  * 컴파일러가 가능하다고 판단한 경우

    1. 코드를 생성하고, 실제 변환 성공 여부는 런타임에 CLR이 검사한다.

       <figure><p>❓ 컴파일러는 객체의 정적 타입만 알고, 런타임 타입을 추론하지는 못하기 때문에</p></figure>

    2. 런타임에 실제 타입 변환이 실패한 경우 `InvlalidCastException` 오류가 발생한다. (예외를 던진다)

  * 컴파일러가 실패한다고 판단한 경우 : 컴파일 에러(CS0030)가 발생한다.

* 사용자 정의 형변환 연산을 수행한다.

  <figure><p>📝&nbsp;사용자 정의 형변환</p><ul><li><p>클래스 내부에 <code>static operator</code>를 선언하여, 서로 다른 타입 간의 형변환 규칙을 직접 설계하는 기능<br>​​​​​​&nbsp;(<code>implicit</code>: 자동 변환, <code>explicit</code>: 캐스트 연산자 사용)</p></li><li><p><strong>원리</strong><br>: 컴파일러가 <strong>정적 타입</strong>을 기준으로 적절한 형변환 연산자를 찾아 코드를 연결한다.</p></li><li><p>주의점</p><ul><li><p>컴파일러는 런타임 타입을 추론하지 않으므로, 정적 타입만 보고 변환을 시도하다가 실제 객체 타입이 다르면 <code>InvalidCastException</code>이 발생한다.</p></li><li><p>사용자 정의 형변환 메서드는 <code>static </code>이어야 한다.&nbsp;<br>(인스턴스가 null일 때도 호출할 수 있어야하기 때문이다.)</p></li></ul></li></ul></figure>

  <figure><p>👨‍🏫&nbsp;사용자 정의 형변환은 코드의 동작을 예측하기 어려워지므로, 특별한 경우가 아니라면 지양하는 것이 좋다.</p></figure>


* `foreach` 문과 일반 캐스트

  * 제너릭이 아닌 `foreach`는 내부적으로 일반 캐스트를 사용한다.

  * 컬렉션 안에 다른 타입이 있으면 런타임에 `InvalidCastException`이 발생할 수 있다.

 

---

### `as` 연산자를 사용한 형변환

```csharp
MyType t = obj as MyType;

if (t != null)
{
    
}

```

* 런타임에 객체의 실제 타입을 검사한다.

* 객체가 그 타입 자체일 때와 그 타입을 상속한 타입일 때 형변환이 가능하다.

* 결과

  * 가능: 형변환

  * 불가능

    * 절대 불가능할 때 (값 타입으로 변환하거나, 상속 관계가 전혀 아닐 때)\
      : 컴파일 에러

    * 가능성이 있는데 불가능할 때 (object 타입/ 인터페이스 등)\
      : `null` 반환 (예외를 던지지 않는다.)

* 특징

  * 예외를 던지지 않기 때문에 비싼 `try-catch` 구문이 필요하지 않아 성능이 좋고, 가독성도 좋다.
    ```csharp
    // 일반 캐스트를 사용한 경우 ❌
    object obj = Factory.GetObject();
    try
    {
         MyType t;
         t = (MyType)obj;
         if(t != null)
            //t 사용
    }
    catch (InvalidCastException)
    {
        // 예외 처리
    }
    ```

    ```csharp
    // as 연산자를 사용한 경우 ✅
    object obj = Factory.GetObject();
    MyType t = obj as MyType;

    if(t != null)
    {
        // t사용
    }
    else
    {
        // 예외 처리
    }
    ```

    
  * 사용자 정의 형변환 연산을 수행하지 않는다.
      * 일반 캐스트와의 비교
         1. 일반 캐스트
  
         ```csharp
         t = (MyType)st;
         
         ```
           * 사용자 정의 형변환이 정의되어 있을 때, `st`의 타입에 따라 코드가 다르게 동작한다.

         2. `as` 연산
        
         ```csharp
         t = st as MyType;
         
         ```
         * `as`​​​​는 사용자 정의 형변환을 무시한다.
           → `st`가 어떤 타입으로 선언되도 항상 동일한 결과를 반환한다.
      
         * 이때 `st`가 `MyType`이나 이를 상속한 타입이 전혀 아니라면 컴파일 오류가 발생한다.

   
  *  nullable 타입(참조 타입)에 대해서만 사용 가능하다.

    ```csharp
    object o = Factory.GetValue();
    
    // ❌ 일반 값 타입 (컴파일 오류)
    int i = o as int; 
    
    // ✅ nullable 값 타입
    var i = o as int?;
    if (i != null)
    	Console.WriteLine(i.Value);
    ```

    * 값 타입을 사용하고 싶은 경우 nullable 값 타입을 사용해야 한다.

      <figure><p>🚨 이때 박싱이 발생할 수 있다.</p></figure>


---



### `is` 연산자

* 객체가 특정 타입이거나, 이를 상속한 타입인지 검사하고 `bool` 값을 반환한다.

* 예시

  ```csharp
  if (obj is MyType)
  {
  	MyType t = (MyType)obj;
  	t.Do();
  }
  
  if (obj is MyType t)
  {
      t.Do(); 
      // t는 이미 MyType으로 캐스트된 상태 (C# 7.0)
  }
  
  ```

 
