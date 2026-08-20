# 최소공배수 / 연속 부분 수열 합 / 영어 끝말잇기 정리

브루트포스로 통과했지만 정석이 아니었던 문제, 시간복잡도가 갈리는 이유, 그리고 반환 수식을 짤 때의 인덱스 처리까지 세 문제를 통해 정리한다.

---

## 문제 1: n개 수의 최소공배수

배열 `arr`이 주어질 때 n개 수 전체의 최소공배수를 구한다.

```python
lcm([2,6,8,14])  # 168
lcm([1,2,3])     # 6
```

### 처음 풀이: 브루트포스 (통과는 했지만 정석 아님)
```python
def solution(arr):
    start = 1
    while True:
        count = 0
        result = arr[0] * start
        for i in arr[1:]:
            if result % i != 0:
                count += 1
        if count == 0:
            return result
        start += 1
```
`arr[0]`을 기준으로 배수를 하나씩 늘려가며 나머지 원소들이 전부 나눠떨어지는지 검증하는 방식. `arr[0] * start`는 항상 `arr[0]`의 배수이므로 `arr[1:]`만 검증해도 결과적으로 전체 원소를 검증한 것과 동일한 효과를 낸다 — **후보를 만드는 데 사용한 값과 검증에서 제외하는 값이 일치**하기 때문에 성립하는 구조다.

### 최적화 시도에서 드러난 핵심 원리: "후보 생성 기준"과 "검증 제외 대상"은 반드시 일치해야 한다
같은 브루트포스를 최댓값 기준으로 최적화하려는 시도:
```python
def solution(arr):
    m = max(arr)
    start = 1
    while True:
        count = 0
        result = m * start
        for i in arr[1:]:      # 버그: m 기준으로 후보를 만들었는데 검증은 arr[0]만 제외
            if result % i != 0:
                count += 1
        if count == 0:
            return result
        start += 1
```
`m = max(arr)`으로 후보를 만들었지만 검증에서는 여전히 `arr[1:]`(위치 기준으로 첫 번째 원소만 제외)을 사용해, `m`이 `arr[0]`과 다른 값일 경우 정작 검증에서 빠지는 원소가 `m`이 아니라 `arr[0]`이 되어버려 특정 원소가 검증 누락되는 구조적 오류가 발생했다. 애초 의도했던 "최댓값 기준으로 정렬한 뒤 그 정렬된 배열에서 첫 원소를 제외하고 검증"이라는 설계를 끝까지 일관되게(정렬된 배열 `s`로) 유지하지 않고 도중에 `arr`/`m`을 혼용한 것이 원인이었다.

### 정석 풀이: GCD 기반 공식
두 수의 최소공배수는 `lcm(a,b) = a*b // gcd(a,b)`로 바로 구할 수 있고, n개는 이 연산을 두 개씩 순차적으로 누적 적용(`lcm(a,b,c) = lcm(lcm(a,b),c)`)하면 된다.
```python
import math
from functools import reduce

def solution(arr):
    return reduce(lambda a, b: a * b // math.gcd(a, b), arr)
```
파이썬 3.9+에서는 `math.lcm(*arr)`로 한 번에 처리 가능. 브루트포스는 최악의 경우(서로소인 큰 수들) 후보를 매우 많이 늘려야 하지만, GCD 기반은 유클리드 호제법으로 배열 길이만큼만 순회하면 끝나 시간복잡도 차원이 다르다.

---

## 문제 2: 원형 수열의 연속 부분 수열 합의 개수

원형으로 연결된 수열에서 만들 수 있는 연속 부분 수열 합의 종류 개수를 구한다.

```python
solution([7,9,1,1,4])  # 18
```

### 정답 코드
```python
def solution(elements):
    l = elements * 2
    result = set()
    total = 0
    for i in range(len(elements)):
        for j in l[i:len(elements)+i]:
            total += j
            result.add(total)
        total = 0
    return len(result)
```
배열을 두 배로 이어붙여(`elements*2`) 원형 구조를 일반 배열 슬라이싱으로 다루는 방식. 시작점 `i`를 고정한 채 안쪽 루프에서 원소를 하나씩 더해가며(`total += j`), 그 시점마다 나온 합을 바로 `set`에 담아 중복 없이 관리한다.

