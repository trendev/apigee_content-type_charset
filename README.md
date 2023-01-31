# APIGEE Proxy demonstrating issue setting `charset` value in `content-type` Http Header  

### Normal behavior :blush:

> curl "http://localhost:8998/myproxy"

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

### Issue providing `charset` with `content-type` :fearful:

> curl --header 'Content-Type: application/json ;charset=utf-8' "http://localhost:8998/myproxy"

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
