# 슬라이딩 윈도우 (Sliding Window) — 2문제 정리

지금까지의 투 포인터와는 다른, "정해진(또는 조건에 따라 변하는) 크기의 구간이 리스트 위를 미끄러지듯 이동하며 훑는" 새 알고리즘 패턴. 고정 크기 → 가변 크기 순서로 다뤘다.

---

## `window`라는 용어

전체 리스트 중 지금 들여다보고 있는 일정 구간을 "창문(window)"에 비유한 이름. 창문을 리스트 위에서 옆으로 미끄러뜨리면, 한쪽 끝이 빠지고 반대쪽 끝이 새로 들어오는 것만 반영하면 되므로, 매번 전체를 다시 계산할 필요가 없다는 게 이 패턴의 핵심.

---

## 문제 1: 길이 k 부분배열의 최대 합 (고정 크기 윈도우)

**예상 난이도: 브론즈 4**

```python
max_sum_k([1, 4, 2, 10, 2, 3, 1], 4)   # 18 ([4,2,10,2])
max_sum_k([2, 3, 5], 2)                # 8
```

### 정답 코드
```python
def max_sum_k(numbers, k):
    window_sum = sum(numbers[:k])
    max_sum = window_sum

    for i in range(len(numbers) - k):
        window_sum = window_sum + numbers[k + i] - numbers[i]
        max_sum = max(max_sum, window_sum)

    return max_sum
```
첫 `k`개의 합을 구해두고(처음엔 반복문으로, 나중에 `sum()`으로 대체), 그다음은 "빠지는 값 빼고 들어오는 값 더하기"만으로 O(1) 갱신. 전체 O(n).

### 변수명 정리
초기 버전에서 `max_sum`(윈도우의 현재 합을 담고 있었음)과 `result`(진짜 전체 최댓값)의 이름이 역할과 어긋나 있었다. `window_sum`(지금 창문 안의 합)과 `max_sum`(지금까지 본 것 중 전체 최댓값, `max_subarray_sum`의 명명 관례와 일관)으로 정리.

### 두 개의 for문(또는 sum()+for문)이 맞는 구조인 이유
"초기 윈도우 채우기"와 "이후 슬라이딩 갱신"은 계산 방식 자체가 다른(하나는 누적, 하나는 차분) 별개의 하위 작업이라, 다이아몬드 문제에서 확립한 기준("이 반복이 원래 하나의 흐름인지, 원래 두 개의 하위 문제인지")에 따라 나뉘어 있는 게 맞는 구조. 억지로 하나의 반복문에 합치면 "아직 첫 윈도우가 안 채워졌는지"를 구분하는 조건문이 반복문 안에 새로 생겨 오히려 복잡해짐(직접 병합 시도로 확인).

### `sum()` 함수
리스트의 모든 값을 더한 합을 반환하는 내장 함수. `sum(numbers[:k])`로 초기 윈도우 합 계산 반복문 자체를 대체.

---

## 문제 2: 합이 target 이상인 최소 길이 부분배열 (가변 크기 윈도우)

**예상 난이도: 실버 3~2**

```python
min_length_subarray([2, 3, 1, 2, 4, 3], 7)   # 2 ([4,3])
min_length_subarray([1, 1, 1, 1], 100)       # 0 (불가능)
min_length_subarray([5], 5)                  # 1
```
고정 크기와 달리, 조건(합이 target 이상)을 만족하는 가장 짧은 구간을 찾아야 하므로 `left`, `right` 두 포인터가 서로 다른 속도·조건으로 움직인다.

### 버전 1: 매번 구간 합을 다시 계산 (느림)
```python
while right < len(numbers):
    window_sum = sum(numbers[left:right + 1])   # 매번 슬라이싱+합산을 새로 함
    ...
```
로직은 맞지만, `sum(numbers[left:right+1])`을 매 반복 다시 계산하는 게 `max_sum_k` 첫 시도에서 확인한 것과 같은 비효율.

