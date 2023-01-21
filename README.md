# Restaurant Reservation Application Project

## Project Introduction

### Purpose

This project is aimed at developing a restaurant web application that can be beneficial for both customers and the business.

### Functionality / Features

Customers and employees will be able to:

- Book tables.
- Browse menu & price.
- Browse the restaurant's location and contact details.

**Only employees will be able to update, delete and read bookings.**

Employees can :

- Login / Logout admin account.
- View a list of reservations group by selected date in time sequence.
- Identify new booking.
- Search a reservation by the customer's mobile number.
- Update reservations details.
- Delete reservations.

When a booking is requested :

- Automatically calculate the availability based on the provided visiting date, time and number of guests and return corresponding message to the client immediately.
- If there's available table, automatically assign a table that can accommodate the number of guests on the given date and time.
- Alert to the admin user when there's a new booking.

### Target Audience

Potential restaurant's customers and restaurant workers.

### Tech Stack

MERN stack will be used for the development.

Frontend:

- React
- Other Dependencies:
  - react-router-dom, axios, dotenv.

Backend:

- NodeJS
- Express
- MongoDB
- Other Dependencies:
  - helmet, jwtwebtoken, bcrypt, dotenv, cors, mongoose.

---

## User Story

### As a **Visitor**

| Action                                                       | Outcome                                      |
| ------------------------------------------------------------ | -------------------------------------------- |
| View menu with photos                                        | Better understanding of food                 |
| View menu with price                                         | Estimate budget                              |
| View menu with a short description                           | Better understanding of food                 |
| View the restaurant's location                               | Know location of the restaurant              |
| View the restaurant's contact                                | Know contact of the restaurant               |
| View the restaurant's business hour                          | Know the restaurant's business hour          |
| Book tables                                                  | No need to call restaurant.                  |
| Receive booking confirmation after booking                   | Know that booking has been made successfully |
| Receive booking fail message when there's no available table | Know that booking has been failed            |

### As an **Admin**

| Action                                                                                                                          | Outcome                                                                                                                                                    |
| ------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Log into admin account                                                                                                          | Only admin is accessible to reservation list                                                                                                               |
| Don't want to login frequently                                                                                                  | It's more convenient                                                                                                                                       |
| Receives information of customer's first name, last name, number of visitors, mobile number, date and time for visit on booking | Prepare table accordingly, Guide customers to their seats on arrival Call to customer when necessary                                                       |
| Assign an available table automatically when a booking is made                                                                  | Employee can read the assigned table information with the customer's information and guide them to their reserved table when they arrive to the restaurant |
| Only receives online booking when the number of guest is not bigger than 6                                                      | Minimize risk of no-show, Table can be arranged accordingly by human for a large number of guests                                                          |
| Confirm a new reservation by clicking a 'Confirm' button                                                                        | Identify incoming reservation                                                                                                                              |
| View reservation list group by date by selected date                                                                            | Easier to identify bookings by date                                                                                                                        |
| View reservation list in two groups: unconfirmed and confirmed                                                                  | Identify unconfirmed reservation so the table can be prepared for unconfirmed reservations                                                                 |
| View reservation lists in time sequence (top: earliest entry time, bottom: latest entry time)                                   | Easier to identify guests coming soon                                                                                                                      |
| Search a reservation by customer's mobile number                                                                                | Easy to find a target reservation                                                                                                                          |
| Update all details of the reservations                                                                                          | Accept customers' plan changes                                                                                                                             |
| Cancel (delete) a reservation                                                                                                   | Table can be available for others                                                                                                                          |

---

## DFD

### **Database Structure**

### Table

```js
{
  _id : String,
  tableNumber : Number, // Unique number to identify tables
  seats: Number // Number of seats that a table has
}
```

The table collection represents all tables that a restaurant have.  
The table collection and documents will be pre seeded into the database because they hardly change.  
In this project, we will seed the tables with 2 seats, 4 seats and 6 seats.

### Reservation

```js
{
  _id: String,
  tableId: ObjectId, // From Table document, Populate
  guest: guestSchema, // Sub document
  isConfirmed: Boolean // To identify unconfirmed reservation and confirmed reservation
}
```

The reservation collection represents all successfully made reservations.  
'isConfirmed' field is necessary to identify reservations that have not been checked or prepared by the restaurant workers.  
The 'guest' field will exist as a sub document form and will have the following schema:

```js
{
  firstName : String,
  lastName : String,
  mobile: String, // Mobile number
  date: Date,  // Visiting date inc time
  guestNumber: Number //Number of required seats
}
```

