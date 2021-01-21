# intro-to-ruby
<hr>
Title: Controllers II<br>
Type: Lesson<br>
Creator: Thom Page<br>
Topics: Rails controllers: Create, Update, Destroy, Postman, cURL<br>
<hr>

# CONTROLLERS II

###### CREATE, UPDATE, and DESTROY --> POST, PUT, and DELETE

### Lesson Objectives
_After this lesson, students will be able to:_

- Write a controller method that creates data
- Send a POST request using either Postman or cURL to the create route
- Write a controller method that updates data
- Send a PUT request using either Postman or cURL to the update route
- Write a controller method that destroys data
- Send a DELETE request using either Postman or cURL to the destroy route

<hr>

## SETUP

* Open the `songs_app_api` project.

## POSTMAN

![](https://i.imgur.com/7XkQ4EB.png)

For our server requests we will look into another useful piece of software called Postman.

Postman allows us to do everything cURL does, but with a nice interface.

* **You probably already have this installed from earlier in the class!**
* Download and install Postman: `https://www.getpostman.com/app/download/osx64`

[Postman](https://www.getpostman.com/app/download/osx64)

You don't have to register / sign up to use it. Skip that step.

We will learn how to use it soon. Armed with Postman, let's make a **create** method in our songs app.

<hr>

## &#x1F4A5; CREATE

In `songs_controller.rb` we need to add a method along with **index** and **show**, one that will create data.

What could it be? `rails routes` will tell us.

```ruby
$ rails routes
Prefix Verb   URI Pattern          Controller#Action
 songs GET    /songs(.:format)     songs#index
       POST   /songs(.:format)     songs#create
  song GET    /songs/:id(.:format) songs#show
       PATCH  /songs/:id(.:format) songs#update
       PUT    /songs/:id(.:format) songs#update
       DELETE /songs/:id(.:format) songs#destroy
```

Looks like the `Controller#Action` is `create` so let's add that method to the controller file. 

```ruby
def create
end
```

With the create method in place let's send some data and confirm it's being received.  Data this is received will be stored in a variable called `params` so let's log params to the console to confirm that it's being received. 

```ruby
def create
    puts "this is params: #{params}"
end
```

Although we will be using Postman for the majority of the lecture there is also another tool that we can use to send data called `curl`

The cURL request will be configured with `model[column]` key name syntax:

```bash
curl -X POST -d "song[title]=Peaks" -d "song[artist_name]=Dictaphone" -d song[artwork]="None" localhost:3000/songs
```

Copy and paste the above curl command into the terminal and you should see the following:

```ruby
Started POST "/songs" for ::1 at 2020-08-02 11:44:51 -0400
   (4.0ms)  SELECT "schema_migrations"."version" FROM "schema_migrations" ORDER BY "schema_migrations"."version" ASC
Processing by SongsController#create as */*
  Parameters: {"song"=>{"title"=>"Peaks", "artist_name"=>"Dictaphone", "artwork"=>"None"}}
this is params: {"song"=>{"title"=>"Peaks", "artist_name"=>"Dictaphone", "artwork"=>"None"}, "controller"=>"songs", "action"=>"create"}
Completed 204 No Content in 0ms (ActiveRecord: 0.0ms | Allocations: 110)
```

As can be seen in the output the `params` object contains not only the data but also info about the `controller` and `action`.

```ruby
this is params: {
  "song"=>{"title"=>"Peaks", "artist_name"=>"Dictaphone", "artwork"=>"None"}, 
  "controller"=>"songs", 
  "action"=>"create"
  }
```

### Create A New Song


Since the data we need is saved in the `song` key let's make sure we can grab just that data. 


```ruby
def create
   puts "this is params: #{params[:song]}"
end
```

The console should output:

```ruby
this is params: {
  "title"=>"Peaks",
  "artist_name"=>"Dictaphone", 
  "artwork"=>"None"
}
```

Now that we have just the song data as an object let's try and use `Song.create()` to create it. 

```ruby
def create
   puts "this is params: #{params[:song]}"
   Song.create(params[:song])
end
```

Now resend the previous curl request. 

```ruby
Started POST "/songs" for ::1 at 2020-08-02 12:03:35 -0400
   (0.7ms)  SELECT "schema_migrations"."version" FROM "schema_migrations" ORDER BY "schema_migrations"."version" ASC
Processing by SongsController#create as */*
  Parameters: {"song"=>{"title"=>"Peaks", "artist_name"=>"Dictaphone", "artwork"=>"None"}}
this is params: {"title"=>"Peaks", "artist_name"=>"Dictaphone", "artwork"=>"None"}
Completed 500 Internal Server Error in 9ms (ActiveRecord: 5.1ms | Allocations: 3368)

ActiveModel::ForbiddenAttributesError (ActiveModel::ForbiddenAttributesError):
  
app/controllers/songs_controller.rb:18:in `create'
```

As you can see from the output, ActiveRecord was not able to create the song.   The reality is that ActiveRecord was not allowed to created the song.  Rails is only trying to help our server be a bit more secure.  

It will however allow us to create the song in the following ways:

- store the song data in a new variable
- update ActionController::Parameters.permit_all_parameters
- using strong params

#### Create A New Variable

```ruby
def create
   puts "this is params: #{params[:song]}"
    song_params = {title: params['title'], artist_name: params[:artist_name], artwork: params[:artwork]}
   Song.create(song_params)
end
```

You should see the following output.

```ruby
Started POST "/songs" for ::1 at 2020-08-02 12:10:12 -0400
Processing by SongsController#create as */*
  Parameters: {"song"=>{"title"=>"Peaks", "artist_name"=>"Dictaphone", "artwork"=>"None"}}
this is params: {"title"=>"Peaks", "artist_name"=>"Dictaphone", "artwork"=>"None"}
   (0.2ms)  BEGIN
  ↳ app/controllers/songs_controller.rb:21:in `create'
  Song Create (0.9ms)  INSERT INTO "songs" DEFAULT VALUES RETURNING "id"
  ↳ app/controllers/songs_controller.rb:21:in `create'
   (0.8ms)  COMMIT
  ↳ app/controllers/songs_controller.rb:21:in `create'
Completed 204 No Content in 9ms (ActiveRecord: 1.9ms | Allocations: 5155)
```

You can even confirm this has been created using `Song.all`

This seems like a lot of work having to always update create a new variable with the data.  So let's configure the `ActionController` settings to provide us the ability to accept the data as is and create the song. 

#### Setting ActionController::Parameters

In the `application_contrroller.rb` file add the following line and resend the data.

```ruby
class ApplicationController < ActionController::API
    ActionController::Parameters.permit_all_parameters = true
end
```

Update the `create` method and remove the song_params variable and go back to our previous code.

```ruby
   def create 
      puts "this is params: #{params[:song]}"
      Song.create(params[:song])
  end
```

Although the second way is much less code to write, it's less secure as we are accepting all column and data.  In the end we want to allow our create method to only accept the data we know should be sent to the server.   

So let's make use of the third option:  `strong params`


### Strong Params

Strong params is a security measure we use to make sure no one can send errant or malicious data to our controllers. 


#### Strong Params Use Case
Consider the scenario. A bank has created an account for a new user which contains the users info along with the account number.  All of that data is stored in the same database however the user should only be able to update their personal info and not the account number or balance without going through the proper channels.  

This is where strong params comes into play.  We can use it to say exactly what fields will be accepted. 

First remove or comment out the previous code added to the `ApplicationController` as we no longer will be using that option. 

```ruby
class ApplicationController < ActionController::API
   # ActionController::Parameters.permit_all_parameters = true
end
```

To work with **strong params**, we write a gatekeeper method inside a `private` section of our controller file. Let's call the private method `song_params`. We can reference this method later in our controller methods.

```ruby
  private # Any methods below here

  def song_params
    # Returns a sanitized hash of the params with nothing extra
    params.required(:song).permit(:title, :artist_name, :artwork)
  end
```

The controller should look like the following:

```ruby
class SongsController < ApplicationController

  def create 
    puts "this is params: #{params[:song]}"
    Song.create(params[:song])
  end

  private # Any methods below here

  def song_params
    # Returns a sanitized hash of the params with nothing extra
    params.required(:song).permit(:title, :artist_name, :artwork)
  end
end
```

All we are doing is returning the globally available **params hash** with some checks on it. We are running methods on the hash:
* We chain `required` to require the specific model
* We chain `permit` to permit specific columns.


### PRIVATE

What does `private` do?

Regular, public methods of a controller class are exposed to the web server through Rails routes, but private methods are not. If you make any helper methods that do not correspond to routes, make them private. This will keep them inaccessible to anyone.

Update the create method to pass the data through strong_params first before creating the song and then use an if/else condition block to send a response. 

### The Create Method

```ruby
  def create
    song = Song.new(song_params)

    if song.save
      render(status: 201, json: { song: song })
    else
      render(status: 422, json: { song: song })
    end
  end
```

**What is this stuff?**

* `song = Song.new(song_params)`

Establishes a new song using the params that were permitted with the private song_params method. Saves it to a variable, `song`.

* `if song.save render(status: 201, json: { song: song })`

If the `song` variable saves without any errors, then send a `201` status code (everything is chill), along with the song data. Note: status code `200` would be fine, too.

![](https://i.imgur.com/bLf0OZA.png)

[HTTP Status Code 201 - Created](https://httpstatuses.com/201)

* `else render(status: 422, json: { song: song })`

Woops! Something went wrong! Send a status code of `422` Unprocessable Entity, along with whatever was in the `song` variable for debugging.

![](https://i.imgur.com/edPAtk6.png)

[HTTP Status codes](https://httpstatuses.com/)

[422 - Unprocessable Entity](https://httpstatuses.com/422)


Our `songs_controller` should look similar to the following:


```ruby
class SongsController < ApplicationController
  def index
    render(json: { songs: Song.all })
  end

  def show
    # Input comes in from the `params` hash
    render(json: Song.find(params[:id]))
  end

  def create
    song = Song.new(song_params)

    if song.save
      render(json: { song: song }, status: 201)
    else
      # Unprocessable Entity
      render(json: { song: song }, status: 422)
    end
  end

  private

  def song_params
    params.required(:song).permit(:title, :artist_name, :artwork)
  end
end
```

## &#x1F50A; SEND REQUESTS 

### &#x1F48C; POSTMAN

We will be using Postman from this point on to make additional requests to the server.  But first let's test that we can post data as well. 

* Open Postman

* Change the request type to POST:

![](https://i.imgur.com/2Rj6dKZ.png)

<br>

* Click on `body`

![](https://i.imgur.com/tRMswrj.png)

<br>

In `body` we can write our key-value pairs. These pairs will look different to how we did it with Node/Express. The key will look like: `model[column]`.

* In `key` put `song[title]`
* In `value` put anything. NO quotes.
* Do the same for `artwork` and `artist_name`

![](https://i.imgur.com/gRri1sB.png)

<br>

Let's send the data to `localhost:3000/songs`

![](https://i.imgur.com/mTNy44C.png)

<br>

Hit the **SEND** button to make the request.

If all goes well, you should see the response below the key-value fields:

![](https://i.imgur.com/HXsOeoY.png)

You can check your data in the browser -- index or show route.

<br>
<hr>

## VALIDATION

In POSTMAN, what if we send data for a column that doesn't exist?

Change `song[artist_name]` to `song[zartist_name]`

![](https://i.imgur.com/04KOq8Y.png)

**SEND**

Response:

![](https://i.imgur.com/VLSjHrf.png)

`zartist_name` was ignored, and `artist_name` is null. Good! It looks like any data that we send that does not conform to the `params.require().permit()` helper method is ignored completely. Thanks, **strong params**.

However, what if we want to prevent that `null` field? Let's make it so there **must** be data sent to the `artist_name` column, otherwise the request should fail.

* Add a simple validation to the `song.rb` file:

`app/models/song.rb`

```ruby
class Song < ApplicationRecord
  validates :artist_name, presence: true
end
```

**SEND THE REQUEST AGAIN FROM POSTMAN**

Remember to send `[song]zartist_name`

![](https://i.imgur.com/vItv6pw.png)

Thanks to our validation, we got that Status Code `422` that we programmed in to our create method. Nothing was entered into the database because the validation failed.

**SEND THE REQUEST AGAIN**

* Change `song[zartist_name]` back to `song[artist_name]`

* The request should work again with a Status `200`.

[More on validations](http://guides.rubyonrails.org/active_record_validations.html)

<hr>

### UPDATE AND DESTROY:

```ruby
  def update
    song = Song.find(params[:id])
    song.update(song_params)
    render(json: { song: song })
  end
```


* find the song by the id in the request url
* update the song according to the permitted params
* render status `200` and the result if successful
* you could add in a check for Unprocessable Entity if you like

```ruby
  def destroy
    song = Song.destroy(params[:id])
    render(status: 204)
  end
```

* destroys the song according the id in the request url
* renders the status code `204`: No Content

![](https://i.imgur.com/7qkK4aV.png)

[Status Code 204](https://httpstatuses.com/204)

<br>

The controller files should look as follows:

```ruby
class SongsController < ApplicationController
  def index
    render(json: { songs: Song.all })
  end

  def show
    render(json: Song.find(params[:id]))
  end

  def create
    song = Song.new(song_params)

    if song.save
      render(json: { song: song }, status: 201)
    else
      render(json: { song: song }, status: 422)
    end
  end

  def update
    song = Song.find(params[:id])
    song.update(song_params)
    render(json: { song: song })
  end

  def destroy
    song = Song.destroy(params[:id])
    render(status: 204)
  end

  private

  def song_params
    params.required(:song).permit(:title, :artist_name, :artwork)
  end
end
```



### SEND A PUT REQUEST

Make a Postman request to update some data:

* Open a new Postman tab
* Select PUT
* Enter the URL: `localhost:3000/songs/6`
* Select Body
* key: `song[title]`
* value: `Fade Into You`

![](https://i.imgur.com/MGTJQa0.png)

Hit **send**

<br>
Do it again, but change a different attribute:

* Open a new Postman tab
* Select PUT
* Enter the URL: `localhost:3000/songs/6`
* Select Body
* key: `song[artist_name]`
* value: `Mazzy Star`

Hit **send**

<br>

Result of both updates:

![](https://i.imgur.com/BOQjlZw.png)

<br>
<hr>


## SEND A DELETE REQUEST

* Open a new Postman tab
* Select DELETE
* Enter the URL: `localhost:3000/songs/6`

![](https://i.imgur.com/Z0CWe2v.png)

Hit **send**

Response 204:

![](https://i.imgur.com/2zz72nP.png)

* In your browser, go to `localhost:3000/songs`. The index route.
* The song with id: 6 should be gone.

![](https://i.imgur.com/oRec1Ih.png)


### cURL

You can also try this using curl.


POST request to create data with cURL:

```bash
curl -X POST -d "song[title]=Peaks" -d "song[artist_name]=Dictaphone" -d song[artwork]="None" localhost:3000/songs
```

PUT request to update data

```bash
curl -X PUT -d "song[title]=Dem Bones" localhost:3000/songs/13
```

DELETE request to delete data

```bash
curl -X DELETE localhost:3000/songs/13
```

### Resources

- [Strong Params](https://learn.co/lessons/strong-params-basics)

<br>


