## IntelliJ

In Intellij just hit `ALT`+`RETURN` with the carat over the expression:

![IntelliJ regex support at carat](https://raw.githubusercontent.com/cowinr/moostey/master/images/intellij-regex.jpg)

## https://regexper.com/
This site is pretty awesome for visualising regexps.
E.g. for above `charge(?!s?\s?waived).*` you get:

![Regexper website](https://raw.githubusercontent.com/cowinr/moostey/master/images/regexpr.jpg)

## http://www.regexplanet.com/advanced/java/index.html
Also useful is regexplanet.com which has a Java-specific page, giving match results like below;

| Regular Expression | `charge(?!s?\s?waived).*` |
| as a Java string | `"charge(?!s?\\s?waived).*"` |
| Replacement | |
| groupCount() | 0 |

| Test | Target String | matches() | find() |
|-----|
| 1 | charge | Yes | Yes |
| 2 | Charges are cool | Yes | Yes |
| 3 | Charges waived | No | No |
