# 1️⃣ Bash Scripting

## Bash Scripting for Beginners

### Introduction

Bash scripting is a fundamental skill for Linux and Unix system administrators, allowing them to automate repetitive tasks, create complex workflows, and efficiently manage systems. This section provides additional content for beginners looking to enhance their understanding of bash scripting.

### Variables and Data Types

In bash scripting, variables are used to store and manipulate data. Here's a quick overview:

```bash
name="John"
age=25
```

* **String Variable (`name`):** Holds textual data.
* **Numeric Variable (`age`):** Stores numerical values.

To access the values:

```bash
echo "Name: $name"
echo "Age: $age"
```

### Conditional Statements

Conditional statements allow you to make decisions in your scripts. The `if`, `else`, and `elif` statements are essential:

```bash
if [ "$age" -lt 18 ]; then
  echo "You are a minor."
elif [ "$age" -ge 18 ] && [ "$age" -lt 65 ]; then
  echo "You are an adult."
else
  echo "You are a senior."
fi
```

* **`-lt`:** Less than
* **`-ge`:** Greater than or equal to

### Loops

Loops are used to iterate over a sequence of values. The `for` and `while` loops are commonly used:

```bash
# For loop
for i in {1..5}; do
  echo "Iteration: $i"
done

# While loop
counter=0
while [ $counter -lt 5 ]; do
  echo "Counter: $counter"
  ((counter++))
done
```

### Functions

Functions allow you to group code into reusable blocks:

```bash
greet() {
  echo "Hello, $1!"
}

greet "Alice"
```

In this example, the `greet` function takes an argument and prints a personalized greeting.

### Input from Users

Accepting input from users is crucial for interactive scripts. Use the `read` command:

```bash
echo "What's your name?"
read userName
echo "Hello, $userName!"
```

The `read` command waits for user input and stores it in the specified variable (`userName`).

### Scripting Best Practices

1. **Comment Your Code:** Add comments to explain complex sections and improve readability.
2. **Indentation:** Maintain consistent indentation to enhance code structure.
3. **Error Handling:** Implement proper error handling to gracefully manage unexpected scenarios.
4. **Modularization:** Break down your scripts into functions for better organization and reusability.
5. **Testing:** Test your scripts in a safe environment before deploying them in a production setting.

### Scripting Data Collection from an IP Scan

#### 1. Save Ping Output to a File

To collect data from a ping command and save it to a file, use the following command:

```bash
ping 10.10.2.15 -c 1 > ip.txt
```

Explanation:

* **`ping 10.10.2.15`**: Sends a single network ping to the IP address 10.10.2.15.
* **`-c 1`**: Specifies to send only one ping.
* **`> ip.txt`**: Redirects the output to a file named "ip.txt."

#### 2. Extract and Process Specific Information

To filter and process specific information from the ping output, use the following command:

```bash
cat ip.txt | grep "PING" | cut -d " " -f 4 | tr -d ":"
```

Explanation:

* **`cat ip.txt`**: Displays the contents of the "ip.txt" file.
* **`|`**: The pipe symbol takes the output of the command on its left and uses it as the input for the command on its right.
* **`grep "PING"`**: Filters lines containing the string "PING" from the output.
* **`cut -d " " -f 4`**: Splits each line into fields based on the space character, then selects the fourth field.
* **`tr -d ":"`**: Removes any colon characters from the output.

#### 3. Automate Network Scan with a Bash Script

Create a bash script named `ipsweep.sh`:

```bash
#!/bin/bash
if [ "$1" == "" ]; then
  echo "You forgot an IP address!"
  echo "Syntax: ./ipsweep.sh 192.168.1"
else
  for ip in `seq 1 254`; do
    ping -c 1 $1.$ip | grep "64 bytes" | cut -d " " -f 4 | tr -d ":" &
  done
fi
```

Explanation:

* **`#!/bin/bash`**: Specifies the script to be interpreted using the Bash shell.
* **`if [ "$1" == "" ]`**: Checks if the first command-line argument is empty.
* Inside the `if` block:
  * Prints an error message if no command-line argument is provided.
  * Provides the correct syntax for using the script.
* The `else` block begins the execution of the script.
* A loop iterates through IP addresses, pings them, and extracts specific information.

#### 4. Scripting Nmap with IP List

To speed up the scanning process, you can script Nmap using a list of valid IP addresses:

```bash
for ip in $(cat ip.txt); do nmap $ip; done
```

This command reads IP addresses from "ip.txt" and runs Nmap on each of them.

### Resources for Learning Bash Scripting

* [**Bash Guide**](https://guide.bash.academy/)**:** A comprehensive guide for learning bash scripting.
* [**Bash Scripting Tutorial for Beginners**](https://linuxconfig.org/bash-scripting-tutorial-for-beginners)**:** Practical tutorial with examples.
* [**ShellCheck**](https://www.shellcheck.net/)**:** An online tool to check your shell scripts for common issues.