The above schema is the information that potential customers are required to provide on reservation request as well.

### Admin

```js
{
  _id: String,
  id: String, //hashed, encrypted
  password: String //hashed, encrypted
}
```

The Admin collection represents an admin account.  
To read, update and delete the 'Reservation' document, the user must be authenticated by successfully logging in.  
It will have only one pre-seeded document because having multiple admin accounts is meaningless in this case.

### **Customer book a table**

![customer booking DFD](docs/booking.png)

When a customer request a reservation, we will receive these information filled by the customer:

1.  First name
2.  Last name
3.  Mobile number
4.  Date (Inc time)
5.  Number of guest

This information is represented as **'Booking Info'** in this DFD.

And the API will process the data as follows:

1. Get an array of reservations from the Reservation collection that have the same date value with the date provided in the Booking Info (Except the time at this stage for simplicity)
2. Filter the reservation array to get another array of reservations that have similar time from (Booking Info's time - 1h 30min) to (Booking Info's time + 1h 30min) and similar number of seats (same number of table seats with the Booking Info's number of guests or one more number of table seats than the Booking Info's number of guests). This will eventually return an array of unavailable tables that is matched with the provided conditions from the Booking Info.
3. Check if there's any reservation reserved by the same mobile number. This will prevent customers book again accidentally. The reason why we implement this after the process of the number 2 is to allow customers to book again on the different dates or time (lunch/dinner).
4. If the reservation found that has been made by the same customer, return error. Otherwise, get all tables from the 'Table' collections and find a table that is not in the unavailable tables. This will return an available table that satisfies the condition provided from the 'Booking Info'. If there's no available table, return error.
5. If an available table is found, insert the provided booking info with the found available tableId into the 'Reservation' collection.
6. Return the newly created document in the 'Reservation' collection.

### Admin login

![admin login DFD](docs/login.png)

The Admin ID and Password will be hashed and encrypted and stored in the 'Admin' collection.  
Therefore, the DFD shows the process of decrypting Admin's credentials from the 'Admin' collection and validate hashed credentials with the provided plain credentials.  
If it's matched, the JWT will be returned.

### **Admin view reservation list**

![admin view reservation list DFD](docs/viewAll.png)

The jwt will be required when the user requests to read the reservation list.
Some API middleware will validate the JWT and the admin credentials are correct.  
If both of them are valid, it will return the reservation list.  
Otherwise, it will return error.

### **Admin update a reservation**

![admin update reservation DFD](docs/update.png)

Same as above, it will validate the JWT and the admin credentials first.  
Then it will check which data is requested to be updated.  
Because if any of the time, date, number of guests or table is requested to be updated, it needs to get an available table that satisfies the condition again.  
Therefore, if any of attempting of changing those values is detected, it will go through the process of finding an available table mentioned in the 'Customer booking DFD'.  
Otherwise, it will update the reservation directly.

### **Admin delete a reservation**

![admin delete reservation DFD](docs/delete.png)

Same as above, it will validate the JWT and the admin credentials first.  
Then it will find the document by its ID from the 'Reservation' collection, and delete it.  
if no matching document is found, it will return an error.

### **Admin search reservation(s) by mobile number**

![admin search reservation by mobile DFD](docs/find.png)

Same as above, it will validate the JWT and the admin credentials first.  
Searching a specific reservation is a necessary function for the restaurant workers.  
Because when they want to update, delete or read a specific reservation, the target reservation is needed to be found first.  
To prepare for such cases, we have decided to make the restaurant workers to be able to search a reservation or reservations (booked by the same person for multiple times) by the customer's mobile number because the mobile number is unique.  
Therefore, if the mobile number is not found from the 'Reservation' collection, it will return an error.  
If it's found, it will return an array of reservation that have the same mobile number.

---

## Application Architecture Diagram

<br>

![AAD](docs/T3A2-A.AAD.drawio.png)

This application consists of three different layers such as the Front-end layer, the Back-end layer, and database layer.

### **Front-end layer**

The `front-end server` can communicate with the `backend server` via `HTTP Request`. The application can be operated through a browser, and the data changed and generated through the event of the application is stored in the database through the backend.
The frontend of this application consists of four main components and components of Header and Footer.

#### **MenuList**

This component represents the menu list that the restaurant has. It include meals that the shop provides and each price of the meal.

#### **Booking**

This component determines whether the customer is already registered and shows its information if registered.
In case of a new customer, the customer can register his/her details and make a reservation at the date and time he/she want. The component can also change the reservation date and time and the number of people. It can also cancel the reservation if he/she don't want to.

