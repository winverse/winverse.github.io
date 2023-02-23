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
```
```

# [블랙잭](https://www.acmicpc.net/submit/2798)
나의 풀이
```

```
해설
```
```

# [음계](https://www.acmicpc.net/submit/2920)
나의 풀이
```python
l = "".join(list(map(str, input().split())))

if l == "12345678":
  print("ascending")
elif l == "87654321":
  print("descending")
else:
  print("mixed")

```
해설
```python
l = list(map(int, input().split()))

isAscending = True
isDescending = True

for i in range(1, 8):
   if l[i - 1] > l[i]:
       isAscending = False
   elif  l[i - 1] < l[i]:
        isDescending = False
        
if isAscending:
    print("ascending")
elif isDescending:
    print("descending")
else:
    print("mixed")
    
```