---
title: "JAXB - Unmarshal XML String into Java Object"
permalink: /jaxb-unmarshal-xml-string-into-java-object.html
excerpt: "A short code sample on how to unmarshal an XML String into a Java Object using JAXB."
date: 2013-07-31
last_modified_at: 2013-07-31
header:
  teaser: "assets/images/teaser/jaxb-teaser.png"
categories: [JAXB]
tags: [Java, JAXB, Object, String, Unmarshal, XML]
redirect_from:
  - /2013/07/jaxb-unmarshal-xml-string.html
  - /2013/07/jaxb-unmarshal-an-xml-string.html
  - /2013/07/jaxb-unmarshalling-xml-string.html
  - /2013/07/jaxb-unmarshal-xml-string-into-java-object.html
published: true
---

<figure>
    <img src="{{ site.url }}/assets/images/logo/jaxb-logo.png" alt="jaxb logo" class="logo">
</figure>

When trying to unmarshal XML to a Java object using JAXB you might want to pass the XML as a `String`. However the `unmarshal()` method of the `Unmarshaller` interface does not support passing an XML String. Following code sample illustrates how to solve this.

Wrap the XML String in a `StringReader` object and pass this to the `unmarshal()` method as shown below. Note that an even [shorter one-liner notation](http://stackoverflow.com/a/9794300/4201470){:target="_blank"} is also possible.

``` java
public static Car unmarshal(String xml) throws JAXBException {
  JAXBContext jaxbContext = JAXBContext.newInstance(Car.class);
  Unmarshaller jaxbUnmarshaller = jaxbContext.createUnmarshaller();

  StringReader reader = new StringReader(xml);
  Car car = (Car) jaxbUnmarshaller.unmarshal(reader);

  LOGGER.info(car.toString());
  return car;
}
```

Next, pass below string representation of an XML to the above `unmarshal()` method.

``` java
xml = "<?xml version=\"1.0\" encoding=\"UTF-8\" "
  + "standalone=\"yes\"?>"
  + "<ns2:Car xmlns:ns2=\"com.codenotfound.jaxb.model\" id=\"ABC-123\">"
  + "<make>Passat</make>"
  + "<manufacturer>Volkswagen</manufacturer></ns2:Car>";
```

JAXB is now able to unmarshal the `StringReader` object and the result is the following:

``` plaintext
Car [make=Passat, manufacturer=Volkswagen, id=ABC-123]
```

---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/jaxb/tree/master/jaxb-unmarshal-string){:target="_blank"}.
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

This concludes the short example on how to unmarshal an XML `String`. If you found this post helpful or have any questions or remarks, please leave a comment.
