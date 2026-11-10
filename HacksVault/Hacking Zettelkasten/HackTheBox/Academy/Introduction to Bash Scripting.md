# Script Execution - Examples

```bash
mdn0x@htb[/htb]$ bash script.sh <optional arguments>
```

```bash
mdn0x@htb[/htb]$ sh script.sh <optional arguments>
```

```bash
mdn0x@htb[/htb]$ ./script.sh <optional arguments>
```

## CIDR.sh

```bash
#!/bin/bash

# Check for given arguments
if [ $# -eq 0 ]
then
    echo -e "You need to specify the target domain.\n"
    echo -e "Usage:"
    echo -e "\t$0 <domain>"
    exit 1
else
    domain=$1
fi

# Identify Network range for the specified IP address(es)
function network_range {
    for ip in $ipaddr
    do
        netrange=$(whois $ip | grep "NetRange\|CIDR" | tee -a CIDR.txt)
        cidr=$(whois $ip | grep "CIDR" | awk '{print $2}')
        cidr_ips=$(prips $cidr)
        echo -e "\nNetRange for $ip:"
        echo -e "$netrange"
    done
}

# Ping discovered IP address(es)
function ping_host {
    hosts_up=0
    hosts_total=0
    
    echo -e "\nPinging host(s):"
    for host in $cidr_ips
    do
        stat=1
        while [ $stat -eq 1 ]
        do
            ping -c 2 $host > /dev/null 2>&1
            if [ $? -eq 0 ]
            then
                echo "$host is up."
                ((stat--))
                ((hosts_up++))
                ((hosts_total++))
            else
                echo "$host is down."
                ((stat--))
                ((hosts_total++))
            fi
        done
    done
    
    echo -e "\n$hosts_up out of $hosts_total hosts are up."
}

# Identify IP address of the specified domain
hosts=$(host $domain | grep "has address" | cut -d" " -f4 | tee discovered_hosts.txt)

echo -e "Discovered IP address:\n$hosts\n"
ipaddr=$(host $domain | grep "has address" | cut -d" " -f4 | tr "\n" " ")

# Available options
echo -e "Additional options available:"
echo -e "\t1) Identify the corresponding network range of target domain."
echo -e "\t2) Ping discovered hosts."
echo -e "\t3) All checks."
echo -e "\t*) Exit.\n"

read -p "Select your option: " opt

case $opt in
    "1") network_range ;;
    "2") ping_host ;;
    "3") network_range && ping_host ;;
    "*") exit 0 ;;
esac
```

As we can see, we have commented here several parts of the script into which we can split it.

1. Check for given arguments
2. Identify network range for the specified IP address(es)
3. Ping discovered IP address(es)
4. Identify IP address(es) of the specified domain
5. Available options

#  Working Components
## Conditional Execution

In summary, this code section works with the following components:

- `#!/bin/bash` - Shebang.
- `if-else-fi` - Conditional execution.
- `echo` - Prints specific output.
- `$#` / `$0` / `$1` - Special variables.
- `domain` - Variables.
### Exercise Script

Create an "If-Else" condition in the "For"-Loop of the "Exercise Script" that prints you the number of characters of the 35th generated value of the variable "var". Submit the number as the answer.

```bash
#!/bin/bash
# Count number of characters in a variable:
#     echo $variable | wc -m

# Variable to encode
var="nef892na9s1p9asn2aJs71nIsm"

for counter in {1..40}
do
        var=$(echo $var | base64)
    if [ "$counter" -eq 35 ]; then
        echo "$var" | wc -m
    fi
done
```

```bash
└─$ bash script.sh 
1197735
```

## Arguments, Variables, and Arrays
### Arguments

The advantage of bash scripts is that we can always pass up to 9 arguments (`$0`-`$9`) to the script without assigning them to variables or setting the corresponding requirements for these. `9 arguments` because the first argument `$0` is reserved for the script. As we can see here, we need the dollar sign (`$`) before the name of the variable to use it at the specified position. The assignment would look like this in comparison:

```bash
mdn0x@htb[/htb]$ ./script.sh ARG1 ARG2 ARG3 ... ARG9
       ASSIGNMENTS:       $0      $1   $2   $3 ...   $9
```

This means that we have automatically assigned the corresponding arguments to the predefined variables in this place. These variables are called special variables. These special variables serve as placeholders. If we now look at the code section again, we will see where and which arguments have been used.
###  Special Variables

| **Special Variable** | **Description**                                                                                                                                                         |
| -------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `$#`                 | This variable holds the number of arguments passed to the script.                                                                                                       |
| `$@`                 | This variable can be used to retrieve the list of command-line arguments.                                                                                               |
| `$n`                 | Each command-line argument can be selectively retrieved using its position. For example, the first argument is found at `$1`.                                           |
| `$$`                 | The process ID of the currently executing process.                                                                                                                      |
| `$?`                 | The exit status of the script. This variable is useful to determine a command's success. The value 0 represents successful execution, while 1 is a result of a failure. |
### Variables

We also see at the end of the if-else loop that we assign the value of the first argument to the variable called "`domain`". The assignment of variables takes place without the dollar sign (`$`). The dollar sign is only intended to allow this variable's corresponding value to be used in other code sections. When assigning variables, there must be no spaces between the names and values.

In contrast to other programming languages, there is no direct differentiation and recognition between the types of variables in Bash like "`strings`," "`integers`," and "`boolean`." All contents of the variables are treated as string characters. Bash enables arithmetic functions depending on whether only numbers are assigned or not. It is important to note when declaring variables that they do `not` contain a `space`. Otherwise, the actual variable name will be interpreted as an internal function or a command.

```bash
mdn0x@htb[/htb]$ variable="Declared without an error."
mdn0x@htb[/htb]$ echo $variable

Declared without an error.
```

### Arrays

There is also the possibility of assigning several values to a single variable in Bash. This can be beneficial if we want to scan multiple domains or IP addresses. These variables are called `arrays` that we can use to store and process an ordered sequence of specific type values. `Arrays` identify each stored entry with an `index` starting with `0`. When we want to assign a value to an array component, we do so in the same way as with standard shell variables. All we do is specify the field index enclosed in square brackets. The declaration for `arrays` looks like this in Bash:

```bash
domains=(www.inlanefreight.com ftp.inlanefreight.com vpn.inlanefreight.com www2.inlanefreight.com)

echo ${domains[0]}
```

It is important to note that single quotes (`'` ... `'`) and double quotes (`"` ... `"`) prevent the separation by a space of the individual values in the array. This means that all spaces between the single and double quotes are ignored and handled as a single value assigned to the array.

#### Exercise

Submit the echo statement that would print "www2.inlanefreight.com" when running the last "Arrays.sh" script.

```bash
echo ${domains[1]}
```
## Comparison Operators

