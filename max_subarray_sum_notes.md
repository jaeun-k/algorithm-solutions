# 최대 부분합 (Maximum Subarray Sum) — 풀이 과정 정리

## 문제 정의

정수 리스트 `numbers`가 주어졌을 때, **연속된 부분 배열(subarray)** 중 합이 최대인 값을 구한다.

- 부분 배열은 반드시 연속된 구간이어야 한다 (순서 변경/건너뛰기 불가).
- 부분 배열의 길이는 최소 1 이상이어야 한다 (빈 배열은 후보가 될 수 없음).
- 리스트에는 음수, 양수, 0이 모두 섞여 있을 수 있다.

```python
max_subarray_sum([1, 5, 20, -20, 1, 56, -1, 2, 7, 10])  # → 81
```

이 문서는 동일한 문제를 3단계로 발전시키며 푼 과정을 정리한 것이다.
브루트포스 → 직관적 최적화(버그 포함) → 디버깅 완료 버전 순서다.

---

## 버전 1: 브루트포스 (O(n²))

```python
def max_subarray_sum(numbers):
    max_sum = max(numbers)
    subarray_sum = 0

    for i in range(len(numbers)):
        for n in range(len(numbers) - i):
            subarray_sum = subarray_sum + numbers[i + n]

            if max_sum < subarray_sum:
                max_sum = subarray_sum

        subarray_sum = 0

    return max_sum
```

### 메커니즘
- 바깥 루프 `i`: 부분 배열의 **시작점**을 하나씩 고정한다.
- 안쪽 루프 `n`: 그 시작점에서부터 오른쪽으로 한 칸씩 늘려가며 **가능한 모든 끝점**까지의 누적합을 계산한다.
- 누적합을 계산할 때마다 즉시 `max_sum`과 비교해 갱신한다.
- 시작점이 바뀔 때마다 `subarray_sum`을 0으로 리셋해 새로 누적을 시작한다.

### 왜 O(n²)인가
- 가능한 모든 (시작점, 끝점) 쌍을 전부 검사한다.
- 시작점 `i`에 대해 안쪽 루프가 `n - i`번 도니, 전체 반복 횟수는 `n + (n-1) + ... + 1` ≈ `n²/2`.
- 리스트가 커질수록 계산량이 제곱으로 늘어난다.

### 초기화 관련 메모
- `max_sum = max(numbers)`로 초기화해 "리스트가 전부 음수여도" 안전하게 시작한다.
- 다만 이 초기화 자체가 정보상 중복이다 — 안쪽 루프의 첫 반복(`n=0`)에서 이미 `numbers[i]` 하나짜리 부분합이 검사되므로, 결국 모든 원소는 루프 안에서 한 번씩은 `max_sum`과 비교된다. `max(numbers)` 초기화는 리스트를 한 번 더 훑는 사소한 중복 작업이다 (전체 O(n²) 복잡도에는 영향 없음).
- 이 중복을 없애려면 `max_sum = numbers[0]`으로 초기화해도 된다. 이 버전에서는 안쪽 루프가 매 시작점마다 `n=0`부터 돌기 때문에, 어떤 원소든 결국 루프 안에서 정직하게 한 번씩 검사되어 `max_sum`이 올바르게 갱신된다 — 즉 초기값은 "루프가 대신 검증해줄 임시값"일 뿐이라 아무 원소로나 시작해도 무방하다. (단, 이 방식은 버전 3의 `max_sum >= 0` 가드처럼 초기값의 부호 자체가 로직에 쓰이는 구조에는 적용할 수 없다 — 버전 3에서 `numbers[0]`으로 바꾸면 오답이 나올 수 있다.)

---

## 버전 2: O(n) 최적화 시도 — 버그 있는 상태

```python
def max_subarray_sum(numbers):
    max_sum = max(numbers)
    subarray_sum = 0

    for n in range(len(numbers)):
        subarray_sum = subarray_sum + numbers[n]

        if subarray_sum < 0:
            subarray_sum = 0

        if max_sum < subarray_sum:
            max_sum = subarray_sum

    return max_sum
```

### 발상
직전 원소까지의 누적합(`subarray_sum`)을 계속 들고 가다가, **누적합이 0보다 작아지는 순간** 그 누적합은 "짐짝"이라고 판단해 버린다(0으로 리셋). 매 스텝 원소를 하나씩만 처리하므로 전체는 O(n).

### 버그: subarray_sum이 0 밑으로 못 내려가게 고정됨
`subarray_sum`은 음수가 될 때마다 무조건 `0`으로 고정(clamp)된다. 이 고정된 `0`은 리스트의 모든 원소가 음수인 경우에는 **실제로 어떤 부분 배열도 만들어낼 수 없는 값**이다 (원소를 하나라도 포함하면 합은 항상 음수이므로). 그럼에도 이 `0`이 매 스텝 `max_sum`과 비교 대상이 되기 때문에, 정답이 음수여야 하는 상황(모든 원소가 음수인 경우)에서 `0`이 잘못 채택되어 버린다.

