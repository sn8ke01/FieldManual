---
layout: post
title: Hunting like Elmer
---

# Real World Bug Hunting 
> Notes from the book of the same name by **Pete Yaworski**.
> The book includes a ton of real world examples pulled from bug bounty reports.  Those examples will not be included in these notes.

[![Awesome](https://img.shields.io/badge/Peter%20Yaworski-RealWorld%20Bug%20Hunting-green)](https://nostarch.com/bughunting)

Table of Contents

=============

* [Open Redirects](#open-redirects)
* [XML External Entity Injection](#xml-external-entity-injection) [*Incomplete Section*]
* [Cross-Site Request Forgery](#csrf) (CSRF)



## Open Redirects

> The visited website sends the user's browser to a different URL that could exist in another domain.  This process can be highjacked by a malicious user and send a user to an evil website w/o the user being aware.

There are 3 main types of Open Redirects
1. URL Parameter
2. HTML <meta> tag
3. DOM (Document Object Model)

#### URL Parameter
Application uses an URL parameter to send a **GET** request to the destination URL.

In the below example the redirect parameter is **redirect_to=** but is could be any number of things.

```html
https://www.google.com/?redirect_to=https://www.attacker.com
```

Other parameters to look out for:
```http
url=
next=
r=
```

#### HTML `<meta>` Tag
HTML <meta> tag instructs the browser to refresh and make a **GET** request to a URL.
```html
<meta http-equiv="refresh" content="0; url=https://evil.com/">
```
This becomes a problem when the attacker can control what the `content` attribute and inject their own URL.

#### DOM (document object model)
The DOM is an API for the HTML and XML documents.  Attacker can redirect users to super evil URLs via JavaScript by inejecting a URL int the `window.location` property.

> The DOM Open Redirect requires the attacker be able to execute JavaScript ethier via XXS or an intential user specifiec URL.

```javascript
window.location = https://www.google.com
window.location.href = https://evil.com
window.location.replace(https://E.Corp)
```

#### Open Redirect Detection
Status Code of 3xx.  Typical Status Code of **302** but any 3xx.

Monitor your proxy for **GET** requests sent to the site you're testing.

## XML External Entity Injection

> This attack abuses systems that parse XML input.
>
> It allows an attacker to get all up in the business of an application's processing of XML data.  It can allow an attacker to view app server filesystems and interact with any backend or external system the application can process.
>
> In short, it takes advantage of how the application processes the inclusion of external entities in its input.

### Document Type Definitions (DTD)

The DTD define what elements can exists, what attributes can be there, and which elements can be enclosed within other elements.  DTD if defined using the `<!Doctype>` element. Each element that is going to be used in the XML document is defined in the DTD using the `!Element` keyword.  There are 2 types of DTD:

1. <u>External</u>: An external .dtd file the XML document references & fetches. 
2. <u>Internal</u>: The DTD exists within the XML document.

[LEARN MORE ABOUT XML & DTD](https://portswigger.net/web-security/xxe/xml-entities)

### XML Entities

An entity is a place holder for information (like a variable in a script).  For example if we wanted to put a URL in the XML multiple times we can instead declare that URL with the DTD `!Element` tag.

```xml
<!Element WEBSITE ANY>
<!Element url SYSTEM "website.txt">
...
<Website>&url;</Website>
```

Abusing XML entities is a primary attack path for XXE Injection.

### XXE Injection Attack

In this example injection attack taken from [PayloadsAllTheThings]([https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XXE%20Injection#classic-xxe)- you would want to find a webapp that alows file uploads and upload an xml file containing the following code...

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<!DOCTYPE foo [  
  <!ELEMENT foo ANY >
  <!ENTITY xxe SYSTEM "file:///c:/boot.ini" >]><foo>&xxe;</foo>
```

## CSRF

Cross-Site Request Forgery happens when an attacker can make the victim's browser send and HTTP request to another website that then performs the action as if it were valid and sent by the victim (user).  In order for the attack to be performed the victim must be authenticated to the website the attacker will send the HTTP request to.

If the victim is a low lever authenticated user of the web application that receives the poorly trusted HTTP request the attacker can accomplish various tasks like password and email changes, fund transfers, and modify other server-side information.  If the victim is an authenticated admin the attacker can takeover the entire web application.

---

When you log into an application the app needs to be able to track the state of that authentication (in or out so to speak) otherwise the you would have to re-authenticate to each new restricted page. Needless to say, some ways of tracking this state are better than others.  





