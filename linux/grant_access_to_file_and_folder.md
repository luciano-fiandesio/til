# Grant access to file and folders for multiple users

In Linux, files and directories have permission sets such as _owner_ (owner or user of the file), _group_ (associated group) and others. However, these permission sets have limitations and doesn’t allow users to set different permissions to different users.

If a user other than the _owner_ needs to access a file or folder (for read, write or both), *ACL* are the list of additional user/groups and their permission to the file. 

Examples:

Grants RW access to user john

```
# setfacl -m u:john:rw /tmp/test
```

Recursive

```
# setfacl -R -m u:john:rw /tmp/test
```

Grant access to group

```
# setfacl -m g:admins:rw /tmp/test
```



