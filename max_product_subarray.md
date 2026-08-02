# 리스트 심화 응용 + 최대 곱 부분배열

리스트 심화(`pop`, `remove`, `sort`, `insert`) 기초를 다진 뒤, 난이도를 올려 두 문제를 풀었다. 하나는 스택 응용, 하나는 지금까지 다룬 것 중 가장 복잡한 알고리즘적 사고(최댓값/최솟값 동시 추적 DP)가 필요했던 문제다.

---

## 문제 1: 카드 게임 시뮬레이션 (스택 응용)


카드 더미(리스트, 맨 뒤가 맨 위)에서 맨 위 카드가 짝수면 그 카드와 다음 카드를 버리고, 홀수면 멈추는 규칙을 시뮬레이션.

```python
card_game([1, 2, 4, 6, 8])   # [1]
card_game([2, 4, 3, 8])      # []
card_game([5, 2, 4])         # [5]
```

### 정답 코드
```python
def card_game(cards):
    while len(cards) > 1:
        if cards[-1] % 2 != 0:
            return cards
        else:
            cards.pop()
            cards.pop()
    return cards
```
`cards[len(cards)-1]`로 처음 작성했던 것을, 음수 인덱스(`cards[-1]`)로 정리. `pop()` 두 번으로 짝을 이루는 두 카드를 동시에 제거하는 스택 활용.

이 문제는 규칙이 곧바로 코드 구조와 1:1로 대응되어 알고리즘적 사고가 거의 필요 없었고, 다음 문제에서 실제로 사고가 필요한 난이도로 조정했다.

---

## 문제 2: 최대 곱 부분배열 (Maximum Product Subarray)


연속된 부분배열 중 원소들을 모두 곱했을 때 최댓값이 되는 곱을 구한다.

```python
max_product_subarray([2, 3, -2, 4])   # 6  ([2,3])
max_product_subarray([-2, 3, -4])     # 24 (전체를 곱하면 음수 두 개가 상쇄되어 양수)
max_product_subarray([-2])            # -2
```

`max_subarray_sum`(최대 부분**합**)과 겉보기엔 비슷하지만, **곱셈은 음수를 만나면 최댓값과 최솟값이 서로 뒤바뀔 수 있다**는 점에서 완전히 다른 사고가 필요했다.

### 버전 1: 브루트포스
```python
def max_product_subarray(numbers):
    max_value = max(numbers)
    count = 0
    for i in numbers:
        current_value = i
        count = count + 1
        for j in numbers[count:]:
            current_value = current_value * j
            if max_value < current_value:
                max_value = current_value
    return max_value
```
모든 (시작점, 끝점) 쌍을 검사하되, 매번 처음부터 곱을 다시 계산하지 않고 이전 값에 새 원소만 곱해 누적한다 (O(n²)).

**중요한 확인**: `max_value = max(numbers)` 초기화가 이 구조에서는 **필수**다. `max_subarray_sum`의 브루트포스 버전에서는 이 초기화가 중복이었는데(안쪽 반복문이 `n=0`부터 시작해 단일 원소 경우를 자동으로 커버했으므로), 이 코드는 안쪽 반복문이 항상 "그다음 원소부터" 곱하기 시작해 **단일 원소 자체는 반복문 안에서 한 번도 비교되지 않는다.** 반례로 확인: `max(numbers)` 초기화 없이 `numbers[0]`으로 시작하면 `[-100, 50]`에서 정답 `50`(마지막 원소가 곧 정답)을 놓치고 `-100`이 나온다. 같은 형태(초기화)라도 코드 구조가 다르면 그 초기화가 필수인지 중복인지가 달라진다는 것을 확인.

브루트포스 카테고리 안에서는 인덱스 접근 대신 슬라이싱(`numbers[count:]`)을 쓰는 지금 형태가 이미 최선이라는 것도 확인 — 인덱스로 바꾸면 이론상 아주 미세하게 빠를 수 있으나 코드가 더 복잡해져 실질적 이득이 없는, "짧음/속도가 항상 가독성을 이기지는 않는다"는 원칙의 사례.

### 핵심 발상: 왜 최솟값까지 같이 추적해야 하는가
직접 손으로 `[-2, 3, -4]`를 굴려서 확인한 과정:
- 위치 0(`-2`): 끝나는 최댓값 `maxEnd=-2`, 최솟값 `minEnd=-2`
- 위치 1(`3`): 후보 `3, -2×3=-6, -2×3=-6` → `maxEnd=3`, `minEnd=-6`
- 위치 2(`-4`): 후보 `-4, 3×-4=-12, minEnd(-6)×-4=24` → **직전의 최솟값(-6, 가장 나빴던 값)이 새로운 음수를 만나 24(최댓값)로 반전됨**

