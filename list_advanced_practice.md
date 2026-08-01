# 리스트 심화 기초 연습 — 예외 처리 이후에 배운 것들

`append`, `.count()`, `in`/`not in`은 이미 충분히 익힌 상태에서, 나머지 리스트 심화 메서드(`pop`, `remove`, `sort`, `insert`)를 실전 문제로 다졌다..

---

## 사전 정리: 새로 배운 메서드

```python
numbers.pop()        # 인자 없으면 맨 뒤 원소를 제거하고 그 값을 반환
numbers.pop(1)       # 인덱스 1번 원소를 제거하고 그 값을 반환

numbers.remove(20)   # "값 20"을 찾아서 제거 (첫 번째로 발견된 것 하나만).
                      # 리스트에 없는 값이면 ValueError 발생 (예외 처리와 직결)

numbers.sort()              # 원본을 직접 오름차순 정렬. 반환값은 항상 None!
numbers.sort(reverse=True)  # 내림차순

numbers.insert(2, 3)   # 인덱스 2 자리에 3을 끼워 넣음 (뒤 원소들은 자동으로 밀림)
                        # 범위를 벗어난 위치를 지정해도 에러 없이 맨 끝에 안전하게 들어감
```

`pop`은 위치로, `remove`는 값으로 제거한다는 것이 핵심 차이. `sort()`를 `numbers = numbers.sort()`처럼 재할당하면 `None`이 되어버리는 함정에 주의.

---

## 문제 1: 상위 3개 점수 (Top 3 Scores)

```python
top_three([5, 2, 9, 1, 7, 3])   # [9, 7, 5]
top_three([4, 1])               # [4, 1]  (3개 미만이면 있는 만큼)
```

### 시행착오: 인덱스 직접 접근의 한계
```python
scores.sort(reverse=True)
return [scores[0], scores[1], scores[2]]
```
리스트 길이가 3보다 짧으면(`[4, 1]`) `scores[2]`가 존재하지 않아 `IndexError`가 난다. 반면 슬라이싱(`scores[:3]`)은 범위를 벗어나도 에러 없이 있는 만큼만 안전하게 반환한다는 걸(`palindrome_three_problems.md`에서 이미 정리한 내용) 이번엔 리스트 인덱싱과 대조하며 재확인.

### 최종 정답 코드
```python
def top_three(scores):
    scores.sort(reverse=True)
    return scores[:3]
```
처음엔 `scores[:3]`을 `for`문으로 하나씩 다시 `append`하는 방식으로 풀었으나, 슬라이싱 결과가 이미 완성된 리스트라는 걸 확인하고 그 불필요한 복사 단계를 제거했다.

---

## 문제 2: 리스트에서 특정 값 전부 제거

```python
remove_all([1, 2, 3, 2, 4, 2], 2)   # [1, 3, 4]
remove_all([5, 5, 5], 5)            # []
```

### 발견: 순회 중인 리스트를 직접 수정하면 값이 누락된다
```python
for i in numbers:
    if i == target:
        numbers.remove(i)   # 순회 중인 리스트 자체를 수정
```
`[5, 5, 5]`처럼 타깃이 연속으로 나오는 경우, 하나를 지우면 뒤 원소들이 당겨지는데 순회 위치는 그대로 다음 칸으로 넘어가면서 **원소 하나를 건너뛰어 버린다.** `RuntimeError` 같은 명시적 에러 없이 조용히 틀린 결과가 나온다는 점에서, 딕셔너리를 순회하며 동시에 삭제했을 때(`RuntimeError`)보다 오히려 더 위험한 함정이었다.

