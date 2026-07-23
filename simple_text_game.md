# 미니 프로젝트: 아주 간단한 텍스트 게임 (완성)

파이썬 기초 커리큘럼의 "미니 콘솔 프로그램" 단계로, 텍스트 기반 턴제 전투 게임을 처음부터 직접 설계했다.

---

## 요구사항 (목표) — 전부 충족

- [x] 플레이어와 몬스터가 각각 체력(HP)을 가짐
- [x] 매 턴 서로 랜덤한 데미지로 공격 (`random.randint` 사용)
- [x] 매 턴 상태(현재 체력)를 출력
- [x] 누군가의 체력이 0 이하가 되면 전투 종료, 승자 출력
- [x] `input()`으로 사용자가 직접 캐릭터 이름/체력을 입력하며 게임을 시작

---

## 최종 완성 코드

```python
import random

def create_character(name, hp):
    character_information = {'name': name, 'hp': hp}
    return character_information

def attack(attacker, defender):
    original_hp = defender['hp']
    defender['hp'] = defender['hp'] - random.randint(1,5)
    current_hp = defender['hp']
    attacked_damaged = original_hp - current_hp
    return f"{attacker['name']}의 공격에 의해 {defender['name']}의 hp가 {attacked_damaged}만큼 감소하여 {current_hp}만큼 남음 "


print("플레이어 정보 입력:")
player = input().split()
player[1] = int(player[1])
print("적 정보 입력:")
enemy = input().split()
enemy[1] = int(enemy[1])

player = create_character(player[0], player[1])
enemy = create_character(enemy[0], enemy[1])

print(f"이름:{player['name']} hp:{player['hp']} 적 이름:{enemy['name']} hp:{enemy['hp']} 게임을 시작합니다")

while player['hp'] > 0 and enemy['hp'] > 0:
    enemy_state = attack(player, enemy)
    if enemy['hp'] > 0:
        print(enemy_state)
    else:
        print(f"{enemy['name']} 사망, {player['name']} 승리")
        break
    player_state = attack(enemy, player)
    if player['hp'] > 0:
        print(player_state)
    else:
        print(f"{player['name']} 사망, {enemy['name']} 승리")
```

---

## 개발 과정에서 겪은 오류들과 원인

### 오류 1: 함수 자체와 그 함수가 만든 결과물을 혼동
```python
def attack(damage):
    create_character['hp'] = create_character['hp'] - damage   # 잘못됨
```
`create_character`는 함수 그 자체이지 딕셔너리가 아니다. 딕셔너리는 그 함수를 **호출해서 나온 결과물**(`player`처럼)이지, 함수 이름 자체를 인덱싱할 수는 없다. `TypeError: 'function' object is not subscriptable` 발생. 해결: `attack`이 캐릭터 딕셔너리 자체를 매개변수로 받도록 수정.

### 오류 2: 함수를 괄호 없이 호출
```python
character['hp'] - random.randint   # 괄호 없음 -> 함수 자체를 가리킴, 실행 안 됨
```
`TypeError: unsupported operand type(s) for -: 'int' and 'method'` 발생. 함수는 항상 `함수이름(필요한 인자들)`처럼 괄호를 붙여 호출해야 실행되어 결과값이 나온다는 것을 확인.

### 오류 3: 문자열과 숫자를 `+`로 직접 이어붙임
```python
return "받은 데미지:" + damaged_hp + "현재 남은 체력:" + current_hp
```
`TypeError: can only concatenate str (not "int") to str`. 숫자를 문자열과 바로 `+`로 못 붙인다. `str()`로 변환하거나(1차 수정), f-string(`f"...{변수}..."`)으로 정리(최종)했다.

### 오류 4: 리스트를 통째로 함수에 넘김 (매개변수 개수 불일치)
```python
create_character(player)   # player는 리스트 하나인데, 함수는 매개변수 2개(name, hp)를 요구
```
`TypeError: create_character() missing 1 required positional argument: 'hp'`. 리스트 안의 값(`player[0]`, `player[1]`)을 각각 따로 꺼내서 넘겨야 했다.

### 오류 5: 함수 호출 결과를 저장하지 않음
```python
create_character(player)   # 반환값을 받을 변수가 없어 결과가 그냥 사라짐
```
함수가 `return`해도 그 결과를 담을 변수가 없으면 버려진다. `player = create_character(...)`처럼 반환값을 다시 변수에 대입해야 실제로 반영된다.

