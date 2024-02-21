
# POSIX ACL NOTES


## Comment & Disclaimer

These notes were prepared based on my experimenting with the POSIX ACL commands on the KAUST Ibex (linux cluster) HPC environment. 
For a good reference on this topic, I would suggest the following literature, mentioned below. 


## Version Info

setfacl 2.3.1
getfacl 2.3.1


## Reference

1. POSIX Access Control Lists on Linux
https://www.usenix.org/legacy/events/usenix03/tech/freenix03/full_papers/gruenbacher/gruenbacher_html/main.html


## Explanation of ACL Commands

1. To revoke all access...

```bash
   setfacl -b /DataPath/ProjectDir/ # ForTesting
```

2. To set default permissions for a user...

```bash
setfacl -d -m user:xyzuserid:rx /DataPath/ProjectDir/xyzuserid/
```

In the above example please note the difference between the string 'xyzuserid' and directory named 'xyzuserid'.

* However, please note that any entity ( file / directory ) that is moved does not inherit the default ACLs. 
* On the otherhand, if we create a subfolder using mkdir or make a new file using touch or cat, then default ACLs 
are inherited. 
* Also, copying a file retains the default ACL. This should be kept in mind. 

3. How to remove default ACLs?

Consider the following ACLs:

CASE 1: TOP-LEVEL-DIR: On the otherhand, the top level dir would just have the following entry:

```bash
   getfacl ../ProjectDir/testdir/ | grep "xyzuserid"
   default:user:xyzuserid:r-x
```

CASE 2: SUB-DIR: The following is the state of every subdir that was issued -d (default) access from some top-level dir

```bash
   getfacl ../ProjectDir/testdir/n1/n2/n3/ | grep "xyzuserid"
   user:xyzuserid:r-x
   default:user:xyzuserid:r-x
```

Because of cases 1 & 2 mentioned above, we have the following situation while cancelling ACLs. 

3A. The following command would only remove ACL entries tagged "default"

```bash
   setfacl -R -d -x u:xyzuserid ../ProjectDir/testdir/
```

Whereas, user ACLs for the same user would still remain and will not go until the next step.

```bash
   setfacl -R -d -x u:xyzuserid ../ProjectDir/testdir/
   
   getfacl ../ProjectDir/testdir/ | grep "xyzuserid"
   getfacl ../ProjectDir/testdir/n1/ | grep "xyzuserid"
   user:xyzuserid:r-x
```

Note: getfacl command piped wtih grep for the username is used to verify if the ACL-entry for a given username still exists. 

3B. Following command is needed to remove all the user:... ACL entries. For example:

```bash
   setfacl -R -x u:xyzuserid ../ProjectDir/testdir/
   
   setfacl -R -x u:xyzuserid ../ProjectDir/testdir/
   getfacl ../ProjectDir/testdir/ | grep "xyzuserid"
   getfacl ../ProjectDir/testdir/n1/ | grep "xyzuserid"
   getfacl ../ProjectDir/testdir/n1/n2/ | grep "xyzuserid"
   getfacl ../ProjectDir/testdir/n1/n2/n3/ | grep "xyzuserid"
   getfacl ../ProjectDir/testdir/n1/n2/n3/n4/ | grep "xyzuserid"
   getfacl ../ProjectDir/testdir/n1/n2/n3/n4/abc.txt | grep "xyzuserid"
```

4. `'c'` or `'C'` flag doesnt work for POSIX ACLs. For example.

```bash
   setfacl -m user:newuserid:rxwc ../ProjectDir/testdir/
   setfacl: Option -m: Invalid argument near character 16
   setfacl -m user:newuserid:rxc ../ProjectDir/testdir/
   setfacl: Option -m: Invalid argument near character 15
   setfacl -m user:newuserid:rc ../ProjectDir/testdir/
   setfacl: Option -m: Invalid argument near character 14
   setfacl -m user:newuserid:c ../ProjectDir/testdir/
   setfacl: Option -m: Invalid argument near character 13
```

Hence, make sure to remove `'c'` and/or `'C'` from existing pipeline codes, like:
permissions = permissions.lower().replace(`'c'`, `''`) if `'c'` in permissions.lower() else permissions.lower()

Hope you found it useful. Thanks for reading till here :-)
