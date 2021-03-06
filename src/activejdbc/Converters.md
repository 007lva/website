Title: Explains ActiveJDBC converters

Introduction
============

Converters can convert an attribute from one type to another. They are very similar to [Validations](Validations) in logic and are declared the same way inside a static block of a class.

Currently there are two converters available: date converter and time stamp converter. Both allow setting an attribute value in a form of a string with appropriate format.

Converters trigger like validators, during validate(), save() and saveIt() methods, and are in fact validators. If a converter cannot convert from one type to another, it will add an appropriate message to the `model.errors()` collection.

Date converter
==============

Date converter can convert value for an attribute from formatted string to `java.sql.Date`. Here is an example of a date converter declaration:

~~~~ {.prettyprint}
public class Person extends Model {
    static{
        convertDate("dob", "yyyy-MM-dd");
    }    
}
~~~~

and here how to use it:

~~~~ {.prettyprint}
Person p = new Person();
p.set("first_name", "Marilyn","last_name", "Monroe");
p.set("dob", "1926/6/1");                     //wrong format
p.validate();
System.out.println(p.errors().get("dob"));    //prints out: attribute dob does not conform to format: yyyy-MM-dd

p.set("dob", "1926-06-01");                   //right format
p.validate();
System.out.println(p.get("dob").getClass()); //prints out: class java.sql.Date
System.out.println(p.get("dob"));            //prints out: 1926-06-01
~~~~

Timestamp converter
===================

Timestamp converter is identical to date converter, but converts to `java.sql.Timestamp`. Here is example of declaration:

~~~~ {.prettyprint}
public class Message extends Model {
    static{
        convertTimestamp("send_time", "yyyy.MM.dd G 'at' HH:mm:ss z");
    }    
}
~~~~

* * * * *

In both cases, the text format for date and timestamp needs to conform to [java.text.SimpleDateFormat](http://download.oracle.com/javase/6/docs/api/java/text/SimpleDateFormat.html)

* * * * *