### 세 가지 해결 방식을 모두 시도해봄
```python
# 방법 A: 원본은 그대로 두고, 새 리스트에 조건에 맞는 것만 담기
def remove_all(numbers, target):
    result = []
    for i in numbers:
        if i != target:
            result.append(i)
    return result

# 방법 B: 복사본을 만들어, 원본(안전하게 순회) 기준으로 복사본에서 제거
def remove_all(numbers, target):
    result = list(numbers)
    for i in numbers:
        if i == target:
            result.remove(i)
    return result

# 방법 C: 원본에 remove를 반복 호출하다가, 더 지울 게 없어 나는 ValueError를 종료 신호로 활용
def remove_all(numbers, target):
    for i in range(len(numbers)):
        try:
            numbers.remove(target)
        except ValueError:
            return numbers
    return numbers
```
세 방법 모두 정답이며, 각각 오늘 배운 도구(필터링, `list()` 복사, 예외 처리)를 서로 다른 조합으로 활용한 결과다.

### 성능 비교: 실측으로 확인한 O(n) vs O(n²)
`target`으로만 가득 찬 최악의 경우(`n`을 2배씩 늘려가며 측정)로 세 방법을 실측했다.
- **방법 A**: `n`이 두 배가 되면 시간도 거의 두 배 → **O(n)**
- **방법 B, C**: `n`이 두 배가 되면 시간이 6~7배 뜀 → **O(n²)**

원인은 `.remove(x)`가 "값을 찾고(O(n)) + 뒤 원소들을 당기는(O(n))" 자체로 O(n)인 연산이라는 데 있다. 이걸 타깃 등장 횟수만큼 반복 호출하면 최악의 경우 O(n²)로 퇴화한다 (`character_pair_count_notes.md`의 "`.count()`는 항상 전체를 훑는다"는 원리와 같은 종류의 함정).

**의도와 결과가 갈렸던 지점**: 애초에 이 문제는 "리스트 심화(`remove`)와 예외 처리(`ValueError`)를 실전에서 조합해보는 연습"이 목적이었고, 그 목적에는 방법 C가 정확히 부합한다. 다만 "실전에서 가장 효율적인 코드는 무엇인가"라는 별도의 질문에는 방법 A가 답이다. 같은 문제라도 "무엇을 묻는가"에 따라 서로 다른 정답이 나올 수 있다는 것을 확인.

---

## 문제 3: 짝수 제거 + 정렬

```python
filter_and_sort([5, 2, 8, 1, 9, 4, 3])   # [1, 3, 5, 9]
```
### 정답 코드
```python
def filter_and_sort(numbers):
    result = []
    for i in numbers:
        if i % 2 != 0:
            result.append(i)
    result.sort()
    return result
```
필터링(문제 2의 방법 A와 동일 패턴)과 `sort()`(오름차순 기본값)를 조합. 필터링 후 복사본에서 `.remove()`로 짝수를 지우는 방식(문제 2의 방법 B와 동일 패턴)도 정답이지만, 같은 이유로 성능이 더 떨어진다.

---

## 문제 4: 괄호 짝 맞추기 (스택, `pop()` 실전 활용)

```python
is_valid_parentheses("(())")   # True
is_valid_parentheses("(()")    # False
```

### 스택(Stack)과 LIFO
`append`(맨 뒤에 추가)와 `pop()`(맨 뒤에서 제거)을 짝으로 쓰면, "가장 나중에 넣은 것이 가장 먼저 나온다"(LIFO, Last In First Out)는 스택 구조를 리스트로 흉내낼 수 있다. 괄호 문제는 "가장 안쪽(가장 나중에 열린)의 괄호가 가장 먼저 닫혀야 한다"는 성질이 정확히 LIFO와 들어맞는 대표적인 스택 활용 예제.

### 정답 코드
```python
def is_valid_parentheses(s):
    open_parentheses = []
    for i in s:
        if i == "(":
            open_parentheses.append(i)
        elif open_parentheses == []:
            return False
        else:
            open_parentheses.pop()
    return open_parentheses == []
```
여는 괄호는 쌓고, 닫는 괄호를 만나면 가장 최근 것을 꺼내 짝을 맞춘다. 스택이 비었는데 닫는 괄호가 나오면 즉시 `False`, 끝까지 다 봤는데 스택에 뭔가 남았으면(여는 괄호가 남음) `False`.

