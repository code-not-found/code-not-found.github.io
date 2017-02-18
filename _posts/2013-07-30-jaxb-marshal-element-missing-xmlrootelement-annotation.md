---
title: JAXB - Marshal Element Missing @XmlRootElement Annotation
tags: [@XmlRootElement, Java, JAXB, Marshal, Unmarshal, XML]
---
Information on the root XML element is required when **marshalling to** or **unmarshalling from** a Java object. JAXB provides this information via the `@XmlRootElement` annotation which contains the name and namespace of the root XML element.

When trying to marshal a class which does not have a `@XMLRootElement` annotation defined, following error will be thrown: "*unable to marshal as an element because it is missing an @XmlRootElement annotation*". Alternatively when trying to unmarshal, the Java runtime will report that an "*unsuspected element*" is found.

For this example let's use following class representing a car with a basic structure. Note that a `XmlRootElement` is not defined!

{% highlight java %}
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
{% endhighlight %}