# 투 포인터(Two Pointers)

## 정의

- 리스트에 순차적으로 접근해야 할 때 **두 개의 점의 위치를 기록하면서 처리**하는 알고리즘
- 1차원 배열에서 각자 다른 원소를 가리키고 있는 **2개의 포인터를 조작해가면서 원하는 값을 찾을 때까지 탐색**하는 알고리즘



## 동작 방식 및 구현

- 보통 left, right나 start, end와 같은 식으로 2개 포인터의 이름을 붙임
- 찾는 값을 `target`이라고 할 때, 포인터의 이름을 start(s), end(e)라고 하자
- 처음에는 `start = end = 0`이고, 두 포인터들의 관계 조건은 항상 `start <= end`이다
- 2개의 포인터의 이름은 현재 부분배열의 시작과 끝을 의미



```python
# 3273 두 수의 합
import sys

input = sys.stdin.readline

n = int(input())
li = sorted(list(map(int, input().rstrip().split())))
x = int(input())

s = 0
e = n - 1
cnt = 0

while s < e < n:
    sums = li[s] + li[e]
    if sums == x:
        cnt += 1
        s += 1

    elif sums > x:
        e -= 1

    else:
        s += 1


print(cnt)

```

- 두 수의 합이 `x`인 경우의 수를 구하는 문제
- 오름차순 정렬 이용
- 두 수의 합이 `x`보다 작으면 `s`의 index를 오른쪽으로 이동
- 두 수의 합이 `x`보다 크면 `e`의 index를 왼쪽으로 이동



## 백준 단계별로 풀어보기

- [3273 두 수의 합](https://www.acmicpc.net/problem/3273)
  - 기초적인 문제
- [2470 두 용액](https://www.acmicpc.net/problem/2470)
  - 3273과 유사하지만, 수학적으로 추가적인 고려가 필요
- [1806 부분합](https://www.acmicpc.net/problem/1806)
  - 시간 제한이 짧은 것이 힌트
  - 중복 연산을 최소화하여 문제를 해결해야 함
- [1644 소수의 연속합](https://www.acmicpc.net/problem/1644)
  - 소수 배열을 우선 만들어야 함
  - 에라토스테네스의 체 활용
- [1450 냅색문제](https://www.acmicpc.net/problem/1450)
  - 해결중
  - Meet In The Middle 알고리즘 활용 필요(시간복잡도)
  - MITM 알고리즘 학습 후 기록



### cf. 슬라이딩 윈도우

- 투 포인터와 유사하지만 슬라이딩 윈도우는 **부분 배열의 크기가 고정**되어 있음
- 즉, 왼쪽 포인터와 오른쪽 포인터가 함께 움직임



