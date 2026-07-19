# 리스트와 딕셔너리 복습 — 4문제 정리

셋(set)과 딕셔너리(dict)를 처음 배운 뒤, 이를 적용해 푼 문제들을 정리한다.
주제: 리스트 기반 풀이(O(n²))를 셋/딕셔너리로 바꿔 O(n)으로 개선하는 연습.

---

## 문제 1: 중복 없는 개수 (Unique Count)
**예상 난이도: 브론즈 5~4**

정수 리스트 `numbers`가 주어졌을 때, 서로 다른 값이 몇 종류인지 세는 함수 `count_unique(numbers)`를 작성한다.

```python
count_unique([1, 2, 4, 5, 2, 6, 3, 1])  # 서로 다른 값: 1,2,3,4,5,6 → 6
```

### 정답 코드
```python
def count_unique(numbers):
    unique_numbers = set()
    count = 0
    for value in numbers:
        if value not in unique_numbers:
            count = count + 1
            unique_numbers.add(value)
    return count
```

### 사용된 핵심 도구
- `set()` — 빈 셋 생성
- `.add()` — 셋에 값 추가
- `not in` — 셋에 값이 있는지 확인 (리스트와 달리 즉시 확인되어 O(1))

### 메커니즘 및 시간복잡도
값을 하나씩 순회하면서, 아직 등록되지 않은 값이면 `count`를 늘리고 셋에 등록한다. 이미 등록된 값이면 건너뛴다. 셋의 `in` 확인이 O(1)이므로 전체는 **O(n)**.

이 문제는 처음 배운 도구(`set`)를 곧바로 적용해보는 연습이라 별도의 실패한 시도는 없었다.

---

## 문제 2: 최빈값 찾기 (Most Frequent Value)
**예상 난이도: 브론즈 2~1**

정수 리스트 `numbers`가 주어졌을 때, 가장 많이 등장하는 값을 반환하는 함수 `find_most_frequent(numbers)`를 작성한다. (동점이면 아무거나 하나 반환)

```python
find_most_frequent([1, 2, 4, 2, 1, 6, 7, 2, 2, 2, 5, 6, 7, 4, 2])  # → 2
```

이 문제는 세 가지 방식으로 풀어보며, 딕셔너리가 왜 필요한지를 단계적으로 확인했다.

### 방법 1: 순수 `for`/`if`만으로 (도구 확인용 브루트포스)
```python
def find_most_frequent(numbers):
    best_value = None
    best_count = 0
    for i in range(len(numbers)):
        count = 0
        for j in range(len(numbers)):
            if numbers[j] == numbers[i]:
                count = count + 1
        if count > best_count:
            best_count = count
            best_value = numbers[i]
    return best_value
```
`append`, `in`, `.count()` 없이도 풀린다는 것을 확인하기 위한 버전. 안쪽 반복문으로 `.count()`가 하는 일을 직접 구현했다. 정답이지만, 같은 값이 여러 번 나올 때마다 매번 처음부터 다시 세는 중복 계산이 발생해 비효율적이다. **O(n²)**.

### 방법 2: 리스트 + `checked` 명단 + `.count()`
```python
def find_most_frequent(numbers):
    checked = []
    best_value = None
    best_count = 0
    for value in numbers:
        if value not in checked:
            checked.append(value)
            count = numbers.count(value)
            if count > best_count:
                best_count = count
                best_value = value
    return best_value
```
문자 빈도 짝 찾기 등에서 썼던 "명단 기록" 패턴을 재사용. 이미 확인한 값은 다시 안 보므로 방법 1보다는 낫지만, `not in checked`와 `.count()`가 여전히 리스트를 훑는 연산이라 **O(n²)**을 벗어나지 못한다.

### 방법 3: 딕셔너리로 개수를 저장하고, `.items()`로 최댓값 추적
```python
def find_most_frequent(numebrs):
    count = {}
    for n in numebrs:
        if n in count:
            count[n] = count[n] + 1
        else:
            count[n] = 1

    best_key = None
    best_value = 0
    for key, value in count.items():
        if value > best_value:
            best_value = value
            best_key = key

    return best_key
```
딕셔너리에 각 값의 등장 횟수를 저장(`n in count`로 존재 여부 확인 후 갱신 또는 초기화)하고, `.items()`로 key-값 짝을 동시에 순회하며 최댓값을 가진 key를 추적한다. **O(n)**.

### 사용된 핵심 도구 (방법 3 기준)
- `{}` — 빈 딕셔너리 생성
- `딕셔너리[key] = 값` — key에 값을 저장/갱신 (key가 이미 있으면 기존 값을 읽어 갱신, 없으면 새로 생성 — 문법은 같고 상황에 따라 의미가 갈림)
- `key in 딕셔너리` — key 존재 여부 확인 (O(1))
- `.items()` — key-값 짝을 튜플로 순회
- 튜플 언패킹 (`for key, value in ...`) — 짝을 두 변수에 동시에 나눠 받는 문법

### 이 문제에서 확인한 것
`max(counts)`를 그냥 쓰면 값이 아니라 **key** 중에서 최댓값을 찾아버리는 함정이 있어, 값 기준 최댓값이 필요할 때는 `.items()`로 직접 순회하며 비교해야 한다는 것을 확인했다.

---

