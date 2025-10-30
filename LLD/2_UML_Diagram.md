# Unified Modeling Language (UML)

# Resources

### GeeksforGeeks Articles
- **Introduction to UML**: [https://www.geeksforgeeks.org/system-design/unified-modeling-language-uml-introduction/](https://www.geeksforgeeks.org/system-design/unified-modeling-language-uml-introduction/)
- **Class Diagrams**: [https://www.geeksforgeeks.org/system-design/unified-modeling-language-uml-class-diagrams/](https://www.geeksforgeeks.org/system-design/unified-modeling-language-uml-class-diagrams/)
- **Sequence Diagrams**: [https://www.geeksforgeeks.org/system-design/unified-modeling-language-uml-sequence-diagrams/](https://www.geeksforgeeks.org/system-design/unified-modeling-language-uml-sequence-diagrams/)
- **Introduction to ER Model**: [https://www.geeksforgeeks.org/dbms/introduction-of-er-model/](https://www.geeksforgeeks.org/dbms/introduction-of-er-model/)

### Videos

#### Hindi
- **UML Overview**: [https://www.youtube.com/watch?v=nPJyyO9pb5s](https://www.youtube.com/watch?v=nPJyyO9pb5s)

#### English
- **Class UML**: [https://www.youtube.com/watch?v=6XrL5jXmTwM](https://www.youtube.com/watch?v=6XrL5jXmTwM)
- **Sequence UML**: [https://www.youtube.com/watch?v=pCK6prSq8aw](https://www.youtube.com/watch?v=pCK6prSq8aw)
- **ER Diagram Part 1**: [https://www.youtube.com/watch?v=xsg9BDiwiJE](https://www.youtube.com/watch?v=xsg9BDiwiJE)
- **ER Diagram Part 2**: [https://www.youtube.com/watch?v=hktyW5Lp0Vo](https://www.youtube.com/watch?v=hktyW5Lp0Vo)

----
<br><br>
# Revision Questions

**Source**: [https://www.c-sharpcorner.com/UploadFile/questpond/uml-interview-questions/](https://www.c-sharpcorner.com/UploadFile/questpond/uml-interview-questions/)

### 1. Define UML?
Unified Modeling Language, a standard language for designing and documenting a system in an object-oriented manner. It has nine diagrams which can be used in design document to express design of software architecture.

### 2. Can you explain use case diagrams?
Use case diagram answers what system does from the user point of view. Use case answer 'What will the system do?'. Use cases are mainly used in requirement document to depict clarity regarding a system. There are three important parts in a use case scenario, actor and use case.  
Scenario: A scenario is a sequence of events which happen when a user interacts with the system.  
Actor: Actor is the who of the system, in other words the end user.  
Use Case: Use case is task or the goal performed by the end user. Below figure shows a simple scenario with 'Actor' and a 'Use Case'. Scenario represents an accountant entering accounts data in the system. As use case's represent action performed they are normally represented by strong verbs.  
Actor's are represented by simple stick man and use case by oval shape as shown in figure below.

### 3. Can you explain primary and secondary actors?
Actors are further classified in to two types primary and secondary actors. Primary actors are the users who are the active participants and they initiate the user case, while secondary actors are those who only passively participate in the use case.

### 4. How does a simple use case look like?
Use case's have two views of representation in any requirement document. One is the use case diagrams and the other is a detail step table about how the use case works. So it's like a pair first an over view is shown using a use case diagram and then a table explaining the same in detail. Below is a simple 'login' use case shown diagrammatically and then a detail table with steps about how the use case is executed.

| Use Case Rel001 | |
|-----------------|--------|
| **Use Case Name** | Login |
| **Description** | This uses depicts the flow of how user will log-in into the chat application. |
| **Primary Actor** | Simple chat user. |
| **Trigger** | User types chat application on URL of the browser. |
| **Pre-condition** | NA |
| **Assumption** | No password is currently present for the system. Rooms will remain constant as explained in the assumption section of this document. |
| **Failed End conditions** | Duplicate user name is not allowed in the chat application. |
| **Action** | User clicks on the log-in button. |
| **Main Scenario** | 1. User types chat application on URL of the browser which in turn opens the main page. 2. In the main page of application user is popped up with 'Enter user name' option and various 'rooms' option drop down menu. 3. User then types the name and selects one of the room from drop down menu and then clicks on the 'Log-in' button. 4. Application then checks whether the user name is unique in the system if not then user is popped up with error message that "user already exist". 5. After entering the unique name the user is finally logged in the application. |
| **Action** | NA |
| **Alternate Scenario** | NA |
| **Success Scenarios** | 1. Opens page of a selected room in that other user names and their messages can be seen. |
| **Note and Open Issues** | NA |

Note: You must be wondering why we have this pair why not just a use case table only. Use case diagrams are good to show relationship between use case and they also provide high over view. The table explanation of a use case talks details about the use case. So when a developer or a user is reading a requirement document, he can get an overview by looking at the diagram if he is interested he can read the use case tables for more details.

### 5. Can you explain 'Extend' and 'Include' in use cases?
'Extend' and 'Include' define relationships between use cases. Below figure shows how these two fundamentals are implemented in a project. The below use case represents a system which is used to maintain customer. When a customer is added successfully it should send an email to the admin saying that a new customer is added. Only admin have rights to modify the customer. First lets define extend and include and then see how the same fits in this use case scenario.  
Include: Include relationship represents an invocation of one use case by the other. If you think from the coding perspective its like one function been called by the other function.  
Extend: This relationship signifies that the extending use case will work exactly like the base use case only that some new steps will inserted in the extended use case.  
Below figure shows that 'add customer' is same as the 'add discounted customer'. The 'Add discounted customer' has an extra process, to define discount for the discounted customer which is not available for the simple customer. One of the requirements of the project was that when we add a customer, the system should send an email. So after the customer is added either through 'Add simple customer' use case or 'Add discounted customer' use case it should invoke 'send a email' use case. So we have defined the same with a simple dotted line with <<include>> as the relationship.

Note: One of the points to be noted in the diagram is we have defined inheritance relationship between simple and admin user. This also helps us defining a technical road map regarding relationships between simple and admin user.

### 6. Can you explain class diagrams?
Class is basically a prototype which helps us create objects. Class defines the static structure of the project. A class represents family of an object. By using Class we can create uniform objects.  
In the below figure you can see how the class diagram looks. Basically there are three important sections which are numbered as shown in the below. Let's try to understand according to the numbering:  
Class name: This is the first section or top most section of the Class which represents the name of the Class (clsCustomer).  
Attributes: This is the second section or the middle section of the class which represents the properties of the system.  
Methods: This section carries operation or method to act on the attributes.

Now in the next section we will have a look on Association relationship between these classes.

### 7. How do we represent private, public and protected in class diagrams?
In order to represent visibility for properties and methods in class diagram we need to place symbols next to each property and method as shown in figure. '+' indicates that it's public properties/methods. '-'indicates private properties which means it can not be accessed outside the class. '#' indicate protected/friend properties. Protected properties can only be seen within the component and not outside the component.

### 8. What does associations in a class diagram mean?
A single Class cannot represent the whole module in a project so we need one or more classes to represent a module. For instance, a module named 'customer detail' cannot be completed by the customer class alone , to complete the whole module we need customer class, address class, phone class in short there is relationship between the classes. So by grouping and relating between the classes we create module and these are termed as Association. In order to associate them we need to draw the arrowed lines between the classes as shown in the below figure.  
In the figure, we can see Order class and the Payment class and arrowed line showing relationship that the order class is paid using payment class in other words order class is going to be used by payment class to pay the order. The left to right marked arrow basically shows the flow that order class uses the payment class.  
In case payment class using the order class then the marked arrow should be right to left showing the direction of the flow.

There are four signs showing the flow.

Multiplicity  
Multiplicity can be termed as classes having multiple associations or one class can be linked to instances of many other classes. If you look at the below figure the customer class is basically associated with the address class and also observes the notations (*, 0 and 1).If you look at the right hand side the (1….*) notation indicates that at least one or many instance of the address class can be present in the customer class. Now towards left hand side we have (0….*) notation indicating that address class can exist without or many customer class can link him.  
In order to represent multiplicity of classes we have to show notations like (1....*), (0....*) as shown in below figure.  
Note: '*' means "many" where as '(0, 1)' means "(zero or at least one)" respectively.

### 9. Can you explain aggregation and composition in class diagrams?
In this Association there are two types mainly Aggregation Association and Composition Association.  
Aggregation Association signifies that the whole object can exist without the Aggregated Object. For example in the below figure we have three classes university class, department class and the Professor Class. The university cannot exist without department which means that university will be closed as the department is closed. In other words lifetime of the university depend on the lifetime of department.  
In the same figure we have defined second Association between the department and the Professor. In this case, if the professor leaves the department still the department continues in other words department is not dependent on the professor this is called as Composition Association.  
Note: The filled diamond represents the aggregation and the empty diamond represents the composition. You can see the figure below for more details.

### 10. What are composite structure diagram and reflexive association in class diagrams?
Composite structure diagram  
When we try to show Aggregation and Composition in a complete project the diagram becomes very complicated so in order to keep it simple we can use Composite structure diagram. In the below figure we have shown two diagrams one is normal diagram other is Composite structure diagram and the simplicity can easily be identified. In the composite diagram the aggregated classes are self contained in the main class which makes it simpler to read.

Reflexive associations  
In many scenarios you need to show that two instances of the same class are associated with each other and this scenario is termed as Reflexive Association. For instance in the below figure shows Reflexive Association in the real project. Here you can see customer class has multiple address class and addresses can be a Head office, corporate office or Regional office. One of the address objects is Head office and we have linked the address object to show Reflexive Association relationship. This is the way we can read the diagram Regional address object is blocked by zero or one instance of Head office object.

### 11. Can you explain business entity and service class?
Business entity objects represent persistent information like tables of a database. Just making my point clearer they just represent data and do not have business validations as such. For instance below figure shows a simple customer table which with three fields 'Customer Code',' Customer Address' and 'Phone Number'. All these fields are properties in 'ClsCustomer' class. So 'ClsCustomer' class becomes the business entity class. The business entity class by itself can not do anything it's just a place holder for data. In the same figure we have one more class 'ClsServiceCustomer'. This class aggregates the business entity class and performs operations like 'Add',' Next' (Move to next record), 'Prev' (Move to previous record) and 'GetItem' (get a customer entity depending on condition).  
With this approach we have separated the data from the behavior. The service represents the behavior while the business entity represents the persistent data.

### 12. Can you explain System entity and service class?
System entity class represents persistent information which is related to the system. For instance in the below figure we have a system entity class which represents information about 'loggedindate' and 'loggedintime' of the system registry. System service class come in two flavors one is it acts like a wrapper in the system entity class to represent behavior for the persistent system entity data. In the figure you can see how the 'ClsAudit' system entity is wrapped by the 'ClsAuditSytem' class which is the system service class. 'ClsAuditSystem' adds 'Audit' and 'GetAudit' behavior to the 'ClsAudit' system entity class.

The other flavor of the system service class is to operate on non-persistent information. The first flavor operated on persistent information. For instance the below figure shows how the class 'ClsPaymentService' class operates on the payment gateway to Check is the card exists , Is the card valid and how much is the amount in the card ?. All these information are non-persistent. By separating the logic of non-persistent data in to a system service class we bring high reusability in the project.

Note: The above question can be asked in interview from the perspective of how you have separated the behavior from the data. The question will normally come twisted like 'How did you separate the behavior from the data?'.

### 13. Can you explain generalization and specialization?
In Generalization and Specialization we define the parent-child relationship between the classes. In many instance you will see some of the classes have same properties and operation these classes are called super class and later you can inherit from super class and make sub classes which have their own custom properties. In the below figure there are three classes to show Generalization and Specialization relationship. All phone types have phone number as a generalized property but depending upon landline or mobile you can have wired or simcard connectivity as specialized property. In this diagram the clsphone represent Generalization whereas clslandline and clsmobile represents specialization.

### 14. How do we achieve generalization and specialization?
By using inheritance.

### 15. Can you explain object diagrams in UML?
Class represents shows the static nature of the system. From the previous question you can easily judge that class diagrams shows the types and how they are linked. Classes come to live only when objects are created from them. Object diagram gives a pictorial representation of class diagram at any point of time. Below figure shows how a class looks in when actual objects are created. We have shown a simple student and course relationship in the object diagram. So a student can take multiple courses. The class diagram shows the same with the multiplicity relationship. We have also shown how the class diagram then looks when the objects are created using the object diagram. We represent object with Object Name: Class Name. For instance in the below figure we have shown 'Shiv : ClsStudent' i.e 'Shiv' is the object and 'ClsStudent' the class. As the objects are created we also need to show data of the properties, the same is represented by 'PropertyName=Value' i.e. 'StudentName=Shiv'.

The diagram also states that 'ClsStudent' can apply for many courses. The same is represented in object diagram by showing two objects one of the 'Computer' and the other of 'English'.  
Note: Object diagrams should only be drawn to represent complicated relationship between objects. It's possible that it can also complicate your technical document as lot. So use it sparingly.

### 16. Can you explain sequence diagrams?
Sequence diagram shows interaction between objects over a specific period time. Below figure shows how a sequence diagram looks like. In this sequence diagram we have four objects 'Customer','Product','Stock' and 'Payment'. The message flow is shown vertically in waterfall manner i.e. it starts from the top and flows to the bottom. Dashed lines represent the duration for which the object will be live. Horizontal rectangles on the dashed lines represent activation of the object. Messages sent from a object is represented by dark arrow and dark arrow head. Return message are represented by dotted arrow. So the figure shows the following sequence of interaction between the four objects:  
Customer object sends message to the product object to request if the product is available or not.  
Product object sends message to the stock object to see if the product exists in the stock.  
Stock object answers saying yes or No.  
Product object sends the message to the customer object.  
Customer object then sends a message to the payment object to pay money.  
Payment object then answers with a receipt to the customer object.  
One of the points to be noted is product and stock object is not active when the payment activity occurs.

Messages in sequence diagrams  
There are five different kinds of messages which can be represented by sequence.  
Synchronous and asynchronous messages  
Synchronous messages are represented by a dark arrow head while asynchronous messages are shown by a thin arrow head as shown in figure.

Recursive message  
We have scenarios where we need to represent function and subroutines which are called recursively. Recursive means the method calling himself. Recursive messages are represented by small rectangle inside a big rectangle with an arrow going from the big rectangle to the small rectangle as shown in figure.

Message iteration  
Message iteration represents loops during sequences of activity. Below figure shows how 'order' calls the 'orderitem' objects in a loop to get cost. To represent loop we need to write 'For each <<object name>>'. In the below figure the object is the 'orderitem'. Also note the for each is put in a box to emphasize that it's a loop.

Message constraint  
If we want to represent constraints it is put in a rectangle bracket as shown in figure. In the below figure the 'customer' object can call 'book tickets' only if the age of the customer is greater than 10.

Message branching  
Below figure shows how 'customer' object have two branches one is when the customer calls save data and one when he cancels the data.

Doing Sequence diagram practically  
Let's take a small example to understand sequence diagram practically. Below is a simple voucher entry screen for accounts data entry. Following are the steps how the accountant will do data entry for the voucher:  
Accountant loads the voucher data entry screen. Voucher screen loads with debit account codes and credit account codes in the respective combo boxes.  
Accountant will then fill in all details of the voucher like voucher description, date, debit account code, credit account code, description, and amount and then click 'add voucher' button.  
Once 'add voucher' is clicked it will appear in the voucher screen below in a grid and the voucher entry screen will be cleared and waiting for new voucher to be added. During this step voucher is not added to database it's only in the collection.  
If there are more vouchers to be added the user again fills voucher and clicks 'add voucher'.  
Once all the vouchers are added he clicks 'submit voucher' which finally adds the group of vouchers to the database.  
Below figure shows pictorially how the screen looks like.

Figure shows how the sequence diagram looks like. Below diagram shows a full sequence diagram view of how the flow of the above screen will flow from the user interface to the data access layer. There are three main steps in the sequence diagram, let's understand the same step by step.  
Step 1:- The accountant loads the voucher data entry screen. You can see from the voucher data entry screen image we have two combo boxes debit and credit account codes which are loaded by the UI. So the UI calls the 'Account Master' to load the account code which in turn calls the data access layer to load the accounting codes.  
Step 2:- In this step the accountant starts filling the voucher information. The important point to be noted in this step is that after a voucher is added there is a conditional statement which says do we want to add a new voucher. If the accountant wants to add new voucher he again repeats step 2 sequence in the sequence diagram. One point to be noted is the vouchers are not added to database they are added in to the voucher collection.  
Step 3:- If there are no more vouchers the accountant clicks submit and finally adds the entire voucher in the database. We have used the loop of the sequence diagram to show how the whole voucher collection is added to the database.
