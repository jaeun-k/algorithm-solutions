# 문자열/단어 처리 (String & Word) — 3문제 정리

`.split()`을 실전에서 처음 적용해본 문제들을 난이도를 올려가며 정리한다. 가장 긴 단어 찾기 → 각 단어 첫 글자 대문자화 → 단어 순서 뒤집기 순서다.

---

## 사전 정리: 새로 배운 문자열 메서드

```python
"the quick brown fox".split()   # → ['the', 'quick', 'brown', 'fox']
```
인자 없이 쓰면 공백(스페이스, 탭, 줄바꿈 등) 기준으로 문자열을 잘라 **리스트**로 반환한다. 공백이 여러 개 연속이거나 앞뒤에 있어도 알아서 깔끔하게 처리된다 (`simple_text_game.md`에서 `input().split()`으로 이미 한 번 사용).

```python
"Hello".upper()   # → "HELLO"
"Hello".lower()   # → "hello"
```
대소문자를 통째로 바꾼다. 숫자/기호처럼 대소문자 구분이 없는 문자는 에러 없이 그대로 둔다.

```python
"  hello world  ".strip()   # → "hello world"
```
문자열 양 끝의 공백만 제거한다 (가운데 공백은 안 건드림).

---

## 문제 1: 가장 긴 단어 찾기 (Longest Word)


```python
longest_word("the quick brown fox jumps")  # "quick" (동률이면 먼저 나온 단어)
```

### 시행착오: 길이만 반환하던 초기 버전
```python
def longest_word(sentence):
    s = sentence.split()
    max_sentence = 0
    for i in s:
        if max_sentence < len(i):
            max_sentence = len(i)
    return max_sentence
```
로직은 맞았으나, `max_sentence`에 **길이(숫자)**만 저장하고 있어서 실제 단어 자체를 잃어버렸다. `find_most_frequent`에서 `best_count`와 `best_value`를 같이 추적했던 것처럼, "가장 긴 길이"와 "그 길이를 가진 단어" 둘 다 추적해야 한다는 것을 확인.

### 정답 코드 (최적화 완료)
```python
def longest_word(sentence):
    words = sentence.split()
    max_sentence = ""

    for word in words:
        if len(word) > len(max_sentence):
            max_sentence = word

    return max_sentence
```
별도의 길이 변수(`max_length`) 없이, `len(max_sentence)`를 그때그때 계산해서 비교했다 — 다이아몬드에서 확인한 "저장 vs 계산" 트레이드오프의 축소판. `<`(초과일 때만 갱신)를 써서, 동률인 단어를 만나도 먼저 발견한 단어가 자동으로 유지된다.

### 부수적으로 확인한 것: 초기값이 뭐든 상관없는 이유
`max_sentence = ""` 대신 `" "`이나 임의의 문자로 시작해도 결과가 똑같다. 판단 기준이 `max_sentence` 자체가 아니라 그 **길이**이기 때문에, 첫 단어를 만나는 즉시 `len(word) > len(초기값)`이 참이 되어 바로 덮어써진다. `max_subarray_sum_notes.md`의 "초기값은 로직이 검증해줄 임시값일 뿐"이라는 원칙과 동일한 사례. 다만 가독성을 위해 관례적으로 `""`을 쓴다 (예외: 빈 문자열 자체가 유효한 데이터로 등장할 수 있는 경우엔 `None`으로 "존재 여부"까지 구분해야 함 — `find_most_frequent`의 `best_value = None`이 그 경우).

---

## 문제 2: 각 단어 첫 글자만 대문자로 (Title Case, 수동 구현)

```python
title_case("the quick brown FOX")  # "The Quick Brown Fox"
```

### 정답 코드
```python
def title_case(sentence):
    words = sentence.split()
    result = ""

    for word in words:
        result = result + word[0].upper() + word[1:].lower() + " "

    return result.strip()
```
각 단어를 `word[0]`(첫 글자)과 `word[1:]`(나머지)로 나눠 각각 `.upper()`/`.lower()`를 적용하고 이어붙인다. 매 단어 뒤에 공백을 붙이다 보니 마지막에 공백이 하나 남는데, 이걸 `.strip()`으로 정리했다.

### 다듬은 지점: 슬라이싱 vs 인덱싱
초기 버전은 `word[:1]`(슬라이싱)으로 첫 글자를 뽑았는데, 결과는 `word[0]`(인덱싱)과 동일하다. 슬라이싱은 원래 "여러 글자로 이루어진 구간"을 위한 도구이므로, 정확히 한 글자만 필요할 때는 인덱싱이 의도를 더 정확히 드러낸다 — `word[1:]`(나머지 전체, 구간이 맞음)은 그대로 슬라이싱 유지.

