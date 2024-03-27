# FiberHome Automated Testing of REST Booking API Newman
This FiberHome project aims to automate the testing of a REST booking API using Postman and Newman. By automating the testing process, I ensure consistent and reliable validation of the API endpoints, reducing manual effort and enabling faster feedback on changes.
### **Features**
- Comprehensive API testing suite covering various HTTP methods including GET, POST, PUT, PATCH, and DELETE.
- Requests with different configurations such as request bodies, pre-request scripts, and tests to ensure comprehensive API endpoint validation.
- Easy-to-use Postman collections for executing tests locally or integrating into continuous integration pipelines.
- Detailed test scripts to validate responses and ensure API functionality meets requirements.
- Modularized structure for easy maintenance and scalability of test suites.

### **API Documentation**
```console
 https://documenter.getpostman.com/view/33749765/2sA35D6Pce
 ```
 
### **Technology Used**
   - **Postman:** Used for creating and managing API requests and collections.
   - **Newman:** Command-line collection runner for Postman, allowing for execution in CI/CD pipelines or terminal environments.

### **Pre-requisite**
  Before running the tests, ensure you have the following installed:
  - **Postman:** <a href="https://www.postman.com/downloads/">Download and install Postman</a> on your machine.
  - **Node.js:** <a href="https://nodejs.org/en">Install Node.js</a> to be able to run Newman.
  - **Newman:** Html Report Library

### **Installation**

1. Postman: If you haven't already, [download and install Postman.](https://www.postman.com/downloads/)
2. Clone the repository:
    ```console 
        git clone https://github.com/abdul-muyeez/FiberHome-Automated-Testing-of-REST-Booking-API-Newman
    ```
3. Import the Postman collection:
    - Open Postman.
    - Click on the Import button.
    - Select the file from the repository.
4. Import the Postman environment:
    - In Postman, click on the gear icon in the top right corner.
    - Select **Import** and choose the file.
5. Newman and Report Installation Process:
    - Newman Install Command:
     ```console 
      npm install -g newman
    ```
    - Newman Html Report Install Command:
     ```console 
      npm install -g newman-reporter-htmlextra
    ```
### **Usage**
1. Select Environment:
    -   In Postman, select the appropriate environment (e.g., Development, Production) from the top-right dropdown.
2. Run Collection:
    -   Select the imported collection from the Collections sidebar.
    -   Click on the Runner button to open the collection runner.
    -   Select the desired environment.
    -   Click Start Test to run the collection.
3. View Results:
    -   Once the tests are complete, view the results in the Runner tab.
    -   Detailed test results can be viewed for each request.
# Testing
  ### 1. Create Booking.
  #### URL: https://restful-booker.herokuapp.com/booking/
  #### METHOD: POST
  #### Pre-request Script:
