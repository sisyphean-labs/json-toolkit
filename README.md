# JSON Toolkit

This is a collection of CLI tools to help manipulate json files in a UNIX-like environment.

## Prerequsites

* UNIX-like operating system
* Bash
* Go
* Python3
* [jq](https://stedolan.github.io/jq/)

## Getting Started

* git clone git@github.com:tyleradams/json-toolkit.git
* make
* ./run-all-tests # On success this should return nothing
* sudo make install

## Tools

### json-diff
json-diff takes in two json files and returns the differences between the files as json.

#### Output format
The output is a json encoded list of difference objects describing the difference between two json files.
If the files are equivalent, the output will be an empty json array.
##### Difference Object
A difference object has a required path, an optional leftValue, and an optional rightValue.
The leftValue and rightValues are the values of the json object at the path for the left file and right files respectively.
If there is no leftValue, this means the path does not exist for the left file, and similarly for the rightValue and right file.
###### Path
The path is the location of a particular a json value nested within a larger json value.
In json-diff, this is encoded as a json array of integers or strings.
This notation is similar to the path notation used by (and plagarized from) jq.
The primary difference is our paths are arrays whereas jq uses . delimited strings

###### Example Path
In the object
```
[
    0,
    {
        "a": [
            -1
        ],
    }
]
```
-1 can be found at [1, "a", 0].
This path should be read as take the second element of top level array.
For this value, which is an object, take the value corresponding to the "a" key.
For this value, which is an array, take the first element.
This final value is the one found at [1, "a", 0].
#### Examples
##### File setup
json-diff operates on files, so here we create a few files we can use later.
```
echo '[]' > empty_array.json
cp empty_array.json other_empty_array.json
echo '[1]' > non_empty_array.json
```

##### Comparing equivalent files
```
json-diff empty_array.json empty_array.json
json-diff empty_array.json other_empty_array.json
json-diff non_empty_array.json non_empty_array.json
```

##### Comparing different files
```
json-diff empty_array.json non_empty_array.json
```

### json-empty
#### Description
json-empty checks if the input is an empty json array. If so, it returns exit code 0 and an empty stdout
If the input is valid json but is not an empty array, json-empty returns exit code 1 and the returns the inputted string to stdout.
If the input is not valid json , json-empty returns exit code 2 and throws an error message to stderr.
#### Examples
##### Passing an empty JSON array into json-empty
```
echo '[]' | json-empty
```

##### Passing a non-empty JSON array into json-empty
```
echo '[1]' | json-empty
```

##### Passing a JSON string into json-empty
```
echo '"non-empty array input"' | json-empty
```

##### Passing non-JSON into json-empty
```
echo 'this is not json' | json-empty
```

### json-to-csv
#### Description
json-to-csv takes a json array of array of strings from stdin and formats the data as a csv on stdout.
#### Examples
```
echo '[["Single cell"]]' | json-to-csv
echo '[["Multiple", "cells", "but", "one", "row"]]' | json-to-csv
echo '[["Multiple", "cells"], ["and"], ["multiple", "rows"]]' | json-to-csv
```

### json-to-dsv
#### Description
json-to-dsv takes a json array of array of strings from stdin, and a delmiter as the first argument, and formats the data as a dsv with the specified delimiter on stdout.
#### Examples
```
echo '[["Single cell"]]' | json-to-dsv :
echo '[["Multiple", "cells", "but", "one", "row"]]' | json-to-dsv :
echo '[["Multiple", "cells"], ["and"], ["multiple", "rows"]]' | json-to-dsv :
```

### json-to-xml
#### Description
json-to-xml takes json from stdin and formats the data as xml on stdout with a top level "root" tag.
#### Examples
```
echo '{"a": "b"}' | json-to-xml
```

### json-to-yaml
#### Description
json-to-yaml takes json from stdin and formats the data as yaml on stdout.
#### Examples
```
echo '{"a": 1, "b": 2}' | json-to-yaml
```

### csv-to-json
#### Description
csv-to-json takes a csv from stdin and formats the data into a json array of array of strings.
#### Examples
```
echo Single cell | csv-to-json
echo Multiple,cells,but,one,row | csv-to-json
echo -e Multiple,cells\\nand\\nmultiple,rows | csv-to-json
```

### dsv-to-json
#### Description
dsv-to-json takes a dsv file from stdin, the delimiter as the first argument, and formats the data into a json array of array of strings.
#### Examples
```
echo Single cell | dsv-to-json :
echo Multiple:cells:but:one:row | dsv-to-json :
echo -e Multiple:cells\\nand\\nmultiple:rows | dsv-to-json :
```

### xml-to-json
#### Description
xml-to-json takes xml from stdin and formats the data as json on stdout.
#### Examples
```
echo '<a>b</a>' | xml-to-json
```

### yaml-to-json
#### Description
yaml-to-json takes yaml from stdin and formats the data as json on stdout.
#### Examples
```
echo 'a: b' | yaml-to-json
```
