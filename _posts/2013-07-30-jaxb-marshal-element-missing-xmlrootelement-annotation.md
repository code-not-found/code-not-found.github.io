---
title: JAXB - Marshal Element Missing @XmlRootElement Annotation
tags: [Java, XML]
---

Information on the root XML element is required when **marshalling** to or **unmarshalling** from a Java object. JAXB provides this information via the `@XmlRootElement` annotation which contains the name and namespace of the root XML element.

When trying to marshal a class which does not have a `@XMLRootElement` annotation defined, following error will be thrown: *"unable to marshal as an element because it is missing an @XmlRootElement annotation"*. Alternatively when trying to unmarshal, the Java runtime will report that an *"unsuspected element"* is found.

For this example let’s use following class representing a car with a basic structure. Note that a `XmlRootElement` is not defined!

~~~ java
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
~~~

* * *

# Marshal when @XMLRootElement is missing

Marshalling is the process of transforming the memory representation of an object to a data format suitable for storage or transmission. In the case of JAXB it means converting a Java object into XML. The below code snippet shows the creation of a new `Car` instance.

~~~ java
car = new Car();
    car.setMake("Passat");
    car.setManufacturer("Volkswagen");
    car.setId("ABC-123");
~~~

The method below takes as input the above car object and tries to marshal it using JAXB.

~~~ java
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
~~~

When running the above method, the runtime returns an error as the `Car` class is missing the required `@XMLRootElement` annotation.

~~~ bash
unable to marshal type "com.codenotfound.jaxb.model.Car" as an
element because it is missing an @XmlRootElement annotation
~~~

In order to be able to marshal the car object we need to provide a root XML element. This is done as shown below by first creating a qualified name which contains the name and namespace of the root XML element. In a next step we create a new `JAXBElement` and pass the qualified name, class and object. Using the created `JAXBElement` we call the `marshal()` method.

~~~ java
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
~~~

~~~ xml
This time JAXB is able to successfully marshal the object and the result is the following:

<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<ns2:car id="ABC-123" xmlns:ns2="com.codenotfound.jaxb.model">
    <make>Passat</make>
    <manufacturer>Volkswagen</manufacturer>
</ns2:car>
~~~