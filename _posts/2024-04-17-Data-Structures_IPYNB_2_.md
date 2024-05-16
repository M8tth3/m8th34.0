---
title: Data Structures Writeup
categories: ['CSP', 'Week 29']
tags: ['hacks']
---

> This blog is really long and can get confusing when trying to keep track of where things. I did my best to organize it but it can still be overwhelming. To keep track of where you are, please use the **sidebar** on the **right** side. It has all the blog's sections and subsections.
{: .prompt-info}

## Project Summary

Last trimester, our CPT project was a video sharing site. It would allow users to create an account and upload their own videos. These videos could be viewed by anyone and shared to others. For this trimester, we integrated all the other projects together. Kyle and Aashray’s project was a social media site while Matthew and Trevor’s project was a baking game.

Our integration project turned into this: a video sharing website (me) where users could comment on videos, like comments, and delete them (Aashray/Kyle). And watching videos would give users points. These points could be used to buy items (Trevor/Matthew).

## Collections

![Image Name](images/sqlite-database.png "sqlite-database")

![Image Name](images/user-creation.png "user-creation")

The first image is of our SQLITE database. The current table in the image is the user database and this holds the user names of people, emails, their real names, DOB, and a hashed password which is used for login and authentication.

To create the users, see the second image. Here, we have a python function called `initUsers()` and this will create all the default users with all of their information.

Each user will have their real name and a username which will be displayed publicly on the site. They will also have their email and an account password. This will be for logging in. They also have a date of birth and an identifier if they’re an admin account. Admin accounts have special privileges like having access to a user dashboard. They can see every user and create or delete other users. This screenshot also demonstrates the integration.

## Lists and Dictionaries

### List and Dictionary 1

![Image Name](images/lists-dictionary.png "lists-dictionary")

This image is from the user query function which will query all the users using a `GET` request. The return is a list of all the users. But within the list is a dictionary which contains all the user information like name, username, DOB, and more. This is an example of a list and dictionary 1.

### List and Dictionary 2

![Image Name](images/dictionary2.png "dictionary2")

This is the second dictionary. It contains video metadata. For example, it has the video’s title and its description. This is used when displaying the frontend so that the user can identify the video. It also contains a unique video ID. While the ID isn’t displayed to the user, it’s used to display the correct video even if two videos have the same title. Other metadata like likes, dislikes, and music is stored there and is tied to the video. So each video will have different metadata.

## API and JSON

### API Code

<!-- ![Image Name](images/get-method.png "get-method") -->
```python
@token_required
def get(self, current_user): # Read Method
    users = User.query.all()    # read/extract all users from database
    json_ready = [user.read() for user in users]  # prepare output in json
    return jsonify(json_ready)  # jsonify creates Flask response object, more specific to APIs than json.dumps
```

<!-- ![Image Name](images/post-request.png "post-request") -->

```python
def post(self):
    body = request.get_json()
    uid = body.get('uid')
    print(uid)
    if uid is None:
        return {'message': f'User ID missing'}, 400
    email = body.get('email')
    print(email)
    if email is None or "@" not in email:
        return {'message': f'Email is blank or has an invalid format'}, 400
    user = User.query.filter_by(_uid=uid).first()
    print(user)
    if user:
        try:
            user.update_email(email)
            return jsonify(user.read())
        except Exception as e:
            return {
                "error": "Something went wrong",
                "message": str(e)
            }, 500
```

<!-- ![Image Name](images/post-method.png "post-method") -->

```python
def put(self):
    body = request.get_json()
    type = int(body.get('type'))
    videoID = int(body.get('videoID'))
    if videoID is None:
        return {'message': f'Video ID is missing'}, 400
    video = Vid.query.filter_by(_videoID=videoID).first()
    if video:
        if type == 0:
            try:
                put_req = video.put()
                return jsonify(video.read())
            except Exception as e:
                return {
                    "error": "Something went wrong",
                    "message": str(e)
                }, 500
        elif type == 1:
            try:
                put_req = video.like()
                return jsonify(video.read())
            except Exception as e:
                return {
                    "error": "Something went wrong",
                    "message": str(e)
                }, 500
        elif type == 2:
            try:
                put_req = video.dislike()
                return jsonify(video.read())
            except Exception as e:
                return {
                    "error": "Something went wrong",
                    "message": str(e)
                }, 500
```

This code show the API code for each request and response using the different methods.

The first piece of code is the get request and this will get the users from the database, prepares the output in JSON, and returns it.

The post request using `self` is for reading the user and for creating the user. It sets the UID and sets the email while checking if either field is valid (long enough and that it’s not missing or faked). And then it adds it to the database. This is also an example of data validation because it’s ensuring that the email and username are valid.

The update method allows you to add likes or dislikes to the video. It will update the database based on the id type.

### Postman

#### Postman Requests

