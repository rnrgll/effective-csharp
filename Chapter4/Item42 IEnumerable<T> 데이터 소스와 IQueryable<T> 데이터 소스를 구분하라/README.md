# 42\. `IEnumerable<T>` 데이터 소스와 `IQueryable<T>` 데이터 소스를 구분하라

`IEnumerable<T>`와 `IQueryable<T>`는 겉보기에는 같은 LINQ 메서드(`Where`, `Select`, `OrderBy` 등)를 제공하지만,

*   **쿼리가 실행되는 위치**
*   **데이터를 가져오는 방식**
*   **성능 특성**

이 근본적으로 다르기 때문에 **데이터 소스의 성격(로컬 vs 원격)에 맞는 인터페이스를 사용해야 한다.**

## 42.1 IQuerayable<T> 과 AsEnumerable() 호출의 차이

### IQuerayable<T>

*   쿼리는 표현식 형태로 누적된다.
*   최종 실행 시 LINQ 제공자가 전체 쿼리를 하나의 SQL로 합성한다.
*   DB에 **한 번만 요청**하고, DB에서 수행한 결과만 가져온다

```csharp
var q =from cin dbContext.Customers
			where c.City =="London"
			select c;
			
			
var finalAnswer =from cin q
			orderby c.Name
			select c;
```

*   `q`도 `IQueryable<Customer>`고 `finalAnswer`도 `IQueryable<Customer>`인 상태로 유지되면, 제공자(LINQ to SQL/EF)가 **where+orderby를 합쳐서 SQL 하나로 만든다.**
*   DB에 **한 번만 요청**하고, DB에서 필터링+정렬까지 끝낸 결과만 가져온다.

### AsEnumerable()

*   `AsEnumerable()` 을 호출하는 순간 쿼리는 `IEnumerable<T>`로 전환된다.
*   원격 쿼리 합성이 끊겨 더 이상 SQL로 번역되지 않는다.
*   이후 연산은 **LINQ to Object로 로컬 실행**된다.
*   결과는 같아도 성능은 크게 달라질 수 있다. (특히 정렬/필터/그룹/집계)

```csharp
var q = (from cin dbContext.Customers
where c.City =="London"
select c).AsEnumerable();  // IEnumerable<T>로 변환

// 여기서부터 로컬 실행
var finalAnswer =from cin q
orderby c.Name
select c;
```

⇒ 내부 매커니즘의 차이는 **38.2 LINQ 처리 방식의 차이 참고**

## 42.2 IQueryable의 장점/제약

### 장점

*   쿼리가 데이터가 있는 곳(DB)에서 실행된다.
*   필터링, 정렬, 집계가 원격에서 수행되므로
    *   네트워크 전송량 감소
    *   메모리 사용 감소
    *   전반적인 성능 향상

⇒ 대량의 데이터 처리에 유리하다.

### 제약

*   `IQueryable<T>`는 **제공자가 이해할 수 있는 연산만** 번역 가능하다.
    
*   쿼리 내부에서:
    
    *   임의의 C# 메서드 호출
    *   제공자가 지원하지 않는 연산
    
    을 사용하면 **주의 시점(순회 시점)에 번역 실패 예외가 발생한다.**
    

⇒ 즉, **표현력이 제한된다.**

성능과 효율이 중요한 경우 `IQueryable<T>` 을, 표현력이나 안전성이 더 중요한 경우 `AsEnumerable()` 로 전환 후 로컬 실행을 하면 된다.