### 이 문제의 시간복잡도가 이미 최적인 이유
`pop()`을 인자 없이 맨 뒤에서 꺼내는 건, 값을 찾을 필요도 뒤 원소를 당길 필요도 없어 **O(1)**이다. 반면 `.remove()`는 값을 찾고 당기는 과정 때문에 O(n)이다. 이 차이 때문에 이 문제는 문자열을 한 번만 훑으며 각 단계가 전부 O(1)이라, 전체가 정확히 **O(n)**으로 더 줄일 여지가 없다.

---

## 문제 5: 정렬된 리스트에 값 삽입하기 (`insert()` 실전 활용)

```python
insert_sorted([1, 3, 5, 7], 4)   # [1, 3, 4, 5, 7]
insert_sorted([2, 4, 6], 10)     # [2, 4, 6, 10]  (가장 큰 값보다 큰 경우)
```

### 처음 놓친 경계 케이스: `value`가 리스트의 모든 값보다 큰 경우
```python
for i in numbers:
    if i > value:
        result.insert(count, value)
        return result
    count = count + 1
# value가 모든 원소보다 크면 이 조건이 한 번도 참이 안 됨 -> return을 못 만나고 None 반환
```
`if i > value:`라는 조건이 이 경우엔 리스트 전체를 훑어도 단 한 번도 참이 되지 않아, 반복문이 끝까지 실행되고도 `return`을 만나지 못해 함수가 `None`을 반환해버린다. `insert()` 자체가 문제가 아니라, 그걸 호출할 조건이 이 특정 입력에서 성립하지 않았던 것 — 이 문제 유형(반복문 안 조건이 특정 입력에서 원천적으로 안 걸리는 경우)은 오늘 두 번 반복해서 나타난 패턴이다.

### 최종 정답 코드
```python
def insert_sorted(numbers, value):
    result = list(numbers)

    if value >= max(numbers):
        result.append(value)
        return result

    count = 0
    for i in numbers:
        if i > value:
            result.insert(count, value)
            return result
        count = count + 1
```
"가장 큰 값보다 큰 경우"를 반복문 진입 전에 미리 걸러내 해결. `max(numbers)`를 반복문 밖에서 딱 한 번만 호출해 불필요한 반복 계산도 피했다.

### 설계 트레이드오프: 원본 보호(복사본) vs 직접 수정
`result = list(numbers)`로 복사본을 만드는 버전과, `numbers`를 직접 수정하는 버전을 비교했다.
- **직접 수정**: 복사 비용이 없어 아주 미세하게 더 효율적이지만, 함수를 호출한 쪽이 넘긴 원본 리스트가 몰래 바뀌어버리는 부작용(side effect)이 생긴다.
- **복사본 사용**: 원본이 절대 바뀌지 않아 예측 가능성이 높다. "정렬된 새 리스트를 반환한다"는 이 문제의 의도에 부합.

이건 시간복잡도의 문제가 아니라 **"함수가 부작용을 일으켜도 되는가"**라는 별도의 설계 축이며, 이번 문제에서는 원본 보호가 더 적절한 선택으로 판단됨.

---

## 부록: `list(리스트)`가 이중 리스트(`[[]]`)를 만드는 게 아닌 이유

`remove_all`의 복사본 버전을 짜다가, "`list(numbers)`가 이중 대괄호 구조를 만드는 것 아닌가"라는 의문이 들었던 것을 계기로 정리.

```python
list(numbers)   # numbers 안의 값들을 하나씩 꺼내 새 리스트에 다시 담음 -> [1, 2, 3]
[numbers]       # numbers 전체를 "원소 하나"로 감쌈 -> [[1, 2, 3]] (진짜 이중 구조)
```
`list()`는 "이미 리스트니까 그냥 감싸기"가 아니라, 순회 가능한 대상이면 무엇이든(문자열, 튜플, 리스트) 그 안의 값들을 하나씩 꺼내 새로운 리스트로 재구성하는 동일한 절차를 적용한다. 원본이 이미 리스트였기 때문에 결과가 "복사"처럼 보이는 것일 뿐, 대괄호로 직접 감싸는 것(`[numbers]`)과는 완전히 다른 동작이다.
