# 행렬 기초 — 리스트 안의 리스트 다루기

처음으로 2차원 리스트(행렬)를 다뤄본 문제들을 정리한다. 대각선 합 → 전체 합/행 합/열 합 → 전치 행렬 순서로 난이도를 올렸다.

---

## 파이썬에서 행렬을 다루는 법 (사전 정리)

파이썬에는 행렬 전용 자료형이 없다. **리스트 안에 리스트를 넣는 방식**으로 표현한다.

```python
matrix = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
]

matrix[0]       # [1, 2, 3]  — 0번째 행 전체 (리스트)
matrix[0][0]    # 1          — 0번째 행에서 0번째 값
matrix[1][2]    # 6          — 1번째 행에서 2번째 값

len(matrix)     # 3 — 행의 개수
len(matrix[0])  # 3 — 열의 개수 (행 하나의 길이)
```

`matrix[i][j]`는 두 단계다: 먼저 `matrix[i]`로 `i`번째 행(리스트)을 통째로 꺼내고, 그 결과에 `[j]`를 한 번 더 붙여서 그 행 안의 `j`번째 값을 꺼낸다.

행렬 전체를 순회하려면 반복문을 두 겹으로 쓴다 (바깥은 "몇 번째 행", 안쪽은 "그 행에서 몇 번째 값"). 또는 `for row in matrix:`로 행 단위로 통째로 꺼내올 수도 있다 — 어느 쪽을 쓸지는 문제가 "행 번호와 열 번호를 각각 따로 다뤄야 하는가"에 달려있다.

행렬을 함수 인자로 넘길 땐, 리스트 여러 개를 그냥 나열하는 게 아니라 **바깥 리스트로 한 번 더 감싸서** 통째로 넘겨야 한다:
```python
diagonal_sum([[1, 2, 3], [4, 5, 6], [7, 8, 9]])   # 올바름
diagonal_sum([1, 2, 3], [4, 5, 6], [7, 8, 9])     # 틀림 — 인자 3개로 인식됨
```

---

## 문제 1: 정사각 행렬의 대각선 합

**예상 난이도: 브론즈 4**

`n×n` 정사각 행렬이 주어졌을 때, 왼쪽 위에서 오른쪽 아래로 이어지는 주대각선 원소들의 합을 구한다.

```python
diagonal_sum([[1,2,3],[4,5,6],[7,8,9]])  # 1+5+9 = 15
```

### 초기 시도 (이중 for문 + 조건으로 골라내기)
```python
def diagonal_sum(matrix):
    total = 0
    for i in range(len(matrix)):
        for j in range(len(matrix)):
            if i == j:
                total = total + matrix[i][j]
    return total
```
정답이지만 O(n²) — 행렬 전체(`n×n`칸)를 다 훑으면서 그중 대각선(`i==j`)인 것만 골라낸다.

### 최적화: 대각선 위치로 바로 접근
```python
def diagonal_sum(matrix):
    total = 0
    for i in range(len(matrix)):
        total = total + matrix[i][i]
    return total
```
필요한 위치(`i == j`)가 무엇인지 이미 알고 있으므로, 굳이 전체를 훑으며 조건으로 걸러낼 필요 없이 `matrix[i][i]`로 바로 접근한다. 이중 for문 → 단일 for문, O(n²) → **O(n)**으로 개선. (merge_sorted에서 슬라이싱으로 조건 분기를 생략했던 것과 같은 결의 개선 — "골라내기"가 아니라 "필요한 곳으로 바로 가기".)

---

## 문제 2: 행렬의 전체 합 / 행의 합 / 열의 합

**예상 난이도: 브론즈 5~4**

```python
matrix_sum([[1, 2, 3], [4, 5, 6]])         # 21 (전체 합)
row_sum([[1, 2, 3], [4, 5, 6], [7,8,9]], 1)  # 15 (1번 행: 4+5+6)
col_sum([[1, 2, 3], [4, 5, 6], [7,8,9]], 1)  # 15 (1번 열: 2+5+8)
```

### matrix_sum (이미 최선)
```python
def matrix_sum(matrix):
    total = 0
    for i in range(len(matrix)):
        for j in range(len(matrix[0])):
            total = total + matrix[i][j]
    return total
```
전체 칸을 다 봐야 하니 O(n×m)이 최소 한도이며, 더 줄일 여지가 없다.

