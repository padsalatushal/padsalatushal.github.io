---
title: "My Approach and Solutions: Navigating the eBhadra CTF Challenge"
date: 2024-02-19 
categories: [CTF,Bug Bounty, Cybersecurity]
tags: [bugbounty-writeup, Infosec, ctf]
---

## Introduction
It’s been a while, but I’m back with another write-up. This time, I’m sharing my approach and solutions to the eBhadra CTF Challenge. Previously, I completed the CyberSeige 2.0 CTF by BugBase and secured an rank of 8. Now, in the eBhadra CTF, I’ll detail my approach and solutions.

![image_of_intro](/assets/img/2024-02-19-my-approach-and-solutions-navigating-the-ebhadra-ctf-challenge/1.webp)


## i. Free Flag
Points: 50
Category: misc

![image_of_challege_1](/assets/img/2024-02-19-my-approach-and-solutions-navigating-the-ebhadra-ctf-challenge/2.webp)

Opening the link will redirect us to BugBase’s discord server. From the announcement channel, we can directly get the flag.

![image_of_discord](/assets/img/2024-02-19-my-approach-and-solutions-navigating-the-ebhadra-ctf-challenge/3.webp)

```bash
ebhadra{w3lc0m3_70_3_bh4dr4_c7f}
```

## ii. API Heist Challenge
Points: 200
Category: Web
Challenge Description: Welcome to BugBase CTF API. We are excited to announce that we plan to launch this API on April 1st, 2024. We have conducted a thorough security assessment of the critical endpoints and addressed some vulnerabilities. However, if you discover any other security issues, please do not hesitate to inform us. We appreciate your cooperation in helping us maintain the security and integrity of our API.

![image_of_challege_2](/assets/img/2024-02-19-my-approach-and-solutions-navigating-the-ebhadra-ctf-challenge/4.webp)

So, when you open up the provided link, you’ll find something pretty interesting. It’s basically a sneak peek into the BugBase CTF API.

![image_of_source_code](/assets/img/2024-02-19-my-approach-and-solutions-navigating-the-ebhadra-ctf-challenge/5.webp)


![image_of_source_code](/assets/img/2024-02-19-my-approach-and-solutions-navigating-the-ebhadra-ctf-challenge/6.webp)
It contains various API endpoints and variables, along with IP addresses and port numbers.

Let’s explore that using Burp Suite.

![image_of_burp](/assets/img/2024-02-19-my-approach-and-solutions-navigating-the-ebhadra-ctf-challenge/7.webp)

The /api/v2/leaderboard endpoint provides details such as user ID, position, score, and username for three users.

![image_of_burp](/assets/img/2024-02-19-my-approach-and-solutions-navigating-the-ebhadra-ctf-challenge/8.webp)

The /api/v2/competitions endpoint gives the details about competition endDate, name (July Jigsaw) , startDate along with little reward.

![image_of_burp](/assets/img/2024-02-19-my-approach-and-solutions-navigating-the-ebhadra-ctf-challenge/9.webp)

The /api/v2/login endpoint, using the username and password obtained from the source code, provides an authentication token.

![image_of_burp](/assets/img/2024-02-19-my-approach-and-solutions-navigating-the-ebhadra-ctf-challenge/10.webp)

We can utilize the authentication token to access the /api/v2/user/{id} endpoint for the third user, ‘phenomenal’. We obtained the user ID from the /api/v2/leaderboard endpoint.

![image_of_burp](/assets/img/2024-02-19-my-approach-and-solutions-navigating-the-ebhadra-ctf-challenge/11.webp)
It give us flag which is tinyurl link.

Too easy, isn’t it?


![image_of_rickroll](/assets/img/2024-02-19-my-approach-and-solutions-navigating-the-ebhadra-ctf-challenge/12.webp)
However, it turns out to be a Rickroll. When we attempt to change the user ID from ‘phenomenal’ to ‘tuhin1729’ in the same request, we receive a message indicating that we don’t have access to this page.

![image_of_burp](/assets/img/2024-02-19-my-approach-and-solutions-navigating-the-ebhadra-ctf-challenge/13.webp)

Within seconds, I realized that I needed to search for other endpoints. However, since the source code was already provided and the challenge seemed to be related to API security, I decided not to waste time fuzzing for additional API endpoints.

