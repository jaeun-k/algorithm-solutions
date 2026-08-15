# 휴대폰 자판 최소 키 입력 횟수 (Keypad Min Presses)

키 하나에 여러 문자가 순서대로 할당된 휴대폰 자판에서, 목표 문자열을 작성하기 위해 필요한 최소 키 입력 횟수를 구하는 문제. "존재 여부 판별의 함정"과 "중복 계산 제거를 통한 최적화"라는 두 단계로 정리한다.

---

## 문제 설명

`keymap[i]`는 `i+1`번 키를 눌렀을 때 순서대로 바뀌는 문자를 의미한다. 예를 들어 `keymap[0] = "ABACD"`이면 1번 키를 한 번 누르면 A, 두 번 누르면 B, 세 번 누르면 다시 A가 된다.

`keymap`과 목표 문자열 배열 `targets`가 주어질 때, 각 목표 문자열을 작성하는 데 필요한 최소 키 입력 횟수를 배열로 반환한다. 작성이 불가능한 문자열은 `-1`을 저장한다.

```python
keymap = ["ABACD", "BCEFD"]
targets = ["ABCD", "AABB"]
solution(keymap, targets)  # [9, 4]

keymap = ["AA"]
targets = ["B"]
solution(keymap, targets)  # [-1]
```

---

## 1차 시도: `zip_longest`로 keymap을 세로로 묶어 탐색

```python
from itertools import zip_longest

def solution(keymap, targets):
    answer = [0] * len(targets)
    for index, target in enumerate(targets):
        for own in target:
            for n, ch in enumerate(zip_longest(*keymap, fillvalue=None)):
                if own in ch:
                    answer[index] += (n + 1)
                    break
        if answer[index] == 0:
            answer[index] = -1
    return answer
```

### 시행착오: "실패 판정 조건"이 잘못됨
target 문자열의 특정 문자가 keymap 어디에도 없을 때, 안쪽 이중 for문은 `break` 없이 그냥 끝까지 돌고 조용히 다음 문자로 넘어간다. 즉 "이 문자를 못 찾았다"는 사실이 어디에도 기록되지 않는다.

그 결과, 실패 여부를 `answer[index] == 0`(총 입력 횟수의 합이 0인가)으로 판단하는 게 문제였다.

- target의 **모든 글자**가 keymap에 없는 경우 → 합이 끝까지 0으로 남아 우연히 `-1` 처리가 맞아떨어짐
- target의 **일부 글자만** keymap에 없는 경우 → 나머지 글자들 덕분에 합이 0이 아닌 값이 되어버려, 원래 `-1`이어야 할 케이스가 엉뚱한 양수로 반환됨

즉 "총합이 0인가"와 "찾아야 할 문자를 전부 찾았는가"는 서로 다른 조건인데, 이걸 같은 것으로 착각한 것이 원인이었다.

### 수정: 찾은 문자 개수를 별도로 추적
```python
count = 0
for own in target:
    for n, ch in enumerate(zip_longest(*keymap, fillvalue=None)):
        if own in ch:
            answer[index] += (n + 1)
            count += 1
            break
if count != len(target):
    answer[index] = -1
```
"몇 개의 문자를 실제로 찾았는가(`count`)"를 target의 전체 길이(`len(target)`)와 비교해, 하나라도 못 찾은 게 있으면 `-1`로 덮어쓰도록 고쳤다.

---

## 2차 시도: 딕셔너리 전처리로 최적화

1차 코드는 target의 문자 하나를 확인할 때마다 `zip_longest(*keymap, ...)`를 매번 새로 만들어 처음부터 순회한다. keymap은 고정되어 있으므로, "문자 → 최소로 눌러야 하는 횟수" 매핑을 미리 한 번만 계산해두면 이 중복 계산을 없앨 수 있다는 데서 출발했다.

```python
def solution(keymap, targets):
    alpha = {}
    result = [0] * len(targets)

    for i in keymap:
        count = 1
        for j in i:
            if j not in alpha:
                alpha[j] = count
            else:
                if count < alpha[j]:
                    alpha[j] = count
            count += 1

    for i in range(len(targets)):
        count2 = 0
        for j in range(len(targets[i])):
            if targets[i][j] in alpha:
                result[i] += alpha[targets[i][j]]
                count2 += 1
        if count2 == 0:
            result[i] = -1
    return result
```

### 같은 함정에 다시 걸림
전처리 로직으로 바꾸는 데 집중하다가, 실패 판정 조건을 또다시 `count2 == 0`으로 썼다. 1차 시도에서 이미 진단했던 것과 정확히 같은 문제(부분 실패를 못 잡아내는 조건)가 코드가 복잡해진 틈을 타 그대로 재발한 것이다.

### 최종 수정
```python
if count2 != len(targets[i]):
    result[i] = -1
```
"찾은 개수"와 "target 전체 길이"를 비교하는 조건으로 바꿔 해결했다.

---

## 최종 정리 코드

로직은 그대로 두고, 표현만 정리한 버전.

```python
def solution(keymap, targets):
    alpha = {}

    for key in keymap:
        for count, ch in enumerate(key, start=1):
            alpha[ch] = min(count, alpha.get(ch, count))

    result = [0] * len(targets)
    for i, target in enumerate(targets):
        count2 = 0
        for ch in target:
            if ch in alpha:
                result[i] += alpha[ch]
                count2 += 1
        if count2 != len(target):
            result[i] = -1

    return result
```

- `count = 1`을 직접 증가시키던 부분 → `enumerate(key, start=1)`로 대체 (인덱스 시작값을 1로 지정)
- `if/else`로 최소값을 비교하던 부분 → `min(count, alpha.get(ch, count))` 한 줄로 축약
- `range(len(...))` + 인덱스 접근 → `enumerate(targets)` / `for ch in target`으로 직접 순회
- 변수명 `i`, `j` → `key`, `ch`, `count`로 의미가 드러나게 정리

---

## 이 문제에서 얻은 교훈

### "총합이 0인가"와 "전부 찾았는가"는 다른 조건이다
target 문자열 중 **일부만** 실패하는 경우를 검증하지 않으면, 전부 실패하거나 전부 성공하는 케이스만으로는 코드가 맞아 보일 수 있다. "실패를 어떻게 판정할 것인가"는 "성공 로직을 어떻게 짤 것인가"만큼 별도로 설계해야 하는 부분이라는 것을 확인했다.

### 같은 실수는 코드가 바뀌어도 재발할 수 있다
1차 시도에서 원인까지 정확히 진단하고 고쳤던 실패 판정 조건이, 구조를 완전히 새로 짠 2차 시도(딕셔너리 전처리)에서도 동일하게 재발했다. 코드의 형태가 바뀌면 이전에 고친 부분도 다시 점검 대상이 된다는 것을 보여준 사례.

### 반복 계산 제거는 로직 검증 이후에
keymap을 매번 순회하는 대신 딕셔너리로 미리 계산해두는 최적화는, 매번 새로 계산하던 1차 코드의 로직이 옳다는 것이 확인된 뒤에 적용했다. 최적화와 정답 검증을 동시에 시도하지 않고 단계를 나눈 것이 문제 해결을 단순하게 만들었다.
