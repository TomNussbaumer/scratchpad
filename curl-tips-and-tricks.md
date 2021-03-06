# curl - Tips and Tricks

For accessing REST apis from scripts `curl` is THE go-to tool.

## Introduction: Simple Oneliners

```shell
## for testing we'll use Github's search api for repositories
## (searching for the term docker)
URL='https://api.github.com/search/repositories?q=docker'

## simplest variant (fetch and grep)
curl $URL | grep "ssh_url"

## silence curl ...
curl -s $URL | grep "ssh_url"

## show HTTP headers, too (and paginate with less)
curl -i -s $URL  | grep "ssh_url" | less

## fetch HTTP headers only
## NOTE: there are links for pagination in the headers!
curl -I -s $URL

## write headers and data to different files (data.00 and data.01)
curl -si $URL | csplit -f data. -s - '/^\s$/'

## BETTER: split result in-memory in different variables
##         (requires two sed runs, but no disk access)
RESULT=$(curl -si $URL)
HEADER=$(echo -n "$RESULT" | sed '/^\s$/q')
BODY=$(echo -n "$RESULT" | sed '1,/^\s$/d')
```

