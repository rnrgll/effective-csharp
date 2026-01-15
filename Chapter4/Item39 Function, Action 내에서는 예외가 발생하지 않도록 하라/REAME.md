# 39. Function, Action 내에서는 예외가 발생하지 않도록 하라

시퀀스를 순차 처리하는 과정에서 `Func` / `Action`(람다) 내부에서 예외가 발생하면 이미 일부 요소만 처리된 **부분 업데이트(partial update)** 상태가 된다.

이 상태는 어떤 요소가 어디까지 변경되었는지 알 수 없기 때문에 **복구가 거의 불가능한 프로그램 상태**를 만든다. (→ Chapter 5. 예외 처리 참고)

따라서 **람다 내부에서는 예외가 외부로 전파되지 않도록 설계**해야 한다.

## 39.1 Function / Action 에서 예외가 발생할 때의 문제점

*   `ForEach(Action)` / `Select(Func)` 같은 구조는 내부적으로 **요소를 하나씩 순차 처리**한다.
*   처리 도중 예외가 발생하면:
    *   이미 처리된 요소는 그대로 남는다
    *   이후 요소는 처리되지 않는다

그 결과:

*   어디까지 처리가 완료되었는지 알 수 없고
*   어떤 요소가 변경되었는지도 알 수 없다

⇒ 결과적으로 **rollback이 사실상 불가능한 상태**가 된다.

⇒ 특히 `ForEach(Action)`처럼 **side effect에 의존하는 로직**은 매우 위험하다.

### Side Effect 기반 로직의 문제점

*   Side Effect 기반 로직이란, **원본 객체나 컬렉션의 상태를 직접 변경하는 코드**를 의미한다.
*   예외 발생 시 :
    *   일부만 변경된 **중간 상태(inconsistent state)가 남는다**
    *   강력한 예외 보증(strong exception guarantee)을 제공할 수 없다
*   객체 상태가 깨졌기 때문에 변경 이력 추적이 어렵고, 복구(rollback)나 재시도가 거의 불가능하다.

> #### 💡모든 LINQ가 예외 발생 시 위험한 것은 아니다.
> <code>Aggregate</code>, <code>Sum</code>, <code>Count</code> 같은 집계 연산이나,
> <code>Select</code>, <code>Where</code>처럼 원본을 수정하지 않는 읽기 전용 연산은 시퀀스의 상태를 변경하지 않는다.
> <p>예외가 발생하더라도 일부만 원본 데이터가 바뀌는 일이 없다. 단순히 연산이 실패하고 결과를 얻지 못하는 것에 그친다.</p>

## 39.2 해결 방법

### 39.2.1 람다가 예외를 절대 발생시지 않게 만들기

가장 간단하고 비용이 적은 해결책은 **람다 내부에서 예외가 발생하지 않도록 구조를 만드는 것이다.**

#### 1) 예외 원인을 사전에 제거하는 전략

*   잘못된 데이터는 **람다에 들어오기 전에 제거한다.**
    
*   `Where`, 필터링 등을 사용해:
    
    *   null
        
    *   상태 이상
        
    *   처리 불가능한 데이터
        
        를 미리 걸러낸다
        
*   판단 로직을 람다 안이 아니라 **람다 밖으로 이동**시킨다
    

```csharp
employees
    .Where(e => e.IsActive)
    .ToList()
    .ForEach(e => e.RaiseSalary()); // RaiseSalary()가 호출되는 시점에 안전한 데이터만 남아 있느 ㄴ상태
```

#### 2) 람다 내부에서 안전하게 처리하기

*   람다 내부에서 조건문, 검증 등을 사용해서 예외가 외부로 전파되지 않도록 한다.

```csharp
employees.ForEach(e =>
{
    if (!e.CanRaiseSalary) return;
    e.RaiseSalary();
});
```

⇒ 가장 간단하고 비용이 적음

### 39.2.2 원본을 건드리지 않기 - 원자적 업데이트 전략

람다 내부에서 예외를 완전하게 제거할 수 없는 경우에는 **원본 데이터를 직접 수정하지 않는 방식**을 사용해야 한다.

#### 방법

1.  원본 시퀀스를 수정하지 않는다.
2.  새 객체 / 컬렉션을 만든다
3.  모든 작업이 성공했을 때만 한 번에 교체한다.

```csharp
var updatedEmployees =
    allEmployees
        .Select(e => new Employee        // 원본을 수정하지 않고 새로운 Employee 객체를 만듦
        {
            EmployeeID = e.EmployeeID,
            Classification = e.Classification,
            YearsOfService = e.YearsOfService,
            MonthlySalary = e.MonthlySalary * 1.05M
        })
        .ToList();

allEmployees = updatedEmployees;

```

#### 장점

*   중간에 예외가 나도 원본이 완전히 안전하기 때문에 rollback이 가능하다.
*   상태의 일관성을 보장한다.

#### 단점

*   코드가 길어지고 복잡해진다.
*   새로운 객체를 할당하는 것이기 때문에 메모리, 성능 비용이 발생한다.
*   대용량 데이터의 경우 부담이 된다.