I had noticed the /api/v2 endpoint from the beginning and thought it was the right time to check for any other versions of the API. After changing the version from v2 to v1, I found the flag.

![image_of_burp](/assets/img/2024-02-19-my-approach-and-solutions-navigating-the-ebhadra-ctf-challenge/14.webp)

Other than expecting a Rickroll, it turned out to be the actual flag.


![image_of_flag](/assets/img/2024-02-19-my-approach-and-solutions-navigating-the-ebhadra-ctf-challenge/15.webp)

## iii. BugBase Employee Directory
Points: 175
Category: Web
Challenge Description: We have created a website where you can learn about the BugBase team, who work tirelessly day and night.


![image_of_challege_3](/assets/img/2024-02-19-my-approach-and-solutions-navigating-the-ebhadra-ctf-challenge/16.webp)

I opened the challenge link and found the BugBase Employee Directory website. It consists of three departments: Security, Development, and Marketing.


![image_of_challege_3](/assets/img/2024-02-19-my-approach-and-solutions-navigating-the-ebhadra-ctf-challenge/17.webp)

After selecting the Marketing department and clicking the filter button, it displays the employee ID, name, and department.

![image_of_challege_3](/assets/img/2024-02-19-my-approach-and-solutions-navigating-the-ebhadra-ctf-challenge/18.webp)


Upon inspecting the source code, I discovered the /sup3r_s3cr3t endpoint, which was commented out.

![image_of_challege_3](/assets/img/2024-02-19-my-approach-and-solutions-navigating-the-ebhadra-ctf-challenge/19.webp)


The `/sup3r_s3cr3t` endpoint contains the source code for the application.


![image_of_challege_3](/assets/img/2024-02-19-my-approach-and-solutions-navigating-the-ebhadra-ctf-challenge/20.webp)

It’s a straightforward Python Flask web application. The initial route, ‘/sup3r_s3cr3t’, is used to disclose the source code of the main file.

The ‘/getEmployee’ POST request executes the ‘get()’ function, which retrieves the dept value from the request. Then, it connects to the ‘employees.db’ database using SQLite3 and executes the SQL query.

Is it possible to perform SQL injection?

Understanding the SQL query.
```sql
"select * from employees where Department LIKE ?", (dept.replace("%", "")
```
The query fetches data from a database table called ‘employees’ based on a department pattern. The ‘?’ symbol acts as a placeholder for the department pattern. The ‘replace’ function removes any ‘%’ characters from the department.

Since ‘%’ is filtered, we cannot use it directly. Therefore, it may be possible to obfuscate it or find another operator that works with the ‘LIKE’ operator.

After reading about SQL ‘LIKE’ operator on https://www.w3schools.com/sql/sql_like.asp, I discovered another operator, ‘_’, which can be used with the ‘LIKE’ operator.”

![image_of_challege_3](/assets/img/2024-02-19-my-approach-and-solutions-navigating-the-ebhadra-ctf-challenge/21.webp)


To make the ‘’ wildcard work, we need to know the exact length of the value of ‘dept’. I quickly wrote the following script, which generates the ‘payload.txt’ file containing all possible payloads up to 30 ‘’ for the combination of ‘a-z’ for brute force.

```python
with open("payload.txt", "w") as file:
    for letter in range(ord('a'), ord('z')+1):
        for underscore_count in range(30):
            file.write(chr(letter) + '_' * underscore_count + "\n")
```
payload.txt :

![image_of_challege_3](/assets/img/2024-02-19-my-approach-and-solutions-navigating-the-ebhadra-ctf-challenge/22.webp)


Now, finally, send the /getEmployee POST request to Intruder and set the ‘dept’ value as position.


![image_of_challege_3](/assets/img/2024-02-19-my-approach-and-solutions-navigating-the-ebhadra-ctf-challenge/23.webp)

Now, use the payloads generated using the Python script.

![image_of_challege_3](/assets/img/2024-02-19-my-approach-and-solutions-navigating-the-ebhadra-ctf-challenge/24.webp)


Sort the results based on the length of the response to find unique results. ‘s_______’ corresponds to security, ‘d__’ is for development, but there is another one starting with ‘e’. Check the response for that.

![image_of_challege_3](/assets/img/2024-02-19-my-approach-and-solutions-navigating-the-ebhadra-ctf-challenge/25.webp)