The following are what each request looks like in Postman. These requests are what our JavaScript code does in the frontend when interacting with the backend.

The first image is a `GET` request with Postman. This will get all the users. This was used in the user panel/admin panel which displayed all the user information. The second image is a `POST` request which is for logging in. The body is a JSON payload with the UID (unique to each user) and a password. It will return a JWT token as a sign of authentication. And the last image is the `PUT` request for liking or disliking a video. The JSON payload has the type and the unique video ID. The video ID tells the backend which video you want to like/dislike. The type tells whether it’s a like or a dislike. In this case, we are disliking a video whose video ID is 1. After these images is another set of 3 images with the returns and success. Check those out to see what each request returns!

![Image Name](images/get-postman.png "get-postman")

![Image Name](images/post-postman.png "post-postman")

![Image Name](images/put-postman.png "put-postman")

#### Postman Success

These are each of the requests and the `200` returns are below:

![Image Name](images/postman-get-success.png "postman-get-success")

![Image Name](images/postman-post-success.png "postman-post-success")

![Image Name](images/postman-put-success.png "postman-put-success")

#### Postman 400

This is the JSON response for a missing body on the post request. So for example, if a user logged in and they didn’t put the correct password in or maybe the wrong username, you’ll get an error message.

![Image Name](images/post-400.png "post-400")

#### Postman 404

This is the 404 error for a `PUT`. Here, we are using the like and dislike function. But the type is an invalid number (can only be 1 or 2. Here, we put 89). And the video ID is also invalid (we don’t have 900 videos). This returns an error because it can’t find the video and also doesn’t understand what we are trying to do (since the type is invalid).

![Image Name](images/put-404.png "put-404")

## Frontend

### JSON returns

These are the JSON returns for each request. It also shows the request in the network tab to the right.

#### Post

This is the `POST` request. Here, I’m logging in on my account. The response status returned a 200 because I typed in the correct username and password. The actual response contains a JWT token.

![Image Name](images/json-post.png "json-post")

#### Put

![Image Name](images/json-put.png "json-put")

#### Get

The first image is the `GET` request. This gets all the users for the user admin panel. Here, we have a JSON list with each user. And within each list is the dictionary containing all the user information. This also reflects the code we looked at in the start of the blog with the lists and dictionaries. The second image is the `GET` request which is formatted (this is the user admin panel I mentioned). All the data is displayed in a table format. The second image also has the request right next to it.

![Image Name](images/json-get.png "json-get")

![Image Name](images/get-formatted.png "get-formatted")

### Code

#### Get Code

```js
// Function to format JSON data into a table
function formatDataIntoTable(data) {
    const table = document.getElementById("data-table");
    // Clear existing table content
    table.innerHTML = "";

    // Create table headers
    const headers = ["Name", "UID", "ID #", "Email", "DOB", "Role"];
    const headerRow = table.insertRow();
    headers.forEach((headerText) => {
        const th = document.createElement("th");
        th.textContent = headerText;
        headerRow.appendChild(th);
    });

    // Iterate over each user's data
    data.forEach((userData) => {
        const row = table.insertRow();
        // Insert data into cells
        row.insertCell().textContent = userData.name;
        row.insertCell().textContent = userData.uid;
        row.insertCell().textContent = userData.id;
        row.insertCell().textContent = userData.email;
        row.insertCell().textContent = userData.dob;
        row.insertCell().textContent = userData.role;
    });
}
```

This code is what is behind the GET request and obtains the array of objects. It will fetch every user and will use JSON to display them in the users table. This code also does iteration to append each element.

This JavaScript function, `formatDataIntoTable`, aims to organize JSON data into an HTML table. It begins by accessing the HTML table element with the id "data-table" where the formatted data will be displayed. Any existing content within this table is cleared to ensure a clean slate. Next, it creates table headers for each piece of data: "Name", "UID", "ID #", "Email", "DOB", and "Role". These headers are appended to a row (`<tr>`) within the table. Subsequently, the function iterates over each user's data within the provided array. For each user, a new row (`<tr>`) is inserted into the table, and their corresponding data is inserted into individual cells (`<td>` elements) within this row. This process continues until all user data has been formatted and displayed in the table.

#### Post Code (Success)

![Image Name](images/post-success.png "post-success")

Here, I logged into my account with the correct password. Now, it returns a success and a popup window to go to the user dashboard. The following is the code which handles this:

```js
// login worked
else {
    // Extract the token from cookies

    // Optionally, you can also store other data if needed
    localStorage.setItem("uid", uid);
    localStorage.setItem("loggedIn", true);

    console.log("Login successful");
    loginSuccess(); //popup window
}
```

