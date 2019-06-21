# JSON Toolkit

A collection of CLI tools which make it easy to write pipelines processing various data files. Using these tools in conjunction with jq, you can write data processing prototypes in seconds!

## Example usecases
### 1 to 10 in json
```bash
seq 10 | newline-to-json | jq 'map(tonumber)'
```
### Select rows from a CSV where the 2nd and 3rd column are equal
```bash
cat file.csv | csv-to-json | jq 'map(select(.[1] == .[2]))' | json-to-csv
```
### Extract the difference between daily XML reports.
```bash
cat yesterday.xml | json-to-xml | json-format > yesterday.json
cat today.xml | json-to-xml | json-format > today.json
json-diff yesterday.json today.json | json-format > difference.json
```
### Extract the difference between daily CSV reports.
```bash
cat yesterday.csv | json-to-csv | json-format > yesterday.json
cat today.csv | json-to-csv | json-format > today.json
json-diff yesterday.json today.json | json-format > difference.json
```
### Testing framework for a web endpoint
If the test passes, this will return exit code 0 and nothing on stdout
If the test fails, this will return exit code 1 and the test difference as JSON on stdout.

```bash
cat test-input.json | json-post https://your-web-server/api/endpoint | json-format > actual-test-output.json
json-diff actual-test-output.json expected-test-output.json | json-format > test-difference.json
test-difference.json | json-empty
```


## Prerequsites

* UNIX-like operating system
* Bash
* Go
* Python3
* [jq](https://stedolan.github.io/jq/)

## Getting Started

```bash
git clone git@github.com:tyleradams/json-toolkit.git
make
./run-all-tests # On success this should return nothing
sudo make install
```

## Tools

### json-diff
json-diff takes in two json files and returns the differences between the files as json.

#### Output format
The output is a json encoded list of difference objects describing the difference between two json files.
If the files are equivalent, the output will be an empty json array.
##### Example
Consider
```json
[ 1, 2, 3 ]
```
and
```json
[ 1, 2, 4 ]
```
Then the difference is
```json
[{"leftValue":3,"path":[2],"rightValue":4}]
```
At path `[2]`, (index 2 element in the top level array), the left value is 3, but the right value is 4

##### Difference Object
A difference object has a required `path`, an optional `leftValue`, and an optional `rightValue`.
The `leftValue` and `rightValue` are the values of the json object at the path for the left file and right files respectively.
If there is no `leftValue`, this means the path does not exist for the left file, and similarly for the `rightValue` and right file. For example, if the left is an array of length 1 and the right is an array of length 2, then path `[1]` (index 1 element) does not exist for the left but it does for the right.
###### Path
The path is the location of a particular a json value nested within a larger json value.
In `json-diff`, this is encoded as a json array of integers for array indicies and strings for object keys.

###### Example Path
For:
```json
[
    0,
    {
        "a": [
            -1
        ],
    }
]
```
-1 can be found at `[1, "a", 0]`.
This path should be read as take the whole array:
Then take the index `1` element of top level array.
Then take the value corresponding to the `"a"` key.
Then take the index `0` element.
This final value is the one found at `[1, "a", 0]`.

### json-empty
#### Description
`json-empty` helps you return the right error code in bash scripts.
* If the input is an empty json array, it returns exit code 0 and an empty stdout.
* If the input is valid json but not an empty array, `json-empty` returns exit code 1 and the returns the inputted string to stdout.
* If the input is not valid json, `json-empty` returns exit code 2 and throws an error message to stderr.
#### Examples
##### Bash script checking if two json files are equivalent
```bash
json-diff left.json right.json | json-empty
```

##### Passing an empty JSON array into json-empty
```bash
echo '[]' | json-empty
```

##### Passing a non-empty JSON array into json-empty
```bash
echo '[1]' | json-empty
```

##### Passing a JSON string into json-empty
```bash
echo '"non-empty array input"' | json-empty
```

##### Passing non-JSON into json-empty
```bash
echo 'this is not json' | json-empty
```

### json-to-csv
#### Description
`json-to-csv` takes a json array of array of strings from stdin and formats the data as a csv on stdout.
#### Examples
```bash
echo '[["Single cell"]]' | json-to-csv
echo '[["Multiple", "cells", "but", "one", "row"]]' | json-to-csv
echo '[["Multiple", "cells"], ["and"], ["multiple", "rows"]]' | json-to-csv
```

### json-to-dsv
#### Description
`json-to-dsv` takes a json array of array of strings from stdin, and a delmiter as the first argument, and formats the data as a dsv with the specified delimiter on stdout.
#### Examples
```bash
echo '[["Single cell"]]' | json-to-dsv :
echo '[["Multiple", "cells", "but", "one", "row"]]' | json-to-dsv :
echo '[["Multiple", "cells"], ["and"], ["multiple", "rows"]]' | json-to-dsv :
```

### json-to-xml
#### Description
`json-to-xml` takes json from stdin and formats the data as xml on stdout. Only an object with a single key can be converted to xml.
#### Examples
```bash
echo '{"root": {"a": "b"}}' | json-to-xml
```

### json-to-yaml
#### Description
`json-to-yaml` takes json from stdin and formats the data as yaml on stdout.
#### Examples
```bash
echo '{"a": 1, "b": 2}' | json-to-yaml
```

### csv-to-json
#### Description
`csv-to-json` takes a csv from stdin and formats the data into a json array of array of strings.
#### Examples
```bash
echo Single cell | csv-to-json
echo Multiple,cells,but,one,row | csv-to-json
echo -e Multiple,cells\\nand\\nmultiple,rows | csv-to-json
```

### dsv-to-json
#### Description
`dsv-to-json` takes a dsv file from stdin, the delimiter as the first argument, and formats the data into a json array of array of strings.
#### Examples
```bash
echo Single cell | dsv-to-json :
echo Multiple:cells:but:one:row | dsv-to-json :
echo -e Multiple:cells\\nand\\nmultiple:rows | dsv-to-json :
```

### xml-to-json
#### Description
`xml-to-json` takes xml from stdin and formats the data as json on stdout.
#### Examples
```bash
echo '<a>b</a>' | xml-to-json
```

### yaml-to-json
#### Description
`yaml-to-json` takes yaml from stdin and formats the data as json on stdout.
#### Examples
```bash
echo 'a: b' | yaml-to-json
```