#### **AboutUs**

This component has information of the restaurant such as the location, opening hours, and its address.

#### **Admin**

This component is logged in by the Admin to confirm and approve the reservation. And if there is a request for modification or deletion from a customer, reservation modification and deletion can be made.

### **Back-end layer**

The Back-end server communicates with the database depending on the request and retrieves the corresponding data to fulfill the request. This front-end request queries the Mongoose to manipulate the desired data. Once receives data from MongoDB, backend will send API response back to the frontend. The backend server manipulates data into mongoDB through APIs for inserting, updating, and deleting data.

### **Database layer**

Data is kept in DB so that uses can retrieve the data they want or update and delete the data.

---

## WireFrame

<br>

`Wireframe` is simplified, low-fidelity representations of a user interface that are used to demonstrate the structure and layout of a website or application. A rough sketch of the designer's idea of the application is used to share his/her thoughts with the relevant stakeholder and developers. We use the `Agile methodology` to efficiently complete this wireframe, allowing for immediate reflection of user requirements changes and better ideas. When a new idea came up, we discussed the pros and cons of it and problems when implementing it, applied it to the draft wireframe, and repeated this process until it was completed. In order to explain the `Agile methodology` of consultation, modification, and reflection, the changed history part is described as follows. Our project team use `Figma` to get wireframes

### **The first draft**

![first draft](docs/WF-firstDraft.png)

After the first meeting of our project team, we drafted the wireframe as above. It consists of a `home page` that can be viewed when a user accesses this application, a `reservation page` that can make a reservation, and an `about us page` that indicates the location and business hours of a restaurant.

#### **Set items in Navbar**

The Navbar has `Menu`, `Reservation`, and `AboutUs` items. It can be moved to corresponding page when clicking on. On the left side of Navbar, a `restaurant logo` is placed so that when clicked, we can move to Home page.

#### **Who handles the application?**

The issue that arose during the first meeting was who would handle this application. I said that the restaurant owner, not the customer, is for reservation management, and Jihyuk, a team member, suggested that the customer make reservations himself and the restaurant owner should manage it. The latter seemed more realistic, so we decided to implement this way.

#### **Implementing changes**

An `Admin` item was added to navbar. The `booking list` on the reservation page of the draft was moved to the `Admin page` to confirm the reservation by the admin. On the reservation page, in order for customers to conveniently register when making a reservation, they searched their mobile number and if it is already registered, the existing their information is displayed. In the case of a new customer, they would register their reservation information. The contents of this change are as follows.

![changed first draft](docs/WF-2nddraft.png)

The yellow part is the newly added part, and the sky blue part is the changed part.

Through the consultation process as above, the `Admin login` screen and the function of modifying and deleting reservations were added, and the screen was implemented so that admins could also modify and delete reservations.
The final of its project design phase is as follows.

### **Desktop Diagrams**

![changed first draft](docs/WF-fin-dt-01.png)

This is the page when you first access this application.

1. **Menu** <br>
   User can go to a page where shows the menu served in the restaurant when click the button. <br>
