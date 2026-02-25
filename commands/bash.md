## Git Bash

## Go through log files

### Desc

I need a maybe complicated bash command, which looping through a folder and subfolders, there are all log files in side. In each log file, there are lines, in each line, like a json string, e.g, {"name": "a123", "duration": 5, ....}, beside these two fields, I don't care other fields. Now, the command needs to go through each line, and file those lines with duration > 3 ... and print out name and duration .... please help

- A1:

```
Error: no jq by default for Git Bash

find . -type f -name "*.log" -exec jq -c 'select(.duration > 3) | {name: .name, duration: .duration}' {} +

```

- A2

```

find . -type f -name "*.log" -exec awk -F'[:,]' '/"duration":[ ]*[4-9]|[1-9][0-9]+/ {for(i=1;i<=NF;i++) if($i ~ /"name"/) n=$(i+1); for(i=1;i<=NF;i++) if($i ~ /"duration"/) d=$(i+1); print "name:" n ", duration:" d}' {} +

```

