---
title: JAXB - Marshal Element Missing @XmlRootElement Annotation
---

Information on the root XML element is required when **marshalling** to or **unmarshalling** from a Java object. JAXB provides this information via the @XmlRootElement annotation which contains the name and namespace of the root XML element.

When trying to marshal a class which does not have a @XMLRootElement annotation defined, following error will be thrown: *"unable to marshal as an element because it is missing an @XmlRootElement annotation"*. Alternatively when trying to unmarshal, the Java runtime will report that an *"unsuspected element"* is found.