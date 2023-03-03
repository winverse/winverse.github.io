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

# [프린터 큐](https://www.acmicpc.net/problem/1966)
나의 풀이
```python
test_count = int(input())

def merge(data, target):
    new_arr = []
    for i in range(len(data)):
        if i == target:
            new_arr.append({ "is_target": True, "value": data[i] })
        else:
            new_arr.append({ "is_target": False, "value": data[i] })
    return new_arr


for _ in range(test_count):
    n, m = map(int, input().split())
    data = list(map(int, input().split()))
    
    queue = merge(data, m)
    data.sort(reverse=True)
    
    count = 0
    while True:
        first = queue[0];
        if first["value"] == data[0]:
            if first["is_target"]:
                count+=1
                print(count)
                break;
            else:
                data.pop(0)
                queue.pop(0)
                count += 1
        else:
            queue.append(queue.pop(0))
```
해설
```python
test_case = int(input())

for _ in range(test_case):
    n, targetIndex = list(map(int, input().split()))
    queue = list(map(int, input().split()))
    queue = [(value, idx) for idx, value in enumerate(queue)]
    
    count = 0
    while True:
        if queue[0][0] == max(queue, key = lambda x: x[0])[0]:
            count+=1
            if queue[0][1] == targetIndex:
                print(count)
                break;
            else:
                queue.pop(0)
        else:
            queue.append(queue.pop(0))
```