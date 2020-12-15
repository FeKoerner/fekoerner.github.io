# Useful Commands

```bash
# show the 30 biggest files or directories
du -hc * | sort -rh|head -n 30

# change rights only for directories or files
find . -type d -exec chmod xxx {} \; # directories
find . -type f -exec chmod xxx {} \; # files
```