## 24. 베이스 클래스나 인터페이스에 대해서 제너릭을 특화하지 말라

### 컴파일러가 메서드를 선택하는 기준

* 오버로딩된 메서드가 여러 개일 때, 컴파일러는 '가장 정확하게 일치하는' 메서드를 최우선으로 선택한다.

* 제네릭 메서드는 요청된 타입에 맞춰 정확히 일치하는 메서드 코드를 생성해 내기 때문에,\
  부모 클래스나 인터페이스로의 암시적 형변환이 필요한 메서드보다 우선순위가 높다.

* 예시
  * 클래스
    ```csharp
    public interface IMessageWriter
    {
        void WriteMessage();
    }
    
    public class MyDerived : MyBase, IMessageWriter
    {
        void IMessageWriter.WriteMessage() 
    		->; WriteLine("Inside MyDerived.WriteMessage");
    }
    
    public class AnotherType : IMessageWriter
    {
        public void WriteMessage() 
        -> WriteLine("Inside AnotherType.WriteMessage");
    }
    ```

    ```csharp
    class Program
    {
        static void WriteMessage(MyBase b)
        {
            WriteLine("Inside WriteMessage(MyBase)");
        }
    
        static void WriteMessage&lt;T&gt;(T obj)
        {
            Write("Inside WriteMessage&lt;T&gt;(T): ");
            WriteLine(obj.ToString());
        }
    
        static void WriteMessage(IMessageWriter obj)
        {
            Write("Inside WriteMessage(IMessageWriter): ");
            obj.WriteMessage();
        }
    }
    ```
    * 메인함수
    ```csharp
    static void Main(string[] args)
    {
    	MyDerived d = new MyDerived();
    	WriteLine("Calling Program.WriteMessage");
    	WriteMessage(d);
    	WriteLine();
    
    	WriteLine("Calling through IMessageWriter interface");
    	WriteMessage((IMessageWriter)d);
    	WriteLine();
    
    	WriteLine("Cast to base object");
    	WriteMessage((MyBase)d);
    	WriteLine();
    
    	WriteLine("Another Type test:");
    	AnotherType anObject = new AnotherType();
    	WriteMessage(anObject);
    	WriteLine();
    
    	WriteLine("Cast to IMessageWriter:");
    	WriteMessage((IMessageWriter)anObject);
    }
    ```
    * 실행결과
      ```
      Calling Program.WriteMessage 
      Inside WriteMessage<T>(T): Item14.MyDerived
      ```
      : `Derived` 타입 기반의 제너릭 코드 생성 및 호출
      </br></br>

      ```
      Calling through IMessageWriter interface 
      Inside WriteMessage(IMessageWriter): Inside MyDerived.WriteMessage 
      ```
      : 인터페이스(`IMessageWriter`) 타입을 인자로 받는 함수 호출\
      → 내부에서 다형성에 의해 실제 인스턴스(`MyDerived`)의 함수 실행
      </br></br>
      
      ```
      Cast to base object 
      Inside WriteMessage(MyBase) 
      ```
      : 부모 클래스(`MyBase`) 타입을 인자로 받는 함수 호출
      </br></br>
      
      ```
      Anther Type test: 
      Inside WriteMessage<T>(T): Item14.AnotherType
      ```
      : `AnotherType` 타입 기반의 제너릭 코드 생성 및 호출
      </br></br>
      
      ```
      Cast to IMessageWriter: 
      Inside WriteMessage(IMessageWriter): Inside AnotherType.WriteMessage
      ```
      : 인터페이스(`IMessageWriter`) 타입을 인자로 받는 함수 호출\
      → 내부에서 다형성에 의해 실제 인스턴스(`AnotherType`)의 함수 실행
          


### 제너릭 특화 (Specialization)

* 제너릭 메서드 내부에서 런타임 형변환(`is`, `as`)과 분기문을 사용해, 특정 타입에 대해 최적화된 별도의 동작을 수행하도록 코드를 작성하는 방식

* 하지만 제네릭을 사용하면서 내부에서 다시 타입을 구분하는 것은 제네릭의 설계 목적인 범용성과 맞지 않기 때문에, 일반적으로 좋지 않다.

* 예시

  ```csharp
  public class Processor<T>
  {
      public void Process(T input)
      {
          if (input is string)
          {
              Console.WriteLine($"[특화] 문자열 처리: {input}");
          }
          else if (input is IMessageWriter)
          {
              ((IMessageWriter)input).WriteMessage();
          }
          else
          {
              // 일반 처리
              Console.WriteLine($"[일반] 타입 처리: {input.GetType().Name}");
          }
      }
  }
  ```



### 제너릭 특화의 단점과 대안

* 컴파일 타임에 결정될 수 있는 타입 정보를 런타임으로 미룬다.

  * 런타임 타입 검사 비용이 발생한다.

  * 잘못된 타입 처리로 인한 런타임 오류 가능성이 생긴다.

  * 제너릭의 장점인 컴파일 타입 안정성이 훼손된다.

* 유지보수가 어렵다.

  * 특화할 타입이 늘어날수록 `if-else` 분기가 계속 증가하고 코드가 복잡해진다.

  * 특히 인터페이스인 경우, 인터페이스를 구현한 모든 클래스를 수정해야 한다.

* 올바른 대안

  * 특정 타입에 대해 다른 동작이 필요하다면, 제너릭 특화 대신 **메서드 오버로딩**을 사용하여 컴파일러가 가장 적합한 메서드를 선택하도록 하는 것이 좋다.

  * 제너릭 특화는 정말 필요한 경우에만 제한적으로 사용한다.

    <figure><p>📌 컴파일 타임 타입 정보만으로는 충분하지 않고, 런타임 확인이 의미 있고 성능상 문제되지 않을 때만 사용한다. (ITEM 19 참고)</p></figure>

     
