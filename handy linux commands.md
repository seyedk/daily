## count the number of first level subdirectories 
```
find . -maxdepth 1 -type d | wc -l
```

## get the used space of a directory (including it's subs)
```
du -sh 
```

## Counting the success messages in a log file
```
cat mylogifle.log | grep SUCC* | wc -l
```