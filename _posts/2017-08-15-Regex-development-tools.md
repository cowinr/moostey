In Intellij just hit `ALT`+`RETURN` with the carat over the expression:

![IntelliJ regex support at carat](https://raw.githubusercontent.com/cowinr/moostey/master/images/intellij-regex.jpg)

This site is pretty awesome for visualising regexps - https://regexper.com/. E.g. for above `charge(?!s?\s?waived).*` you get:

![Regexper website](https://raw.githubusercontent.com/cowinr/moostey/master/images/regexpr.jpg)

Also sometimes useful is - http://www.regexplanet.com/advanced/java/index.html which is Java-specific.

| Regular Expression | `charge(?!s?\s?waived).*` |
| as a Java string | `"charge(?!s?\\s?waived).*"` |
| Replacement | |
| groupCount() | 0 |

| Test | Target String | matches() | replaceFirst() | replaceAll() | lookingAt() | find() | n [start(n),end(n)] group(n) |
|-----|
| 1 | charge | Yes | Yes | Yes | 0: [0,6] charge |
| 2 | Charges are cool | Yes | Yes | Yes | 0: [0,16] Charges are cool |
| 3 | Charges waived | No | Charges waived | Charges waived | No | No |
