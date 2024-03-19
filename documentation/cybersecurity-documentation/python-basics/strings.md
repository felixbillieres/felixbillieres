# 🔠 Strings

## Python Strings for Beginners 🐍🔤

### Creation:

* Example: `my_string = 'Hello, World!'` or `my_string = "Hello, World!"`

### Accessing Characters:

* You can access individual characters within a string using indexing, starting from 0.
  * Example: `print(my_string[0])` would output 'H'.

### String Concatenation:

* You can concatenate (join) two or more strings using the `+` operator.
  * Example: `greeting = 'Hello' + ' ' + 'World!'` would result in 'Hello World!'.

### String Length:

* The `len()` function can be used to determine the length (number of characters) of a string.
  * Example: `print(len(my_string))` would output the length of the string.

### String Slicing:

* You can extract a substring from a string using slicing, specifying the start and end indices.
  * Example: `substring = my_string[7:12]` would extract the substring 'World'.

### String Methods:

* Python provides various built-in methods to manipulate and transform strings.
  * Examples include `upper()`, `lower()`, `strip()`, `split()`, `replace()`, `find()`, `count()`, and more.
  * Example: `print(my_string.upper())` would output 'HELLO, WORLD!'.

### String Formatting:

* String formatting allows you to embed values within a string. This can be done using the `%` operator or the `format()` method.
  * Example
* ```python
  name = 'Alice'
  age = 30
  print("My name is %s and I'm %d years old." % (name, age))
  # Output: My name is Alice, and I'm 30 years old.
  ```

### Escape Characters:

* Strings can include special characters using escape sequences. Common escape characters include  (newline),  (tab), `\\` (backslash), etc.

```python
escaped_string = "This is a multi-line string.\nIt spans multiple lines using the escape character."
```
