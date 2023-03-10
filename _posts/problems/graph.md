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
# 못품
# [친구 네트워크](https://www.acmicpc.net/problem/4195)
나의 풀이
```python
```
해설
```python
def find(x):
	if x == parent[x]:
		return x
	else:
		p = find(parent[x])
		parent[x] = p
		return p

def union(x, y):
	x = find(x)
	y = find(y)

	if x != y:
		parent[y] = x
		number[x] += number[y]
		
test_case = int(input())

for _ in range(test_case):
    parent = dict()
    number = dict()
    
    f = int(input())
    
    for _ in range(f):
        x, y  = input().split()
        
        if x not in parent:
            parent[x] = x
            number[x] = x
            
        if y not in parent:
            parent[y] = y
            number[y] = y
            
        union(x, y)
        print(number[find(x)])
```