```
//Random Values Generation and Set it into Environment
pm.environment.set("firstname", pm.variables.replaceIn("{{$randomFirstName}}"))
pm.environment.set("lastname", pm.variables.replaceIn("{{$randomLastName}}"));

//Totalprice
var min = 500
var max = 1000
var totalprice = Math.floor(Math.random() * (max - min + 1)) + min;
pm.environment.set("totalprice", totalprice)

//DepositPaid
var depositpaid= pm.variables.replaceIn("{{$randomBoolean}}")
pm.environment.set("depositpaid", depositpaid)

//Date Format (YYYY-MM-DD)
const date= require('moment')
const today= date()
var checkin= today.format("YYYY-MM-DD")
pm.environment.set("checkin", checkin)
var checkout= today.add(5,'d').format("YYYY-MM-DD")
pm.environment.set("checkout", checkout)

//Randomly Select Additional Needs from Data
const service = ["Breakfast", "Lunch", "Dinner", "Airport Pickup", "Car Parking"]
const selected = service[Math.floor(Math.random() * service.length)]
pm.environment.set("extraNeeds", selected)

```
#### Request Body:
```
{
	"firstname" : "{{firstname}}",
	"lastname" : "{{lastname}}",
	"totalprice" : {{totalprice}},
	"depositpaid" : {{depositpaid}},
	"bookingdates" : {
    	"checkin" : "{{checkin}}",
    	"checkout" : "{{checkout}}"
	},
	"additionalneeds" : "{{extraNeeds}}"
}
```
#### Tests:
```
var responseCode= pm.response.code
console.log(responseCode)

if (responseCode==200){
 
    var fiberdata= pm.response.json()
        pm.environment.set("id", fiberdata.bookingid)
        var id = pm.environment.get("id")
         pm.test(`Booking IS Successful. ID-${id}`, function(){ 
            pm.expect(pm.environment.get("id")).to.eql(fiberdata.bookingid)
    })

}else if(responseCode==404){
    pm.test("Booking is Unsuccessful !")
}else if(responseCode==500){
    pm.test("Server Error")
}

```
#### Response Body:
```
{
    "bookingid": 1083,
    "booking": {
        "firstname": "Giovanni",
        "lastname": "Wyman",
        "totalprice": 553,
        "depositpaid": true,
        "bookingdates": {
            "checkin": "2024-03-26",
            "checkout": "2024-03-31"
        },
        "additionalneeds": "Car Parking"
    }
}
```
### 2. Get Booking.
#### URL: https://restful-booker.herokuapp.com/booking/3078
#### METHOD: GET
#### Pre-request Script: None
#### Request Body: None
#### Tests:
```
// Get the response code
const id = pm.environment.get("id")
var responseCode= pm.response.code
console.log(responseCode)
// Handle the response based on the code
if(responseCode==200){
    // Parse the response as JSON
    var fiberdata= pm.response.json()
     pm.test(`Booking ID-${id}`)
    // Test validations for each data point
    pm.test("Inserted First Name Validation", function(){ 
        pm.expect(pm.environment.get("firstname")).to.eql(fiberdata.firstname)
    })

    pm.test("Inserted Last Name Validation", function(){
        pm.expect(pm.environment.get("lastname")).to.eql(fiberdata.lastname)
    })

    pm.test("Inserted Checkin Date Validation", function(){
     pm.expect(pm.environment.get("checkin")).to.eql(fiberdata.bookingdates.checkin)
    })
    pm.test("Inserted Checkput Date Validation", function(){
     pm.expect(pm.environment.get("checkout")).to.eql(fiberdata.bookingdates.checkout)
    })
    pm.test("Inserted Total Price Validation", function(){
        pm.expect(pm.environment.get("totalprice")).to.eql(fiberdata.totalprice)
    })
    pm.test("Deposit Paid Validation", function(){
        pm.expect(pm.environment.get("depositpaid")).to.eql(fiberdata.depositpaid.toString())
    })

}else if(responseCode==404){
    pm.test("Not Found")
}else if(responseCode==500){
    pm.test("Server Error")
}else {
  pm.test("Unexpected Status Code", function () {
    pm.expect(responseCode).to.be.oneOf([200, 404, 500])
  })
}

```
#### Response Body:
```
{
    "firstname": "Giovanni",
    "lastname": "Wyman",
    "totalprice": 553,
    "depositpaid": true,
    "bookingdates": {
        "checkin": "2024-03-26",
        "checkout": "2024-03-31"
    },
    "additionalneeds": "Car Parking"
}
```
### 3. Acces Token.
#### URL: https://restful-booker.herokuapp.com/auth
#### METHOD: POST
#### Pre-request Script: None
#### Request Body:
```
{
	"username": "admin",
	"password": "password123"
}

```
#### Tests:
```
var responseCode= pm.response.code
console.log(responseCode)

if (responseCode==200){
 
    var fiberdata= pm.response.json()

        pm.environment.set("token", fiberdata.token)

         pm.test("Access Token Created", function(){ 
        pm.expect(pm.environment.get("token")).to.eql(fiberdata.token)
    })

}else if(responseCode==404){
    pm.test("Acces Token not Created")
}else if(responseCode==500){
    pm.test("Server Error")
}else {
    pm.test("Something went wrong")
}

```
#### Response Body:
```
{
    "token": "533ebaa322135af"
}

```
### 4. Update Booking.
#### URL: https://restful-booker.herokuapp.com/booking/3078
#### METHOD: PUT
#### Pre-request Script:
```
//Generate Random Values tehn set it into Environment

var firstname= pm.variables.replaceIn("{{$randomFirstName}}")
pm.environment.set("ufirstname", firstname)
var lastname= pm.variables.replaceIn("{{$randomLastName}}")
pm.environment.set("ulastname", lastname)

//totalprice Update
var min = 600
var max = 900
var totalprice = Math.floor(Math.random() * (max - min + 1)) + min;
pm.environment.set("utotalprice", totalprice)
//DepositPaid
var depositpaid= pm.variables.replaceIn("{{$randomBoolean}}")
pm.environment.set("udepositpaid", depositpaid)

//Date
const date= require('moment')
const today= date()
var checkin= today.add(4,'d').format("YYYY-MM-DD")
pm.environment.set("ucheckin", checkin)
var checkout= today.add(6,'d').format("YYYY-MM-DD")
pm.environment.set("ucheckout", checkout)

// RANDOMLY SELECT MEAL THEN SET IT TO ENVIRONMENT
const service = ["Breakfast", "Lunch", "Dinner", "Airport Pickup", "Pet Service"]
const selected = service[Math.floor(Math.random() * service.length)]
pm.environment.set("uextraNeeds", selected)

```
#### Request Body:
```
{
	"firstname" : "{{ufirstname}}",
	"lastname" : "{{ulastname}}",
	"totalprice" : {{utotalprice}},
	"depositpaid" : {{udepositpaid}},
	"bookingdates" : {
    	"checkin" : "{{ucheckin}}",
    	"checkout" : "{{ucheckout}}"
	},
	"additionalneeds" : "{{uextraNeeds}}"
}

```
#### Tests:
```
const id = pm.environment.get("id")
var responseCode= pm.response.code
console.log(responseCode)

if (responseCode==200){
 
    var fiberdata= pm.response.json()
    
        pm.environment.set("Updated", fiberdata.token)
        
         pm.test(`Successfully Updated Booking ID-${id}`, function(){ 
        pm.expect(pm.environment.get("Updated")).to.eql(fiberdata.Updated)
    })

}else if(responseCode==404){
    pm.test("Not Updated")
}else if(responseCode==500){
    pm.test("Server Error")
}

```
#### Response Body:
```
{
    "firstname": "Mariano",
    "lastname": "Gorczany",
    "totalprice": 627,
    "depositpaid": false,
    "bookingdates": {
        "checkin": "2024-03-30",
        "checkout": "2024-04-05"
    },
    "additionalneeds": "Airport Pickup"
}
```
#### Request Headers:
```
Cookie          token
```
### 5. Verification After Update
#### URL: https://restful-booker.herokuapp.com/booking/3078
#### METHOD: GET
#### Pre-request Script: None
#### Request Body: None
#### Tests: 
```
const id = pm.environment.get("id")
var responseCode= pm.response.code
console.log(responseCode)

if(responseCode==200){

    var fiberdata= pm.response.json()
    pm.test(`Modification of Booking ID-${id}`)
    pm.test("First Name Updated", function(){ 
        pm.expect(pm.environment.get("ufirstname")).to.eql(fiberdata.firstname)
    })

    pm.test("Last Name Updated", function(){
        pm.expect(pm.environment.get("ulastname")).to.eql(fiberdata.lastname)
    })

    pm.test("Checkin Date Updated", function(){
     pm.expect(pm.environment.get("ucheckin")).to.eql(fiberdata.bookingdates.checkin)
    })
    pm.test("Checkput Date Updated", function(){
     pm.expect(pm.environment.get("ucheckout")).to.eql(fiberdata.bookingdates.checkout)
    })
    pm.test("Total Price Updated", function(){
        pm.expect(pm.environment.get("utotalprice")).to.eql(fiberdata.totalprice)
    })
    pm.test("Deposit Paid Updated", function(){
        pm.expect(pm.environment.get("udepositpaid")).to.eql(fiberdata.depositpaid.toString())
    })

}else if(responseCode==404){
    pm.test("Not Found")
}else if(responseCode==500){
    pm.test("Server Error")
}else {
  pm.test("Unexpected Status Code", function () {
    pm.expect(responseCode).to.be.oneOf([200, 404, 500])
  })
}

```
#### Response Body:
```
{
    "firstname": "Mariano",
    "lastname": "Gorczany",
    "totalprice": 627,
    "depositpaid": false,
    "bookingdates": {
        "checkin": "2024-03-30",
        "checkout": "2024-04-05"
    },
    "additionalneeds": "Airport Pickup"
}
```
### 6. Partial Update.
#### URL: https://restful-booker.herokuapp.com/booking/3078
#### METHOD: PATCH
#### Pre-request Script:
```

const moment = require('moment');

// Get the current date
const today = moment();

// Define the desired update offset (number of days to add)
const updateOffset = 10;  // Adjust this value as needed

// Calculate the updated check-in and check-out dates
const updatedCheckin = today.add(updateOffset, 'days').format('YYYY-MM-DD');
const updatedCheckout = today.add(updateOffset + 3, 'days').format('YYYY-MM-DD');  // Add offset + 3 days

// Set the updated check-in and check-out dates in the environment
pm.environment.set("pcheckin", updatedCheckin);
pm.environment.set("pcheckout", updatedCheckout);

const service = ["Pet Service", "Tour Guide", "Rental Car"]
const selected = service[Math.floor(Math.random() * service.length)]
pm.environment.set("pextraNeeds", selected)

```
#### Request Body:
```
{
"bookingdates" : {
    	"checkin" : "{{pcheckin}}",
    	"checkout" : "{{pcheckout}}"
	},
	"additionalneeds" : "{{pextraNeeds}}"
}

```
#### Tests: 
```
const id = pm.environment.get("bID")
switch(pm.response.code)
    {
        case 200:
            pm.test(`Successful To Modify Details of Booking ID- ${id}.`)
        break
        case 403:
            pm.test(`Unsuccessful To Modify Details of Booking ID- ${id}`)
        break
        case 405:
            pm.test(`There's No Details of Booking ID- ${id}. Create First`)
        break
    }
