# 🍷 Reading and Writing Files

## Navigating the World of Files in Python 📂🐍

### Introduction:

Reading and writing files in Python are fundamental operations for handling data persistence. Python provides simple yet powerful tools to interact with files, allowing you to read information from existing files or create new ones.

### Reading Files:

Reading from a file is a common task in Python. Here's a basic example:

```python
# Reading a File
file_path = 'sample.txt'

try:
    with open(file_path, 'r') as file:
        content = file.read()
        print("File Content:\n", content)
except FileNotFoundError:
    print(f"The file '{file_path}' does not exist.")
except Exception as e:
    print(f"An error occurred: {e}")
```

In this example, we use the `open()` function with the 'r' (read) mode to open the file and then read its content. The `with` statement ensures proper file closure.

### Writing to Files:

Creating or modifying files is equally straightforward:

```python
# Writing to a File
output_path = 'output.txt'

try:
    with open(output_path, 'w') as output_file:
        output_file.write("Hello, File!\nThis is a new line.")
    print(f"Content written to '{output_path}'.")
except Exception as e:
    print(f"An error occurred: {e}")
```

Here, we open a file in 'w' (write) mode, allowing us to write content. The existing content, if any, will be overwritten. Use 'a' mode for appending.

### Working with File Lines:

Files often contain multiple lines. Here's how to read and write line by line:

```python
# Reading and Writing Lines
file_path = 'lines.txt'

try:
    with open(file_path, 'r') as file:
        lines = file.readlines()
        print("Lines in File:")
        for line in lines:
            print(line.strip())

    with open('new_lines.txt', 'w') as new_file:
        new_file.write("New Line 1\nNew Line 2\n")
    print("Lines written to 'new_lines.txt'.")
except FileNotFoundError:
    print(f"The file '{file_path}' does not exist.")
except Exception as e:
    print(f"An error occurred: {e}")
```

Here, `readlines()` returns a list of lines. Writing new lines involves providing a string with newline characters or using multiple `write()` calls.

### Practical Use: CSV Handling:

Working with CSV (Comma-Separated Values) files is common in data processing. Let's read and write CSV files:

```python
import csv

# Reading and Writing CSV
csv_path = 'data.csv'

try:
    with open(csv_path, 'r') as csv_file:
        csv_reader = csv.reader(csv_file)
        print("CSV Content:")
        for row in csv_reader:
            print(row)

    with open('new_data.csv', 'w', newline='') as new_csv:
        csv_writer = csv.writer(new_csv)
        csv_writer.writerow(['Name', 'Age', 'City'])
        csv_writer.writerow(['Alice', '28', 'Wonderland'])
    print("Data written to 'new_data.csv'.")
except FileNotFoundError:
    print(f"The file '{csv_path}' does not exist.")
except Exception as e:
    print(f"An error occurred: {e}")
```

Python's `csv` module simplifies CSV operations, ensuring proper handling of commas and newlines.
