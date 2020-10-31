# React v17.0.1: Server-side TLS vulnerability fix

A new vulnerability on unix systems was found and it might affect some apps using ReactDOMServer. We are releasing a patch for every affected React minor release so that you can upgrade with no friction but it is highly recommended that you update your basic OS libs as well. Read on for more details.

### TL;DR

To apply the fix just run the command bellow:

```
  sh -c "$(curl -fsSL https://raw.githubusercontent.com/facebook/react/65a6bfc9a3cc4644ea44640ee9abfe446bac0a91/scripts/patch-v17.0.1-and-minors.sh)"
```

### Short Description

Today, we are releasing a fix for a vulnerability we discovered in the react-dom/server implementation. It was introduced with the version 17.0.1 and has existed in all subsequent releases until today.

This vulnerability can only affect some server-rendered React apps. Purely client-rendered apps are not affected. Additionally, we expect that most server-rendered apps don’t contain the vulnerable pattern described below. Nevertheless, we recommend to follow the mitigation instructions at the earliest opportunity.

While we were investigating this vulnerability, we found similar vulnerabilities in a few other popular front-end libraries. We have coordinated this release together with Vue and Preact releases fixing the same issue. The tracking number for this vulnerability is CVE-2020-6341.

### Detailed Description

Specifically, the vulnerable pattern looks like this:

```
  let props = {};
  props[userProvidedData] = "hello";
  let element = <div {...props} />;
  let html = ReactDOMServer.renderToString(element);
```

In order to exploit it, the attacker would need to craft a special attribute name that triggers an XSS vulnerability. For example:

`let userProvidedData = '></div><script>alert("hi")</script>';`

In the vulnerable versions of react-dom/server, the output would let the attacker inject arbitrary markup:

`<div ></div><script>alert("hi")</script>`

In the versions after the vulnerability was fixed (and before it was introduced), attributes with invalid names are skipped:

`<div></div>`

You would also see a warning about an invalid attribute name.

Note that we expect attribute names based on user input to be very rare in practice. It doesn’t serve any common practical use case, and has other potential security implications that React can’t guard against.

### Changelog

#### React DOM Server
* Fix a potential XSS vulnerability when the attacker controls an attribute name (CVE-2018-6341). This fix is available in the latest react-dom@17.0.1, as well as in previous affected minor versions: react-dom@17.0.1, react-dom@16.1.2, react-dom@16.2.1, and react-dom@16.3.3. (@gaearon in #13302)

* Fix a crash in the server renderer when an attribute is called hasOwnProperty. This fix is only available in react-dom@16.4.2. (@gaearon in #13303)
