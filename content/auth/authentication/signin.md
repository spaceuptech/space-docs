---
title: "Email Signin"
date: 2019-09-23T09:21:49+05:30
draft: true
weight: 1
toc: false
---

You can easily allow users to log in to your app via email using the `db.signIn` function. This is how you could add a simple log in feature in your app. Make sure you have enabled the `auth` module. Here's a code snippet to implement basic email sign in:

<div class="row tabs-wrapper">
  <div class="col s12" style="padding:0">
    <ul class="tabs">
      <li class="tab col s2"><a class="active" href="#signin-js">Javascript</a></li>
      <li class="tab col s2"><a href="#signin-java">Java</a></li>
      <li class="tab col s2"><a href="#signin-python">Python</a></li>
      <li class="tab col s2"><a href="#signin-golang">Golang</a></li>
    </ul>
  </div>
  <div id="signin-js" class="col s12" style="padding:0">
{{< highlight javascript >}}
import { API } from 'space-api';

// Initialize api with the project name and url of the space cloud
const api = new API('todo-app', 'http://localhost:4122');

// Initialize database(s) you intend to use
const db = api.Mongo();

// SignIn
db.signIn('demo@example.com', '1234').then(res => {
  if (res.status === 200) {
    // Set the token id to enable operations of other modules
    api.setToken(res.data.token)
    
    // res.data contains request payload
    console.log('Response:', res.data);
    return;
  }
  // Request failed
}).catch(ex => {
  // Exception occured while processing request
});
{{< /highlight >}}     
  </div>
  <div id="signin-java" class="col s12" style="padding:0">
{{< highlight java >}}
API api = new API("books-app", "localhost", 4124);
SQL db = api.MySQL();
db.signIn("email", "password", new Utils.ResponseListener() {
    @Override
    public void onResponse(int statusCode, Response response) {
        if (statusCode == 200) {
            try {
                System.out.println("Token: " + response.getResult(Map.class).get("token"));
            } catch (Exception e) {
                e.printStackTrace();
            }
        } else {
            System.out.println(response.getError());
        }
    }

    @Override
    public void onError(Exception e) {
        System.out.println(e.getMessage());
    }
});
{{< /highlight >}}     
  </div>
 <div id="signin-python" class="col s12" style="padding:0">
{{< highlight python >}}
from space_api import API

// Initialize api with the project name and url of the space cloud
api = API("books-app", "localhost:4124")

// Initialize database(s) you intend to use
db = api.my_sql()

// Sign In
response = db.sign_in("user_email", "user_password")
if response.status == 200:
    print(response.result)
else:
    print(response.error)

api.close()
{{< /highlight >}}    
  </div>
  <div id="signin-golang" class="col s12" style="padding:0">
{{< highlight golang >}}
import (
	"github.com/spaceuptech/space-api-go/api"
	"fmt"
)

func main() {
	api, err := api.New("books-app", "localhost:4124", false)
	if(err != nil) {
		fmt.Println(err)
	}
	db := api.MySQL()
	resp, err := db.SignIn("email", "password")
	if err != nil {
		fmt.Println("Error:", err)
	} else {
		if resp.Status == 200 {
			var v map[string]interface{}
			err:= resp.Unmarshal(&v)
			if err != nil {
				fmt.Println("Error Unmarshalling:", err)
			} else {
				fmt.Println("Result:", v)
			}
		} else {
			fmt.Println("Error Processing Request:", resp.Error)
		}
	}
}
{{< /highlight >}}     
  </div>
</div>

As you would have noticed, the above function is asynchronous in nature. The `signIn` method takes 2 parameters:

- **email** - Email of the user.
- **pass** - Password of the user.

## Response

On getting the log in request, Space Cloud validates whether such an user exists and sends a response accordingly. A response object sent by the server contains the **status** and **data** fields explained below.

**status:** Number describing the http status code of the response. Following values are possible:

- 200 - Successful sign in
- 404 - No user with the given email
- 401 - The given credentials are not correct
- 500 - Internal server error

**data:** The data object consists of the following fields:

- **token** - The JWT token generated by the authentication module. The token contains the following claims - **id** (the unique id for the user), **role** and **email**
- **user** - Row / document corresponding to the signed in user

## Next steps

The next step would be checking out the client reference to register a new user.