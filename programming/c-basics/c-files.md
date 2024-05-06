# 📁 C Files

In C, you can create, open, read, and write to files by declaring a [pointer](https://www.w3schools.com/c/c\_pointers.php) of type `FILE`, and use the `fopen()` function:

<figure><img src="../../.gitbook/assets/image (997).png" alt=""><figcaption></figcaption></figure>

```
FILE *fptr
fptr = fopen(filename, mode); 
```

Here are the 3 modes you can specify:

```
w - Writes to a file
a - Appends new data to a file
r - Reads from a file 
```

Here is an example:

```
FILE *fptr;

// Create a file
fptr = fopen("filename.txt", "w");

// Close the file
fclose(fptr); 
```

**Writing to a file:**

if you want to write to a file, you first specify the path and the **W** mode and use fprintf:

```
 FILE *fptr;

// Open a file in writing mode
fptr = fopen("filename.txt", "w");

// Write some text to the file
fprintf(fptr, "Some text");

// Close the file
fclose(fptr); 
```

if there is already content it will overwrite it, if you want to append, you use the **a** mode:

```
// Open a file in append mode
fptr = fopen("filename.txt", "a");

// Append some text to the file
fprintf(fptr, "\nHi everybody!");
```
