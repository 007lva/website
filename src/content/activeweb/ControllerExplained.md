Title: Learn all about ActiveWeb controllers

[Introduction](#Introduction)

[Controller actions](#Controller_actions)

[Controllers and HTTP methods](#Controllers_and_HTTP_methods)

[RESTful controllers](#RESTful_controllers)

[Location of controllers in project](#Location_of_controllers_in_project)

[Controller paths](#Controller_paths)

[V in MVC](#V_in_MVC)

[Mapping to views](#Mapping_to_views)

-   [Default mapping to views](#Default_mapping_to_views)
-   [Override mapping to relative views](#Override_mapping_to_relative_views)
-   [Override mapping to absolute views](#Override_mapping_to_absolute_views)

[Passing data to views](#Passing_data_to_views)

[Getting request parameters](#Getting_request_parameters)

-   [getting a single parameter](#getting_a_single_parameter)
-   [getting all parameters](#getting_all_parameters)
-   [getting all values for a single parameter](#getting_all_values_for_a_single_parameter)
-   [getting a first value from each submitted parameter](#getting_a_first_value_from_each_submitted_parameter)

[Ajax APIs](#Ajax_APIs)

-   [detecting if a request is Ajax](#detecting_if_a_request_is_Ajax)
-   [Responding to Ajax call directly](#Responding_to_Ajax_call_directly)
-   [Responding to Ajax call with a view](#Responding_to_Ajax_call_with_a_view)

[Downloading of files](#Downloading_of_files)

[Uploading files](#Uploading_files)

[Session management](#Session_management)

[Cookies management](#Cookies_management)

-   [Sending cookies to a client](#Sending_cookies_to_a_client)
-   [Getting cookies from request](#Getting_cookies_from_request)

[Logging](#Logging)

[Threading issues](#Threading_issues)

Introduction
============

Controllers are at the heart of an ActiveWeb application. These are classes wich designed to process an HTTP request. They are somewhat similar to Java Servlets, but even more similar to Ruby on Rails, Grails anall sind SpringMVC controllers. A simplest controller looks like this:

~~~~ {.prettyprint}
public class GreetingController extends AppController{
   public void index(){}
}
~~~~

A controller above is a working controller, even though it does not have any code in it. Such a controller is automatically mapped to a URL"

~~~~ {.prettyprint}
http://host:port/context/greeting
~~~~

Controller actions
==================

A controller action is a method that is designed to process an HTTP request. For example, in the example above, the URL will be mapped to the "action" index by default. However, if the URL looked like this:

~~~~ {.prettyprint}
http://host:port/context/greeting/hello
~~~~

then it would map to action "hello":

~~~~ {.prettyprint}
public class GreetingController extends AppController{
   public void hello(){}
}
~~~~

Controllers and HTTP methods
============================

The HTTP specification defines methods: GET, POST, DELETE, PUT, CONNECT and HEAD. Currently ActiveWeb supports GET, POST, DELETE, PUT and HEAD as the most used methods.

An HTTP request is not only mapped to an action, but also its HTTP method must correspond to an action's HTTP method. Action HTTP methods are configured with annotations, `@GET, @POST, @PUT, and @DELETE, @HEAD`.

* * * * *

However, if annotation is missing, an action is mapped to HTTP GET method by default

* * * * *

This makes it unnecessary to use annotations except cases when you need so-called "destructive" action. A destructive action is the one that is designed to change a state of a resource (in REST style of web programming).

Here is an example of a destructive action is:

~~~~ {.prettyprint}
public class BooksController extends AppController{

   @PUT
   public void updateBook(){}
}
~~~~

ActiveWeb promotes REST-style web programming and will not allow to access an action that is configured for one HTTP method with another. For instance, if you try to access the `GreetingController#index` with HTTP method POST, you would get an exception:

~~~~ {.prettyprint}
activeweb.ControllerException: Cannot access action 
app.controllers.GreetingController.index with HTTP method: 'POST' because it is 
configured for method: 'GET'
~~~~

RESTful controllers
===================

A restful controller is almost the same as a regular controller. The difference is that in a standard controller you can define any number of actions and configure them with annotations to accept any HTTP methods. RESTful controllers by contrast have a fixed set of actions and these actions will accept certain HTTP methods. In order to create a RESTful controller, you need to do two things:

1.  Add `@RESTful` annotation to the controller class
2.  Provide a fixed set of action methods in the body of controller

Here is an example of a RESTful controller taken from Kitchensink application:

~~~~ {.prettyprint}
@RESTful
public class RpostsController extends AppController {

    public void index(){...}

    public void newForm(){...}

    public void create(){...}

    public void show(){...}

    public void destroy(){...}
~~~~

A RESTful controller is allowed to have 7 methods, and they all are automatically mapped to the following URSs and HTTP methods:

~~~~ {.prettyprint}
==========================================================================================
verb    path                   action          used for
==========================================================================================
GET     /books                 index           display a list of all books
GET     /books/new_form        new_form        return an HTML form for creating a new book
POST    /books                 create          create a new book
GET     /books/id              show            display a specific book
GET     /books/id/edit_form    edit_form       return an HTML form for editing a books
PUT     /books/id              update          update a specific book
DELETE  /books/id              destroy         delete a specific book
~~~~

There is no need to add any other annotations to RESTful controllers.

If you examine the table above, you will see that a combination of a path and HTTP method is mapped to an action. For instance, a path '/books' is mapped twice, with GET HTTP method to action `index`, an with POST to action `create`. This makes for elegant, REST-style URLs and leads ultimately to better user experience.

Location of controllers in project
==================================

The page [Structure of a web project](StructureOfActiveWebProject) states that controllers are located in a package:

~~~~ {.prettyprint}
app.controllers
~~~~

ActiveWeb specifies that all controllers are located in this package or sub-packages.

Controller paths
================

A controller path is made of a sub-package and a controller name. A controller name is *not* a class name, but rather underscored, flattened name part of a class simple name.

Example 1

~~~~ {.prettyprint}
package app.controllers;
class BooksController{..} // ===> controller path: /books
~~~~

Example 2:

~~~~ {.prettyprint}
package app.controllers;
class UniversityBooksController{..} // ===> controller path: /university_books
~~~~

Example 3

~~~~ {.prettyprint}
package app.controllers.depaul;
class UniversityBooksController{..} // ===> controller path: /depaul/university_books
~~~~

Example 4

~~~~ {.prettyprint}
package app.controllers.depaul.chicago;
class UniversityBooksController{..} // ===> controller path: /depaul/chicago/university_books
~~~~

V in MVC
========

ActiveWeb does not use JSPs. Period. Instead it uses [FreeMarker](http://freemarker.sourceforge.net/) (FM). The FM templates are located in this directory:

~~~~ {.prettyprint}
src/main/webapp/WEB-INF/views
~~~~

Mapping to views
================

The directory: `src/main/webapp/WEB-INF/views` has subdirectories. These subdirectories are named after controller paths. This makes it easy to find FM templates (views) associated with controllers.

Default mapping to views
------------------------

Under normal circumstances, the HTTP request is applied to an action, and then the framework passes control to a view. Under these conditions, there is no need for any configuration or code.

Example: if a HTTP GET request is sent to this URL: `http://hostname/context/greeting`, then the framework will invoke a GreetingControoller, and by default action `index`:

~~~~ {.prettyprint}
public class GreetingController extends AppController{
   public void index(){}
}
~~~~

After execution of action, the ActiveWeb will find a template:

~~~~ {.prettyprint}
src/main/webapp/WEB-INF/views/greeting/index.ftl
~~~~

will render it, and send results to the browser.

ActiveWeb in the same spirit as ActiveJDBC does not have any configuration files. Most actions are based on conventions.

Override mapping to relative views
----------------------------------

In some cases, you need to override default mapping to views. You will use a `render()` method for this:

~~~~ {.prettyprint}
public class GreetingController extends AppController{
   public void index(){
      //some code here
      render("show");
   }
}
~~~~

The `render("show");` will signal to ActiveWeb that instead of "index", you want to render "show". Since you did not provide any other information, ActiveWeb will assume that this view will be found at the same location:

~~~~ {.prettyprint}
src/main/webapp/WEB-INF/views/greeting/show.ftl
~~~~

Override mapping to absolute views
----------------------------------

Sometimes you need to call a view that "belongs" to a different controller, or even some shared view. In that case, you can specify an "absolute" path to a view like this:

~~~~ {.prettyprint}
public class GreetingController extends AppController{
   public void index(){
      //some code here
      render("/shared/show");
   }
}
~~~~

ActiveWeb will use the following view to render:

~~~~ {.prettyprint}
src/main/webapp/WEB-INF/views/shared/show.ftl
~~~~

Passing data to views
=====================

Passing data to views is done with two methods:

-   assign(name, value)
-   view(name, value)

There is no difference between these methods, they are aliases.

Example:

~~~~ {.prettyprint}
public class GreetingController extends AppController{
   public void index(){
       view("name", "John Doe");
   }
}
~~~~

The corresponding view can look like this:

~~~~ {.prettyprint}
Hello, my name is: ${name}
~~~~

You could have guessed that the output will look like:

~~~~ {.prettyprint}
"Hello, my name is: John Doe"
~~~~

Getting request parameters
==========================

Getting request parameters is the most important part of any web application. ActiveWeb provides a few methods to achieve this goal:

getting a single parameter
--------------------------

~~~~ {.prettyprint}
public class GreetingController extends AppController{
   public void index(){
       String name = param("name");
   }
}
~~~~

getting all parameters
----------------------

~~~~ {.prettyprint}
public class GreetingController extends AppController{
   public void index(){
      Map<String, String[]> allParams = params();
   }
}
~~~~

getting all values for a single parameter
-----------------------------------------

This is in cases for submit parameters with multiple values, such as selects:

~~~~ {.prettyprint}
public class GreetingController extends AppController{
   public void index(){
      List<String> states = params("states");
   }
}
~~~~

getting a first value from each submitted parameter
---------------------------------------------------

This is in cases for submit parameters with multiple values, such as selects:

~~~~ {.prettyprint}
public class GreetingController extends AppController{
   public void index(){
      Map<String, String> firstValues = params1st()
   }
}
~~~~

The return value is a map where keys are names of all parameters, while values are first value for each parameter, even if such parameter has more than one value submitted.

Ajax APIs
=========

detecting if a request is Ajax
------------------------------

Controllers (and filters alike) provide a simple way to detect if a request is a from an XmlHttpRequest in browser:

~~~~ {.prettyprint}
public class GreetingController extends AppController{
   public void index(){
      if(xhr()){
         ...
      }else{
         ...
      }
   }
}
~~~~

The `xhr();` method also has an alias: `isXhr();`.

Responding to Ajax call directly
--------------------------------

ActiveWeb provides a simple method `respond(..)`:

~~~~ {.prettyprint}
public class GreetingController extends AppController{
   public void index(){
      respond("<message>hello</message>").contentType("text/xml").status(200);
   }
}
~~~~

It is easy to use this method to send plain text, XML, JSON, or any other text format as a response to Ajax call.

Responding to Ajax call with a view
-----------------------------------

Method `respond()` is usually used to respond with quick HTML snippet, or generated JSON or other format. However, if you need to respond with a more complex structure, and potentially generate it with a view and use a full power of a view expression language for condition, iteration, etc, you can use a regular `render()` method, the one that is also used for rendering HTML pages. However, for Ajax calls, layouts are not necessary, so the call will look like this:

~~~~ {.prettyprint}
public class AjaxController extends AppController{
   public void index(){
      //some code here
      render().noLayout();
   }
}
~~~~

The URL for this call will look like this: `http://host/context/ajax`. The Ajax actions act just like any other actions, they support all HTTP methods and annotations.

Downloading of files
====================

There are a few calls you can use for file download:

-   sending files to a client:

~~~~ {.prettyprint}
public class GetPdfController extends AppController{
   public void index(){   
     File f;
      //... obtain file
     sendFile(f).contentType("application/pdf").status(200);
   }
}
~~~~

-   Streaming bytes to a client:

~~~~ {.prettyprint}
public class GetPdfController extends AppController{
   public void index(){   
     byte[] bytes;
      //... obtain data
     outputStream("application/pdf").write(bytes);
   }
}
~~~~

-   Reading from InputStream and sending to client:

~~~~ {.prettyprint}
public class GetPdfController extends AppController{
   public void index(){   
     InputStream in;
      //... point input stream to data
     streamOut(in).contentType("applicaiton/pdf");
   }
}
~~~~

Uploading files
===============

Controllers have a built-in facility for uploading data:

~~~~ {.prettyprint}
public class UploadController extends AppController{
   public void index(){   

        Iterator<FormItem> iterator = uploadedFiles();
        
        if (iterator.hasNext()){
            FormItem item = iterator.next();
            String name = item.getFileName();
            if(item.isFile()){
                InputStream in = item.getInputStream();
                ///process data
            }               
        }
}
~~~~

Session management
==================

A session object is accessed with a `session()` call inside controllers:

~~~~ {.prettyprint}
public class HomeController extends AppController{
   public void index(){   
     // get a value
     session().put("name", value);
   }
~~~~

Other methods available on a session object are:

~~~~ {.prettyprint}
  session().get(String);            // to get a single value
  session().remove(name);           // remove object by name
  session().names();                // to get a list of all objects in session
  session().getCreationTime();      // obvious
  session().invalidate();           // invalidate session
  session().setTimeToLive(seconds); // set time to live
~~~~

Cookies management
==================

Sending cookies to a client
---------------------------

~~~~ {.prettyprint}
sendCookie(Cookie);               //sends a cookie to client
sendCookie(name, value);          //simple short cut to do the same as above 
sendPermanentCookie(name, value); //will send a cookie with time to live == 20 years
~~~~

Getting cookies from request
----------------------------

~~~~ {.prettyprint}
List<Cookie> cookies();//gets a list of all cookies sent in request
Cookie cookie(name);   //retrieve an individual cookie
cookieValue(name);     //retrieve a cookie value by name of cookie
~~~~

Logging
=======

Controllers already have a way to log information to a log system. These methods are available for logging:

~~~~ {.prettyprint}
logDebug(..);
logWarning(..);
logError(..);
logInfo(..);
~~~~

Threading issues
================

Controllers are thread safe. An instance of a controller is created to process each and every request, and then discarded. This means that if you create instance variables in controller, this variable will not interfere with another value form another request.

However, creation of instance variables in controller classes is considered a bad coding practice in general, because it is vulnerable to side effects. While controllers are objects in OO language (Java), they need to be treated as procedural devices when it comes to processing HTTP requests. This means that every action should be completely self-sufficient and not rely on some instance variable set by another action or that a method invoked from an action should not depend on an instance variable set higher up the stack.