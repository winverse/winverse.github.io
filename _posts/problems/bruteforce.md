---
layout: post
author: winverse
category: problems
is_published: false
---
나의 풀이
```python

```
해설
```python

```

# [블랙잭](https://www.acmicpc.net/submit/2798)
나의 풀이
```python
import itertools

n, m = map(int, input().split())
cards = list(map(int, input().split()))

nums = list(filter(lambda x: x <= m, map(sum, itertools.combinations(cards, 3))))
nums.sort(reverse=True)
print(nums[0])
```
해설
```python
n, m = map(int, input().split())
data = list(map(int, input().split()))

result = 0
length = len(data)

count = 0
for i in range(0, length):
    for j in range(i + 1, length):
        for k in range(j + 1, length):
            sum_value = data[i] + data[j] + data[k]
            if sum_value <= m:
                result = max(result, sum_value)
print(result)
  
```

# 배운점
- 전체 카드의 개수가 100개 이므로 최대 100^3인 1백만개가 전체 숫자이다.
- 이렇게 되면 초당 2천만개 정도를 처리하는 python에서는 무리 없이 처리가 가능한 숫자이다.
- 그래서 삼중 for 문을 사용하여 처리하였다.