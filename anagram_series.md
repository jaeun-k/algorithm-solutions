# 애너그램(Anagram) 3문제 정리

문자별 개수를 비교하는 기본 판별부터, 슬라이딩 윈도우 응용, 그리고 24시간에 걸친 시행착오 끝에 도달한 "최소 삭제 횟수" 문제까지 세 단계로 정리한다.

---

## 문제 1: 애너그램 판별 (Anagram Check)

**예상 난이도: 브론즈 4**

두 문자열이 애너그램(같은 문자를 재배열해서 만들 수 있는 관계)인지 판별한다.

```python
is_anagram("listen", "silent")   # True
is_anagram("hello", "world")     # False
```

### 시행착오: "있냐 없냐"로는 안 된다는 것을 깨달음
처음엔 "하나의 딕셔너리 + 존재 여부(있다/없다)"로 접근하려 했으나 감이 오지 않았다. 원인은 이 문제가 **존재 여부가 아니라 "존재하는데 정확히 몇 개인가"를 물어보는 문제**였기 때문이다. `list_dict_review_four_problems.md`에서 이미 정리한 원칙("있다/없다만 확인하면 set, 개수 등 추가 정보가 필요하면 dict")이 정확히 이 지점에서 다시 확인됐다 — set/존재 확인 방식의 틀로는 이 문제를 풀 수 없었다.

### 정답 코드 (딕셔너리 버전)
```python
def is_anagram(s1, s2):
    if len(s1) != len(s2):
        return False

    s1_dict = {}
    s2_dict = {}
    for i in s1:
        if i not in s1_dict:
            s1_dict[i] = 1
        else:
            s1_dict[i] = s1_dict[i] + 1
    for i in s2:
        if i not in s2_dict:
            s2_dict[i] = 1
        else:
            s2_dict[i] = s2_dict[i] + 1

    return s1_dict == s2_dict
```
길이가 다르면 조기 종료, 두 딕셔너리(문자별 개수)를 만들어 `==`로 통째로 비교. 딕셔너리끼리의 `==` 비교는 key와 value가 전부 같은지를 한 번에 확인해준다.

### 리스트만으로 푸는 버전 (dict 없이)
dict가 필수 도구는 아니라는 것을 확인하기 위해, `character_pair_count_notes.md`의 "명단 기록" 패턴을 재활용해 dict 없이도 풀어봤다.
```python
def is_anagram(s1, s2):
    if len(s1) != len(s2):
        return False

    checked = []
    for char in s1:
        if char not in checked:
            checked.append(char)
            if s1.count(char) != s2.count(char):
                return False

    return True
```
정답이지만 시간복잡도는 dict 버전(O(n))보다 불리하다(O(n×d), d=서로 다른 문자 종류 수) — `character_pair_count_notes.md`에서 이미 확인한 트레이드오프와 동일.

---

## 문제 2: 부분 문자열 애너그램 존재 확인 (Contains Anagram)

**예상 난이도: 실버 4~3**

문자열 `s` 안에 `p`의 애너그램인 부분 문자열이 존재하는지 확인한다.

```python
contains_anagram("cbaebabacd", "abc")  # True ("cba"가 "abc"의 애너그램)
contains_anagram("af", "be")           # False
```

### 정답 코드
```python
def is_anagram(s1, s2):
    if len(s1) != len(s2):
        return False
    checked = []
    for char in s1:
        if char not in checked:
            checked.append(char)
            if s1.count(char) != s2.count(char):
                return False
    return True

def contains_anagram(s, p):
    if len(s) < len(p):
        return False
    for i in range(len(s) - len(p) + 1):
        if is_anagram(s[i:i+len(p)], p):
            return True
    return False
```
`p`와 길이가 같은 부분 문자열을 슬라이딩하며 `is_anagram`을 그대로 재사용했다 (`rotate_by_reverse`에서 이미 연습한 "함수 안에서 다른 함수 호출" 패턴).

---

## 문제 3: 애너그램으로 만들기 위한 최소 삭제 횟수

두 문자열을 애너그램 관계로 만들기 위해 삭제해야 하는 최소 문자 개수를 구한다.

```python
min_deletions_to_anagram("bcadeh", "hea")     # 3
min_deletions_to_anagram("cde", "abc")        # 4
min_deletions_to_anagram("listen", "silent")  # 0
```

이 문제는 24시간에 걸친 시행착오 끝에 완전히 스스로의 힘으로 완성한 문제다. 아래는 그 과정 중 핵심적인 단계만 추린 것이다.

### 시행착오 1: "존재 여부"만으로 접근 — 실패
```python
for i in s1:
    if i not in s2:
        count_s1 = count_s1 + 1
# (s2 쪽도 동일하게)
```
서로에게 없는 문자의 개수를 센 다음, 남은 길이 차이(`abs`)까지 더하는 방식으로 접근했으나, `min_deletions_to_anagram("aab", "abb")`에서 `0`이라는 오답이 나왔다. 원인은 `not in`이 **존재 여부만 확인하고 개수 차이는 전혀 반영하지 못하기 때문**이었다 — `"aab"`의 `a`가 몇 개든, `s2`에 `a`라는 글자가 하나라도 있으면 그냥 통과해버린다.