### row_sum — 초기 버전 (인덱스로 우회)
```python
def row_sum(matrix, row_index):
    total = 0
    for i in range(len(matrix[0])):
        total = total + matrix[row_index][i]
    return total
```

### row_sum — 최적화 (이미 완성된 리스트를 그대로 순회)
```python
def row_sum(matrix, row_index):
    total = 0
    for value in matrix[row_index]:
        total = total + value
    return total
```
`matrix[row_index]` 자체가 이미 그 행 전체를 담은 리스트라는 걸 이용해, 인덱스로 하나씩 접근하는 대신 그 리스트를 바로 값으로 순회한다.

### col_sum (이미 필요한 최소 구조)
```python
def col_sum(matrix, col_index):
    total = 0
    for i in range(len(matrix)):
        total = total + matrix[i][col_index]
    return total
```
열은 행과 달리 "미리 만들어진 리스트"가 존재하지 않는다. 각 행에서 `col_index`번째 값을 하나씩 따로 꺼내와야 하므로, 인덱스로 각 행에 접근하는 이 구조가 이미 필요한 최소 형태다. `row_sum`과 겉보기엔 비슷해 보여도, "지름길(이미 존재하는 리스트)이 있는가"의 차이 때문에 하나만 개선 여지가 있었다.

---

## 문제 3: 전치 행렬 (Transpose)

**예상 난이도: 실버 5**

`n×m` 행렬이 주어졌을 때, 행과 열을 뒤바꾼 `m×n` 행렬을 반환한다. `matrix[i][j]`의 값은 결과에서 `result[j][i]` 위치로 간다.

```python
transpose([[1, 2, 3], [4, 5, 6]])
# → [[1, 4], [2, 5], [3, 6]]
```

### 막힌 지점: 빈 리스트에 바로 인덱스로 대입 시도
```python
def transpose_matrix(matrix):
    new_matrix = []
    for i in range(len(matrix[0])):
        for j in range(len(matrix)):
            new_matrix[i][j] = matrix[j][i]   # IndexError
    return new_matrix
```
`new_matrix = []`는 텅 빈 리스트라, `new_matrix[i][j] = ...`처럼 값을 대입하려면 그 자리가 이미 존재해야 하는데(리스트는 존재하지 않는 인덱스에 새로 끼워 넣는 방식으로 동작하지 않음) 그 전제가 없어 에러가 났다. 알고리즘(어느 값이 어디로 가야 하는지) 자체는 맞았고, "목적지 자리가 미리 마련되어 있어야 한다"는 전제를 놓친 것이 원인이었다.

### 해결 방법 1: 미리 정확한 크기의 빈 틀을 만들어둔다
```python
def transpose_matrix(matrix):
    rows = len(matrix)
    cols = len(matrix[0])

    new_matrix = []
    for i in range(cols):
        new_row = []
        for j in range(rows):
            new_row.append(0)
        new_matrix.append(new_row)

    for i in range(rows):
        for j in range(cols):
            new_matrix[j][i] = matrix[i][j]

    return new_matrix
```
일단 크기만 맞는 틀(전부 `0`)을 만들어두고, 그다음 진짜 값으로 하나씩 덮어쓰는 2단계 구조.

### 해결 방법 2 (최종, 정석): `append`만으로 한 줄씩 완성해서 채운다
```python
def transpose_matrix(matrix):
    rows = len(matrix)
    cols = len(matrix[0])

    new_matrix = []
    for i in range(cols):
        new_row = []
        for j in range(rows):
            new_row.append(matrix[j][i])
        new_matrix.append(new_row)

    return new_matrix
```
자리를 미리 확보해두지 않고, 한 행(`new_row`)을 처음부터 끝까지 완전히 채운 다음 그 완성된 행 전체를 `new_matrix`에 `append`한다. 방법 1의 "미리 0으로 채웠다가 나중에 덮어쓰는" 불필요한 이중 작업이 없어, 지금 배운 도구 안에서는 이쪽이 정석이자 최적이다.

(참고: `zip(*matrix)`를 쓰면 반복문 없이 한 줄로 전치 행렬을 만들 수 있으나, `zip`과 `*` 언패킹은 아직 배우지 않은 도구라 이번엔 배제.)

