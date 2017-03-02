---
title: JAXB - Marshal Element Missing @XmlRootElement Annotation
permalink: /2013/07/jaxb-marshal-element-missing-xmlrootelement-annotation.html
excerpt: How to handle element missing @XmlRootElement annotation errors when trying to marshal a Java object using JAXB.
date: 2013-07-30 21:00
categories: [JAXB]
tags: [XmlRootElement, Java, JAXB, Marshal, Unmarshal, XML]
redirect_from: "/2013/07/jaxb-marshal-unmarshal-with-missing.html"
---

<figure>
    <img src="{{ site.url }}/assets/images/logos/jaxb-logo.png" alt="jaxb logo" class="logo">
</figure>

Information on the root XML element is required when **marshalling** to or **unmarshalling** from a Java object. JAXB provides this information via the `@XmlRootElement` annotation which contains the name and namespace of the root XML element.

When trying to marshal a class which does not have a `@XMLRootElement` annotation defined, following error will be thrown: <var>"unable to marshal as an element because it is missing an @XmlRootElement annotation"</var>. Alternatively when trying to unmarshal, the Java runtime will report that an <var>"unsuspected element"</var> is found.

For this example let’s use following class representing a car with a basic structure. Note that a `XmlRootElement` is not defined!

``` java
package com.codenotfound.jaxb.model;

import javax.xml.bind.annotation.XmlAttribute;
import javax.xml.bind.annotation.XmlElement;

public class Car {

    private String make;
    private String manufacturer;
    private String id;

    public String getMake() {
        return make;
    }

    @XmlElement
    public void setMake(String make) {
        this.make = make;
    }

    public String getManufacturer() {
        return manufacturer;
    }

    @XmlElement
    public void setManufacturer(String manufacturer) {
        this.manufacturer = manufacturer;
    }

    public String getId() {
        return id;
    }

    @XmlAttribute
    public void setId(String id) {
        this.id = id;
    }

    public String toString() {
        return "Car [" + "make=" + make + ", manufacturer=" + manufacturer
                + ", id=" + id + "]";
    }
}
```

# Marshal when @XMLRootElement is missing

Marshalling is the process of transforming the memory representation of an object to a data format suitable for storage or transmission. In the case of JAXB it means converting a Java object into XML. The below code snippet shows the creation of a new `Car` instance.
``` java
    car = new Car();
    car.setMake("Passat");
    car.setManufacturer("Volkswagen");
    car.setId("ABC-123");
```

The method below takes as input the above car object and tries to marshal it using JAXB.

``` java
public static String marshalError(Car car) throws JAXBException {
    StringWriter stringWriter = new StringWriter();

    JAXBContext jaxbContext = JAXBContext.newInstance(Car.class);
    Marshaller jaxbMarshaller = jaxbContext.createMarshaller();

    // format the XML output
    jaxbMarshaller.setProperty(Marshaller.JAXB_FORMATTED_OUTPUT, true);

    jaxbMarshaller.marshal(car, stringWriter);

    String result = stringWriter.toString();
    LOGGER.info(result);
    return result;
}
```

When running the above method, the runtime returns an error as the `Car` class is missing the required `@XMLRootElement` annotation.

``` plaintext
unable to marshal type "com.codenotfound.jaxb.model.Car" as an
element because it is missing an @XmlRootElement annotation
```

In order to be able to marshal the car object we need to provide a root XML element. This is done as shown below by first creating a qualified name which contains the name and namespace of the root XML element. In a next step we create a new `JAXBElement` and pass the qualified name, class and object. Using the created `JAXBElement` we call the `marshal()` method.

``` java
public static String marshal(Car car) throws JAXBException {
    StringWriter stringWriter = new StringWriter();

    JAXBContext jaxbContext = JAXBContext.newInstance(Car.class);
    Marshaller jaxbMarshaller = jaxbContext.createMarshaller();

    // format the XML output
    jaxbMarshaller.setProperty(Marshaller.JAXB_FORMATTED_OUTPUT, true);

    QName qName = new QName("com.codenotfound.jaxb.model", "car");
    JAXBElement<Car> root = new JAXBElement<Car>(qName, Car.class, car);

    jaxbMarshaller.marshal(root, stringWriter);

    String result = stringWriter.toString();
    LOGGER.info(result);
    return result;
}
```

This time JAXB is able to successfully marshal the object and the result is the following:

``` xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<ns2:car id="ABC-123" xmlns:ns2="com.codenotfound.jaxb.model">
    <make>Passat</make>
    <manufacturer>Volkswagen</manufacturer>
</ns2:car>
```

# Unmarshal when @XMLRootElement is missing

Unmarshalling in JAXB is the process of converting XML content into a Java object. Lets reuse the XML representation of a car that we generated in the previous section.

``` xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<ns2:car id="DEF-456" xmlns:ns2="com.codenotfound.jaxb.model">
    <make>Golf</make>
    <manufacturer>Volkswagen</manufacturer>
</ns2:car>
```

In the method below we pass the above XML file and try to unmarshal it to an instance of the `Car` class. Note that it is also possible to [use JAXB to create an object from an XML String]({{ site.url }}/2013/07/jaxb-unmarshal-xml-string-into-java-object.html) instead of using a file.

``` java
public static Car unmarshalError(File file) throws JAXBException {
    JAXBContext jaxbContext = JAXBContext.newInstance(Car.class);
    Unmarshaller jaxbUnmarshaller = jaxbContext.createUnmarshaller();

    Car car = (Car) jaxbUnmarshaller.unmarshal(file);

    LOGGER.info(car.toString());
    return car;
}
```

When running the above code, the runtime returns an error as a root XML element is found (the XML has a root element) but the `Car` class does not define a `@XMLRootElement` and as such it is not expected.

``` plaintext
unexpected element (uri:"com.codenotfound.jaxb.model", local:"car").
Expected elements are (none)
```

In order to be able to unmarshal the object we need to define a root XML element. This is done as shown below by first manually creating the root `JAXBElement` of type `Car` by using the XML file and the class we are trying to unmarshal to. Then we create a `Car` object and assign the value of the previous created root `JAXBElement`.

``` java
public static Car unmarshal(File file) throws JAXBException {
    JAXBContext jaxbContext = JAXBContext.newInstance(Car.class);
    Unmarshaller jaxbUnmarshaller = jaxbContext.createUnmarshaller();

    JAXBElement<Car> root = jaxbUnmarshaller.unmarshal(new StreamSource(
            file), Car.class);
    Car car = root.getValue();

    LOGGER.info(car.toString());
    return car;
}
```

This time JAXB is able to successfully unmarshal the object and the result is the following:

``` plaintext
Car [make=Golf, manufacturer=Volkswagen, id=DEF-456]
```

---

{% capture notice-github %}
![github mark](/assets/images/logos/github-mark.png){: .align-left}
If you would like to run the above code sample you can get the full source code [here](https://github.com/code-not-found/jaxb/tree/master/jaxb-missing-rootelement).
{% endcapture %}
<div class="notice--info">{{ notice-github | markdownify }}</div>

This concludes the marshal & unmarshal with missing `@XmlRootElement` annotation code samples. If you found this post helpful or have any questions or remarks, please leave a comment below.