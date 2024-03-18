# Hitch
> *Any person, any place*

Hitch is a ride-hailing app that aims to help the Cornell community by providing easy and reliable transportation anywhere in Ithaca. After logging in and inputting a destination, Hitch matches a user with a student driver and quickly schedules a ride. Addtional features include a sidebar with ride history and a profile page where the user can change their information.

Requirement fullfillment:

**Backend**
1. Four endpoints
   - Nine endpoints created, detailed in full in the API below
2. API specifications (see below!)
3. Relational database schema
   - Using SQLAlchemy, we created a similar relational database structure to the CMS assignment
   - We have two tables, users and rides, with relational tables splitting the users into driver and rider for each ride, along with rides where the user rode and rides where the user drove
   - These tables allow us to easily populate the ride history page
4. Heroku deployment
   - The frontend sends all requests to our Heroku deployment page
   - The url (as in the API) is https://hitchhackchall.herokuapp.com/
   
**iOS**
1. All of the structs and classes MakeRide, User, and PastRides conform to Codable.
2. The MenuViewController and PastRidesViewController use UITableView.
3. All views use NSLayoutConstrant.
4. The menu button in the UITabBarController navigates between screens as do the objects in MenuViewController.
5. Integration with API
   - Hitch uses the MapKit API and CoreLocation for all location and mapping.
   - All the networking and backend integration conforms to the API created by the backend team to log in, 
   access profile data, past rides, and a real time database of ongoing rides.

## Backend API

Send requests to Heroku deployment at: https://hitchhackchall.herokuapp.com/

If a given endpoint fails, the API returns a JSON as below, along with an error code (frequently 404).
```
{
   "success": false,
   "error": <ERROR DESCRIPTION>
}
```

### Register and create user

Register a user in the database, passing their login credentials, name, and current location. An example request is shown below.

`POST /register/`

Request:
```
{
    "email": "pi@gma.com",
    "password": "password",
    "name" : "me",
    "latitude" : 55.5,
    "longitude" : 60.7
}
```
Returns basic user info

Response:
```
{
   "success": true,
   "data": {
             "id": <ID>,
             "name": <USER INPUT FOR NAME>,
             "email": <USER INPUT FOR EMAIL>,
             "longitude": <USER INPUT FOR LONGITUDE>,
             "latitude": <USER INPUT FOR LATITUDE>,
             "session_token": <SESSION TOKEN>
   }
}
```

### Login

Verify username and password associated with a user, returns the expiration time for that user's current login.

`POST /login/`

Request:
```
{
   "email": <EMAIL>,
   "password": <PASSWORD>
}
```

Response:
```
{
   "success": true,
   "data": {
            "id": <USER ID>,
            "name": <NAME>,
            "email": <EMAIL>,
            "longitude": <LONGITUDE>,
            "latitude": <LATITUDE>,
            "session_token": <SESSION TOKEN>
   }
}
```

### Logout

Logs out specified user.

`POST /logout/`

Request:
```
{
   "email": <EMAIL>
}
```

Response:
```
{
   "success": true,
   "data": {
      "email": <USER EMAIL>
   }
}
```

### Edit user info

Only some of these parameters are required - session token is always required to make edits.

`POST /api/user/<email>/`

Example request:
```
{
    "name": "pete",
    "current_session": "333109b965ec75ffdcf21342cf8deb7707111218"
}
```

Response:
```
{
   "success": true,
   "data": {
            "id": <USER ID>,
            "name": <NAME>,
            "email": <EMAIL>,
            "longitude": <LONGITUDE>,
            "latitude": <LATITUDE>
          }
}
```

### Get user info

`GET /api/user/<email>/`

Response:
```
{
   "success": true,
   "data": {
            "id": <USER ID>,
            "name": <NAME>,
            "email": <EMAIL>
          }
}
```

### Change user location by email

`POST /api/user/<email>/changecoords/`

Request:
```
{
  "latitude": <USER LATITUDE>,
  "longitude": <USER LONGITUDE>,
  "current_session": <SESSION TOKEN>
}
```
Latitude and longitude are both optional - if you don't include one, it won't update.

Response:
```
{
   "success": true,
   "data": {
            "id": <USER ID>,
            "name": <NAME>,
            "email": <EMAIL>,
            "longitude": <LONGITUDE>,
            "latitude": <LATITUDE>
          }
}
```

### Make a new ride

`POST /api/rides/<email>/`

```
{
  "end_lat": <RIDE END LATITUDE>,
  "end_long": <RIDE END LONGITUDE>,
  "session_token": <SESSION TOKEN>
}
```
Creates a ride from user's current location to the given endpoints.

Success Response:
```
{
  "success": true,
  "data": {
            "id": <ride id>,
            "begin_lat": <start point latitude>,
            "begin_long": <start point longitude>,
            "end_lat": <end point latitude>,
            "end_long": <end point longitude>,
            "status": <ride status where 0 = waiting, 1 = active, 2 = completed>
        }
}
```

### Mark ride as complete

`POST /api/rides/<email>/complete/`

Request:
```
{
   "current_session": <SESSION TOKEN>
}
```

Response:
```
{
  "success": true,
  "data": {
            "id": <ride id>,
            "begin_lat": <start point latitude>,
            "begin_long": <start point longitude>,
            "end_lat": <end point latitude>,
            "end_long": <end point longitude>,
            "status": <ride status where 0 = waiting, 1 = active, 2 = completed>
        }
}
```

### Get ride history

`GET /api/user/<email>/history/`

Response has a list of completed rides as the "data" element.

Request:
```
{
   "current_session": <SESSION TOKEN>
}
```

Example response with two rides in ride history :
```
{
  "success": true,
  "data": [{
            "id": <ride id>,
            "begin_lat": <start point latitude>,
            "begin_long": <start point longitude>,
            "end_lat": <end point latitude>,
            "end_long": <end point longitude>,
            "status": 2
         },
         {
            "id": <ride id>,
            "begin_lat": <start point latitude>,
            "begin_long": <start point longitude>,
            "end_lat": <end point latitude>,
            "end_long": <end point longitude>,
            "status": 2
         }
        ]
}
```
