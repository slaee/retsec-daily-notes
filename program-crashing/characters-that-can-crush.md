
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


# References

https://rafa.hashnode.dev/influxdb-nosql-injection