### 시간복잡도가 갈리는 지점: 루프 순서에 따라 누적합 가능 여부가 결정된다
동일한 문제를 다르게 푼 풀이:
```python
def solution(elements):
    l = elements * 2
    sums = set()
    for length in range(len(elements)):
        for i in range(len(elements)):
            current_sum = sum(l[i:i+length+1])   # 매번 슬라이싱+합산을 새로 계산
            sums.add(current_sum)
    return len(sums)
```
이 방식은 **바깥 루프를 길이로, 안쪽 루프를 시작점으로** 잡았기 때문에, 안쪽 루프를 한 바퀴 돌 때마다 시작점이 계속 바뀌어 직전에 구한 합을 재사용할 수 없다. 그래서 매번 `sum(slice)`를 처음부터 다시 계산해야 하고, 이는 이중 루프(O(n²))에 슬라이싱+합산 비용(O(n))이 곱해져 사실상 O(n³)이 된다.

반대로 정답 코드는 **바깥 루프를 시작점으로, 안쪽 루프를 길이 증가로** 잡아서, 안쪽 루프가 진행될 때마다 "이전 길이의 합"에 원소 하나만 더하면 되는 누적합(running sum) 구조가 자연스럽게 성립한다. 이 덕분에 전체 시간복잡도가 O(n²)로 유지된다. 즉 같은 이중 루프라도 **어느 축을 바깥/안쪽에 두느냐가 누적합 적용 가능 여부, 나아가 시간복잡도 자체를 결정**한다.

---

## 문제 3: 영어 끝말잇기

n명이 순서대로 돌아가며 끝말잇기를 할 때, 가장 먼저 탈락하는 사람의 번호와 차례를 구한다.

```python
solution(3, ["tank","kick","know","wheel","land","dream","mother","robot","tank"])  # [3,3]
solution(2, ["hello","one","even","never","now","world","draw"])                    # [1,3]
```

### 정답 코드
```python
def solution(n, words):
    w = []
    seen = set()
    for index, word in enumerate(words):
        if w and (word in seen or word[0] != w[-1][-1]):
            return [index % n + 1, index // n + 1]
        w.append(word)
        seen.add(word)
    return [0, 0]
```

### 자료구조를 역할별로 분리: 리스트(순서 유지)와 set(중복 조회)를 함께 사용
`w`(리스트)와 `seen`(set)이 서로 다른 역할을 맡는다.
- `w[-1][-1]`: 직전 단어의 마지막 글자를 확인해야 하므로 **순서가 있는 리스트**가 필요
- `word in seen`: 이전에 나온 단어인지 반복적으로 조회해야 하므로 **O(1) 조회가 되는 set**이 유리 (리스트의 `in`은 O(n)이라 반복문 안에서 누적 호출 시 O(n²)로 커질 수 있음)

즉 set으로 리스트를 완전히 대체하는 게 아니라, "중복 조회"라는 역할만 떼어내 더 빠른 자료구조로 옮기고 "순서 유지"가 필요한 역할은 리스트에 남겨두는 방식이다.

### 조기 종료(early return) 패턴
탈락 조건을 만족하는 순간 바로 `return`하기 때문에 이후 단어는 검사할 필요가 없고, `else` 없이도 `if` 아래로 내려온 흐름 자체가 "탈락 아님"을 의미하게 되어 코드가 한 단계 평평해진다.

### 반환 수식: 0-indexed와 1-indexed 변환
`enumerate`가 주는 `index`는 0부터 시작하지만 사람 번호/차례는 1부터 세야 한다.
- 사람 번호: `index % n + 1`
- 차례: `index // n + 1`

이런 식은 감으로 한 번에 짜기보다, 작은 예시(`n=3`일 때 `index=0,1,2,3...`)를 직접 대입해 결과가 맞는지 검증하며 확정하는 방식이 실수를 줄이는 데 효과적이었다.

---

## 세 문제를 관통하는 공통 원리

- **탐색/조회가 반복되는 곳에는 리스트의 `in`보다 set/dict를 쓴다.** (O(n) 반복 조회 → O(1) 조회)
- **이중 루프에서 누적 가능한 값이 있다면, 그 값을 재사용할 수 있도록 루프 순서(바깥/안쪽 축)를 설계한다.** 같은 이중 루프라도 축을 어떻게 잡느냐에 따라 시간복잡도 차원 자체가 달라질 수 있다.
- **최적화를 시도할 때는 "무엇을 기준으로 값을 만들었는지"와 "무엇을 제외하고 검증하는지"가 항상 일치해야 한다.** 둘 중 하나만 바꾸고 나머지를 그대로 두면 논리가 깨진다.
- **반환 수식(특히 인덱스 변환)은 감으로 짜기보다 작은 값을 대입해 검증하며 확정한다.**
