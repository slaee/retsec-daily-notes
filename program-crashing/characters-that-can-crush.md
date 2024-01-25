
## Rust
If we are ask to input something with the rust program we can try to crash it by sending non utf-8 encoded text.

Non utf-8 character candidates for rusty program crashing:
`é` (Latin1 small letter e with acute)
`ñ` (Latin1 small letter n with tilde)
`Б` (Cyrillic capital letter Be)
`и` (Cyrillic small letter I)
`漢` (Kanji character for "kan") 
`字` (Kanji character for "ji")
`ع` (Arabic letter Ain)
`م` (Arabic letter Meem)
`अ` (Devanagari letter A)
`` (U+F8FF, Apple logo)

## InfluxDB QL
Sometimes query strings are not secured enough.

Character candidates:
`"` Double quotes, sample error:
```
HttpError: compilation failed: error @1:82-1:158: expected RPAREN, got EOF error error @1:155-1:158: got unexpected token in string expression @1:158-1:158: EOF
```

## Expression Language (EL) in JSP

Candidates:
`${333 + 333}` returns `ELException`
```
 Failed to parse expression [${333 333}]
```

## URL Scan
Intercepts requests and reject them if it detects invalid characters. It is meant to protect website from malicious attacks and SQL injection. If doesn't configure it correctly then it will reject perfectly reasonable requests.

Candidates:
`%`  from query: `results.aspx?criteria=Search%20Criteria%20is%20%25Be`
```
Error Code: 500 Internal Server Error. The request was rejected by the HTTP filter. Contact the server administrator. (12217)
```

## Evals

`eval()` 
Candidates that can crash a ruby program that calls eval to evaluate our input:
Syntax errors:
`-`  , ` `` ` , `~`  , `++`  , `--`  , `**` ,  `(` , any single symbol characters.

# References

https://rafa.hashnode.dev/influxdb-nosql-injection

https://youtu.be/qA8KB6KndrE?si=NOxIJd9UHm5By0eQ&t=355

https://stackoverflow.com/a/2156180