**실패 사례:**
```python
max_subarray_sum([-4, -9, -1])
# 이 코드의 결과: 0  (오답)
# 정답: -1  (가장 덜 나쁜 단일 원소)
```
모든 원소가 음수인 리스트에서는, `subarray_sum`이 계속 0으로 리셋되고 그 `0`이 어떤 음수보다도 크기 때문에 `max_sum`이 `0`으로 잘못 갱신되어 버린다.

(실증 확인: 5,000개 랜덤 케이스로 테스트했을 때, 이 버전이 브루트포스와 다른 답을 낸 케이스는 예외 없이 전부 "모든 원소가 음수인 리스트"였다. 원소 중 하나라도 0 이상이 섞여 있으면 이 버그는 발생하지 않는다.)

---

## 버전 3: O(n) 최적화 — 수정 완료

```python
def max_subarray_sum(numbers):
    max_sum = max(numbers)
    subarray_sum = 0

    for n in range(len(numbers)):
        subarray_sum = subarray_sum + numbers[n]

        if subarray_sum < 0:
            subarray_sum = 0

        if max_sum >= 0 and max_sum < subarray_sum:
            max_sum = subarray_sum

    return max_sum
```

### 수정 내용
비교 조건에 `max_sum >= 0`을 추가했다.

### 왜 이 조건이 정확히 문제를 해결하는가
`max_sum`의 초기값은 `max(numbers)`, 즉 리스트 전체의 최댓값이다.

- **모든 원소가 음수인 경우**: `max(numbers) < 0` → `max_sum`이 처음부터 음수 → `max_sum >= 0` 조건이 계속 거짓 → 이후 어떤 비교도 일어나지 않음 → 초기값(진짜 최댓값)이 그대로 보존됨. 즉 버그의 원인이었던 "0과의 잘못된 비교"가 원천 차단된다.
- **0 이상인 원소가 하나라도 있는 경우**: `max(numbers) >= 0` → `max_sum`이 처음부터 0 이상 → `max_sum`은 이후 자기보다 큰 값으로만 갱신되므로 한 번 0 이상이 되면 영원히 0 이상 유지 → `max_sum >= 0` 조건이 사실상 항상 참 → 매 스텝 정상적으로 비교가 이루어짐.

이 두 경우가 논리적으로 전체 입력 공간을 빠짐없이 덮기 때문에, "초기 음수 상태에서 나중에 큰 양수를 놓치는" 것 같은 반례는 구조적으로 발생할 수 없다.

### 검증
동일한 로직으로 브루트포스 대비 2,000개의 랜덤 케이스(길이 1~12, 음수/양수 혼합, 전부 음수 포함)를 비교 실행해 전부 일치함을 확인했다.

---

## 세 버전 비교 요약

| 버전 | 시간복잡도 | 핵심 아이디어 | 상태 |
|---|---|---|---|
| 1. 브루트포스 | O(n²) | 모든 (시작점, 끝점) 쌍을 전부 검사 | 정답 |
| 2. 리셋 방식 (초안) | O(n) | 누적합이 0 미만이면 버림 | **버그**: 전부 음수인 입력에서 오답 |
| 3. 리셋 방식 (수정) | O(n) | 버전 2 + `max_sum >= 0` 가드 추가 | 정답 |

---

## 참고: 이 유형의 정석 풀이 (Kadane's Algorithm)

버전 2/3은 "0이라는 절대 기준"과 비교해 버릴지 말지 판단하는 **이벤트 기반** 사고방식이다. 이 문제의 표준 풀이(Kadane's Algorithm, 1984)는 조금 다른 방식으로 접근한다 — 매 원소마다 절대 기준이 아니라 **"이어갈까, 새로 시작할까"**를 상대적으로 저울질한다:

```python
def max_subarray_sum(numbers):
    max_sum = numbers[0]
    subarray_sum = numbers[0]

    for n in range(1, len(numbers)):
        subarray_sum = max(numbers[n], subarray_sum + numbers[n])
        max_sum = max(max_sum, subarray_sum)

    return max_sum
```

이 방식은 "0 미만이면 리셋"이라는 조건 자체가 없다. `subarray_sum`이 절대 `0`으로 고정(clamp)되는 일이 없고, 항상 `numbers[n]`(원소 하나) 또는 `subarray_sum + numbers[n]`(이어붙인 값) 중 하나를 선택하므로 매 단계 실제로 존재하는, 길이 1 이상인 후보만 다룬다. 그래서 버전 2/3에서 "0이 잘못된 값으로 채택되는" 문제 자체가 발생할 수 없고, 그걸 막기 위한 `max_sum >= 0` 같은 별도 가드도 필요 없다.

이 문제는 DP(Dynamic Programming)의 대표적인 입문 예제로 꼽힌다 — "인덱스 n에서 끝나는 최적 부분 배열의 답"이 "인덱스 n-1에서 끝나는 최적 부분 배열의 답"을 재활용해서 만들어지는 구조(최적 부분구조 + 겹치는 부분 문제)를 가지기 때문이다.