```
#### Response Body:
```
{
    "firstname": "Mariano",
    "lastname": "Gorczany",
    "totalprice": 627,
    "depositpaid": false,
    "bookingdates": {
        "checkin": "2024-04-05",
        "checkout": "2024-04-18"
    },
    "additionalneeds": "Rental Car"
}
```
#### Request Headers:
```
Cookie          token
```
### 7. Verification Partial Update
#### URL: https://restful-booker.herokuapp.com/booking/3078
#### METHOD: GET
#### Pre-request Script: None
#### Request Body: None
#### Tests:
```

const id = pm.environment.get("id")
var responseCode= pm.response.code
console.log(responseCode)
// Handle the response based on the code
if(responseCode==200){
    // Parse the response as JSON
    var fiberdata= pm.response.json()
     pm.test(`Booking ID-${id}`)
    // Test validations for each data point

    pm.test("Modified Checkin Date Validation", function(){
     pm.expect(pm.environment.get("pcheckin")).to.eql(fiberdata.bookingdates.checkin)
    })
    pm.test("Modified Checkput Date Validation", function(){
     pm.expect(pm.environment.get("pcheckout")).to.eql(fiberdata.bookingdates.checkout)
    })
    pm.test("Modified Additional Need Validation", function(){
     pm.expect(pm.environment.get("pextraNeeds")).to.eql(fiberdata.additionalneeds)
    })

}else if(responseCode==404){
    pm.test("Not Found")
}else if(responseCode==500){
    pm.test("Server Error")
}else {
  pm.test("Unexpected Status Code", function () {
    pm.expect(responseCode).to.be.oneOf([200, 404, 500])
  })
}
```
#### Response Body:
```
{
    "firstname": "Mariano",
    "lastname": "Gorczany",
    "totalprice": 627,
    "depositpaid": false,
    "bookingdates": {
        "checkin": "2024-04-05",
        "checkout": "2024-04-18"
    },
    "additionalneeds": "Rental Car"
}

