---
title: Hack the box SecretRezipe
date: 2023-11-26 00:00:01 +0000
categories: [hack the box, challenge]
tags: [hack the box, challenge, misc, htb, SecretRezipe, RSA]
img_path: /assets/img/posts
---

# Challenge description

> We have launched a startup that produces soft drinks. We use special ingredients to make them very tasty, so we have a lot of protections on our files to prevent our competitors from copying our ideas.

# The web

If we spawn the system, and browse to it, we will find a drinks website called "Refreshing startup".

![Refreshing startup web](secret-rezipe-refreshing-startup-web.png){: width="700" height="600" .shadow}
*Refreshing startup web*

The only thing that we can do is suggest a recipe and download it.

![Suggestions](secret-rezipe-suggestions.png){: width="700" height="600" .shadow}
*Suggestions*

But, the zip file will be password protected, and we don't have the password.

# The files

When we extract the `SecretRezipe.zip` file, we found the `misc_secret_rezipe` folder.
This folder contains a docker container configuration and source files.
So, this will be a grey box approach.
And, since the objective is not the web, let's analyze these files.
After a while analyzing files, we will have found a pair of interesting things:

## route.js

```javascript
const { FLAG, PASSWORD } = require('./config/config')
```

`FLAG` and `PASSWORD` constants requires './config/config', we will check this file later.

```javascript
let data = `Secret: ${FLAG}`
```

`data` will always start with "Secret: ${FLAG}".

```javascript
if (req.body.ingredients) {
  data += `\n${req.body.ingredients}`
}
```

If there are no ingredients, nothing will be appended to `data`.

```javascript
  const tempPath = os.tmpdir() + '/' + crypto.randomBytes(16).toString('hex')
  fs.mkdirSync(tempPath);
  fs.writeFileSync(tempPath + '/ingredients.txt', data)
  child_process.execSync(`zip -P ${PASSWORD} ${tempPath}/ingredients.zip ${tempPath}/ingredients.txt`)
  return res.sendFile(tempPath + '/ingredients.zip')
```

The content of the `data` variable will be written to the `ingredients.txt` file in a random temporary directory.
The the temporary directory will be zipped using the `${PASSWORD}` constant as password.
And, finally, the zipped file will be sent.

## config.js

```javascript
try {
     var FLAG = fs.readFileSync("/flag.txt")
} catch (e) {
    var FLAG = "HTB{fake_flag_for_testing}"
}
```

The flag is read from the `flag.txt` file.
And, obviously, "HTB{fake_flag_for_testing}" is a bait.

```javascript
PASSWORD: crypto.randomUUID()
```

Ok. So the password is... A random [UUID](https://en.wikipedia.org/wiki/Universally_unique_identifier)...
As the UUID (or Universally Unique Identifier) is 128 bit label, if we want to bruteforce it we will have to try 2^128 combinations...
So much time for this life.

# ingredients.zip

How we have ruled out brute forde (or I hope so...), let's analyze the recipe, the zip file:

```
$ 7z l -slt ingredients.zip
...
Path = tmp/ccc1959a8a4ed397b8ef86e26aba47f0/ingredients.txt
...
Method = ZipCrypto Store
```

A simple password-based symmetric encryption system generally known as ZipCrypto. It is documented in the ZIP specification, and known to be seriously flawed. In particular, it is vulnerable to known-plaintext attacks,

`ZipCrypto` is a simple password-based symetric encryption system known to be seriously flawled.
And, in particular is vulnerable to kpa (or known-plaintext attacks).
I have already talked about the KPAs here, in the post [XOR known-plaintext attack](https://rubenhortas.github.io/posts/xor-known-plaintext-attack), so I won't stop at this and we will intent exploit this vulnerability.

# KPA (kwnown-plaintext attack)

The first thing we will do is generate an empty recipe.
We only want the flag, so wil be enough.
And, then, we will attack the `ingredients.zip` file using [bkcrack](https://github.com/kimci86/bkcrack).
The attack will require at least 12 bytes of known plaintext, and at least 8 of them must be contiguous.

So, what we do know about our flag?
As we have seen in the `route.js`file, the flag will starts by "Secret: ".
And we known that all flags in [hack the box](https://www.hackthebox.com) starts with "HTB{".
So, we will create the `plaintext.txt` file containing only the string "Secret: HTB{".

> I checked this file with a hexadecimal editor to make sure that there was not another characters in the file. Some editor added new characters at the end, and, if there are other characters at the end, the attack will not work.
{: .prompt-warning}

Now, we launch the attack:

```
$ bkcrack -C ingredients.zip -c tmp/ccc1959a8a4ed397b8ef86e26aba47f0/ingredients.txt -p plaintext.txt
````

After a while, and, if everithing went well, we will see the keys:

```
[00:00:00] Keys
104d144a d3d97394 18e6b1a5
```

Now, we can get our flag:

```
$ bkcrack -C ingredients.zip -c tmp/ccc1959a8a4ed397b8ef86e26aba47f0/ingredients.txt -k KEYS -f decrypted_file.txt
```

![Challenge results](owned-secretrezipe.png){: width="700" height="600" .shadow}
*SecretRezipe has been Pwned*

*Enjoy! ;)*