2. **Reservations** (http://localhost:3000/reservation/booking) <br>
   User can go to the above URL where you can make a reservation.
3. **AboutUs** (http://localhost:3000/aboutUs) <br>
   A button to go to the page that shows restaurant's location and detail information. <br>
4. **Admin** (http://localhost:3000/admin) <br>
   A button that can be moved to a page where admin can log in <br>
5. **Logo** (http://localhost:3000) <br>
   A button to go to the home page <br>

![client booking](docs/WF-fin-dt-02.png)

It is a page where existing and new customers can make reservations

1. **Customer search** <br>
   When existing customers enter their mobile number, their information is displayed in number 2 area on the right, and they can enter the desired date and time for reservation. New customers can enter their reservation information.
2. **Input area** <br>
   Customer can enter the desired date, time, and number of guests for reservation. You can enter up to 6 people for the number of people.
3. **Button** <br>
   `Booking button` is user can make a reservation after entering the information. `Update button` is user can fix the reservation information. `Delete button` is user can cancel the reservation.

![aboutus](docs/WF-fin-dt-03.png) <br>
This AboutUs page shows about the location of the restaurant and its details. <br>

![admin login](docs/WF-fin-dt-04.png) <br>

After project meeting, this page is the new function and screen we got. The purpose of this page is to prevent anyone from accessing the admin page.

![admin page](docs/WF-fin-dt-05.png) <br>
The admin can use this page to manage the bookings.

1. **Check Box** <br>
   If admin click the checkbox the booking turns to completion status. So the admin can see this booking in completed part (the part said number 3 in the picture). <br>
2. **A table needs to conform** <br>
   This is a table showing the reservations made by the customer for the admin to check the customer's registered reservation.
3. **A table shows booking completed** <br>
   This is a table showing the reservations confirmed by the admin.
4. **Update and Delete Button** <br>
   This is a button for the admin to update or delete the information of the booking that has been confirmed.
5. **Logout Button** <br>
   For the safety reasons, this button is designed to log out if admin is away for a long time.

### **Tablet Diagrams**

Contents shown in AAD for desktop should be displayed vertically on tablet(824X1194) so that they do not cross the window. The function description of each content is the same as that of the desktop.

![tablet 01](docs/WF-fin-tb-01.png) <br>

![tablet 02](docs/WF-fin-tb-02.png) <br>

### **Mobile Diagrams**

All content should be adjusted to fit the mobile size.

![tablet 02](docs/WF-fin-mb-01.png) <br>

Our team completed our wireframes via `Facetime` and `chat`. When new features and restrictions came up, we tried to derive good results through an `iterative process of consultation, implementation, and testing`.

## Project Management

<br>

Trello for Restaurant booking application: https://trello.com/b/yeE67wpk/t3a2-a

![trello](docs/Trello-main.png)

## PLAN

It is the stage of establishing an overall schedule and plan for the project. For each step, decide on the due date and things to do, and plan to finish the project without any issues. The PLAN consists of `Creating Project Plan` and `Setting deliverables`.

![trello_plan](docs/Trello-plan.png)

### Creating Project Plan

Create the project plan and schedule in Trello according to Agile methodology.

- Create the due data and check list of each step in the project.
- Plan and schedule and prioritize.

### Setting deliverables

- Deliverables comes out in each step of project are very important. This project has lots of deliverables so we have to concern ourselves not to miss any deliverables. Thus, we put this step in Plan.
  <br>

## Design

Program, interface, and database should be designed for implementing applications based on User story. All of these are top priority tasks during the project.

![trello_design](docs/Trello-design-01.png)

### Design MongoDb

To implement the database, define collections and schemas. We have 3 collections to cover the transactions.

#### **Admin**

Represents admin user’s ID and Password.
No CRUD
Pre seed a single doc into the DB

#### **Reservation**

Represents all reservations.
Read, Update, Delete by only Admin.
Create by Anyone.
SubDoc: Guest

#### **Table**

Represents all table a restaurant have.
No CRUD
Pre seed 6 of 2 seaters, 8 of 4 seaters, 4 of 6 Seaters into the DB

### Dataflow Diagram

DFD is implemented to understand the flow of data for the entire application.

DFD URL:
https://lucid.app/lucidchart/8eaa9e24-8138-4385-9a71-400adc39f12d/edit?viewport_loc=-1599%2C-14924%2C2047%2C1129%2C0_0&invitationId=inv_6d4f3244-6c95-473e-8022-36e45e2ffadd

### Application Architecture Diagram

Create an AAD to understand the overall structure of the application.

![trello_design](docs/Trello-design-02.jpeg)

### Creating UserStory

Provides multiple user stories that use ‘persona, what and why’ that outline meaningful features of project.

### Creating Wireframes

The application is implemented to correspond to all screen sizes for the application to be implemented.

### Design API

Create API to communicate between the frontend and the backend.

#### **/reservation**

POST
For customers' booking request.

GET
To get a list of all reservations.

‌#### **/reservation/:mobileNumber**

GET
To get a list of reservations that match with the provided mobile number.

#### **/reservation/:id**

DELETE
To delete one reservation

PUT
To update one reservation

#### **/admin**

POST
To log in as admin

![trello_design](docs/Trello-design-03.jpeg)

### Design Workflow

Design workflow for source code to understand the application structure easily and create the program list.
Setting person in charge of each program based on the program list.

### Design Testcase for user Test

Testcases should be created to verify that each function of the application works normally.

### ReadMe document

Descript ReaMe file according to Rubric

## Develop

### Coding

Implement db and programs based on design. Conduct functional testing during development. Following up the assignment requirement when coding.

![trello_develop](docs/trello_coding.png)

## Test

### Testing

Functional tests and unit tests are performed frequently during development according to the agile methodology, and when an error is found, the development and test stages are repeated until the development is completed by correcting the error part and performing the test. Perform the unit test according to the test case and verify that the function is accurately implemented.

![trello_test](docs/trello_test.png)

## Deploy

### Deployment

Ensure all requirements are carried out as planned.

![trello_deploy](docs/trello_deploy.png)