## 문제 3: 두 수의 합 (Two Sum / has_pair_sum)
**예상 난이도: 실버 5~4**

정수 리스트 `numbers`와 목표값 `target`이 주어졌을 때, 합이 정확히 `target`이 되는 서로 다른 인덱스의 두 원소 조합이 존재하는지 판별한다. (브루트포스, 원본 리스트 직접 대조, 리스트 기반 "지나온 흔적 기록" 등 O(n²) 풀이 전 과정은 별도 문서(`seen_pattern_three_problems.md`)에 정리되어 있으므로, 여기서는 셋을 적용한 최종 O(n) 버전만 정리한다.)

```python
has_pair_sum([2, 7, 11, 15], 9)  # → True
has_pair_sum([3, 3], 6)          # → True
```

### 정답 코드 (셋 적용, O(n))
```python
def has_pair_sum(numbers, target):
    seen = set()
    for value in numbers:
        if target - value in seen:
            return True
        seen.add(value)
    return False
```

### 리스트 버전에서 바뀐 부분
이전에 짠 리스트 기반 버전(`seen = []`, `seen.append(value)`)에서 **딱 두 곳만 교체**했다: `[]` → `set()`, `.append()` → `.add()`. 로직(`target - value in seen` 체크, 순회 순서)은 전혀 바꾸지 않았다.

### 사용된 핵심 도구
- `set()`, `.add()`, `in` — 문제 1과 동일

### 확인된 성능 차이
정답이 존재하지 않는 최악의 케이스로 실측한 결과, 리스트 버전은 n이 8배(2000→16000) 커질 때 시간이 약 47배 늘어난 반면(O(n²)), 셋 버전은 거의 변화가 없었다(O(n)).

---

## 세 문제의 공통 패턴: "그릇만 바꿔서 O(n²) → O(n)"

세 문제 모두 리스트로 풀면 O(n²)에 머무르지만, **알고리즘의 뼈대(로직)는 전혀 바꾸지 않고, 값을 담는 자료구조만 교체**하는 것으로 O(n)까지 개선되었다.

| 문제 | 필요했던 것 | 쓰인 자료구조 | 이유 |
|---|---|---|---|
| 중복 없는 개수 | 값이 이미 나왔는지만 확인 | `set` | 존재 여부만 필요, 개수/추가 정보 불필요 |
| 최빈값 찾기 | 각 값이 몇 번 나왔는지 개수 | `dict` | 존재 여부뿐 아니라 값(개수)까지 저장해야 함 |
| 두 수의 합 | 지나온 값 중에 특정 값이 있는지 | `set` | 존재 여부만 필요 |

**`set`과 `dict`를 고르는 기준:** 단순히 "있다/없다"만 확인하면 `set`, 그 값에 대해 추가 정보(개수 등)를 같이 저장해야 하면 `dict`.

**리스트와 셋/딕셔너리의 근본적 차이:** 리스트에서 `in`/`.count()`는 값을 찾기 위해 처음부터 하나씩 훑어야 하지만(순차 탐색), 셋/딕셔너리는 내부적으로 값을 저장할 때 위치를 미리 계산해두는 방식(해시 테이블)을 써서 훑지 않고 즉시 확인 가능하다. 이 차이가 반복문 안에서 누적되어 O(n²) vs O(n)이라는 차이를 만든다.

---

## 부수적으로 정리된 개념들

- **셋(set)과 리스트의 차이는 속도뿐만이 아니다.** 셋은 중복을 자동으로 제거하고, 순서를 보장하지 않으며, 인덱싱이 불가능하다. "리스트를 셋으로 바꿔도 되는가"는 매번 "이 문제가 순서나 중복 개수 자체를 신경 쓰는가"를 먼저 확인해야 한다. 위 세 문제는 모두 순서/중복 개수가 결과에 영향을 주지 않아 교체가 안전했다.
- **딕셔너리 대입(`d[key] = 값`)은 문법이 하나뿐이지만, key가 이미 있었는지 없었는지에 따라 "새로 생성"과 "기존 값 갱신"이라는 다른 의미로 동작한다.**
- **`max(딕셔너리)`는 값이 아니라 key 중 최댓값을 반환한다.** 값 기준 최댓값은 `max(딕셔너리.values())`, 최댓값을 가진 key까지 찾으려면 `.items()`로 직접 순회하며 비교해야 한다.
- **"배열"과 "리스트"는 같은 개념을 가리키는 다른 층위의 용어다.** "배열"은 프로그래밍 전반에서 쓰이는 일반 개념이고, "리스트"는 파이썬이 그 개념을 구현한 실제 자료형 이름이다.
- **"브루트포스"는 반복문 개수로 정의되지 않는다.** 지름길 없이 가능한 모든 경우를 다 확인하는 접근 방식 자체를 가리키며, 이중 for문은 그 결과로 자주 등장하는 형태일 뿐이다. 반복문이 한 겹이어도 내부에서 `.count()` 같은 O(n) 연산을 매번 호출하면 실질적으로 브루트포스적 성격을 띨 수 있다.
- **반복문이 한 겹이라고 시간복잡도가 항상 O(n)인 것은 아니다.** 반복문 안에서 그 자체로 O(n)이 걸리는 연산을 호출하면 O(n²) 이상이 될 수 있다.