```js
// 200 success
function loginSuccess() {
    // the login form elements
    var popupwindow200 = document.getElementById("popupWindow200");
    var closeButton200 = document.getElementById("closeButton200");

    popupWindow200.style.display = "block"; // display the window

    // redirect to dashboard
    closeButton200.addEventListener("click", function () {
        window.location.replace("/MKAAT-frontend/dashboard");
    });
}
```

#### Post Code (Failure)

![Image Name](images/post-failure.png "post-failure")

Here, I tried to login on my account but used a fake password (the wrong password). You can see that it returns a 401 error after and it displays a popup window to the user showing this. The 401 error can also be seen in the network tab. The following code details the error handling and displaying of the popup window. When a 401 error is detected, it will change the style element of a certain div from being hidden to being shown. This allows it to be a popup rather than a page redirect. This is a far better solution than redirecting the user to a different page because the user would have to go back to the previous page if they mess up their password. Instead, it makes more sense to give a popup and a close option. This is much cleaner and is better for UX.

```js
// note; code is cropped to be shorter
// error occured and login didn't work
if (!loginResponse.ok) {
    if (loginResponse.status === 401) {
        // wrong creds
        console.log(loginResponse.status);

        // below is the POPUP version of the error
        loginFailure(); //popup window
    }
}
```

```js
function loginFailure() {
    // the login form elements
    var popupWindow401 = document.getElementById("popupWindow401");
    var closeButton401 = document.getElementById("closeButton401");

    popupWindow401.style.display = "block"; // display the window

    // close the window if you click close
    closeButton401.addEventListener("click", function () {
        popupWindow401.style.display = "none";
    });
}
```

## ML

### Summary

Our machine learning algorithm was to predict inflation rates in the future (ex: 2030, 2040, and so on). Our project this trimester integrated a shop feature where users could buy items from points earned such as an adblocker for 5 minutes. The inflation feature can modify the cost of the items based on the predicted inflation rates of the future. This is what our machine learning algorithm does.

### Preparation

The following is the preparation of our data. Here’s how it works: we are first creating the variable for the model within the class. Afterwards, we open a dataset with inflation rates of the past. Using this, we calculate the percentage change between each year. After analyzing this, we can predict what the inflation rate may be in the future.

```python
def __init__(self):
    self.dt = None
    self.model = None
    self.X_train = None
    self.X_test = None
    self.y_train = None
    self.y_test = None
    self.encoder = None
    self.initInflation()

def initInflation(self):
    inflation_data = pd.read_csv('datasets/us_cpi.csv', header=None)
    self.td = inflation_data
    self.td.columns = ['Date', 'CPI']
    
    self.td['CPI'] = pd.to_numeric(self.td['CPI'], errors='coerce')
    self.td = self.td.dropna(subset=['CPI'])
    
    #convert yearmon to datetime objects
    self.td['Date'] = pd.to_datetime(self.td['Date'], format='%d-%m-%Y', errors='coerce')
    self.td['Date'] = self.td['Date'] + pd.offsets.MonthEnd(0)
    self.td['Timestamp'] = (self.td['Date'] - pd.Timestamp("1970-01-01")) // pd.Timedelta('1s')

    #Inflation rate calculation
    self.td['Inflation Rate'] = self.td['CPI'].pct_change() * 100
    
    self.td = self.td.dropna(subset=['Inflation Rate'])
```

### Using the Model

```python
def runLinearRegression(self, X, y):
    X = self.td['Timestamp'].values.reshape(-1,1)
    y = self.td['Inflation Rate'].values
    self.X_train, self.X_test, self.y_train, self.y_test = train_test_split(X, y, test_size=0.2, random_state=42)
    self.model = LinearRegression()
    self.model.fit(self.X_train, self.y_train)
    
    self.y_pred = self.model.predict(self.X_test)
    
def predictInflation(self, date):
    X = self.td['Timestamp'].values.reshape(-1,1)
    y = self.td['Inflation Rate'].values
    self.X_train, self.X_test, self.y_train, self.y_test = train_test_split(X, y, test_size=0.2, random_state=42)
    self.model = LinearRegression()
    self.model.fit(self.X_train, self.y_train)
    future_date = pd.to_datetime(date["date"], format='%Y-%m-%d')
    future_timestamp = (future_date - pd.Timestamp("1970-01-01")) // pd.Timedelta('1s')
    future_inflation_rate = self.model.predict(np.array([future_timestamp]).reshape(1,-1))
    
    randomInt = random.randint(1,50)
    
    if randomInt == 1:
        future_inflation_rate[0]*=2
    return future_inflation_rate[0]*10
```

These two functions are what actually let us predict inflation in the future. We use linear regression to predict inflation rates based on certain timestamps. The second function will call the previous one using a future timestamp. It then returns the predicted inflation rate which is amplified. If we were to remove the amplifier, the inflation would actually take many years to have a significant effect (since it takes a long time in real life). To speed things up, we just amplify it to a large amount.
