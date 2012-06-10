---
layout: post
title: "JAXB broken by JDK 7"
date: 2012-05-12 21:12
comments: true
categories: [Java, JAXB, Development]
---

I upgraded to Mac OS [JDK 7](http://bit.ly/IV8sKV) [last week](http://bit.ly/IV96bb) right after it got released, and wanted to be a happy user. 

And my happniess went out as quickly as my [JAXB](http://jaxb.java.net/) related modules failed to work. I kept getting following exception
```java NPE from JAXB
Exception in thread "main" java.lang.NullPointerException
	at com.sun.xml.internal.bind.v2.runtime.ClassBeanInfoImpl.checkOverrideProperties(ClassBeanInfoImpl.java:190)
	at com.sun.xml.internal.bind.v2.runtime.ClassBeanInfoImpl.<init>(ClassBeanInfoImpl.java:171)
	at com.sun.xml.internal.bind.v2.runtime.JAXBContextImpl.getOrCreate(JAXBContextImpl.java:494)
	at com.sun.xml.internal.bind.v2.runtime.JAXBContextImpl.getOrCreate(JAXBContextImpl.java:513)
	...
	at com.sun.xml.internal.bind.v2.runtime.ClassBeanInfoImpl.<init>(ClassBeanInfoImpl.java:166)
	at com.sun.xml.internal.bind.v2.runtime.JAXBContextImpl.getOrCreate(JAXBContextImpl.java:494)
	at com.sun.xml.internal.bind.v2.runtime.ClassBeanInfoImpl.<init>(ClassBeanInfoImpl.java:153)
	at com.sun.xml.internal.bind.v2.runtime.JAXBContextImpl.getOrCreate(JAXBContextImpl.java:494)
	at com.sun.xml.internal.bind.v2.runtime.ClassBeanInfoImpl.<init>(ClassBeanInfoImpl.java:153)
```
<!-- more -->

I checked with [Google](http://www.google.com) and found [there](http://java.net/projects/jaxb/lists/users/archive/2011-09/message/2) [are](http://stackoverflow.com/questions/7810599/null-pointer-when-parsing-camelcontext-xml-file-during-startup) already hunderds of similar reports about this, especially those from [Camel community](http://camel.465427.n5.nabble.com/Occational-NPE-in-CamelNamespaceHandler-td4726744.html). This is sad. I still remembered the [big](http://www.zdnet.com/blog/burnette/oracle-rebrands-java-breaks-eclipse/2012) [regression](http://bugs.sun.com/bugdatabase/view_bug.do?bug_id=6969236) when JDK 6 first came out.

I could not find some good solutions for this except for those suggestion about upgrading to JAXB 2.2.5. I dislike the endorsed idea very much. So no other choice, I had to dig into code myself.

```java ClassBeanInfoImpl.checkOverrideProperties http://bit.ly/IVmRqm OpenJDK 7
private void checkOverrideProperties(Property p) {
        ClassBeanInfoImpl bi = this;
        while ((bi = bi.superClazz) != null) {
            for (Property superProperty : bi.properties) {
                if (superProperty == null) break;
                String spName = superProperty.getFieldName();
                if ((spName != null) && (spName.equals(p.getFieldName()))) {
                    superProperty.setHiddenByOverride(true);
                }
            }
        }
    }
```
The NPE is thrown from line 4 in above snippet. And through some dirty debugging, seems to me in certain occation, if a class A inherits a class B which has no JAXB related properties defined, such exception gets thrown out. And actually after I modified some classes hierarchy which do have similar pattern, the NPE is gone. 