To compare specific values with each other, we need elements that are called [comparison operators](https://www.tldp.org/LDP/abs/html/comparison-ops.html). The `comparison operators` are used to determine how the defined values will be compared. For these operators, we differentiate between:

- `string` operators
- `integer` operators
- `file` operators
- `boolean` operators

It is important to note here that we put the variable for the given argument (`$1`) in double-quotes (`"$1"`). This tells Bash that the content of the variable should be handled as a string.
### Exercise Script

 Create an "If-Else" condition in the "For"-Loop that checks if the variable named "var" contains the contents of the variable named "value". Additionally, the variable "var" must contain more than 113,450 characters. If these conditions are met, the script must then print the last 20 characters of the variable "var". Submit these last 20 characters as the answer.

```bash
#!/bin/bash

var="8dm7KsjU28B7v621Jls"
value="ERmFRMVZ0U2paTlJYTkxDZz09Cg"

for i in {1..40}
do
    var=$(echo "$var" | base64)

    if [[ "$var" == *"$value"* ]] && [[ ${#var} -gt 113450 ]]; then
        echo "$var" | tail -c 20
    fi
done
```
## Arithmetic

We can summarize all the operators in a small script:

```bash
#!/bin/bash

increase=1
decrease=1

echo "Addition: 10 + 10 = $((10 + 10))"
echo "Subtraction: 10 - 10 = $((10 - 10))"
echo "Multiplication: 10 * 10 = $((10 * 10))"
echo "Division: 10 / 10 = $((10 / 10))"
echo "Modulus: 10 % 4 = $((10 % 4))"

((increase++))
echo "Increase Variable: $increase"

((decrease--))
echo "Decrease Variable: $decrease"
```

```bash
mdn0x@htb[/htb]$ ./Arithmetic.sh

Addition: 10 + 10 = 20
Subtraction: 10 - 10 = 0
Multiplication: 10 * 10 = 100
Division: 10 / 10 = 1
Modulus: 10 % 4 = 2
Increase Variable: 2
Decrease Variable: 0
```

If we look at our `CIDR.sh` script, we will see that we have used the `increase` and `decrease` operators several times. This ensures that the while loop, which we will discuss later, runs and pings the hosts while the variable "`stat`" has a value of `1`. If the ping command ends with code `0` (successful), we get a message that the `host is up` and the "`stat`" variable, as well as the variables "`hosts_up`" and "`hosts_total`" get changed.

---
# Script Control
## Input & Output Control

We may get results from our sent requests and executed commands, which we have to decide manually on how to proceed. Another example would be that we have defined several functions in our script designed for different scenarios. We have to decide which of them should be executed after a manual check and based on the results. It is also quite possible that specific scans or activities may not be allowed to be performed. Therefore, we need to be familiar with how to get a running script to wait for our instructions. If we look at our `CIDR.sh` script again, we see that we have added such a call to decide further steps.

The first `echo` lines serve as a display menu for the options available to us. With the `read` command, the line with "`Select your option:`" is displayed, and the additional option `-p` ensures that our input remains on the same line. Our input is stored in the variable `opt`, which we then use to execute the corresponding functions with the `case` statement, which we will look at later. Depending on the number we enter, the `case` statement determines which functions are executed.

We have already learned about the output redirections of output in the `Linux Fundamentals` module. Nevertheless, the problem with the redirections is that we do not get any output from the respective command. It will be redirected to the appropriate file. If our scripts become more complicated later, they can take much more time than just a few seconds. To avoid sitting inactively and waiting for our script's results, we can use the [tee](https://man7.org/linux/man-pages/man1/tee.1.html) utility. It ensures that we see the results we get immediately and that they are stored in the corresponding files. In our `CIDR.sh` script, we have used this utility twice in different ways.

When using `tee`, we transfer the received output and use the pipe (`|`) to forward it to `tee`. The "`-a` / `--append`" parameter ensures that the specified file is not overwritten but supplemented with the new results. At the same time, it shows us the results and how they will be found in the file.
## Flow Control - Loops

The control of the flow of our scripts is essential. We have already learned about the `if-else` conditions, which are also part of flow control. After all, we want our script to work quickly and efficiently, and for this, we can use other components to increase efficiency and allow error-free processing. Each control structure is either a `branch` or a `loop`. Logical expressions of boolean values usually control the execution of a control structure. These control structures include:

- Branches:
    - `If-Else` Conditions
    - `Case` Statements
- Loops:
    - `For` Loops
    - `While` Loops
    - `Until` Loops
### Exercise Script

Create a "For" loop that encodes the variable "var" 28 times in "base64". The number of characters in the 28th hash is the value that must be assigned to the "salt" variable.

```bash
#!/bin/bash

# Decrypt function
function decrypt {
    MzSaas7k=$(echo $hash | sed 's/988sn1/83unasa/g')
    Mzns7293sk=$(echo $MzSaas7k | sed 's/4d298d/9999/g')
    MzSaas7k=$(echo $Mzns7293sk | sed 's/3i8dqos82/873h4d/g')
    Mzns7293sk=$(echo $MzSaas7k | sed 's/4n9Ls/20X/g')
    MzSaas7k=$(echo $Mzns7293sk | sed 's/912oijs01/i7gg/g')
    Mzns7293sk=$(echo $MzSaas7k | sed 's/k32jx0aa/n391s/g')
    MzSaas7k=$(echo $Mzns7293sk | sed 's/nI72n/YzF1/g')
    Mzns7293sk=$(echo $MzSaas7k | sed 's/82ns71n/2d49/g')
    MzSaas7k=$(echo $Mzns7293sk | sed 's/JGcms1a/zIm12/g')
    Mzns7293sk=$(echo $MzSaas7k | sed 's/MS9/4SIs/g')
    MzSaas7k=$(echo $Mzns7293sk | sed 's/Ymxj00Ims/Uso18/g')
    Mzns7293sk=$(echo $MzSaas7k | sed 's/sSi8Lm/Mit/g')
    MzSaas7k=$(echo $Mzns7293sk | sed 's/9su2n/43n92ka/g')
    Mzns7293sk=$(echo $MzSaas7k | sed 's/ggf3iunds/dn3i8/g')
    MzSaas7k=$(echo $Mzns7293sk | sed 's/uBz/TT0K/g')

    flag=$(echo $MzSaas7k | base64 -d | openssl enc -aes-128-cbc -a -d -salt -pass pass:$salt)
}

# Variables
var="9M"
salt=""
hash="VTJGc2RHVmtYMTl2ZnYyNTdUeERVRnBtQWVGNmFWWVUySG1wTXNmRi9rQT0K"

# Base64 encoding 28 times
for i in {1..28}
do
    var=$(echo "$var" | base64)
done

# Calculate salt
salt=$(echo "$var" | wc -m)

# Check if salt is empty
if [[ ! -z "$salt" ]]
then
    decrypt
    echo "$flag"
else
    exit 1
fi

└─$ bash decrypt.sh 
*** WARNING : deprecated key derivation used.
Using -iter or -pbkdf2 would be better.
HTBL00p5r0x
```
## Flow Control - Branches

As we have already seen, the branches in flow control include `if-else` and the `case` statements. We have already discussed the `if-else` statements in detail and know how this works. Now we will take a closer look at the case statements.
### Case Statements

`Case` statements are also known as `switch-case` statements in other languages, such as C/C++ and C#. The main difference between `if-else` and `switch-case` is that `if-else` constructs allow us to check any boolean expression, while `switch-case` always compares only the variable with the exact value. Therefore, the same conditions as for `if-else`, such as "greater-than," are not allowed for `switch-case`. The syntax for the switch-case statements looks like this:

```bash
case <expression> in
    pattern_1 ) statements ;;
    pattern_2 ) statements ;;
    pattern_3 ) statements ;;
esac
```

The definition of switch-case starts with `case`, followed by the variable or value as an expression, which is then compared in the pattern. If the variable or value matches the expression, then the statements are executed after the parenthesis and ended with a double semicolon (`;;`).
# Execution Flow
## Functions

The bigger our scripts get, the more chaotic they become. If we use the same routines several times in the script, the script's size will increase accordingly. In such cases, `functions` are the solution that improves both the size and the clarity of the script many times. We combine several commands in a block between curly brackets ( `{` ... `}` ) and call them with a function name defined by us with `functions`. Once a function has been defined, it can be called and used again during the script.

`Functions` are an essential part of scripts and programs, as they are used to execute recurring commands for different values and phases of the script or program. Therefore, we do not have to repeat the whole section of code repeatedly but can create a single function that executes the specific commands. The definition of such functions makes the code easier to read and helps to keep the code as short as possible. It is important to note that functions must always be defined logically `before` the first call since a script is also processed from top to bottom. Therefore the definition of a function is always `at the beginning` of the script. There are two methods to define the functions:

```bash
Method 1:
function name {     <commands> }

Method 2:
name() {     <commands> }
```
### Parameter Passing

Such functions should be designed so that they can be used with a fixed structure of the values or at least only with a fixed format. Like we have already seen in our `CIDR.sh` script, we used the format of an IP address for the function "`network_range`". The parameters are optional, and therefore we can call the function without parameters. In principle, the same applies to the passed parameters as to parameters passed to a shell script. These are `$1` - `$9` (`${n}`), or `$variable` as we have already seen. Each function has its own set of parameters. So they do not collide with those of other functions or the parameters of the shell script.

An important difference between bash scripts and other programming languages is that all defined variables are always processed `globally` unless otherwise declared by "[local](https://www.tldp.org/LDP/abs/html/localvar.html)." This means that the first time we have defined a variable in a function, we will call it in our main script (outside the function). Passing the parameters to the functions is done the same way as we passed the arguments to our script and looks like this:

```bash
#!/bin/bash

function print_pars {
    echo $1 $2 $3
}

one="First parameter"
two="Second parameter"
three="Third parameter"

print_pars "$one" "$two" "$three"
```
### Return Values

When we start a new process, each `child process` (for example, a `function` in the executed script) returns a `return code` to the `parent process` (`bash shell` through which we executed the script) at its termination, informing it of the status of the execution. This information is used to determine whether the process ran successfully or whether specific errors occurred. Based on this information, the `parent process` can decide on further program flow.

| **Return Code** | **Description**                |
| --------------- | ------------------------------ |
| `1`             | General errors                 |
| `2`             | Misuse of shell builtins       |
| `126`           | Command invoked cannot execute |
| `127`           | Command not found              |
| `128`           | Invalid argument to exit       |
| `128+n`         | Fatal error signal "`n`"       |
| `130`           | Script terminated by Control-C |
| `255\*`         | Exit status out of range       |
To get the value of a function back, we can use several methods like `return`, `echo`, or a `variable`. In the next example, we will see how to use "`$?`" to read the "`return code`," how to pass the arguments to the function and how to assign the result to a variable.

```bash
#!/bin/bash

function given_args {

        if [ $# -lt 1 ]
        then
                echo -e "Number of arguments: $#"
                return 1
        else
                echo -e "Number of arguments: $#"
                return 0
        fi
}

# No arguments given
given_args
echo -e "Function status code: $?\n"

# One argument given
given_args "argument"
echo -e "Function status code: $?\n"

# Pass the results of the funtion into a variable
content=$(given_args "argument")

echo -e "Content of the variable: \n\t$content"
```

Execution:

```bash
mdn0x@htb[/htb]$ ./Return.sh

Number of arguments: 0
Function status code: 1

Number of arguments: 1
Function status code: 0

Content of the variable:
    Number of arguments: 1
```
# Debugging

Bash gives us an excellent opportunity to find, track, and fix errors in our code. The term `debugging` can have many different meanings. Nevertheless, [Bash debugging](https://tldp.org/LDP/Bash-Beginners-Guide/html/sect_02_03.html) is the process of removing errors (bugs) from our code. Debugging can be performed in many different ways. For example, we can use our code for debugging to check for typos, or we can use it for code analysis to track them and determine why specific errors occur.

This process is also used to find vulnerabilities in programs. For example, we can try to cause errors using different input types and track their handling in the CPU through the assembler, which may provide a way to manipulate the handling of these errors to insert our own code and force the system to execute it. This topic will be covered and discussed in detail in other modules. Bash allows us to debug our code by using the "`-x`" (`xtrace`) and "`-v`" options. Now let us see an example with our `CIDR.sh` script.

```
mdn0x@htb[/htb]$ bash -x CIDR.sh
```

**Verbose Debugging**

```bash
mdn0x@htb[/htb]$ bash -x -v CIDR.sh
```

n comparison to normal debugging, we see the entire code section that has been processed so far and then the individual steps that have been taken.

---


