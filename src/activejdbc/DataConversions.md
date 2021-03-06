Title: How ActiveJDBC performs data conversions

-   [Pass-through framework](#Pass_through_framework)
-   [How to see mapped type](#How_to_see_mapped_type)
-   [Convenience conversion methods](#Convenience_conversion_methods)
-   [Custom setters and getters](#Custom_setters_and_getters)
-   [What happens if I stick a wrong type?](#What_happens_if_I_stick_a_wrong_type_)
-   [Different data types for the same attribute](#Different_data_types_for_the_same_attribute)
-   [Should I care about types?](#Should_I_care_about_types_)

Pass-through framework
======================

ActiveJDBC is a **pass-through framework**. This means that for the most part, it does not attempt to convert data types and relies on the underlying driver for most data conversions during read and write operations. Most database drivers do a great deal of conversion

-   **select operations**: during select operations, the driver will populate your models with whatever types that are mapped from DB table columns to appropriate Java types.
-   **insert/update operations**: during inserts or updates, the DBMS driver will perform conversions where necessary and possible. In cases a conversion is not possible, you are going to get a runtime exception from DBMS (see below)

How to see mapped type
======================

This is pretty easy, insert a record into a table using standard DBMS tools, and then query it like this:

~~~~ {.prettyprint}

Person p = Person.findAll().get(0);
System.out.println("DOB type: " + p.get("dob").getClass());
~~~~

Convenience conversion methods
==============================

ActiveJDBC provides a number of getter and setter methods for conversion. Most conversions use the underlying [Convert](http://code.google.com/p/activejdbc/source/browse/trunk/javalite-common/src/main/java/javalite/common/Convert.java) class. Here are some examples:

~~~~ {.prettyprint}

Date date = person.getDate("dob");
Boolean projectCompleted = project.getBoolean("completed");
...
~~~~

Most all types can be converted to String, including a CLOB. Consider this example, where column CONTENT is defined as CLOB:

~~~~ {.prettyprint}
String text = article.getString("content");
~~~~

Custom setters and getters
==========================

If you like more control over types, you can provide typed getters and setters:

~~~~ {.prettyprint}
public class Person extends Model{
...
public void setName(String name){
   set("name", name);
}
...
}
~~~~

and then use it like this:

~~~~ {.prettyprint}
Person p = new Person();
p.setName("John");
p.setDob(new Date(12345L));
~~~~

For more information on setters and getters, see [SettersAndGetters](SettersAndGetters)

What happens if I stick a wrong type?
=====================================

If you do this:

~~~~ {.prettyprint}
Student p = new Student();
p.set("first_name", "John");
p.set("last_name", "Doe");
p.set("dob", 1); // ===>>  this is wrong type for DOB field
p.saveIt();
~~~~

you are going to get an exception from the DBMS itself, if it cannot convert the type:

~~~~ {.prettyprint}

activejdbc.DBException: com.mysql.jdbc.MysqlDataTruncation: Data truncation: Incorrect date value: '1' for column 'dob' at row 1, Query: INSERT INTO students (first_name, dob, last_name) VALUES (?, ?, ?), params: John,1,Doe
    at com.mysql.jdbc.MysqlIO.checkErrorPacket(MysqlIO.java:2868)
    at com.mysql.jdbc.MysqlIO.sendCommand(MysqlIO.java:1573)
    at com.mysql.jdbc.ServerPreparedStatement.serverExecute(ServerPreparedStatement.java:1169)
    at com.mysql.jdbc.ServerPreparedStatement.executeInternal(ServerPreparedStatement.java:693)
~~~~

Different data types for the same attribute
===========================================

Is this possible? Yes, for instance, take this example:

~~~~ {.prettyprint}
p.set("dob", "2011-12-3");
p.saveIt();
~~~~

This will work for MySQL, and the driver will convert the String "2011-12-3" during insert or update. However, if you retain a reference to this model, the internal value type is going to remain String.

Should I care about types?
==========================

Not so much. Just use the models with types you think are appropriate, and ActiveJDBC together with the driver will handle most of what you need. In case you get a conversion exception from the driver, do a due diligence then.

back to [Features](Features)
