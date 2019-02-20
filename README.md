# tower-cli export

Export configuration from configured tower host and automate filing the recieved assets.json into separate files for each type.

```bash
tower-cli receive --all > assets.json
types=`cat assets.json | jq -r '.[].asset_type'`
for t in $types; do cat assets.json | jq --arg t "$t" 'select(.[].asset_type==$t)' | jq '.' > $t.json; done
rm assets.json
```

# tower-cli import

Import the configuration, some objects might need to be staged to allow for dependencies.

```bash
fails=0

for f in `ls *json`; do echo "Sending $f" && (! tower-cli send $f) && fails=$(($fails + 1)) ; done

count=`ls *json | wc | awk '{print $1}'`
pass=$(($count - $fails))

echo "count: $count"
echo "fails: $fails"
echo "pass: $pass"


```

# configure tower-cli

```bash
$ tower-cli config host http://<new-awx-host.example.com>
$ tower-cli config username <user>
$ tower-cli config password <pass>
$ tower-cli send assets.json
```

# query asset_type directly

Filtering for project. 

```bash
cat assets.json | jq --arg t "project" '.[] |select(.asset_type==$t)'
```


# jq slurp

jq outputs the filtered objects as individual objects and not apart of an array. Tower cli needs an array of objects fed in in a single file, we can pipe the output of jq to another jq process to perfrom the ```--slurp``` argument that outputs the objects into an array. 


# references 

AWX reference on [data migrations](https://github.com/ansible/awx/blob/devel/DATA_MIGRATION.md).