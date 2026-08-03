# 이진 탐색(Binary Search) 입문 — 2문제 정리

지금까지의 알고리즘 패턴(투 포인터, 슬라이딩 윈도우 등)과 달리, "정렬된 데이터"에서만 쓸 수 있는 대신 매번 탐색 범위를 절반씩 줄여 O(log n)에 답을 찾는 새 패턴. 기본 탐색 → 중복값 처리 순서로 다뤘다.

---

## 개념: 이진 탐색이란

전화번호부에서 이름을 찾을 때 앞에서부터 한 장씩 넘기지 않고, 책 한가운데를 펼쳐 "찾는 이름이 이 페이지보다 앞인지 뒤인지"를 확인한 뒤, 그 절반만 다시 살펴보는 방식. 정렬된 리스트에서만 성립하며, 매 단계 탐색 범위가 절반씩 줄어드므로 리스트가 아무리 커도 확인 횟수는 log(n)번(100만 개도 최대 20번)뿐이다.

### 기본 구조
```python
left = 0
right = len(numbers) - 1

while left <= right:
    mid = (left + right) // 2
    if numbers[mid] == target:
        return mid
    elif numbers[mid] < target:
        left = mid + 1
    else:
        right = mid - 1

return -1
```
`left`, `right` 두 포인터를 쓴다는 점에서 투 포인터와 비슷해 보이지만, 한 칸씩 움직이는 게 아니라 매번 절반씩 건너뛴다는 게 다르다.

---

## 문제 1: 정렬된 리스트에서 값 찾기

**예상 난이도: 브론즈 4**

```python
binary_search([1, 3, 5, 7, 9, 11], 7)   # 3
binary_search([1, 3, 5, 7, 9, 11], 4)   # -1
```

### 정답 코드
```python
def binary_search(numbers, target):
    left = 0
    right = len(numbers) - 1

    while left <= right:
        mid = (left + right) // 2
        if numbers[mid] == target:
            return mid
        elif numbers[mid] < target:
            left = mid + 1
        else:
            right = mid - 1
    
    return -1
```

### 시행착오: 반복문 뒤에 남겨둔 죽은 코드
```python
while left <= right:
    ...
if mid != target:
    return -1
return mid
```
`while` 안에서 값을 찾으면 이미 `return mid`로 함수가 끝나므로, 반복문을 **빠져나왔다는 것 자체가 이미 실패(못 찾음)가 확정된 상태**라는 것을 놓쳤다. 게다가 `mid`(인덱스)와 `target`(값)을 비교하는 것 자체가 의미상 맞지 않는 비교였는데, 우연히 테스트 케이스들에서 인덱스와 값이 달라 정답처럼 보였을 뿐이었다. 반복문을 빠져나오면 무조건 `return -1`로 정리.

빈 리스트, 원소 하나짜리 리스트 등 극단적 경계 케이스까지 검증 완료.

---

## 문제 2: 중복된 값의 첫 번째 위치 찾기

**예상 난이도: 실버 4~3**

```python
find_first([1, 2, 2, 2, 3, 4], 2)   # 1 (2가 처음 나오는 위치)
find_first([1, 2, 2, 2, 3, 4], 5)   # -1
find_first([1, 1, 1, 1], 1)         # 0
```

값이 여러 번 중복될 수 있을 때, "값을 찾으면 즉시 반환"하는 기본 이진 탐색의 습관을 비틀어야 하는 문제. `numbers[mid] == target`을 만나도 바로 반환하지 않고, 일단 후보로 기억해두되 혹시 더 앞쪽에도 같은 값이 있는지 계속 왼쪽을 살펴봐야 한다.

### 시행착오: `.count()`로 사전 확인 (O(n)으로 퇴화)
```python
if numbers.count(target) == 0:
    return -1
```
정답은 맞지만, `.count()` 자체가 리스트 전체를 훑는 O(n) 연산이라, 애써 만든 O(log n) 이진 탐색의 장점을 이 한 줄이 무색하게 만든다.

### 최종 정답 코드
```python
def find_first(numbers, target):
    left = 0
    right = len(numbers) - 1
    result = None

    while left <= right:
        mid = (left + right) // 2
        if numbers[mid] == target:
            result = mid
            right = mid - 1
        elif numbers[mid] > target:
            right = mid - 1
        else:
            left = mid + 1

    if result is None:
        return -1
    return result
```
`.count()`로 사전 확인하는 대신, `result = None`으로 "아직 한 번도 못 찾았다"는 상태를 표현했다. 값을 찾아도 즉시 반환하지 않고 `result`에 저장한 뒤 `right = mid - 1`로 계속 왼쪽을 탐색하고, 반복문이 끝난 뒤 `result`가 여전히 `None`이면 실패로 간주한다. `find_most_frequent`에서 이미 쓴 `best_value = None` 패턴(존재 여부까지 구분해야 할 때 `None`을 쓰는 것)을 여기서도 재활용했다. `.count()` 없이 완전한 O(log n) 달성.

### 검증
`find_most_frequent`류의 랜덤/경계 테스트를 포함해 모든 케이스(중복, 미존재, 전부 동일값)에서 정답 확인.