### 시행착오 2: 딕셔너리로 서로 없는 key 삭제 — 구조적으로 막힘
```python
for i in s1_dict:
    if i not in s2_dict:
        del s1_dict[i]
```
딕셔너리를 순회하면서 그 안에서 동시에 key를 지우려 하자 `RuntimeError: dictionary changed size during iteration`가 발생했다. "순회 중인 대상을 순회 도중에 수정할 수 없다"는 원칙을 이 과정에서 확인했다. 이 우회로(딕셔너리 값의 합으로 길이 구하기 등)까지 설계했으나 "for문이 6번이나 필요하고 너무 복잡하다"고 판단해 전략 자체를 폐기했다.

### 시행착오 3: "명단 기록 후 개수 비교"까지는 도달, 그러나 적용 지점이 안 맞음
```python
final_list = []
save = 0
for i in s1:
    if i not in final_list:
        final_list.append(i)
        if s1.count(i) != s2.count(i):
            save = save + abs(s1.count(i) - s2.count(i))
```
"공통 문자의 개수 차이를 센다"는 핵심 아이디어에 근접했으나, 이 방식은 지금까지 써온 "원본은 건드리지 않고, 서로가 가지지 않은 문자들의 개수를 센 뒤 길이 차이를 구하는" 접근에는 적용할 수 없다는 걸 깨달았다. 그래서 방향을 틀어, 원본을 직접 건드리는(줄여나가는) 방식으로 다시 접근하기로 했다.

### 노선 전환: 원본을 직접 "줄인" 상태로 만들기
"원본을 직접 건드린다"는 걸, 실제로 원본 문자열 자체를 수정하는 게 아니라 **"서로 공통으로 가진 문자만 남긴 새로운 리스트"를 만드는 것**으로 구현했다. `s1`을 훑으며 그 문자가 `s2`에도 있으면 `s1_common`에 담고, `s2`도 같은 방식으로 `s2_common`에 담는다. 이렇게 만들어진 `s1_common`, `s2_common`은 "상대에게 아예 없는 문자가 전부 제거된, 축소된 원본"에 해당한다.

- `len(s1) - len(s1_common)`: 원본에서 이 축소 과정을 통해 사라진 개수 = **상대에게 아예 없어서 지워야 하는 문자 수**
- 이렇게 축소된 `s1_common`, `s2_common`을 놓고, 그 안에서 다시 문자별 개수를 비교하면 = **양쪽에 다 있지만 개수가 안 맞아서 추가로 지워야 하는 문자 수**

이 두 종류의 "지워야 할 개수"를 각각 구해서 더하면 최종 답이 된다는 게 이 노선 전환의 핵심 아이디어였다.

### 최종 정답: `s1_common`/`s2_common`을 이용한 이중 계산
```python
def min_deletions_to_anagram(s1, s2):
    s1_common = []
    s2_common = []

    for i in s1:
        if i in s2:
            s1_common.append(i)
    for i in s2:
        if i in s1:
            s2_common.append(i)

    s1_remove_number = len(s1) - len(s1_common)
    s2_remove_number = len(s2) - len(s2_common)

    final_list = []
    save = 0
    for i in s1_common:
        if i not in final_list:
            final_list.append(i)
            if s1_common.count(i) != s2_common.count(i):
                save = save + abs(s1_common.count(i) - s2_common.count(i))

    return s1_remove_number + s2_remove_number + save
```
`save`는 원본이 아니라 `s1_common`, `s2_common`(공통 문자만 남긴 리스트) 안에서 개수를 비교한다. `s1_remove_number`/`s2_remove_number`가 이미 "상대에게 아예 없는 문자"를 처리했으므로, `save`는 그 문자들과 겹치지 않고 순수하게 "양쪽에 다 있지만 개수가 다른 문자"만 추가로 잡아낸다.

### 검증
```
"bcadeh" vs "hea"      -> 3   (정답)
"cde" vs "abc"         -> 4   (정답)
"aab" vs "abb"         -> 2   (정답)
"listen" vs "silent"   -> 0   (정답)
```
추가로 2,000개의 랜덤 케이스를 표준 해법(Counter 기반)과 대조해 전부 일치함을 확인했다.

---

## 이 문제(문제 3)에서 얻은 교훈

### 24시간의 시행착오가 헛되지 않았던 이유
막힌 매 순간마다 "이 접근이 왜 안 되는지"를 정확히 진단하고 다음 시도로 넘어갔다: `in`은 존재 여부만 본다 → 딕셔너리 삭제는 순회 중 수정이 안 된다 → "세는 것"과 "직접 줄이는 것"은 다른 접근이다 → 마지막엔 "공통 원소만 남긴 리스트"라는, 원본도 아니고 단순 카운트도 아닌 제3의 중간 표현을 스스로 고안해내며 노선을 전환했다.

### "직접 시행착오로 도달한 지저분한 코드"와 "우아하지만 낯선 정석 코드"의 차이
이 문제를 통해 도달한 코드는 짧지도, 우아하지도 않다. 그러나 이 코드의 매 줄(`s1_common`을 왜 만들어야 하는지, `save`가 왜 원본이 아니라 `_common` 기준이어야 하는지)이 전부 실제로 겪은 실패와 그 원인 진단에서 나온 결과다. 코드의 길이나 간결함과, 그 코드에 대한 이해도는 서로 다른 축이라는 것을 확인한 사례.
