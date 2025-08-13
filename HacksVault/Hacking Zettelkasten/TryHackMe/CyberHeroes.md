- - - 
created : 17-07-2025 

Tags : #easy  
- - - 
# Objective

Catch the flag.

# Enumeration

let’s view the page source code in /login :

![fbbcd8556d9159dfd7b3bd309da0f30d.png](../../2%20-%20Resources/Others/Flameshots/fbbcd8556d9159dfd7b3bd309da0f30d.png)

This is where the whole thing comes into play :

```
a = document.getElementById('uname')
b = document.getElementById('pass')
```

In this section, the username and password we entered in the form are taken.

then here it is checked whether the uname (i.e. username) is equal to “h3ck3rBoi” and the pass (i.e. password) is equal to the opposite of “54321@terceSrepuS”, i.e. “<span style="color: rgb(45, 194, 107);">**SuperSecret@12345**</span>”.

```
f (a.value=="h3ck3rBoi" & b.value==RevereString("54321@terceSrepuS"))
```

So, the username we need to log in is h3ck3rBoi and the password is SuperSecret@12345. Now let’s try logging in with this credential :

![8d4cb4911ea6f07914380ebc15bddc52.png](../../2%20-%20Resources/Others/Flameshots/8d4cb4911ea6f07914380ebc15bddc52.png)

Pwned !!