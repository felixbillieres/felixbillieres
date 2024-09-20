# 🍯 Environment-based Privilege Escalation

[PATH](http://www.linfo.org/path\_env\_var.html) is an environment variable that specifies the set of directories where an executable can be located. An account's PATH variable is a set of absolute paths, allowing a user to type a command without specifying the absolute path to the binary.

```shell-session
echo $PATH
```

Creating a script or program in a directory specified in the PATH will make it executable from any directory on the system.
