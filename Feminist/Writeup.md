## Introduction

![](images/image.png)

As the name in the website suggests, This is an RCE challenge, and we have to somehow execute some Linux commands in a way that we can get the flag. 
We have an input field where we can enter some commands, and we have a button to execute them. with a Feminist theme. And most importantly, a link to the source code.

## Analysis

The source code is pretty straightforward, a small backend in Flask:

```python

from flask import Flask, render_template, request, send_file
from subprocess import PIPE, Popen

# create a flask app
app = Flask(__name__)

# this function executes the command. if there is an output, it returns it. otherwise, it returns the error.
def exec(q):
    p = Popen(q, shell=True, stdout=PIPE, stderr=PIPE)
    stdout, stderr = p.communicate()
    if stdout:
        return stdout.decode()
    return stderr.decode()

# this is the index page that we have visited
@app.get("/")
def index():
    return render_template('index.html', output="")

# this is a POST Endpoint that will execute once we click the button
@app.post("/")
def query():
    # our query
    q = request.form['q'] 
    # split the query and add "she" to each word
    q = ["she" + word for word in q.split(' ')] 
    # join the words again, but with no space
    q = ''.join(q)
    # execute the command
    result = exec(q)
    # return the result
    return render_template('index.html', output=result)

# this is the endpoint that returns the source code
@app.get("/code")
def code():
    return send_file(__file__)

```

So simply, if we send the command `ls`,  it will execute `shels` and throws and error.
and if we send the command `cat flag.txt`, it will execute `shecatsheflag.txt` which doesn't make sense to linux as well.
So we have a couple of things to be bypass in this task. First the `she` addition to the beginning, second the space issue.

## Exploitation

### 1. The `she` addition

The first thing to think of in term of linux commands, is the seperators `&&`, `||`, `;`
Which are used to execute multiple commands in one line. So we can bypass the `she` addition by using `||` or `;` to seperate the commands, because `||` will execute if the first command fails, and `;` will execute the second command no matter what. and we should not seperate the commands with space between them.

After we bypassed the `she` addition, we can use the `ls` command to see the files in the directory.

```sh
;ls
# feminist.py requirements.txt run.sh shefiles templates
```

We can see few files out there, with an untypical name `shefiles`. So we can try to read the content of this file.
Yet we have the Space Issue yet to solve..

### 2. The Space Issue

If you can't use a space, you can use something that represents a space. And the first one that comes to mind is `$IFS`, the Internal Field Seperator in Linux. It is used to seperate the fields in a line, and by default its value is a space. So we can use it to seperate the commands.

One thing to note, is that the commands with $IFS will fail, because since there's no spaces, anything after `$IFS` will be appended to the command, resulting to a variable name called `$IFSshefiles` for example, so we have to seperate the variable name from whats next, a way to do so is by using `*`.

```sh
;cat$IFS*shefiles # or cat$IFS* files, since it adds the she to the beginning
# /bin/sh: 1: she: not found cat: shefiles: Is a directory
```

So we can see that the `shefiles` is a directory, and we can try to list its content.

```sh
;ls$IFS*shefiles
# flag.txt
```

So, Let's try to read the flag.

```sh
;cat$IFS*shefiles/flag.txt
# ENIT{th3_best_g0_gir1_moM3nt_1_sAw_waS_in_tHe_boys}
```

## Conclusion
The Challenge is pretty simple and Linux commands are your best friend in this task. I hope you enjoyed reading this writeup, and I hope you learned something new.


## Flag
> ENIT{th3_best_g0_gir1_moM3nt_1_sAw_waS_in_tHe_boys}

I Recommand watching the show. It is top notch.

![](images/feminist.jpeg)


## Other Writeups
- [unintended](unintended.md)