### 버전 2: O(1) 갱신 + 예외 처리로 경계 우회
```python
def min_length_subarray(numbers, target):
    if sum(numbers[:len(numbers)]) < target:
        return 0

    left = 0
    right = 0
    min_length = len(numbers)
    window_sum = numbers[0]

    while right < len(numbers):
        if window_sum >= target:
            if min_length > right - left + 1:
                min_length = right - left + 1
            window_sum = window_sum - numbers[left]
            left = left + 1
        else:
            right = right + 1
            try:
                window_sum = window_sum + numbers[right]
            except IndexError:
                return min_length

    return min_length
```
`window_sum`을 빼고 더하는 식으로 O(1) 갱신하는 데는 성공했으나, `right`가 리스트 끝을 넘어가는 경계를 `try`/`except IndexError`로 우회함. 정답이지만, 이는 이 문제 유형의 정식 구조라기보다 임시방편에 가까움.

### 버전 3 (최종): `for` + `while` 조합, 정석 슬라이딩 윈도우 구조
```python
def min_length_subarray(numbers, target):
    left = 0
    window_sum = 0
    min_length = len(numbers) + 1

    for right in range(len(numbers)):
        window_sum = window_sum + numbers[right]

        while window_sum >= target:
            min_length = min(min_length, right - left + 1)
            window_sum = window_sum - numbers[left]
            left = left + 1

    if min_length == len(numbers) + 1:
        return 0
    return min_length
```
- **바깥 `for right in range(...)`**: `right`가 한 칸씩 늘며 창문을 넓힘 (`range()`가 범위를 자동으로 보장하므로 예외 처리 불필요)
- **안쪽 `while window_sum >= target`**: 조건을 만족하는 동안 `left`를 최대한 밀어 창문을 좁힘
- **`min_length = len(numbers) + 1`**: "절대 나올 수 없는 값"으로 초기화해, 끝까지 이 값이면 조건을 한 번도 만족 못 한 것이므로 `0`을 반환 — 함수 맨 앞의 사전 확인(`sum(numbers) < target`)이 아예 필요 없어짐. `top_three`/`insert_sorted`에서는 "특별 케이스를 미리 걸러내는" 방식이었다면, 이번엔 "초기값 설계 자체가 특별 케이스까지 포함해서 처리"하는 반대 방향의 접근.

---

## 핵심 논의: 왜 이중 반복문인데 O(n²)이 아닌가

`for` 안에 `while`이 있는 구조를 보고 "이거 n²아니냐"는 의문이 들었으나, 실측(각 포인터의 총 이동 횟수를 세어봄)으로 확인한 결과 `right`는 정확히 n번, `left`는 최대 n번만 움직였다.

### 최종적으로 정리된, 가장 명확한 설명 (수식이 아니라 구간 분할로)
`right`가 전체 구간(0~n)을 지나는 동안, `left`가 움직이는 구간을 몇 개로 쪼개서 보든(예: 0~3구간에서 3번, 3~6구간에서 3번, 6~n구간에서 n-6번), 그 조각들을 다 더하면 결국 `left`가 갈 수 있는 자리(`0`~`n`) 총량인 `n`을 절대 못 넘는다. **`left`가 되돌아가지 않는다는 사실 하나만으로, 개별 반복에서 몇 번 움직였는지 하나하나 셀 필요 없이 "총합은 최대 n"이라는 게 보장된다.**

`right`와 `left`가 극단적으로 매번 같이 1씩 늘어나는 최악의 상상 케이스로 봐도: `right`가 n번 도는 동안 `left`도 최대 n번 도니, 전체 합쳐진 일의 양은 `n(right 몫) + n(left 몫) = 2n`이며, 이는 n²과 비교하면 n이 커질수록 압도적으로 작다.

이 원리는 `merge_sorted`("i, j 둘 다 절대 되돌아가지 않으니 안쪽 반복의 총 이동 횟수를 합쳐도 O(n)")에서 이미 확인한 것과 정확히 같으며, "반복문 개수만 보고 시간복잡도를 판단하면 안 된다"는 원칙이 "겉보기엔 이중 구조인데 실제로는 O(n)"이라는 방향으로 다시 확인된 사례. 이런 분석 방식을 분할상환분석(amortized analysis)이라 부르며, 핵심은 "각 반복에서 안쪽이 몇 번 도는지 개별로 따지지 않고, 안쪽 포인터가 전체 실행 동안 갈 수 있는 최대 총량이 얼마인지"만 보면 된다는 것.
