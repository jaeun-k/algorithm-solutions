# 딕셔너리 매핑과 큐 시뮬레이션 2문제 정리

신고 집계 문제에서 언패킹과 딕셔너리 압축을, 프로세스 우선순위 문제에서 "순회 중인 리스트를 직접 수정하면 안 되는 이유"와 큐 시뮬레이션의 정석 도구(deque)를 정리한다.

---

## 문제 1: 신고 결과 받기

각 유저가 신고한 내용을 취합해, k번 이상 신고당한 유저를 신고한 사람들에게 각각 몇 번의 결과 메일이 가는지 구한다.

```python
solution(["muzi","frodo","apeach","neo"],
          ["muzi frodo","apeach frodo","frodo neo","muzi neo","apeach muzi"], 2)
# [2,1,1,0]
```

### 최종 코드
```python
def solution(id_list, report, k):
    reported = {}
    report_dict = {}
    for i in set(report):
        reporter, reportee = i.split()
        report_dict[reporter] = report_dict.get(reporter, []) + [reportee]
        reported[reportee] = reported.get(reportee, 0) + 1

    result = [
        sum(1 for j in report_dict.get(i, []) if reported[j] >= k)
        for i in id_list
    ]
    return result
```

### 핵심 통찰

**언패킹(unpacking)으로 인덱스 접근을 없앨 수 있다**
```python
reporter, reportee = i.split()
```
`split()`은 항상 리스트를 반환하지만, 그 결과의 원소 개수가 정확히 맞으면(여기선 항상 2개) `a, b = 리스트`처럼 여러 변수에 한 번에 나눠 담을 수 있다. 이건 `split()`만의 특별한 기능이 아니라, 리스트·튜플 등 원소 개수가 맞는 모든 반복 가능한 대상에 적용되는 일반적인 파이썬 문법이다. 이 발상 전환 하나로, `i[0]`, `i[1]`처럼 인덱스로 꺼내 쓰던 것을 의미 있는 이름의 변수로 바로 받을 수 있게 되고, 중간 저장용 리스트(`report_list`) 자체도 필요 없어져 한 번의 순회로 여러 딕셔너리를 동시에 채울 수 있다.

**`dict.get(key, default)`는 하나의 메커니즘이 두 가지 역할로 쓰인다**
- 카운팅 초기화: `reported.get(reportee, 0) + 1` — 없으면 0에서 시작
- 안전한 조회: `report_dict.get(i, [])` — 없으면 빈 리스트를 줘서, `for` 순회가 자연스럽게 0번 돌고 끝나게 함으로써 `if i in report_dict: ... else: ...` 분기 자체를 없앨 수 있음

두 경우 다 "있으면 실제 값, 없으면 기본값"이라는 동일한 규칙인데, 어디에 쓰이느냐에 따라 "값을 만드는 용도"로도 "에러 방지용 안전장치"로도 보일 수 있다.

**딕셔너리 값으로 여러 개를 담아야 하면 리스트(또는 set)가 필수**
"한 신고자가 여러 명을 신고할 수 있다"는 1:다(多) 관계를 표현하려면, 값 자리에 여러 개를 묶어 담을 수 있는 컨테이너가 있어야 한다. 이 문제는 이미 `set(report)`로 원본에서 중복을 제거해뒀기 때문에, 값에 다시 set을 쓸 실익은 없고 리스트로 충분하다. set을 쓰려면 `+` 대신 `|`(합집합), `[x]` 대신 `{x}`를 써야 한다는 것도 확인함.

---

## 문제 2: 프로세스 (우선순위 큐 시뮬레이션)

대기 큐에서 하나를 꺼내, 더 높은 우선순위가 남아있으면 다시 큐에 넣고 없으면 실행한다. 특정 위치의 프로세스가 몇 번째로 실행되는지 구한다.

```python
solution([2,1,3,2], 2)  # 1
solution([1,1,9,1,1,1], 0)  # 5
```

### 시행착오: `for index, value in enumerate(...)` 안에서 리스트를 직접 수정
```python
for index, value in enumerate(priorities):
    ...
    priorities.pop(index)
    priorities.append(value)
```
`pop`으로 리스트가 앞으로 밀리는 동안, `for`문의 `index`는 그 밀림과 무관하게 그냥 0, 1, 2...로 기계적으로 증가한다. 그 결과 몇 번의 회전 뒤에는 `index`가 가리키는 자리와 큐가 실제로 봐야 하는 "맨 앞"이 완전히 어긋나, 특정 원소를 건너뛰는 오류가 생겼다. 이 문제는 "순서대로 끝까지 훑는" 순회가 아니라 "항상 맨 앞만 반복해서 보는" 큐 구조이므로, 애초에 `for`+`enumerate`라는 도구 자체가 이 문제의 본질과 맞지 않았다.

