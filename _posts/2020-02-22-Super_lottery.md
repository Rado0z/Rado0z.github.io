---
title: Super Lottery
published: true
categories: Ringzer0CTF
---


### [](#header-2)Inroduction
supper lottery is one of web challenge from rinzer0ctf platform which is really really fun challenge.

### [](#header-3)1- understanding the web application

![](https://raw.githubusercontent.com/Rado0z/Rado0z.github.io/master/assets/rinzer-1.png)

there is combination of nubmers we should predicte the right combination numbers to win the prize. so first i used the burpsuite to get the idea how this web application works.

![](https://raw.githubusercontent.com/Rado0z/Rado0z.github.io/master/assets/ring-burp.png)

the first thing comes to my mind is to try to fuzz the combination parameter but it seems bad idea because this parameter seems only take this combination and compare with the right one.

so i moved to next step which is looking for web directories.

### [](#header-3)2- Content Discovery / Directory Bruting

as always when we try to make content discovery, the first thing we do is to check the robots.txt

![](https://raw.githubusercontent.com/Rado0z/Rado0z.github.io/master/assets/robots_rinzer0.png)

and i found model folder and when i check it i found python script which is give us the idea how this the web application works.

![](https://raw.githubusercontent.com/Rado0z/Rado0z.github.io/master/assets/lottery_code.png)

so the next step is to understand how this code works and find the way to get the combination numbers

### [](#header-3)3- Cracking the way :)

so lets take a look about how this code works:

```python

#!/usr/bin/env python2
import random, os, sys

def generate_combination():
    combination = []
    for i in range(3):
        numbers = ""
        for _ in range(10):
            rand_num = random.randint(0, 99)
            if rand_num < 10:
                numbers += "0"
            numbers += str(rand_num)
            if _ != 9:
                numbers += "-"
        combination.append(numbers)
    return combination


if __name__ == '__main__':
    sessionId = str(sys.argv[1])
    random.seed(os.getpid())
    combList = generate_combination()
    f = open('/tmp/combination-' + sessionId + '.txt', 'w+')
    for comb in combList:
        f.write(str(comb)+'\n')
    f.close()

```

first, from the main function, it seems the php script ( which is web application langauge is used ) create a sessionid and give to python script and create three combination numbers in put them in arrary. and save it in file inside `/tmp` folder. the good thing is this code create a three of combination numbers. so if we know the first one that mean we can get the second and win the prize. so we understand the goal and we are in two option to reach this goal which is:
1. to prediect the numbers from the first combination.
2. get the file but we should find LFI vulnerability to get /tmp folder. and at first i feel this option is good because the we have the sessionid and it's easy to get the file.

after deep looking i could not get and find any parameters got lead the to LFI.
so backing to option 1 i will and deep understanding how this script works.
so first i edit this script to understand how these number is generated

```python

import random, os, sys

def generate_combination():
    combination = []
    for i in range(3):
        numbers = ""
        for _ in range(10):
            rand_num = random.randint(0, 99)
            if rand_num < 10:
                numbers += "0"
            numbers += str(rand_num)
            if _ != 9:
                numbers += "-"
        combination.append(numbers)
    print combination

generate_combination()

```
and here is the result
```
['70-76-48-93-93-89-18-72-26-27', '99-11-09-80-66-71-37-87-96-34', '83-03-93-33-96-95-02-53-78-31']

```
these numbers is always changed so it's really hard to predict.

and then i looked and i found this function `random.seed(os.getpid())`. after doing some research, i found that if we put for example `random.seed(1)` before the the generation function, will give us the same result in every time. so if we know what the proccess id that got excute in web servier, then we will the combiantion numbers. it's great but there is no way to get the pid while we don't have any shell or rce vulnerablity.


### [](#header-3)4- the winnig.

as i understand that the maxiumun process id that can ubuntu os create is 32768. so one of them is right one for the combination. and here is the steps to solve this part:
1. get the first win number from the webapplication
2. write a python script and compare it with first number

```python

import random, os, sys

def generate_combination():
  for i in range(1,30000):
    random.seed(i)
    combination = []
    for i in range(3):
        numbers = ""
        for _ in range(10):
            rand_num = random.randint(0, 99)
            if rand_num < 10:
                numbers += "0"
            numbers += str(rand_num)
            if _ != 9:
                numbers += "-"
        combination.append(numbers)
        if combination[0] == '22-90-10-28-89-59-35-10-06-92':
                print combination


generate_combination()

```

and after i run it got the reslut

```
['22-90-10-28-89-59-35-10-06-92', '21-57-14-02-01-10-83-75-31-35', '87-64-43-17-88-19-02-95-92-90']
```
which the second one is the right one


![](https://raw.githubusercontent.com/Rado0z/Rado0z.github.io/master/assets/win.png)

and we got it !.

thanks for reading and i will be very happy for any feedback.