### 오류 6: 이미 죽은 캐릭터가 계속 공격하는 논리 버그 ("좀비 공격")
```python
while player['hp'] > 0 and enemy['hp'] > 0:
    enemy_state = attack(player, enemy)
    if enemy['hp'] > 0:
        print(enemy_state)
    else:
        print(f"{enemy['name']} 사망, {player['name']} 승리")
    player_state = attack(enemy, player)   # <- enemy가 이미 죽었어도 무조건 실행됨
    ...
```
첫 번째 공격으로 `enemy`가 죽었는데도, 반복문이 멈추지 않고 그 다음 줄(`enemy`가 `player`를 공격하는 코드)이 조건 없이 실행되어, 죽은 캐릭터가 한 번 더 공격하는 버그가 있었다. 실제로 `random.randint`를 크게 조작해 즉사 시나리오를 강제로 재현해 확인함.

**해결:** `enemy`가 죽은 분기에 `break`를 추가해, 그 즉시 반복문을 완전히 빠져나오도록 수정. `break`는 반복문(`while`/`for`)을 조건과 상관없이 즉시 종료시키는 명령문이라는 것을 이 과정에서 익힘.

---

## 새로 배운 도구들

### `random` 모듈
- `random.randint(a, b)` — a부터 b까지(양쪽 포함) 랜덤 정수 하나 반환
- 표준 라이브러리에 속한 모듈이며, `len()`/`max()` 같은 내장 함수와 달리 `import random`으로 명시적으로 불러와야 사용 가능하다는 것을 확인

### f-string
```python
f"받은 데미지: {damaged_hp}  현재 남은 체력: {current_hp}"
```
문자열 앞에 `f`를 붙이고 `{변수}`로 감싸면 값이 자동으로 문자열에 끼워 넣어진다. `str()` 변환과 `+` 이어붙이기를 대체하는 더 깔끔한 방법.

### `input()`
- `input("안내문구")`는 사용자 입력을 **항상 문자열로** 반환한다. 숫자를 입력해도 문자열이므로, 숫자로 쓰려면 `int()`/`float()` 변환이 필요하다.
- 여러 값을 한 줄에 입력받고 싶을 때는 `.split()`으로 쪼갠다.
- `.split()`의 결과는 **리스트**이지 딕셔너리가 아니다. 리스트는 원래 "여러 값을 하나의 변수에 순서대로 담아두는" 자료형이므로, `player = input(...).split()`처럼 변수 하나에 값 여러 개가 들어있는 게 자연스러운 동작이다.

### `break`
반복문(`while`/`for`) 안에서 만나는 즉시, 그 반복문 전체를 조건과 무관하게 종료시키는 명령문. "이미 승부가 났으니 더 이상 진행할 필요가 없다"는 상황에서 반복문을 즉시 빠져나오는 데 사용했다.

### 변수 재사용에 대한 메모
`player = create_character(player[0], player[1])`처럼, 원래 리스트였던 변수를 딕셔너리로 재할당하는 것은 문법적으로 완전히 정상이다. 파이썬 변수는 "값을 담는 상자"가 아니라 "값에 붙는 이름표"에 가까워, 같은 이름에 다른 타입의 값을 다시 붙이는 것이 자유롭다. 이번처럼 "같은 대상(플레이어)이 원시 데이터에서 가공된 최종 형태로 정제되는" 경우엔 이름을 재사용하는 것이 자연스러운 흐름이다.

---

## 학습 방법론 메모

- 프로젝트는 알고리즘 문제보다 훨씬 더 잘게 쪼개서 접근해야 막막함이 줄어든다는 것을 확인함 (설계와 구현을 동시에 요구하는 "무에서 통째로 만들기"는 스캐폴딩 원칙과 어긋남 — 함수 하나씩 조각내서 진행하는 것으로 조정).
- 막혔을 때 "스스로 고민할지 바로 물어볼지"의 기준: **논리/알고리즘을 몰라서 막힌 것이면 스스로 고민, 처음 보는 문법이나 에러 메시지면 바로 물어보는 것**이 효율적이라는 원칙을 다시 확인함.
- 반복문 안에서 "조건 확인 후 상태 출력"과 "반복문 자체를 종료하는 것"은 서로 다른 일이라는 것을 좀비 공격 버그를 통해 확인함 — 상태를 확인하는 `if`만으로는 반복문이 멈추지 않으며, 정말로 멈춰야 한다면 `break`처럼 반복문 자체를 종료시키는 명령이 별도로 필요하다.
