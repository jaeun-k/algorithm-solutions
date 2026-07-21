# 회문(Palindrome) 3문제 정리

문자열을 "양 끝에서 다가가며 비교"하는 새로운 패턴을 익히고, 이를 확장해 더 어려운 문제까지 단계적으로 풀어간 과정을 정리한다. 슬라이싱 개념을 처음 제대로 배운 계기이기도 하다.

---

## 문제 1: 회문 판별 (Palindrome Check)
**예상 난이도: 브론즈 5~4**

문자열 `s`가 주어졌을 때, 회문(앞으로 읽으나 뒤로 읽으나 똑같은 문자열)인지 판별하는 함수 `is_palindrome(s)`를 작성한다.

```python
is_palindrome("level")   # True
is_palindrome("hello")   # False
```

### 정답 코드
```python
def is_palindrome(s):
    for value in range(len(s) // 2):
        if s[value] != s[len(s) - value - 1]:
            return False
    return True
```

### 사용된 핵심 도구
- 인덱싱 (`s[value]`, `s[len(s)-value-1]`)
- `len(s) // 2` — 절반까지만 비교하면 충분하다는 것 (양 끝에서 만나므로 중간을 넘어설 필요 없음)

### 접근 방식
"명단을 기록하고 대조"하는 기존 패턴과 완전히 다른, **양 끝에서 서로를 향해 다가가며 비교**하는 새로운 패턴. 별도의 실패 시도 없이 바로 정답에 도달했다.

---

## 문제 2: 인덱스 하나 빼고 만들기 (Remove at Index)
**예상 난이도: 브론즈 5**

문자열 `s`와 정수 `i`가 주어졌을 때, 인덱스 `i`번째 글자만 뺀 새 문자열을 반환하는 함수 `remove_at(s, i)`를 작성한다.

```python
remove_at("abcde", 2)  # "abde"
```

### 초기 시도와 방향 전환
처음엔 "빈 리스트를 만들고, 두 개의 `for`문으로 각각 앞부분/뒷부분을 `append`해서 채운 뒤 다시 문자열로 합친다"는 접근을 시도했다. 논리적으로는 작동하지만, **결과가 리스트로 나오는데 문제가 요구하는 건 문자열**이라는 불일치가 있었다. 이 과정에서 "슬라이싱은 문자열에 적용하면 결과도 문자열로 나온다"는 것을 확인하고, 리스트 조립 없이 슬라이싱 두 조각을 바로 이어붙이는 방향으로 전환했다.

### 정답 코드
```python
def remove_at(chars, number):
    left_chars = chars[0:number]
    right_chars = chars[number + 1:]
    full_chars = left_chars + right_chars
    return full_chars
```

### 사용된 핵심 도구
- 슬라이싱 (`chars[0:number]`, `chars[number+1:]`) — 범위를 잘라 부분 문자열을 만듦
- 문자열 `+` 이어붙이기

### 인덱싱과 슬라이싱의 차이 (이 문제를 계기로 정리)
| | 인덱싱 (`s[i]`) | 슬라이싱 (`s[i:j]`) |
|---|---|---|
| 반환하는 것 | 값 하나 | 부분 문자열/리스트 |
| 끝 인덱스 | 정확히 그 위치 | 그 위치 **앞까지** (포함 안 됨) |
| 범위 벗어나면 | 에러 발생 | 에러 없이 안전하게 처리 |
| 세 번째 옵션 | 없음 | `[시작:끝:간격]` — 간격을 `-1`로 주면 뒤집힘 |

---

## 문제 3: 하나만 지워도 회문 (Almost Palindrome)
**예상 난이도: 실버 4~3**

문자열 `s`가 주어졌을 때, 문자를 최대 한 개까지 지웠을 때 회문이 될 수 있는지 판별하는 함수 `can_be_palindrome(s)`를 작성한다.

```python
can_be_palindrome("abca")  # True ('b' 또는 'c' 하나만 지우면 회문)
can_be_palindrome("abc")   # False (뭘 지워도 회문 안 됨)
```

### 워밍업: 두 함수 조합해보기
본 문제로 가기 전, `remove_at`과 `is_palindrome`을 조합해 "특정 인덱스를 뺐을 때 회문인지"를 확인하는 연습을 거쳤다. 두 함수를 그대로 갖다 쓰는 대신, 로직 자체를 하나의 함수 안에서 처음부터 다시 짜보는 방식으로 진행했다 (기존 함수 재사용 여부는 "코드 품질" 관점과 "이 문제를 다시 풀어보는 연습" 관점이 서로 다른 목적을 가지므로, 후자를 택한 것도 유효한 선택이었음을 확인함).

### 시도 1: 불일치 개수만 세는 방식 — 실패
```python
def can_be_palindrome(s):
    diff_char = 0
    for value in range(len(s) // 2):
        if s[value] != s[len(s) - value - 1]:
            diff_char = diff_char + 1
        if diff_char == 2:
            return False
    return True
```
**실패 이유:** "불일치가 1번만 있었다"는 사실이 "그 1번을 지우면 회문이 된다"는 것을 보장하지 않는다. 한 글자를 실제로 지우면 문자열 길이가 줄어들며 비교해야 할 짝 자체가 완전히 달라지는데, 이 버전은 그 재검증을 전혀 하지 않고 단순히 불일치 횟수만 세고 있었다. 반례: `"abc"`, `"abcda"` 둘 다 지워도 회문이 안 되는데 `True`를 반환함.

