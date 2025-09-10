# Cheat-sheat for coverage

## Clearing out-old run data:
```
coverage combine
coverage erase
```
Note that these are no-op if your coverage data was already combined or you didn't have any.
So I just do these commands reflexively whenever starting a new coverage analysis.

## Running your script:
Replace  `python`  with `coverage run --parallel-mode`  in you command. This way you can 
run several scripts, adding their coverage together.

## Combining data from several script:
Using this command after using the `--parallel-mode` option, even if  you wound up using just one.
```
coverage combine --append
```

## Viewing data
### In Terminal:
```
coverage report  -m
```
### As a web page:
This will allow you to click on reported page ranges and  be  shown the corresponding source:
```
coverage  html
cd htmlcov
open  index.html
```

## Getting the coverage of your unit test suite:
```
coverage run -m pytest  .
```

