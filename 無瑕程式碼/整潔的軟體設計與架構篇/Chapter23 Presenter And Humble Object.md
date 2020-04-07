# **Presenter And Humble Object**

## **The Humble Object Pattern**
### The idea is very simple: Split the behaviors into two modules or classes. One of those modules is humble; it contains all the hard-to-test behaviors stripped down to their barest essence. The other module contains all the testable behaviors that were stripped out of the humble object.

<br>

## **Presenter And View**
### The View is the humble object that hard to test. The code in this object is kept as simple as possible. It moves data into the GUI but does not process that data.

### The Presenter is the testable object. Its job is to accept data from the application and format it for presentation so that View can simply move it to the screen.

### Nothing is left for the View to do other than to load the data from the View Model into the screen. Thus the View is humble.

<br>

## **Database Gateways**
### Between the use case interactors and the database are the database gateways. These gateways are polymorphic interfaces that contain methods for every create, read, update, or delete operations that can be performed by the application on the database.

### Recall that we do not allow SQL in the use cases layer; instead, we use gateway interfaces that have appropriate methods. Those gate ways are implemented by classes in the database layer. That implementation is the humble object.

### The interactors, in contrast, are not humble beacuse they encapsulate application-specific business rules. Although they are not humble, those interactors are *testable*, because the gateways can be replaced with appropriate stubs and test-doubles.

<br>

## **Data Mappers**
### Going back to the topic of databases, in which layer do you think ORMs like Hibernate belong?

### In the database layer of course. Indeed, ORMs form another kind of *Humble Object* boundary between the gateway interfaces and the database.

<br>

## **Service Listeners**
### What about services? If your application must communicate with other services, or if your application provides a set of services, will we find the *Humble Object* pattern creating a service boundary?

### Of course! The application will load data into simple data structures and then pass those structures across the boundary to modules that properly format the data and send it to external services.

### On the input side, the service listeners will receive data from the service interface and format it into a simple data structure that can be used by the application. That data structure is then passed across the service boundary.
