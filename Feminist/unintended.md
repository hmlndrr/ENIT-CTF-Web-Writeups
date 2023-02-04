# Unintended Solution


As mentioned in the original walkthrough it is a **command injection** task.

We can bypass the ` ` condition with wildcards `*` or we can also use `stdin input <` to specify the commands params .

*Note: in this case we have to specify the total filename*

```sh
;cat<shefiles/flag.txt
```

> [Original Writeup](Writeup.md)

