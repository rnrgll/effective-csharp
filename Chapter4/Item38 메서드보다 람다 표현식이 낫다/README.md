

# 38\. 메서드보다 람다 표현식이 낫다

## 38.1 중복 코드를 메서드로 분리

LINQ 쿼리에서 동일한 조건이 반복되는 경우, 일반적인 리팩터링 원칙에 따라 조건을 메서드로 분리하고 싶어진다.

중복 코드를 메서드로 분리하면 ‘재사용성과 유지보수성이 향상’되는게 일반적이지만, LINQ에서는 LINQ 쿼리 처리 방식 때문에 다르다.

### 예시 코드

```csharp
var allEmployees = FindAllEmployees();

// 20년 이상 근속자
var earlyFolks =
		from e in allEmployees
		where e.Classification == EmployeeType.Salary  // 중복
		where e.YearsOfService >=20
		where e.MonthlySalary <4000  // 중복
		select e;

// 20년 미만 근속자
var newest =
		from ein allEmployees
		where e.Classification == EmployeeType.Salary  // 중복
		where e.YearsOfService <20
		where e.MonthlySalary <4000 // 중복
		select e;
```

중복된 where의 조건 부분을 메서드로 분리하면 다음과 같다.

```csharp
private static bool LowPaidSalaried(Employee e) =>
    e.MonthlySalary <4000 &&
    e.Classification == EmployeeType.Salary;

var earlyFolks =
		from ein allEmployees
		where LowPaidSalaried(e) && e.YearsOfService >=20
		select e;
		
var newest =
		from ein allEmployees
		where LowPaidSalaried(e) && e.YearsOfService < 20
		select e;
```

## 38.2 LINQ 처리 방식의 차이

LINQ 쿼리는 **<span class="fontColorRed">시퀀스 타입에 따라 두 가지 방식으로 처리된다.</span>**

1.  쿼리 표현식 내의 람다 표현식을 **델리게이트로 변환(LINQ to Objects)**하여 실행

2.  람다를 **표현식 트리**로 분석 후 **SQL을 생성**한다. **(LINQ to SQL)**

### 38.2.1 IEnumerable<T> — LINQ to Objects

*   **메모리 내 컬렉션**(List, Array 등)을 대상으로 쿼리를 수행한다.
*   람다 표현식을 전달하면 → **델리게이트(실제 실행 코드)로 컴파일한다.**
*   필터링/정렬 등은 C# 실행 엔진이 수행한다.

### 38.2.2 IQueryable<T> — LINQ to SQL

*   **원격 데이터 소스(DB)** 쿼리를 수행하기 위한 확장 인터페이스다.
*   `IEnumerable<T>`를 상속하지만 `Expression` 기반의 `Expression<Func<T, bool>>` 형태의 람다를 받는다.
*   **람다 표현식을→ 표현식 트리(Expression<TDelegate>)로 캡처하여 SQL로 변환한다.**
*   C#은 **쿼리를 만들기만 하고,** 실제 실행은 DB(혹은 다른 원격 소스)가 수행한다.

```csharp
source.Where(e => e.Id > 10)
```

1.  source가 IEnumerable → Where는 Enumerable.Where 호출

*   `Func<T, bool>` 타입(델리게이트)으로 받음
*   C#에서 직접 e.Id > 10을 실행
*   메모리 컬렉션에서 필터링됨

2.  source가 IQueryable → Where는 Queryable.Where 호출

*   `Expression<Func<T, bool>>` 타입(표현식 트리)으로 받음
*   e.Id > 10이라는 구조를 트리 형태로 저장
*   이 구조를 기반으로 SQL을 생성하여 DB에서 실행


> ⇒ IEnumerable 기반 LINQ는 C# 코드로 실행되는 LINQ to Objects가 된다.
> ⇒ IQueryable 기반 LINQ는 표현식 트리를 SQL로 번역해서 DB에서 실행하는 LINQ to SQL가 된다.



### 38.2.3 메서드 호출의 문제점

표현식 트리는 구조 정보만을 담고 있기 때문에 <span class="fontColorRed"><strong>메서드 호출(Expression.MethodCall)을 SQL로 변환할 수 없다.</strong></span>

쿼리 구문 내에 메서드 호출이 등장하면, 표현식 트리에는 `MethodCallExpression` 노드로 기록된다. 

하지만 표현식 트리는 메서드의 **구현 로직을 포함하지 않으므로**, LINQ to SQL 엔진은 해당 메서드를 T-SQL로 어떻게 번역해야 하는지 알 수 없다.

따라서 사용자 정의 메서드 호출이 포함된 표현식은 SQL 변환 과정에서 예외가 발생한다.

```csharp
where LowPaidSalaried(e)
```

이건 SQL로 번역하면:

```csharp
WHERE LowPaidSalaried(e)
```

→ 이런 SQL은 존재하지 않는다

→ 런타임 예외 발생한다.

## 38.3 IQueryable 확장 메서드 사용하기

조건 로직을 `bool` 메서드로 분리하는 대신, **<span class="fontColorRed">입력 시퀀스를 받아 필터링된 시퀀스를 반환하는 메서드</span>를 작성한다.**

`IQueryable -> IQueryable` 형태의 <span class="fontColorRed"><strong>확장 메서드</strong></span>를 사용하면:

*   쿼리 조각이 **표현식 트리로 유지**되고
*   여러 쿼리가 **하나의 트리로 결합**되며
*   **원격 데이터 소스에서도 정상적으로 실행**된다.

```csharp
private static IQueryable<Employee>LowPaidSalariedFilter(
			this IQueryable<Employee> sequence) =>
			from sin sequence
			where s.Classification == EmployeeType.Salary
			       && s.MonthlySalary <4000
			select s;

```

```csharp
source
  .LowPaidSalariedFilter()
  .Where(e => e.YearsOfService >=20)

```

### IEnumerable / IQueryable 모두 지원하기

실제 코드에서는 다음을 모두 제공하는 것이 바람직하다.

*   `IQueryable<T>` 버전 → DB, 원격 쿼리
*   `IEnumerable<T>` 버전 → 로컬 컬렉션

```csharp
// 1) IQueryable용 (DB, 원격 쿼리용)
public static IQueryable<Employee>LowPaidSalariedFilter(
this IQueryable<Employee> source) =>
		    source.Where(s =>
        s.Classification == EmployeeType.Salary &&
        s.MonthlySalary <4000);

// 2) IEnumerable용 (로컬 컬렉션용)
public static IEnumerable<Employee>LowPaidSalariedFilter(
this IEnumerable<Employee> source) =>
		    source.Where(s =>
        s.Classification == EmployeeType.Salary &&
        s.MonthlySalary <4000);

```

*   DB에서 쓸 땐: `IQueryable` 오버로드가 선택 → **표현식 트리 유지 + SQL 변환 OK**
*   로컬 리스트에서 쓸 땐: `IEnumerable` 오버로드가 선택 → **LINQ to Objects로 정상 동작**
