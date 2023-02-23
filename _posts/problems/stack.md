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

# [스택 수열](https://www.acmicpc.net/submit/1874)
- stack, greedy
나의 풀이
```python
n = int(input())

stack = [];
temp = [];
answers = [];

for _ in range(n):
    x = int(input())
    temp.append(x)
    
for i in range(1, n + 1):
    stack.append(i)
    answers.append("+")
    
    while True:
        if not stack or not temp:
            break;
            
        last_value = stack[-1]
        first = temp[0]
        
        if last_value == first:
            temp.pop(0)
            stack.pop()
            answers.append("-")
        else:
            break;

if len(stack) > 0 or len(temp) > 0:
    print("NO")
else:
    print("\n".join(answers))

```
해설
```python
n = int(input())

count = 1
stack = []
result = []

for i in range(1, n + 1):
    data = int(input())
    while count <= data:
        stack.append(count)
        count+=1
        result.append("+")
    if stack[-1] == data:
        result.append("-")
        stack.pop()
    else:
        print("NO")
        exit(0)
        
print("\n".join(result))
```