### 부수적으로 정리된 개념: 함수 반환값을 "한 번만 쓸 때"의 저장 방식
```python
result = result.strip()   # 방법 1: 재할당 (기존 변수에 다시 저장)
return result

return result.strip()     # 방법 2: 저장 없이 바로 반환
```
`.strip()`도 원본을 바꾸지 않고 새 문자열을 반환하므로, 그 결과를 어딘가에 담아야 반영된다. 다만 결과를 그 자리에서 한 번만 쓰고 끝난다면(다시 안 쓴다면), 별도 변수에 저장하지 않고 바로 다음 연산에 넘겨도 된다. 두 방식 다 정답이며, 방법 1은 "지금 이 변수에 최종적으로 뭐가 들어있는지"가 각 단계마다 명시적으로 보여 가독성과 디버깅(중간에 `print`로 확인하기 쉬움) 면에서 유리하다.

---

## 문제 3: 단어 순서 뒤집기 (Reverse Word Order)

```python
reverse_word("the sky is blue")  # "blue is sky the"
```
슬라이싱 뒤집기(`[::-1]`)나 `reversed()` 없이, 투 포인터로 직접 순서를 뒤집는다.

### 정답 코드
```python
def reverse_word(sentence):
    s = sentence.split()

    left = 0
    right = len(s) - 1
    while left < right:
        s[left], s[right] = s[right], s[left]
        left = left + 1
        right = right - 1

    result = ""
    for word in s:
        result = result + word + " "

    return result.strip()
```
`palindrome`/`rotate_by_reverse`에서 문자를 스왑하던 투 포인터 로직을, 이번엔 리스트의 원소(문자가 아니라 단어)에 그대로 적용했다. 단어가 하나뿐인 경우 `left < right`가 처음부터 거짓이라 뒤집기 자체가 일어나지 않고 그대로 반환되는 것도 자연스럽게 처리된다.

### 변수명 다듬기
초기 버전에서 결과를 담는 변수 이름을 `char`로 뒀는데, `char`는 "문자 하나"를 뜻하는 이름이라 "여러 단어가 합쳐진 문장 전체"를 담기엔 맞지 않았다. `result`로 수정. 순회 변수도 `i` 대신 `word`로 바꿔 "지금 단어 하나씩을 다루고 있다"는 걸 명확히 했다.

### 부수적으로 정리된 개념: 단수/복수 변수명 규칙
| 이름 | 의미 |
|---|---|
| `char`, `letter` | 문자 하나 (단수) |
| `chars`, `letters` | 문자 여러 개 (복수) |

순회 대상이 "문자"가 아니라 "단어"인 경우엔 애초에 `char`/`letter` 계열 이름 자체가 안 맞으므로, `word`처럼 실제로 다루는 단위에 맞는 이름을 골라야 한다는 것을 재확인.

---

## 사용된 핵심 도구

- `.split()` — 문장을 단어 리스트로 분리
- `.upper()`, `.lower()` — 대소문자 변환
- `.strip()` — 양 끝 공백 제거
- 문자열 인덱싱(`word[0]`)과 슬라이싱(`word[1:]`)의 구분 사용
- 투 포인터 스왑을, 문자 단위가 아니라 리스트 원소(단어) 단위로 적용

## 이 시리즈에서 얻은 교훈: 문제 난이도는 "직전 맥락"에 따라 달라진다
문제 3(`reverse_word`)이 예상보다 쉽게 풀린 이유는, 직전 문제(`title_case`)에서 "문장을 쪼개고 다시 합친다"는 뼈대(`.split()` → 순회 → 이어붙이기 → `.strip()`)를 이미 손에 익힌 상태였기 때문이다. 문제 3은 그 뼈대에 "투 포인터로 순서 뒤집기"라는 조각 하나만 추가하면 되는 상황이었다. 만약 이 문제만 단독으로 마주쳤다면, "단어 단위로 다뤄야 한다"는 것과 "문자 스왑 로직을 단어 스왑에 그대로 전이할 수 있다"는 것 둘 다 스스로 떠올려야 했을 것이다. 즉 체감 난이도는 문제 자체의 복잡도뿐 아니라 직전에 어떤 재료가 준비되어 있었는가에 크게 좌우되며, 실전에서는 이런 연결고리 없이 문제가 단독으로 나올 수 있으므로 "이 로직, 예전에 다른 문제에서 본 패턴 아닌가?"를 스스로 먼저 점검하는 습관이 유효하다.
