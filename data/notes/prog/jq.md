# JSON data types

JavaScript Object Notation is a popular text based format for exchanging data through APIs. It consists of these data types:

* objects (aka dictionaries) `{}`
* arrays `[]`
* numbers
* strings
* booleans
* "null"

# jq

Command line options:

`-r` (`--raw-output`) -- emit raw (unquoted) strings as output

[Basic filters](https://stedolan.github.io/jq/manual/#Basicfilters):

```
'.'         # pretty print everything

'.foo'      # value at key foo
'.foo.bar'

'.[]'       # all elems of an array
```

You can join filters using `|`:

```
jq '.data.result | .[] | .values | .[] | .[]'
```

Get values of [multiple keys](https://stackoverflow.com/questions/28164849/using-jq-to-parse-and-display-multiple-fields-in-a-json-serially):

```
jq '.rrsets[] | "\(.name) \(.type)"'
```

## Examples

Get ModSecurity rule [IDs](https://www.netnea.com/cms/core-rule-set-inventory/) and description:

```
cat /var/log/modsec_audit.log | jq '.transaction.messages[].details | "\(.ruleId) \(.match)"'
```

Get number of pods per node:

```
kubectl get pods -o json --all-namespaces | jq '.items | group_by(.spec.nodeName) | map({"nodeName": .[0].spec.nodeName, "count": length}) | sort_by(.count) | reverse'
```

# More

* [jq playground](https://jqplay.org/)