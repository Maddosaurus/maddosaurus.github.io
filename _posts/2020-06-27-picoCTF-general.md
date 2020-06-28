---
layout: post
title:  "picoCTF 2019 - General Skills"
date:   2020-06-28 23:00 +0100
tags: writeups

The picoCTF 2019 contained multiple challenges in the "General" category. As most of these were rather short, they are documented in a collective post rather than single ones.  

<!--more-->


Name: Let's Warm Up  
Points: 50  
Challenge: If I told you a word started with 0x70 in hexadecimal, what would it start with in ASCII?  
Solution: A quick lookup in a [table](http://www.asciitable.com/) yields "p", so the answer would be `picoCTF{p}`.  

---

Name: Warmed Up  
Points: 50  
Challenge: What is 0x3D (base 16) in decimal (base 10)?  
Solution: For example, open the Windows Calculator (calc.exe), set it to "Programmer", select HEX, then enter 3D.  

---

Name: 2Warm  
Points: 50  
Challenge: Can you convert the number 42 (base 10) to binary (base 2)?  
Solution: Same as "Warmed Up", using calc in Programmer by selecting DEC, entering 42 and reading solution from BIN.  

---

Name: Bases  
Points: 100  
Challenge: What does this bDNhcm5fdGgzX3IwcDM1 mean? I think it has something to do with bases.  
Solution: By educated guessing, it looks like a base64-encoded string. On Linux, this can be decrypted by using `echo "bDNhcm5fdGgzX3IwcDM1" | base64 -d` which yields the flag.  

---

Name: First Grep  
Points: 100  
Challenge: Can you find the flag in [file](https://2019shell1.picoctf.com/static/c36821c1ffbd5ed01f10ba2ed05ab413/file)? This would be really tedious to look through manually, something tells me there is a better way. You can also find the file in /problems/first-grep_6_c2319e8af66fa6bec197edc733dd52dd on the shell server.  
Solution: As the challenge name hints, the use of grep helps solving this challenge. By executing `cat file | grep picoCTF` the flag gets yielded.  

---

Name: Resources  
Points: 100  
Challenge: We put together a bunch of resources to help you out on our website! If you go over there, you might even find a flag! https://picoctf.com/resources ([link](https://picoctf.com/resources))  
Solution: Open the link and read the page. The flag can be found in the text.

---

Name: strings it  
Points: 100  
Challenge: Can you find the flag in [file](https://2019shell1.picoctf.com/static/7963880d17a07ff2009afa1687fda1cc/strings) without running it? You can also find the file in /problems/strings-it_5_1fd17da9526a76a4fffce289dee10fbb on the shell server.  
Solution: As the challenge name implies, the use of the Linux command "strings" helps in solving this challenge. `strings strings | grep picoCTF` 

---

Name: what's a net cat?  
Points: 100  
Challenge: Using netcat (nc) is going to be pretty important. Can you connect to 2019shell1.picoctf.com at port 4158 to get the flag?  
Solution: Calling netcat with the given parameters yields the flag: `nc 2019shell1.picoctf.com 4158`  

---

Name: Based  
Points: 200
Challenge: To get truly 1337, you must understand different data encodings, such as hexadecimal or binary. Can you get the flag from this program to prove you are on the way to becoming 1337? Connect with nc 2019shell1.picoctf.com 44303.  
Solution:  
When connecting, you are greeted with multiple messages that are asking you to transcode mutliple formats into ASCII.  
It starts with binary, followed by octal and then hex without spacers:
```
Let us see how data is stored
chair
Please give the 01100011 01101000 01100001 01101001 01110010 as a word.
...
you have 45 seconds.....

Input:
chair
Please give me the  154 151 147 150 164 as a word.
Input:
light
Please give me the 706965 as a word.
Input:
pie
You've beaten the challenge
Flag: ...
```
Interestingly enough the words change on reconnection, so it is unlikely to just learn the words.
This is best done with an ASCII table at hand (i.e. http://www.asciitable.com/).  

---

Name: First Grep: Part II  
Points: 200  
Challenge: Can you find the flag in /problems/first-grep--part-ii_4_ca16fbcd16c92f0cb1e376a6c188d58f/files on the shell server? Remember to use grep.  
Solution:  
This challenge needs you to log in to the webshell. Once there, one can cd to the directory and do a recursive grep on all files:  
```
cd /problems/first-grep--part-ii_4_ca16fbcd16c92f0cb1e376a6c188d58f/files
grep -iR picoCTF* .
```
which yields the flag.

---

Name: plumbing  
Points: 200  
Challenge: Sometimes you need to handle process data outside of a file. Can you find a way to keep the output from this program and search for the flag? Connect to 2019shell1.picoctf.com 21957.  
Solution:
On connect, the console is spammed with a lot of output. As the challenge name hints, one can redirect the output into a file and the grep that file.  
```
nc 2019shell1.picoctf.com 21957 > out.txt
cat out.txt | grep picoCTF
```
which yields the flag.

---

Name: whats-the-difference  
Points: 200  
Challenge: Can you spot the difference? [kitters](https://2019shell1.picoctf.com/static/473cf765877f28edf95140f90cd76b59/kitters.jpg) [cattos](https://2019shell1.picoctf.com/static/473cf765877f28edf95140f90cd76b59/cattos.jpg) They are also available at /problems/whats-the-difference_0_00862749a2aeb45993f36cc9cf98a47a on the shell server  
Solution: The pictures differ at multiple places by one byte. By comparing them byte by byte and printing the differing bytes from cattos.jpg, we get the flag. `cmp -bl cattos.jpg kitters.jpg | awk '{print $3}' | tr -d "\n"`

---

Name: where-is-the-file  
Points: 200  
Challenge: I've used a super secret mind trick to hide this file. Maybe something lies in /problems/where-is-the-file_2_f1aa319cafd4b55ee4a60c1ba65255e2.  
Solution:  
The file in question is a "hidden" file whose name begins with a dot (.). To list these, the ls command needs the -h parameter:  
```
cd /problems/where-is-the-file_2_f1aa319cafd4b55ee4a60c1ba65255e2
ls -lah
cat .cant_see_mee
```

---

Name: flag_shop  
Points: 300  
Challenge: There's a flag shop selling stuff, can you buy a flag? [Source](https://2019shell1.picoctf.com/static/23b8f90691073c4466b11fe2bae8d6ae/store.c). Connect with nc 2019shell1.picoctf.com 29250.  
Solution: Upon connection, a text-based menu is presented to the user. There is not enough money in the account to buy the "real" flag.  
Upon inspection of the source code, it becomes apparent that the total cost for buying a flag is stored as an integer and could be overflowed by buying a very large number of flags, as this is the calculation base for `total_cost`:
```c
if(number_flags > 0){
    int total_cost = 0;
    total_cost = 900*number_flags;
    printf("\nThe final cost is: %d\n", total_cost);
    if(total_cost <= account_balance){
        account_balance = account_balance - total_cost;
        printf("\nYour current balance after transaction: %d\n\n", account_balance);
    }
```
By doing so, the cost turns negative, effectively adding money into the users' account:
```
Welcome to the flag exchange
We sell flags

(...)

Enter a menu selection
2
Currently for sale
1. Defintely not the flag Flag
2. 1337 Flag
1
These knockoff Flags cost 900 each, enter desired quantity
3579138

The final cost is: -1073743096

Your current balance after transaction: 1073744196

Welcome to the flag exchange
We sell flags

(...)

Enter a menu selection
2
Currently for sale
1. Defintely not the flag Flag
2. 1337 Flag
2
1337 flags cost 100000 dollars, and we only have 1 in stock
Enter 1 to buy one1
YOUR FLAG IS: 
```

---

Name: mus1c  
Points: 300  
Challenge: I wrote you a [song](https://2019shell1.picoctf.com/static/e0b32d09ed9e6cf0d4a7ded906a29e21/lyrics.txt). Put it in the picoCTF{} flag format  
Solution: The song turns out to be lyrics. On closer inspection, it turns out there is a programming language called [rockstar](https://codewithrockstar.com/online). When used with the online sandbox of rockstar, the lyrics turn out to be a running program that produces output:
```
114
114
114
111
99
107
110
(...)
```
This output can then be translated into the flag (decimal to character).

---

Name: 1_wanna_b3_a_r0ck5tar  
Points:  
Challenge: I wrote you another [song](https://2019shell1.picoctf.com/static/c7fa1eda3444e700dfd8addb3cf8e806/lyrics.txt). Put the flag in the picoCTF{} flag format.  
Solution: This is again a valid rockstar program. This time it requires input. The easiest way is to patch it and remove all input and parts that rely on the input:  
```
Rocknroll is right              
Silence is wrong                
A guitar is a six-string        
Tommy's been down               
Music is a billboard-burning razzmatazz!
Tommy is rockin guitar
Shout Tommy!                    
Music is amazing sensation 
Jamming is awesome presence
Scream Music!                   
Scream Jamming!                 
Tommy is playing rock           
Scream Tommy!       
They are dazzled audiences                  
Shout it!
Rock is electric heaven                     
Scream it!
Tommy is jukebox god            
Say it!                                     
Break it down
Shout "Bring on the rock!"
```
which yields the desired output:  
```
66
79
(...)
```