---

## 사용된 핵심 도구

- 2차원 리스트(리스트 안의 리스트) 인덱싱: `matrix[i][j]`
- `len(matrix)` (행 개수) vs `len(matrix[0])` (열 개수)
- 이중 for문으로 행렬 전체 순회, 또는 `for row in matrix:`로 행 단위 순회
- 빈 리스트에 값을 채우는 두 가지 전략: "미리 틀을 만들고 나중에 대입" vs "완성된 조각을 append로 쌓기"

---

## 핵심 논의: "이해"란 무엇인가 (전치 행렬 공식을 계기로)

전치 행렬의 성질(`matrix[i][j] ↔ matrix[j][i]`)을 숫자를 직접 넣어보며 발견한 뒤, "이 공식을 떠올릴 때마다 그 유도 과정 전체가 머릿속에서 다시 빠르게 계산되어야 진짜 이해한 것 아닌가"라는 질문이 나왔다. 이는 학창 시절 "공식을 암기하지 말고 이해해라"는 말의 실체가 무엇인지에 대한 오래된 의문과 연결됐다.

### 결론: "이해"는 "암기의 반대"가 아니라 "더 잘 저장하는 방식"이다
- **순수 암기**: 결론(공식)만 고립된 채로 저장. 다른 어떤 것과도 연결선이 없어, 그 하나의 접근 경로가 끊기면(까먹으면) 완전히 접근 불가능해짐.
- **이해**: 결론과 함께 "왜 그렇게 되는지"에 대한 도출 과정, 이미지, 손으로 확인한 경험까지 같이 저장됨. 여러 경로(도출 과정, 시각적 이미지, 직접 확인한 경험)로 같은 결론에 접근할 수 있어, 하나의 경로가 끊겨도 다른 경로로 우회해 재도달 가능.
- 즉 "이해했다"는 건 "매번 유도 과정을 처음부터 재현한다"는 뜻이 아니다. 이해한 사람도 실전에서는 결론(공식)만 빠르게 꺼내 쓴다 — 다만 그 결론이 여러 갈래로 연결되어 저장되어 있어서, 더 유연하고 잘 안 잊히고, 필요하면 재유도도 가능하다는 차이가 있을 뿐이다.

### "촤르르 계산되는 경지"는 실제로 존재하지 않는다
암산 천재(폰 노이만 등)의 사례를 들어 "패턴을 자주 보면 계산 자체가 순식간에 이루어지는 경지에 도달하는가"를 논의했다. 결론은 아니다 — 그런 사례도 실제로는:
1. 방대한 중간 결과를 이미 암기해둔 상태에서 "조합"하는 것이거나
2. 계산을 더 쉬운 형태로 바꾸는 훈련된 절차(지름길)를 자동으로 적용하는 것

즉 "암기에 의존하지 않는 순수 계산 속도의 향상"이 아니라, **암기된 조각 + 자동화된 절차**의 조합이다.

### 실제로 존재하는 현상: 절차화(proceduralization)
"의식적으로 한 단계씩 계산하던 절차가, 충분한 반복을 거치면 의식적인 단계 인식 없이 자동으로 실행되는 하나의 덩어리로 압축되는 것." 체감상 이것이 "촤르르"에 가장 가까운 실제 현상이지만, 이것도 결국 "그 절차를 무수히 반복해서 자동화시킨 결과"이지, 반복 없이 즉각 도달하는 지름길은 없다.

### 전치 행렬에 적용하면
지금 숫자를 넣어보며 `matrix[i][j] ↔ matrix[j][i]`를 확인한 것은 정확히 "이해" 쪽 경로였다. 이 공식을 까먹어도 필요하면 숫자 몇 개로 금방 재유도할 수 있고, 변형된 문제(예: 반대 대각선 기준)에도 같은 확인 방식으로 대응할 수 있다는 점에서, 결론만 뚝 떼어 외운 것과는 질적으로 다른 저장 방식이다. 다음번에 비슷한 대칭/구조 문제를 몇 번 더 마주치면, 지금의 "일단 암기" 단계가 반복을 거쳐 절차화되어, 계산 없이 즉시 결론이 튀어나오는 체감으로 바뀔 것으로 예상된다.
