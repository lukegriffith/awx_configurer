# awx_config

Repository holds documentation on exporting and importing awx configuration.

Example json files have been exported via the [example](#tower-cli-export)

## tower-cli export

Export configuration from configured tower host and automate filing the recieved assets.json into separate files for each type.

```bash
tower-cli receive --all > assets.json
types=`cat assets.json | jq -r '.[].asset_type'`
for t in $types; do cat assets.json | jq --arg t "$t" 'select(.[].asset_type==$t)' | jq '.' > $t.json; done
rm assets.json
```

## tower-cli import

Import the configuration, some objects might need to be staged to allow for dependencies.

```bash
fails=0

for f in `ls *json`; do echo "Sending $f" && (! tower-cli send $f) && fails=$(($fails + 1)) && echo "$f failed" ; done

count=`ls *json | wc | awk '{print $1}'`
pass=$(($count - $fails))

echo "count: $count"
echo "fails: $fails"
echo "pass: $pass"
```

## Configure tower-cli
Config can be set via environment variables or command line.
Via the command line sensitive information is stored to the users home directory.

```bash
TOWER_HOST=http://<new-awx-host.example.com>
TOWER_USERNAME=<user>
TOWER_PASSWORD=<pass>
```

OR

```bash
tower-cli config host http://<new-awx-host.example.com>
tower-cli config username <user>
tower-cli config password <pass>
tower-cli send assets.json
```

## Query asset_type directly

Filtering for project. 

```bash
cat assets.json | jq --arg t "project" '.[] | select(.asset_type==$t)'
```


## jq piping

jq outputs the filtered objects as individual objects and not apart of an array. I've piped the output of the jq filter in the export to another jq, what re-creates the objects from the original as elements of a json array.


## References 

AWX reference on [data migrations](https://github.com/ansible/awx/blob/devel/DATA_MIGRATION.md).