# 예외 처리(Exception Handling) 입문

지금까지 `IndexError`, `KeyError`, `RuntimeError` 등 여러 에러를 실전에서 마주쳤지만, 그때마다 프로그램이 그대로 멈췄다. 이번엔 그 에러들을 미리 대비하고 우아하게 처리하는 정식 도구(`try`/`except`)를 배웠다. 

---

## 기본 구조

```python
try:
    numbers[10]        # 에러가 날 수도 있는 코드
except IndexError:
    print("범위를 벗어났습니다")
```
`try` 블록에서 에러가 나면 그 즉시 중단되고 `except` 블록으로 넘어간다. 에러가 안 나면 `except`는 아예 실행되지 않는다.

---

## 문제 1: 안전한 나눗셈

```python
def safe_divide(a, b):
    try:
        return a / b
    except ZeroDivisionError:
        return "나눌 수 없습니다"
```
`ZeroDivisionError`를 처음 다룸.

### 시행착오: 종료 조건 없이 변수 미할당
```python
try:
    result = a / b
except ZeroDivisionError:
    print("나눌 수 없습니다")   # result에 아무것도 저장 안 함
return result   # UnboundLocalError!
```
`except` 블록에서 `print`만 하고 `result`에 값을 저장하지 않으면, `try`가 실패했을 때 `result`라는 변수 자체가 존재하지 않게 되어 마지막 `return result`에서 에러가 난다. `try`가 성공하든 실패하든, 반환할 변수에는 항상 값이 채워지도록 각 분기에서 명시적으로 할당해야 한다.

### 스타일 비교: 변수에 담기 vs 각 분기에서 바로 반환
```python
# 변수 사용
try:
    result = a / b
except ZeroDivisionError:
    result = "나눌 수 없습니다"
return result

# 각 분기에서 바로 return (더 간결, try/except가 각자 독립적으로 완결됨)
try:
    return a / b
except ZeroDivisionError:
    return "나눌 수 없습니다"
```

---

## 문제 2: 안전한 정수 변환

```python
def safe_int(s):
    try:
        return int(s)
    except ValueError:
        return None
```
`int("abc")`처럼 변환 불가능한 문자열을 정수로 바꾸려 할 때 나는 에러가 `ValueError`라는 것을 확인.

---

## 문제 3: 리스트에서 여러 인덱스 안전하게 접근

반복문 **안에서** `try`/`except`를 매번 사용해, 일부 인덱스가 잘못돼도 전체가 멈추지 않고 그 자리만 `None`으로 채우며 계속 진행하는 문제.

```python
def safe_get_all(numbers, index):
    new_list = []
    for i in index:
        try:
            new_list.append(numbers[i])
        except IndexError:
            new_list.append(None)
    return new_list
```

### 파이썬 스타일 참고: EAFP
"일단 시도해보고 안 되면 예외로 처리한다"는 이 방식을 **EAFP**(Easier to Ask Forgiveness than Permission)라 부르며, 파이썬이 권장하는 스타일이다. 반대로 미리 조건을 확인하는 **LBYL**(`if 0 <= i < len(numbers):`) 방식도 동일한 결과를 내지만, 이번 문제 취지(예외 처리 연습)엔 EAFP가 더 맞다.

### 참고: 음수 인덱스는 에러가 아니다
`numbers[-1]`, `numbers[-2]`는 "뒤에서부터 센 유효한 위치"라 정상 처리되고, 리스트 크기를 초과하는 음수(`-10` 등)만 `IndexError`가 난다.

---

## 부록: `list()`와 튜플 — 이 문제를 풀다가 파생된 논의

`safe_get_all`을 리스트 `+` 방식으로 다시 짜보다가 `new_list + list(numbers[i])`에서 `TypeError: 'int' object is not iterable`을 만난 것을 계기로 정리된 개념들.

### `list()`는 "순회 가능한 대상"만 받는다
```python
list("abc")       # ['a','b','c']  문자열은 순회 가능
list((1,2,3))     # [1,2,3]        튜플도 순회 가능
list(numbers[i])  # 에러! 정수는 "여러 값의 묶음"이 아니라 값 그 자체
```
값 하나를 리스트에 추가하고 싶다면 `list()`로 감쌀 게 아니라, `.append()`를 쓰거나 `[값]`(대괄호로 직접 만드는 원소 하나짜리 리스트)을 써야 한다.

### `list((1,3,4))`는 되고 `list(1,3,4)`는 안 되는 이유
바깥 괄호(함수 호출)와 안쪽 괄호(튜플)의 역할이 다르다. `list((1,3,4))`는 "튜플 하나"를 인자로 넘긴 것이고, `list(1,3,4)`는 값 세 개를 각각 넘기려 한 것이라 "인자를 하나만 받는 `list()`에 세 개를 줬다"는 에러가 난다.

### 튜플(tuple) — 새로운 자료형
- 리스트와 거의 같지만 소괄호(`()`)를 쓰고, **불변(immutable)**이다 (`t[0] = 100`은 에러).
- 원소가 하나면 반드시 콤마를 붙여야 진짜 튜플이다(`(5,)`). 콤마 없는 `(5)`는 그냥 정수 5.
- **함수가 값을 여러 개 반환할 때 자동으로 튜플이 만들어진다.** `a, b = 2, 4`도 사실 오른쪽(`2, 4`)이 먼저 튜플 `(2, 4)`로 만들어지고, 그걸 왼쪽이 두 조각으로 나눠 받는(언패킹) 2단계 과정이다. `.items()`로 얻는 `(key, value)` 쌍도 이 원리와 같다.
- **딕셔너리의 key로 쓸 수 있다** (리스트는 불가능 — key는 불변이어야 하므로). 좌표 `(x, y)`를 key로 쓰는 경우가 대표적.