Got the flag.

## iv. Crazy Encoding
Points: 100
Category: Reversing
Challenge Description: Our security Engineers came up with a crazy encoding , can you decode the flag if we gave you it’s source code?


![image_of_challege_4](/assets/img/2024-02-19-my-approach-and-solutions-navigating-the-ebhadra-ctf-challenge/26.webp)


So, when you open up the provided link, you’ll find python script.

```python
import random

def encode_flag(flag, random_string):
    encoded_flag = ""
    random_len = len(random_string)
    
    for i in range(len(flag)):
        flag_char = flag[i]
        random_char = random_string[i % random_len]
        
        # Calculate the encoded value
        encoded_value = (ord(flag_char) + ord(random_char)) ^ ord(random_char)
        
        # Convert to binary and append to the final string
        encoded_flag += format(encoded_value, '08b')
        
        # Check for every 8th character to cycle
        if (i + 1) % 8 == 0:
            random_string = random_string[1:] + random_string[0]
    
    # Replace 0 with "." and 1 with "-"
    encoded_flag = encoded_flag.replace('0', '.').replace('1', '-')
    
    return encoded_flag

# Example usage:
flag = "ebhadra{REDACTED}"
random_string = '_3]|4twR><0<Y[&snV;j.hFfTqgpv]"&SNFgpA7<Z~"cpP6NfI*M;[mtzy/B'
start = random.randint(0, len(random_string) - 8)
end = start+8
encoded_flag = encode_flag(flag, random_string[start:end])
print(encoded_flag)
```

This Python script defines an `encode_flag` function used to encode a given flag using a provided random string. The process involves XOR operations between ASCII values of the flag characters and corresponding characters from the random string. Then, the result is converted to binary format, with ‘0’s represented as ‘.’ and ‘1’s as ‘-’.

flow of encryption script :

  1. Iterate through each character in the flag.
  2. Obtain the corresponding character from the random string.
  3. Perform an XOR operation between the ASCII values of the flag character and the random character.
  4. Convert the resulting value to binary format and append it to the encoded flag string.
  5. Repeat steps 2–4 until all flag characters are processed.
  6. Replace ‘0’s with ‘.’ and ‘1’s with ‘-’ in the final encoded flag.

Writing the decryption script in python that decode the cipher text which is encrypted by upon script.

```python
from Crypto.Util.number import *
def decode_flag(cipher,random_string):
    cipher=long_to_bytes(int(cipher.replace(".","0").replace("-","1"),2))
    random_len = len(random_string)
    decoded_flag=""
    for i in range(len(cipher)):
        cipher_char = cipher[i]
        random_char = random_string[i % random_len]
        # Calculate the encoded value
        decoded_value = (cipher_char^ ord(random_char))-ord(random_char)
        # Convert to binary and append to the final string
        decoded_flag += chr(abs(decoded_value))
        # Check for every 8th character to cycle
        if (i + 1) % 8 == 0:
            random_string = random_string[1:] + random_string[0]
    return decoded_flag
cipher=" -..---.----...-.-..--...-.-....--.-..-..-...---.---....--...--.--------.--.-.--..-.---.--..-------.-..----..--..-.-.-.--.-.-...----.-----.--.--.-.-..-.-----...----..--.-.--.-..-.--..-.-..-----.-.-..-.-..----.-.-.--.--..--...-.-.---.-.-.--------.-----.-.--.-.-------.-.-.--------.--...---.-.--.-.----.-..----.-.---.---.----.-.-.---.----.-.---.--.-.-..-----.-.----.-.--..-.-.-.---.-.-------..--.-...-..-.--...-------.--..------.-..--.--.-.-.---.-.--.-..---..-.-.-..----.-...----.----.-.-.---.-..----.--.-.---.--..--..--..------.-------..--.--.-..-..--.----.--..---..-.---.-.---.-..-....---..--.-.--.-.--.--.--.-.--....---.---.-..-..---.-...---..--..--.-.---.-.---..--.-.--.----.-...-.-------.----.--..-.-...---...--.-----.--.--.----.--..--...---.-.---..---...--.-------.-.-.-...-.----.--..-.-.-.-..-----...-..----..-.------..-.-..-.-.--..--.---..-..--.---...-..------..--..-----.----.--..-.-.---.-.-..-..-.-.-.-----..-.-.---...--.-...---.-.-..--.-.--.----.--..---.....--"
random_string = '_3]|4twR><0<Y[&snV;j.hFfTqgpv]"&SNFgpA7<Z~"cpP6NfI*M;[mtzy/B'
for start in range(len(random_string) - 9):
    flag=decode_flag(cipher,random_string[start:start+8])
    if "ebhadra" in flag:
        print(flag)
        break34
```