"지금은 나쁜 값이지만, 나중에 음수를 만나면 최고의 값으로 바뀔 수 있는 후보"를 버리지 않고 계속 들고 다녀야 한다는 것이 핵심이다. `max_subarray_sum`(Kadane)의 "값 하나만 추적" 패턴을 그대로 재사용하려던 첫 시도가 이 지점에서 통하지 않았다.

### 점화식
```
maxEnd[i] = max(numbers[i], maxEnd[i-1] * numbers[i], minEnd[i-1] * numbers[i])
minEnd[i] = min(numbers[i], maxEnd[i-1] * numbers[i], minEnd[i-1] * numbers[i])
```

### 시행착오 1: 갱신 순서 문제
```python
max_end = max(numbers[i], max_end * numbers[i], min_end * numbers[i])
min_end = min(numbers[i], max_end * numbers[i], min_end * numbers[i])  # 버그
```
두 번째 줄의 `max_end`가 이미 첫 번째 줄에서 갱신된 "새 값"이라, `min_end` 계산이 "직전 위치의 max_end"가 아니라 "방금 만든 값"을 잘못 참조한다. `before_max_end`라는 임시 변수로 갱신 전 값을 붙잡아두는 방식으로 1차 해결.

### 시행착오 2: "여기서 끝나는 값"과 "전체 답"의 혼동
```python
return max_end   # 버그: 마지막 위치의 max_end만 반환
```
`[3, -1]`처럼 앞쪽에서 이미 최선의 답(`3`)이 나왔는데 뒤에서 더 낮은 값(`-1`)으로 이어지면, 마지막 위치의 `max_end`만 반환하는 바람에 앞서 찾은 더 좋은 값을 잃어버린다. `max_subarray_sum`의 `max_sum`(전체 최고 기록을 별도로, 절대 줄어들지 않게 유지하는 변수)과 정확히 같은 역할의 변수(`total`)가 빠져 있었던 것. `total`을 추가해 매 위치마다 `max_end`와 비교 갱신하도록 수정.

### 최종 정답 코드 (임시 변수 없이 동시 대입으로 정리)
```python
def max_product_subarray(numbers):
    max_end = 1
    min_end = 1
    total = max(numbers)

    for n in numbers:
        candidates = (n, max_end * n, min_end * n)
        max_end, min_end = max(candidates), min(candidates)
        total = max(total, max_end)

    return total
```
`max_end, min_end = max(candidates), min(candidates)`처럼 **동시 대입**을 쓰면, 오른쪽의 모든 식이 먼저 다 계산된 뒤에야 왼쪽에 나눠 담기므로(어제 튜플 논의에서 확인한 원리), `before_max_end` 같은 임시 변수 없이도 "갱신 전 값을 참조해야 하는" 문제가 원천적으로 해결된다. 5000개의 랜덤 케이스로 브루트포스와 대조해 검증 완료.

---

## 핵심 논의: 이 문제가 왜 유독 어려웠는가

### "발상을 못 떠올린 것"과 "패턴을 안 배운 것"의 구분
최솟값을 같이 추적해야 한다는 아이디어는, 곱셈이라는 연산의 부호 반전 성질을 실제로 반례를 만들어 겪어보기 전에는 알아채기 매우 어려운 종류였다. `character_pair_count_notes.md`의 원칙(발상 부재 vs 안 배운 도구)을 다시 확인한 사례이며, 이 패턴("최댓값/최솟값 동시 추적 DP")은 스스로 재발명하기보다 한 번 배우고 나면 유사 상황에서 알아보는 방식으로 습득되는 표준 패턴 중 하나임을 확인.

### 재귀의 "믿고 넘어가기"와 이 문제의 차이
`max_end`/`min_end`/`before_max_end`가 서로 얽혀 있는 게 재귀처럼 복잡하게 느껴졌으나, 둘은 다르다.
- **재귀**: 재귀 호출 "내부에서 실제로 벌어지는 실행"을 아예 안 보고, 그 함수가 약속한 결과만 믿는다 (안 봐도 되는 것을 블랙박스로 취급).
- **이 문제**: 안을 안 봐도 되는 블랙박스가 없다 — 이 몇 줄 자체가 로직의 전부다. 대신 "낱개 변수 하나하나"가 아니라 **"(max, min) 한 쌍을 갱신하는 하나의 단계"**로 개념을 압축해서 이해해야 한다.

정리하면: 재귀는 "볼 필요 없는 부분을 안 본다"이고, 이 문제는 "봐야 할 여러 낱개를 하나의 더 큰 단위로 묶어서 본다"는, 둘 다 "세부를 전부 실시간 추적하지 않는다"는 공통점은 있지만 서로 다른 종류의 압축(청킹)이라는 것을 확인했다.