### 시도 2: 불일치 지점에서 실제로 제거해보고 확인 — 첫 번째 버그 (조기 종료 실패)
```python
def can_be_palindrome(s):
    count = 0
    for value in range(len(s)//2):
        if s[value] != s[len(s) - value - 1]:
            if remove_at(s,value) == True or remove_at(s,len(s) - value - 1) == True:
                count = count + 1
            else:
                return False
        if count == 2:
            return False
    return True
```
**실패 이유:** 불일치 지점에서 제거가 성공해도 즉시 반환하지 않고 `count`만 늘린 채 반복을 계속 진행했다. 그러다 원본 문자열의 다른 지점에서 또 불일치를 만나면, 이미 답을 찾았음에도 그 지점을 별도로 재검증하려다 잘못된 `False`를 반환하는 문제가 생겼다. 반례: `"baba"` (인덱스 0을 지우면 `"aba"`로 회문이 되는데도 `False` 반환).

### 최종 정답 (슬라이싱 + 기존 함수 재사용 없이 자체 구현)
```python
def remove_at(chars, number):
    left_chars = chars[0:number]
    right_chars = chars[number + 1:]
    full_chars = left_chars + right_chars
    for value in range(len(full_chars) // 2):
        if full_chars[value] != full_chars[len(full_chars) - value - 1]:
            return False
    return True

def can_be_palindrome(s):
    for value in range(len(s)//2):
        if s[value] != s[len(s) - value - 1]:
            if remove_at(s, value) == True or remove_at(s, len(s) - value - 1) == True:
                return True
            else:
                return False
    return True
```
불일치를 찾는 즉시 `return True`/`return False`로 끝내도록 정리해 시도 2의 조기 종료 문제를 해결했다. 쓰이지 않던 `count` 변수도 제거했다.

### 대안 버전: 포인터(범위) 방식 — 문자열을 새로 만들지 않는 방법
슬라이싱으로 매번 새 문자열을 만드는 대신, "확인할 범위(시작/끝 인덱스)"만 조정하는 방식으로도 재구현했다.
```python
def is_palindrome_range(s, left, right):
    while left < right:
        if s[left] != s[right]:
            return False
        left = left + 1
        right = right - 1
    return True

def can_be_palindrome(s):
    for n in range(len(s) // 2):
        if s[n] != s[len(s) - 1 - n]:
            return is_palindrome_range(s, n+1, len(s)-1-n) or is_palindrome_range(s, n, len(s)-n-2)
    return True
```
`remove_at`처럼 문자열을 실제로 잘라 새로 만들지 않아 메모리를 아낄 수 있고, `if/else`로 True/False를 감싸는 대신 `or` 표현식의 결과를 그대로 `return`해 코드가 더 간결하다.

### 사용된 핵심 도구
- 슬라이싱, 인덱싱 (문제 1, 2와 동일)
- `while` 반복문과 두 개의 포인터(`left`, `right`)를 직접 증감시키는 방식
- `or` 연산자의 단락 평가(short-circuit) — 왼쪽이 참이면 오른쪽은 계산도 안 하고 즉시 그 값을 반환. 왼쪽이 거짓이어야만 오른쪽을 계산해 그 값을 그대로 반환
- `if`와 `return`의 관계: `if`는 "조건이 참일 때 아래 블록에 진입할지"만 결정하는 문지기 역할이고, `return`은 그 블록 안에 놓인 완전히 독립적인 명령문. `if`가 반환값을 만들어주는 게 아니라, 이미 계산된 값(`A or B`)을 담은 `return`문이 그 블록에 진입했을 때 실행되는 것뿐임

---

## 세 문제의 공통 패턴: "양 끝에서 다가가며 비교하기"

오늘까지 다룬 패턴이 하나 더 늘었다.

| 패턴 | 예시 문제 |
|---|---|
| 러닝 값 + 매 스텝 갱신 | 최장 증가 구간, 최대 부분합 |
| 명단 기록 후 대조 | 문자 빈도 짝 찾기, 공통 문자 개수, 두 수의 합 |
| **양 끝에서 다가가며 비교** | **회문 판별, 하나만 지워도 회문** |

이 패턴의 핵심은, `value`(혹은 `offset`)라는 **하나의 변수만 0부터 늘려가면서, 왼쪽 인덱스(`value`)와 오른쪽 인덱스(`len-1-value`)를 동시에 계산**하는 것이다. `left`, `right` 두 변수를 각자 따로 움직이는 `while` 버전도, 결국 이 계산을 명시적인 두 변수로 나눠서 표현한 것일 뿐 본질은 같다.

---

## 학습 과정에 대한 메모: 스캐폴딩(단계적 접근)의 효율

`can_be_palindrome`을 처음부터 붙잡는 대신, 아래 순서로 쪼개서 접근했다.

```
회문 판별 (인덱싱만 필요)
    ↓
인덱스 하나 빼고 문자열 만들기 (슬라이싱 필요)
    ↓
(워밍업) 두 함수 조합해보기
    ↓
하나만 지워도 회문 (앞의 모든 것을 조합 + 새로운 판단 로직)
```

이렇게 쪼갠 결과:
- 막힘의 원인이 매번 정확히 좁혀졌다 (인덱싱 문제인지, 슬라이싱 문제인지, 조합 발상 문제인지 구분 가능).
- 작은 성공이 누적되어 다음 단계로 넘어갈 자신감과 재료를 동시에 제공했다.
- 실수가 나더라도 작은 단위 안에서 발생해 원인을 즉시 짚어낼 수 있었다 (`- value` 누락, 조기 종료 실패, `return` 누락 등 모두 작은 범위에서 빠르게 진단됨).

이는 "처음부터 고난도 문제에 바로 도전하는 것"보다 학습 효율이 높은 접근으로 확인되었다.