### 전환: `while`문으로 항상 `priorities[0]`만 확인
인덱스를 추적할 필요 자체가 없다는 걸 깨닫고, 매 반복마다 그냥 맨 앞만 보는 방식으로 전환해 해결했다.
```python
while True:
    if priorities[0] == max(priorities):
        priorities.pop(0)
        count += 1
        if location == 0:
            return count
        location -= 1
    else:
        priorities.append(priorities[0])
        priorities.pop(0)
        if location == 0:
            location = len(priorities) - 1
        else:
            location -= 1
```
이건 정답이고 이 문제(큐 시뮬레이션)의 표준적인 접근이다. 다만 `location`을 매번 손으로 밀어주고, 맨 뒤로 갈 때 재계산해야 하는 번거로움이 있었다.

### 정석 개선: 값에 "원래 위치" 정보를 함께 실어서 이동시키기
`location`을 계속 손으로 갱신하는 대신, 큐에 넣는 원소 자체를 `(값, 원래 인덱스)` 쌍으로 만들면, 그 정체성이 원소를 따라 계속 이동하므로 위치를 재계산할 필요가 사라진다.
```python
from collections import deque

def solution(priorities, location):
    count = 0
    que = deque((index, value) for index, value in enumerate(priorities))
    while True:
        current = que.popleft()
        if any(current[1] < q[1] for q in que):
            que.append(current)
        else:
            count += 1
            if current[0] == location:
                return count
```

### 새로 정리한 도구들

**`deque`**: `collections`의 양방향 큐. 리스트는 맨 앞에서의 추가/제거(`insert(0,x)`, `pop(0)`)가 O(n)인데, `deque`는 `appendleft()`/`popleft()`가 O(1)이라 "맨 앞을 계속 다뤄야 하는" 큐 시뮬레이션이나 BFS에서 표준적으로 쓰인다. 아무 문제에나 자주 등장하는 도구는 아니고, "큐 구조다" 혹은 "BFS다"라는 신호가 보일 때 떠올려야 하는 특화 도구다.

**`any()` / `all()`**: `any(조건)`은 하나라도 만족하면 True, `all(조건)`은 전부 만족해야 True. 둘 다 반복문+조건 확인을 한 줄로 압축해주는 범용 도구로, `any()`/`all()`은 특정 유형에 국한되지 않고 폭넓게 쓰인다. 빈 이터러블에서 `any([])`는 False, `all([])`는 True라는 차이가 있다.

**튜플 순서를 의도적으로 설계하기**: `(index, value)` 대신 `(value, index)` 순서로 튜플을 만들면, 파이썬 튜플 비교/`max()`가 첫 번째 원소부터 비교하는 특성을 이용해 값 기준 비교를 더 간결하게 쓸 수 있다.

---

## 이 두 문제에서 얻은 교훈

### 순회 중인 자료구조를 직접 수정하면, 늘어나는 경우와 줄어드는 경우가 다르게 깨진다
- 원소가 계속 늘어나면(append 등): `for`문이 매 순간의 길이를 다시 확인하므로 종료 조건이 계속 밀려 **무한루프**에 빠질 수 있다.
- 원소가 줄어들면(pop 등): 뒤 원소들이 앞으로 당겨지는데 `for`의 인덱스는 그와 무관하게 증가하므로, **일부 원소를 건너뛰는** 버그가 생긴다.

### 어떤 반복 구조가 필요한지 먼저 파악한 뒤 도구를 골라야 한다
"처음부터 끝까지 순서대로, 각자 원래 자리를 유지하며 훑어야 하는" 순회에는 `for`+`enumerate`가 맞지만, "항상 맨 앞만 반복해서 확인하고 회전시키는" 큐 구조에는 애초에 인덱스 추적이라는 개념 자체가 불필요하다. 문제의 반복 구조(순차 순회 vs 큐 회전)를 먼저 식별하는 것이 도구 선택보다 우선한다.

### 위치를 계속 계산해서 추적하는 대신, 정체성 정보를 데이터에 함께 실어 보내는 것이 더 안정적이다
"이 값이 지금 몇 번째 자리에 있는가"를 매번 손으로 재계산하는 대신, 애초에 값과 "원래 위치"를 하나의 튜플로 묶어 함께 이동시키면, 자료구조가 아무리 재배치되어도 그 정체성 확인은 항상 정확하다. 이는 순서 추적이 필요한 스택/큐 시뮬레이션 전반에 재사용 가능한 패턴이다.
