---
layout : post
title : "에라토스테네스의 체, 베르트랑 공준, 골드바흐의 추측"
date : 2020-11-23 +0900
description : Mathmatics Algorithm for Prime number
tag : [Algorithm]
---

### 에라토스테네스의 체, 베르트랑 공준, 골드바흐의 추측

 백준에서 소수관련 문제를 풀다가 나중에 언젠가는 쓸까? 싶기도하고 개인적인 공부를 위해 남겨본다.

 사실 소수관련해서는 무조건 에라토스테네스의 체다. 압도적인 속도를 자랑하기 때문.

 난 이걸 모르고 그냥 막무가내로 코드짜서 몇몇 소수관련 문제를 넘겼는데, 어느순간부턴 안되었다.

 아래는 2581번 소수 문제를 풀때의 코드.

``` python
import sys

M = int(sys.stdin.readline())
N = int(sys.stdin.readline())
count = N
primeNum = []
while count != M-1:
    checkPrime = 0
    for i in range(2, count):
        if count%i == 0:
            checkPrime = 0
            break
        else :
            checkPrime += 1
    if checkPrime != 0:
        primeNum.append(count)
    count -= 1

if M <= 2 and N >= 2:
    primeNum.append(2)

if not primeNum:
    sys.stdout.write(str(-1))
else :
    sumPrime = sum(primeNum)
    minPrime = min(primeNum)

    sys.stdout.write(str(sumPrime)+"\n"+str(minPrime))
```

 이후 다른 소수문제를 접하면서 에라토스테네스의 체를 쓰기 시작했다.

 아래는 기록용으로 위 두 문제를 풀면서 작성한 코드.

 먼저 4948번 베르트랑 공준 문제.

```python
import sys

# 에라토스테네스의 체 이용
def primeNum(m, n):
    prime = [True] * (n+1)
    k = int(n ** 0.5)
    for i in range(2, k+1):
        if prime[i] == True:
            for j in range(i+i, n+1 , i):
                prime[j] = False
    return [i for i in range(m,n+1) if prime[i] == True]


while True:
    n = int(sys.stdin.readline())
    if n == 0:
        break

    primeList = primeNum(n + 1, 2 * n)
    #check prime number list
    #print('primeList : ', primeList)
    if 1 in primeList:
        primeList.pop(0)

    sys.stdout.write(str(len(primeList))+"\n")
```

 시간이 좀 오래걸리는데, 아무래도 while True: 안에서 계속 primeList를 만드는 것 보다 아예 밖에서 primeNum(0, 123456*2) 를 보내놓고 거기서 빼다 쓰는게 시간이 훨씬 더 절약될 것 같다.

 다음으로 9020번 골드바흐의 추측

```python
import sys

# 에라토스테네스의 체 이용
def primeNum(m, n):
    prime = [True] * (n+1)
    k = int(n ** 0.5)
    for i in range(2, k+1):
        if prime[i] == True:
            for j in range(i+i, n+1 , i):
                prime[j] = False
    return [i for i in range(m,n+1) if prime[i] == True]
primeSet = primeNum(0,10000)
T = int(sys.stdin.readline())
if 1 in primeSet:
    primeSet.pop(0)
while T:
    n = int(sys.stdin.readline())
    aWouldFixed = 0

    if n/2 in primeSet:
        sys.stdout.write("{0} {1}\n".format(int(n/2), int(n/2)))
    else :
        i = 0
        while True:
            if n/2 > primeSet[i] and n/2 < primeSet[i+1]:
                a = i
                b = i + 1
                break
            i += 1

        b_backup = b
        while n != primeSet[a] + primeSet[b]:
            if n < primeSet[a] + primeSet[b]:
                a -= 1
                b = b_backup
            else :
                b += 1
        sys.stdout.write("{0} {1}\n".format(primeSet[a], primeSet[b]))

    T -= 1
```

 단순히 소수를 불러오는게 아니라 어떤 소수보다 크거나 작은 케이스를 골라내는 건데, 베르트랑 공준에서 얻은 실패를 여기서 약간 보완해서 while문 밖에서 미리 소수 배열들을 만들어놓고 진입했다. 그리고 받는 숫자의 중간부분을 기점으로 좌우로 벌려나가면서 해당 숫자를 찾은 케이스. 저기서 b = b_backup은 공간을 점점 벌리기만 할 가능성을 고려해서 추가한 케이스. 다행히 한방에 맞았습니다! 가 떴다.



![img1](https://raw.githubusercontent.com/ReaperMaKNaE/reapermaknae.github.io/main/assets/img/20201123-1.png)






### Reference

에라토스테네스의 체, [https://www.acmicpc.net/problem/1978](https://www.acmicpc.net/problem/1978)

베르트랑 공준, [https://www.acmicpc.net/problem/4948](https://www.acmicpc.net/problem/4948)

골드바흐의 추측, [https://www.acmicpc.net/problem/9020](https://www.acmicpc.net/problem/9020)