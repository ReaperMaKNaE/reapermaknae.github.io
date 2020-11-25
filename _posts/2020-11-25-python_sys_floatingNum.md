---
layout : post
title : "python에서 floating num 표현"
date : 2020-11-25 +0900
description : Expression of floating number in Python
tag : [Algorithm]
---

### Floating number 소수점 자리수 고정

 백준 3053번을 풀다가 문득 print를 쓰기 싫어 찾아보게 되었다.

 방법은 간단했다.

```python
'%0.6f' % float(something what you want to express in float)
```

 대충 위와 같은 꼴이다.

 이런 저런 표현들이 참 많다. 일단 3053 택시 기하학을 해결한 코드는 아래와 같다.

```python
import sys

PI = 3.14159265359
R = int(sys.stdin.readline())

sys.stdout.write(str('%0.6f' % float((R ** 2) * PI))+"\n")
sys.stdout.write(str('%0.6f' % float(2*R*R)))
```

 PI의 경우, 3.141592를 썼는데 틀렸다고 나오더라. 찾아보니 뒤에 몇 개 더 붙이길래 65359를 붙여서 해결했다. 어릴 땐 3.14만 썼었고, 어느순간 부터는 3.141592를 썼는데 이제 다섯개가 더 붙어야 한다니. 왠지모르게 또 더 늘려서 써야할 것 같은 느낌이다.