### `list(리스트)`는 복사본을 만든다
```python
a = [1, 2, 3]
b = list(a)      # a와 값은 같지만 완전히 다른(독립된) 리스트
b.append(4)      # a는 영향 없음
```

---

## 문제 4: 여러 종류의 에러를 각각 다르게 처리

```python
def safe_calculate(numbers, i, divisor):
    try:
        return numbers[i] / divisor
    except IndexError:
        return "잘못된 인덱스입니다"
    except ZeroDivisionError:
        return "0으로 나눌 수 없습니다"
```
하나의 `try` 뒤에 `except`를 여러 개 이어 붙여, 서로 다른 종류의 에러를 각각 다른 방식으로 처리할 수 있다는 것을 확인.

---

## 예외 처리 문법의 나머지 기본 요소

`try`/`except`만으로 부족한 지점들을 마저 채웠다.

### `else` — `try`가 성공했을 때만 실행
```python
try:
    average = sum(scores) / count
except ZeroDivisionError:
    return None
else:
    bonus = average / (count - count)   # 여기서 나는 에러는 위 except가 안 잡음
    return average, bonus
```

**왜 필요한지 실증한 사례:** `else` 없이 `average` 계산과 `bonus` 계산(버그로 항상 0으로 나눔)을 한 `try` 블록에 몰아넣으면, `average`는 멀쩡히 성공했는데도 `bonus`의 버그가 우연히 같은 종류(`ZeroDivisionError`)라서 `except`에 걸려버려 "계산 실패"로 잘못 보고되고, 진짜 계산 결과(`average`)도 버려지며 진짜 버그(`bonus`)의 위치도 숨겨진다. `else`로 "지켜야 할 계산(try)"과 "성공 이후의 처리(else)"를 분리하면, 후속 코드의 버그가 앞의 `except`에 잘못 잡히지 않고 정확한 에러로 그대로 드러난다.

### `finally` — 성공/실패와 무관하게 항상 실행
```python
try:
    result = a / b
except ZeroDivisionError:
    result = None
finally:
    print("계산 시도 완료")   # 항상 실행됨, return으로 빠져나가는 중에도 실행
```
파일 닫기, 연결 정리처럼 에러 여부와 무관하게 반드시 해야 하는 뒷정리에 사용한다 (파일 입출력 학습 시 다시 등장할 개념).

### 에러 객체 잡기 (`as e`)
```python
except ValueError as e:
    print("변환 실패:", e)   # e에는 실제 에러 메시지가 담김
```
고정된 메시지 대신, 파이썬이 실제로 생성한 에러 메시지 내용을 그대로 활용하고 싶을 때 사용.

### 부수 개념: `e`는 문자열이 아니라 예외 객체다
```python
"계산 오류: " + e        # 에러! e는 str이 아니라 예외 객체
f"계산 오류: {e}"        # 됨 — f-string의 {}는 값을 자동으로 문자열로 변환
print("계산 오류:", e)   # 됨 — print가 각 인자를 알아서 문자열로 바꿔 출력
"계산 오류: " + str(e)   # 됨 — str()로 명시적 변환
```
`+`는 반드시 양쪽이 진짜 문자열이어야 하지만, f-string의 `{}`와 `print()`의 콤마 구분 인자는 값을 자동으로 문자열로 변환해준다는 것을 재확인 (`simple_text_game.md`에서 숫자를 `+`로 못 붙였던 것과 같은 원리).

### 미룬 것: `raise`, 여러 에러 묶어서 처리(`except (A, B):`)
`raise`(직접 에러를 발생시키는 것)는 에러를 잡는 게 아니라 만들어 던지는 것이라 별개 주제이며, 직접 함수를 설계하며 유효성 검사가 필요해질 때 배우는 것으로 미룸. `except (A, B):`는 `except`를 여러 번 따로 쓰는 것의 단축 표현일 뿐이라 몰라도 지장 없어 마찬가지로 미룸.

---

## 문제 5 (종합): `try`/`except`(2종)/`else`/`finally` 전부 조합

```python
def calculate_report(numbers, i, divisor):
    try:
        result = numbers[i] / divisor
    except ZeroDivisionError as e:
        print("계산 오류:", e)
        return None
    except IndexError:
        print("잘못된 인덱스입니다")
        return None
    else:
        print("계산 완료")
        return result
    finally:
        print("작업 종료")
```

### 검증
```
calculate_report([2,3,4,1,6], 3, 0)   -> "계산 오류: division by zero" / "작업 종료" / None
calculate_report([10,20,30], 1, 2)    -> "계산 완료" / "작업 종료" / 10.0
calculate_report([10,20,30], 10, 2)   -> "잘못된 인덱스입니다" / "작업 종료" / None
calculate_report([10,20,30], 1, 0)    -> "계산 오류: division by zero" / "작업 종료" / None
```
`return`이 `except`/`else` 안에 있는데도, `finally`("작업 종료")가 그 `return`보다 먼저 실행된다는 것까지 정확히 확인 — 함수가 어떤 경로로 빠져나가든 `finally`가 반드시 먼저 실행된다는 특성을 실제로 검증한 사례.