flow of decryption script :

  1. Convert the cipher from binary format to ASCII characters.
  2. Iterate through each character in the cipher.
  3. Obtain the corresponding character from the random string.
  4. Perform reverse operations to obtain the original flag character.
  5. Repeat steps 2–4 until all cipher characters are processed.
  6. Check for every 8th character to cycle through the random string.

Got the flag.

## V. Cave Python
Points: 175
Category: misc
Challenge Description: Oh no! The python fell into a cave. Can you help him make his escape?


![image_of_challege_5](/assets/img/2024-02-19-my-approach-and-solutions-navigating-the-ebhadra-ctf-challenge/27.webp)


![image_of_challege_5](/assets/img/2024-02-19-my-approach-and-solutions-navigating-the-ebhadra-ctf-challenge/28.webp)

![image_of_challege_5](/assets/img/2024-02-19-my-approach-and-solutions-navigating-the-ebhadra-ctf-challenge/29.webp)


When we test different inputs, we notice that some are allowed while others cause Python errors. By testing each letter and special character one by one, we figure out that the challenge only accepts specific ones.

Whitelists characters : [a, c, h, i, n, r, s, t, (, ), [, ], +, -, /, ‘, ‘, ]

Now, we can try using functions like chr() and int() within these allowed characters.

![image_of_challege_5](/assets/img/2024-02-19-my-approach-and-solutions-navigating-the-ebhadra-ctf-challenge/30.webp)


It appears that the system is attempting to evaluate our input if it’s accepted. Additionally, we’re unable to utilize numbers, which limits our use of chr() function. However, we still have access to the following functions:

  1. ascii
  2. chr
  3. hash
  4. int
  5. str

The hash function is interesting because it produces large integers when given function names as input. These integers represent the memory addresses of those functions.

Using hash(), we can obtain integers representing ‘1’ and ‘0’. We can then use these with int() to convert them into their respective ASCII code points. By utilizing chr(), we can convert these code points back into characters. This allows us to create unrestricted input for the second eval. Here’s the plan:

  1. Use int() with ‘1’ and ‘0’ to get ASCII code points in binary format.
  2. Convert these code points to characters using chr().
  3. Pass these characters as input to the next eval.

The script below demonstrates this process.
```python
from pwn import *

r = remote("20.235.243.131",13337)
r.recvuntil(b'>')

def lvl1(inp):
 word=""
 for i in inp:
  char = "ascii(int())"
  b = bin(ord(i))[2:]
  for bit in b:
   if bit == '1':
    char += '+ascii(hash(())//hash(()))'
   elif bit == '0':
    char += '+ascii(int())'
  word += f"chr(int({char},(hash(())//hash(())+hash(())//hash(()))))+"

 print(eval(word[:-1]))
 
 return word[:-1].encode()

r.sendline(lvl1("print(eval())"))
r.interactive()
```
let’s investigate the ‘globals’ array

![image_of_challege_5](/assets/img/2024-02-19-my-approach-and-solutions-navigating-the-ebhadra-ctf-challenge/31.webp)

Hmm, there’s a suspicious function called ‘blackflag’. Let’s attempt to execute it…

![image_of_challege_5](/assets/img/2024-02-19-my-approach-and-solutions-navigating-the-ebhadra-ctf-challenge/32.webp)


Got the flag.
## Note on other challenges
Due to technical problems of the Welcoming Hackers As A Service bot and time constraints, I was unable to fully solve the following two challenges:

  1. Welcoming Hackers As A Service
  2. Do You Know RSA?

Suggestions are most welcome as always. I will try to keep posting my findings. If you got anything from it, you can press the clap icon below, and don’t forget to follow me on [Twitter](https://twitter.com/padsalatushal) & [LinkedIn](https://www.linkedin.com/in/padsalatushal/) as well. See you all next time. :)


