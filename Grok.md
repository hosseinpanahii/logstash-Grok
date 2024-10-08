![image](https://github.com/user-attachments/assets/648c7c37-2846-446c-b6ad-fa0e95efbbb7)# Grok Pattern Examples
Grok is a powerful tool for extracting structured data from unstructured text. Grok syntax is composed of reusable elements called Grok patterns that enable parsing for data such as timestamps, IP addresses, hostnames, log levels, and more.The prebuilt patterns make Grok easier to use than defining new regular expressions to extract structured data, especially for long text strings.

## Key Concepts for Using Grok

Grok patterns follow the syntax: %{PATTERN TO MATCH:output label}
- PATTERN TO MATCH: this is the pattern to match on

        - Many patterns are predefined and available for review within the Logstash repository.
        - Example: %{TIMESTAMP_ISO8601} extracts an ISO 8601 timestamp
        - Example: %{IP} extracts any IPv4 or IPv6 value
- output label: the name of the key to use for the parsed data (optional)

          - While the output label is optional, if you don't specify a label, the extracted data is dropped and will not be included in the output
          - Example: %{TIMESTAMP_ISO8601:timestamp} extracts an ISO 8601 timestamp, and then outputs the result with the output label as timestamp: {"timestamp": "2022-09-27 18:00:00.000"}
By combining multiple Grok patterns you can parse a single line of text and extract the relevant information into structured data.

| Pattern | Usage |
| --- | --- |
| %{GREEDYDATA} | This pattern serves as an excellent starting point for building any Grok expression. This will match an entire line of data, or until you specify another expression to buffer it.This pattern is often used for the remainder of any line not parsed |
|  %{DATA}  | This pattern uses a lazy loading with any character count from zero to many, until it is succeeded by another expression, or it reaches the end of the line.This means you can use this pattern with other patterns to "bookend" a larger set of interstitial data. |

Sometimes you will be unable to span the appropriate characters using these patterns. You can also employ the following patterns to aid in more specific advancement:

| Command | Description |
| --- | --- |
| %{SPACE} | Use this pattern to explicitly jump over whitespace if needed. |
| %{NOTSPACE} | This pattern is useful for skipping sets of characters between whitespaces. |
| %{WORD} | This will match whole words within text. |
| %{QS} or %{QUOTEDSTRING} | Use this pattern to extract sets of characters between quotes. |

## Let’s start Grok
Let’s start with an example unstructured log message, which we will then structure with a Grok pattern:

```config
192.168.1.1 - - [07/Oct/2024:12:50:33 +0000] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:131.0) Gecko/20100101 Firefox/131.0" "-"
```

Imagine searching through millions of log lines that look like that! It seems terrible. And that’s why we have parsing languages like Grok – to make the data easier to read, and easier to search. 
A much easier way to view that data – and to search through it with analysis tools like Kibana – is to break it down into fields with values, like the list below:
```jsonc
{
  "request": "/ HTTP/1.1",
  "engin": "Gecko/20100101",
  "ide_version": "Firefox/131.0",
  "MONTH": "Oct",
  "os": "Windows NT 10.0",
  "HOUR": "12",
  "TIME": "12:50:33",
  "mohammad": "GET",
  "ide": "Mozilla/5.0",
  "INT": "+0000",
  "remote_ip": "192.168.1.1",
  "YEAR": "2024",
  "DATA": "x64; rv:131.0",
  "bytes": "0",
  "MINUTE": "50",
  "SECOND": "33",
  "time": "07/Oct/2024:12:50:33 +0000",
  "arch": "Win64",
  "MONTHDAY": "07",
  "status": "304"
```

Let’s use an example Grok pattern to generate these fields. in this section we use [Online Grok Pattern Generator / Debugger Tool](https://www.javainuse.com/grok) inorder to extract data by data.
PLEASE NOTE: It’s not recommended to use space to describe the field’s name.

###  Extracting an IP

Let’s say we want to extract the IP, we can use the IP method:

%{IP:ip}

Then we have these:  – – 

To tell Grok to ignore them, we just add them to our pattern.

%{IP:ip} – –

This pattern gives us this field:

Ip:192.168.1.1

###  Timestamps and Arrays
In the next part of our unstructured log message, we have the timestamp “trapped” inside an array:

[07/Oct/2024:12:50:33 +0000] 
To extract it, we need to use regex and the HTTPDATE method, while adding the brackets on the outside so Grok knows to ignore them:

\[%{HTTPDATE:time}\]

Building on our previous pattern, we now have:

%{IP:ip} – – \[%{HTTPDATE:time}\]

That gives us:

ip: 192.168.1.1
timestamp: 07/Oct/2024:12:50:33 +0000

Going back to our original unstructured message, it looks like we have a space where the timestamp ends, and when the “GET starts. We need to tell Grok to ignore spaces as well. To do that, we just hit the spacebar on our keyboard or we can use %{SPACE} -> that catches until 4 spaces.

### Extracting Verbs

Time to extract the GET field. First, we need to tell Grok to ignore the quotation marks – Then use the WORD method, we will do that by writing:

"%{WORD:verb}

So, now our pattern reads:

%{IP:ip} – – \[%{HTTPDATE:time}\] "%{WORD:verb}

That gives us these fields. 
ip: 192.168.1.1
timestamp: 07/Oct/2024:12:50:33 +0000
verb:GET

### Extracting Request 

In order to extract the request -> /category/electronics HTTP/1.1″, we need to use the DATA method, which is essentially the wild card in regex. 

This means we need to add a stopping point to extract this information to tell the DATA method where to stop – otherwise, it won’t capture any of the data. We can use the quotation marks as a stop mark: 

%{DATA:request}"
Now, we have the following grok pattern:
%{IP:ip} – – \[%{HTTPDATE:time}\] "%{WORD:verb} %{DATA:request}"

ip: 192.168.1.1
timestamp: 07/Oct/2024:12:50:33 +0000
verb:GET
request: / HTTP/1.1

### Extracting the Status 
Next up is the status, but again we have a space between the end of the request and the status, we can add a space or %{SPACE}. To extract numbers, we use the NUMBER method. 

%{NUMBER:status}

Now our pattern extends to: 
%{IP:ip} – – \[%{HTTPDATE:time}\] "%{WORD:verb} %{DATA:request}" %{NUMBER:status} 

ip: 192.168.1.1
timestamp: 07/Oct/2024:12:50:33 +0000
verb:GET
request: / HTTP/1.1
status:304

### Extracting the Bytes 

In order to extract bytes we need to use the NUMBER method again, but before that, we need to use a regular space or %{SPACE}
%{NUMBER:bytes}
%{IP:ip} – – \[%{HTTPDATE:time}\] "%{WORD:verb} %{DATA:request}" %{NUMBER:status} %{NUMBER:bytes}

ip: 192.168.1.1
timestamp: 07/Oct/2024:12:50:33 +0000
verb:GET
request: / HTTP/1.1
status:304
bytes:0

### Ignoring data and extracting OS

we can  IGNORE this data -> "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:131.0) Gecko/20100101 Firefox/131.0" "-"
but we extract word by word

NOTE: in order to ignore some words , we will use the DATA method  without writing the field it will ignore it. like: %{DATA}

To explain this pattern a bit more…

\( -> stops until it reaches (
; ->  stops until it reaches ;

 "-" "%{DATA:ide} \(%{DATA:os}; %{DATA:arch}; %{DATA}\) %{DATA:engin} %{DATA:ide_version}" "-"


That’s it! Now we’re left with the following grok pattern to structure our data.


final:


```jsonc
{
  "request": "/ HTTP/1.1",
  "engin": "Gecko/20100101",
  "ide_version": "Firefox/131.0",
  "MONTH": "Oct",
  "os": "Windows NT 10.0",
  "HOUR": "12",
  "TIME": "12:50:33",
  "verb": "GET",
  "ide": "Mozilla/5.0",
  "INT": "+0000",
  "remote_ip": "192.168.1.1",
  "YEAR": "2024",
  "DATA": "x64; rv:131.0",
  "bytes": "0",
  "MINUTE": "50",
  "SECOND": "33",
  "time": "07/Oct/2024:12:50:33 +0000",
  "arch": "Win64",
  "MONTHDAY": "07",
  "status": "304"
}
```

