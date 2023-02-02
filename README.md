# APIGEE Proxy demonstrating issue setting `charset` value in `Content-Type` Http Header    

## Issue Description

Looks like APIGEE forces URL encoding of `QueryParam` content if `charset` value is set in Http Header `Content-Type`. 

Providing data in `QueryParam`, containing '`+`' character, could be an **issue** for backend services leveraging accurate data.

## Mitigation

:thumbsup: Provide data in Http Request `Headers` instead of `QueryParams`. 

:point_right: Escaping characters or using javascript functions (like  `encodeURIComponent()`) does not solve the problem and BTW data is URL encoded twice...
 
## Proxy's principles

**Very simple pass-through proxy**, reading a value in a `KVM`, adding the value in `QueryParam` and `Header` and forwarding the Request to `https://httpbin.org/get`. 

## Requirements
-  docker installed :white_check_mark:
-  apigee vscode plugin setup :white_check_mark:
-  apigee emulator up & running :white_check_mark:

## :blush: Normal behavior

```
curl "http://localhost:8998/myproxy"
```
:arrow_down:
```
{
  "args": {
    "apiKey": "a+bcde123/zzzyyy/1mAnZaRVB1an"
  }, 
  "headers": {
    "Accept": "*/*", 
    "Apikey": "a+bcde123/zzzyyy/1mAnZaRVB1an", 
    "Host": "httpbin.org", 
    "User-Agent": "curl/7.85.0", 
    "X-Amzn-Trace-Id": "Root=1-63d7f3b9-438e59ba620bd31e1d61452d"
  }, 
  "origin": "87.218.110.186", 
  "url": "https://httpbin.org/get?apiKey=a%2Bbcde123%2Fzzzyyy%2F1mAnZaRVB1an"
}
```

We can note in `https://httpbin.org/get` Response, in _args_ property, that `apiKey` **is not interpreted** by APIGEE.

We can observe the same thing in _headers_ property for `Apikey` value.

## :fearful: Issue providing `charset` with `content-type`

```
curl --header 'Content-Type: application/json ;charset=utf-8' "http://localhost:8998/myproxy"
```
:arrow_down:
```
{
  "args": {
    "apiKey": "a bcde123/zzzyyy/1mAnZaRVB1an"
  }, 
  "headers": {
    "Accept": "*/*", 
    "Apikey": "a+bcde123/zzzyyy/1mAnZaRVB1an", 
    "Content-Type": "application/json ;charset=utf-8", 
    "Host": "httpbin.org", 
    "User-Agent": "curl/7.85.0", 
    "X-Amzn-Trace-Id": "Root=1-63d7f3af-0c4e462100ce0dda55103818"
  }, 
  "origin": "86.218.110.186", 
  "url": "https://httpbin.org/get?apiKey=a bcde123%2Fzzzyyy%2F1mAnZaRVB1an"
}
```

This time, we can note in `https://httpbin.org/get` Response, in _args_ property, that `apiKey` **is interpreted** and in _headers_ property that `Apikey` **is not interpreted** !

Usually, '`+`' character is interpreted into '`SPACE`' character in `QueryParams` but it remains interesting to see that APIGEE is only performing this translation when `charset` value is set in `Content-Type`.

## :rotating_light: This could be an abnormal behavior...

## :tada: Update on 02/02/2023: we have a fix from Google's support !!!
Adding a `property` in apiproxy's defintion (_default.xml_ file) fixes the problem:
```xml
<HTTPProxyConnection>
 <BasePath>/myproxy</BasePath>
 <Properties>
  <Property name="request.queryparams.ignore.content.type.charset">true</Property>
 </Properties>
</HTTPProxyConnection>
```
