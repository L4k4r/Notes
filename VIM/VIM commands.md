
Remove all whitespaces
```vim
:%s/\s\+//g
```

Remove all newline
```vim
:%s/\n//g
```

Get only the users from an nxc --users command
```vim
:%s/.*INLANEFREIGHT.LOCAL\\\([^[:space:]]*\).*/\1/
```
The INLANEFREIGHT.LOCAL is everything that comes in front of the username.



```bash
echo -n "    app.inlanefreight.local
    dev.inlanefreight.local
    drupal-dev.inlanefreight.local
    drupal-qa.inlanefreight.local
    drupal-acc.inlanefreight.local
    drupal.inlanefreight.local
    blog.inlanefreight.local
" | tr -d '\n' | tr -d '[:blank:]' | sed 's/\.local/ /g'
```
- One line seperated by spaces.


FIND COMMAND
```bash
find / -type f -name flag.txt 2>/dev/null
```