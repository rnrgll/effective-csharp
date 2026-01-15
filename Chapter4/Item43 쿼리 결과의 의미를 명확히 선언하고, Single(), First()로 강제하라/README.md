# 43\. 쿼리 결과의 의미를 명확히 선언하고, Single()/First()로 강제하라

LINQ 쿼리는 기본적으로 **시퀀스(IEnumerable)** 를 반환하지만, 실제로 원하는 결과가 다음과 같을 때가 있다.

*   결과가 **정확히 하나여야 한다**
*   결과가 **0개 또는 1개일 수 있다**
*   여러 개 중 **첫 번째 하나만 필요하다**

이러한 결과 개수에 대한 기대를 코드로 명확히 표현하는 것이 중요하다.

그 역할을 하는 것이 `Single()`, `SingleOrDefault()`, `First()`, `FirstOrDefault()` 이다.

## 43.1 Single() / SingleOrDefault()

### 43.1.1 Single() : 정확히 1개

*   결과가 **반드시 1개를 강제한다.**
*   0개 ❌ → 예외 / 2개 이상 ❌ → 예외

```csharp
var answer =
    (from pin somePeople
		 where p.FirstName =="Bill"
		 select p).Single();
```

⇒ ‘Bill이라는 이름을 가진 사람은 **반드시 한 명**이어야 한다’는 의미

*   데이터에 Bill이 3명 → **즉시 예외 → `InvalideOperationException`**
*   데이터에 Bill이 0명 → **즉시 예외 → `InvalideOperationException`**

즉시 예외가 발생함으로써 잘못된 데이터 상태를 빠르게 드러내기 때문에 추가적으로 데이터 손상이 발생하는 것을 미연에 방지할 수 있다.

<div class="textBox"><div class="textBoxTitle"><p>❗Single()은 즉시 평가된다</p></div><div class="textBoxContent"><ul><li><p><code>Single()</code>은 <strong>지연 평가가 아니라 즉시 평가(eager evaluation)</strong> 된다.</p><p>→ 시퀀스를 끝까지 검사해 ‘정확히 한 개인지’ 확인해야 하기 때문이다.</p></li><li><p>값을 반환하기 위해 <strong data-end="561" data-start="544">즉시 전체 시퀀스를 검사</strong>하며,&nbsp;요소가 0개이거나 2개 이상이면 <strong data-end="617" data-start="587">메서드를 호출한 시점에 즉시 예외를 발생시킨다.</strong></p></li><li><p>데이터의 유일성이 깨진 시점을 명확히 알려준다.</p></li></ul></div></div>

### 43.1.2 SingleOrDefault : 0개 또는 1개

*   결과가 **0개 또는 1개여야 한다.**
*   0개 → 기본값 (참조 타입은 `null`, 값 타입은 `default(T)`)
*   1개 → OK
*   2개 이상 → ❌ 예외

```csharp
var person = people.Where(p => p.Name == "Larry").SingleOrDefault();
```

⇒ ‘Larry 이라는 이름을 가진 사람이 없을 수도 있지만, 있다면 **반드시 한 명**이어야 한다’는 의미

*   Larry가 없으면 → 기본값(null)
*   Larry가 1명 → 그 사람
*   Larry가 2명 이상 → 즉시 예외

## 43.2 First() / FirstOrDefault

*   여러 개여도 OK
*   그 중에 첫 번째를 가져온다.
*   순서(ORDER BY 또는 원본 순서)에 의존한다

```csharp
var answer =
    (from pin Forwards
		where p.GoalsScored >0
		orderby p.GoalsScoreddescending
		select p).First();

```

⇒ ‘득점을 한 포워드 중 가장 먼저 나오는 한 명만 필요하다’

*   결과가 0개
    
    → `First()` ❌ 예외
    
    → `FirstOrDefault()` → null (또는 default(T))
    
*   **여러 명이 있어도 문제 없음**
    
*   첫 번째를 어떻게 정하느냐(정렬, 조건)가 핵심
    

### 43.2.1 Single VS First

| 구분 | Single | First |
| --- | --- | --- |
| 허용 개수 | 정확히 1개 | 1개 이상 |
| 0개일 경우 | 예외 | 예외 / default(왜 default야) |
| 2개 이상일 경우 | 예외 | 허용 |
| 의미 | **유일성 보장** | **대표 하나 선택** |
| 사용 예 | ID, 유니크 키 | 최신 데이터, 최고값 |

### 43.2.2 Take(1) VS First()

#### Take()

*   `Take(1)`은 LINQ의 **시퀀스 조작 연산자**이다.
*   결과 타입: `IEnumerable<T>`
    *   결과는 **항상 시퀀스**다. 요소가 있든 없든 항상 시퀀스를 반환한다.
    *   요소가 없으면 → **빈 시퀀스**
    *   요소가 많아도 → **첫 1개만 포함**

##### Take(1) 결과 예시

```csharp
query.Take(1)
```

| 입력 시퀀스 | `Take(1)` 결과 |
| --- | --- |
| `[ ]` | 빈 시퀀스 |
| `[A]` | `[A]` |
| `[A, B, C]` | `[A]` |

⇒ 즉, Take(1)은 단일 값 추출이 아니라 **시퀀스를 잘라내는 연산이다.**

#### Take(1) vs First()

*   목적
    *   Take(1) : 시퀀스를 자르는 연산. 앞에서 최대 1개만 포함한 **새 시퀀스 생성**
    *   First() : 단일 값을 얻는 연산. 첫 번째 요소 **하나를 반환** (없으면 오류)
*   반환 타입
    *   Take(1) : `IEnumerable<T>`
    *   First() : T
*   빈 시퀀스일 때 동작
    *   Take(1) : 빈 시퀀스 반환
    *   First() : 예외

| 항목 | Take(1) | First() |
| --- | --- | --- |
| 결과 타입 | `IEnumerable<T>` | `T` |
| 목적 | 시퀀스 조작 | 단일 값 요청 |
| 빈 결과 | 빈 시퀀스 | 예외/기본값 |
| 의미 표현 | 모호 | 명확 |
| 용도 | 파이프라인 단계 | 최종 결과 추출 |