```
### 8. Delete Booking
#### URL: https://restful-booker.herokuapp.com/booking/3078
#### METHOD: DELETE
#### Pre-request Script: None
#### Request Body: None
#### Tests:
```
const id = pm.environment.get("id")
switch(pm.response.code)
    {
        case 200,201:
            pm.test(`Successfully Deleted Details of Booking ID- ${id}.`)
        break
         default:
            pm.test(`Unable To Delete Details of Booking ID- ${id}`)
    }
```
#### Response Body:
```
Created
```
#### Request Headers:
```
Cookie          token
```
### 9. Verification After Delete
#### URL: https://restful-booker.herokuapp.com/booking/3078
#### METHOD: GET
#### Pre-request Script: None
#### Request Body: None
#### Tests:
```
const id = pm.environment.get("id")
var responseCode= pm.response.code
console.log(responseCode)

if(responseCode==200){

    var fiberdata= pm.response.json()
    pm.test(`Unsuccesful to delete Booking ID-${id}`)

}else if(responseCode==404){
    pm.test(`Successfully Deleted Booking ID-${id}`)
}else if(responseCode==500){
    pm.test(`Server Error`)
}else {
  pm.test(`Unexpected Status Code`, function () {
    pm.expect(responseCode).to.be.oneOf([200, 404, 500])
  })
}

```
#### Response Body:
```
Not Found
```
# Run Through Command-Prompt
  • Run Command for Console:
  ```
    newman run Fiberhome.postman_collection.json -e Fiber1.postman_environment.json
  ```
  • Run Command for Report:
  ```
    newman run Fiberhome.postman_collection.json -e Fiber1.postman_environment.json -r cli,htmlextra
  ```
# Newman Report Summary:
![image](https://github.com/abdul-muyeez/FiberHome-Automated-Testing-of-REST-Booking-API-Newman/assets/136342156/5a5f076a-9651-4110-a7f4-3d54c3147299)
![image](https://github.com/abdul-muyeez/FiberHome-Automated-Testing-of-REST-Booking-API-Newman/assets/136342156/c38c61fc-7bc4-4c8d-a0e9-7939b92427d3)
![image](https://github.com/abdul-muyeez/FiberHome-Automated-Testing-of-REST-Booking-API-Newman/assets/136342156/7f335aec-b33d-40c6-bda9-4c5e62d5c9b5)
![image](https://github.com/abdul-muyeez/FiberHome-Automated-Testing-of-REST-Booking-API-Newman/assets/136342156/32cda565-e085-40b3-b40d-6a1789f22f9d)
![image](https://github.com/abdul-muyeez/FiberHome-Automated-Testing-of-REST-Booking-API-Newman/assets/136342156/4d884888-6c85-4680-b1c5-4d08cbd28e19)

## Have a good look in this project


