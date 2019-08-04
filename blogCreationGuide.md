# Steps to creating an app
For more details, see [Ruby on Rails](https://guides.rubyonrails.org/getting_started.html)

## Create a new Rails app
`$ rails new <appName>`

## Start up webserver
`$ rails server`

## See your app in action. Open a browser...
`http://localhost:3000`

## Routes, controllers, and views
- A _controller_ receives specific requests for the application

- _Routing_ decides which controller receives a request

- There are often more than one route to a controller

- Different routes serve different _actions_ (where actions to a controller are methods to a class)

- An action collects information to send to the view

- The _view_ displays the information

## Generate a new controller
```
$ rails generate controller <CONTROLLER_NAME> <ACTION>
# for example...
$ rails generate controller Welcome index
```

See `app/controllers` and `app/views` folders for relevant files. Note, the creation of the<br/> 
controller adds an entry to the application routing file in `config/routes.rb`. In the routing file<br/> 
you will see an entry which maps the new controller and action to a specific URL:<br/> 


`get 'welcome/index'` maps requests to `http://localhost:3000/welcome/index` to the welcome controllers index action


## Set the application homepage
Here we set homepage by defining a root in the `config/routes.rb` file, where we define the routes.

We can define the `root` as:

```
$ root controller#action
# or, in our example
$ root welcome#index
```

## Getting up and running with _resources_
_Resources_ are collections of related things, e.g., articles, people, places.<br/>
You can create, read, update and destroy (_CRUD_) items for a resource.<br/>
Rails provides a *resources* method for declaring a standard REST resource.<br/>
This would be set in your `config/routes.rb` file...<br/>
```
Rails.application.routes.draw do
  get 'welcome/index'
  resources :articles
  root 'welcome#index'
end
```

The outcome will be a series of routes created for the *articles* resource.<br/>
You can list all available routes using <br/>

`$ rails routes`

Some of the routes created are:<br/>
- index
- new
- create
- edit
- show
- update
- destroy

The next step is to create the controllers and associated actions for your *articles* resources.

## Creating controllers for resources
With the routes already created, we can make an `articles` controller to map<br/>
the actions to those ready-made routes. These actions will perform CRUD operations...

`$ rails generate controller Articles`

## Creating a `new` action

- Go to your new controller for `articles` in `app/controllers`
- Add a new action for `new`, but leave it empty
- Define the view in `app/view`. Set some HTML, anything you like
- navigate to `http://localhost:3000/articles/new`

## Creating actions with more substance!
Add the form code to the app/views/articles/new.html.erb file.<br/>
Form needs a different URL in order to go somewhere else, e.g., to **create** a form.<br/>
Be sure to add the url parameter to `form_with` using the base path arg for articles - `articles_path`.<br/>
The default action for creating new forms in rails is **create**, and is a route created for you.<br/>
Adding the `articles_path` as the URL tells Rails to point the form to the URI associated with the `articles` prefix.

```
$ bin/rails routes
      Prefix Verb   URI Pattern                  Controller#Action
welcome_index GET    /welcome/index(.:format)     welcome#index
     articles GET    /articles(.:format)          articles#index
              POST   /articles(.:format)          articles#create
```
The form will by default send a `POST` request to that route, which is associated with a **create** action.<br/>
So, lets define what happens in **create** actions.

## Creating articles
First, define a new action in the `articles` controller.<br/>
Make a new action called `create`. Add the following to print out <br/>
the parameters of the form object

```
def create
  render plain: params[:article].inspect
end
```

This is rendering to screen a hash (like Python dict) using key, value pairs of <br/>
`plain` and `params` respectively, where `params` is the method for representing the parameters <br/>
(or fields) of a form object.

## Creating the model
Models use a singular name, their corresponding database tables use a plural name.<br/>
In our example, we will create an Article model, with a _title_ attribute of type `string`,<br/>
and _text_ attribute of type `text`:

`$ rails generate model Article title:string text:text`

This will create two files of interest:

- `app/models/article.rb`
- `db/migrate/**some_number**_create_articles.rb  # creates database structure`

## Database migrations
Migrations are ruby classes that are designed to make it simple to create and modify database tables.<br/>
Rails uses rake commands to run migrations. Migrations are timestamped and ran in timestamped order.<br/>
Take a look at the `db/migrate/*create_articles.rb` file to see the migration class. You will see that <br/>
an articles table was created with your title and text params, as well as the timestamps as fields/columns.

You can now use rails to run the migration and create the **articles** table.

`$ rails db:migrate`

Congratulations, you have created somewhere to save your data!

## Saving data in the controller
We use the _POST_ verb to create (Crud)
We use the ArticlesController to define our _create_ action associated with the *POST* verb.<br/>
You will find the controller for articles in the `app/controller` folder.
However, Rails has security features that help keep your apps secure. One of them is <strong>strong parameters</strong>.<br/>
Strong parameter control entry into the database, by specifying what parameters are required, and allowed <br/>
```
def create
  @article = Article.new(params[:article])  # Creates new model with respective attributes
 
  @article.save  # save model in db
  redirect_to @article  # redirect to the show action, to be defined

  def article_params
    # Require and permit params
    params.require(:article).permit(:title, :text)
  end
end
```

## Showing articles
We use the _GET_ verb to show or read (cRud).

`article GET    /articles/:id(.:format)      articles#show`

`:id` is a required parameter, the ID of the article.<br/>

The show action is defined in the ArticleController...<br/>

```
def show
  @article = Article.find(params[:id])  # find article associated with :id and assign to instance variable
end
```

The *controller* has retrieved the information from the *model*, and this can now be shown in the *view*<br/>

```
<p>
  <strong>Title:</strong>
  <%= @article.title %>
</p>
<p>
  <strong>Text:</strong>
  <%= @article.text %>
</p>
```

## Listing all articles
The verb and action for getting all articles is listed in the `$ rails routes`.<br/>

`articles GET    /articles(.:format)          articles#index`

We must now define an index action in the ArticlesController<br/>

```
def index
  @articles = Article.all  # return all articles
end
```

Now the controller has extracted the information from the model, it can be shown in the `articles#index` view.<br/>
Note the added link to html via the embedded Ruby.<br/>

```
<h1>Listing articles</h1>
 
<table>
  <tr>
    <th>Title</th>
    <th>Text</th>
    <th></th>
  </tr>
 
  <% @articles.each do |article| %>
    <tr>
      <td><%= article.title %></td>
      <td><%= article.text %></td>
      <td><%= link_to 'Show', article_path(article) %></td>
    </tr>
  <% end %>
</table>
```

## Add more links
So far, we can create, show and list articles. Lets add some more navigation links...<br/>

To the welcome page (`views/welcome/index.html.erb`), add a "My Blog" link to take us to the list of articles<br/>
```
<h1>Hello, Rails!</h1>
<%= link_to 'My Blog', controller: articles_path %>
```

To the articles list (`views/articles/index.html.erb`), add a link to create a new article<br/>
```
<%= link_to 'New article', new_article_path  %>

# Also add the option to go back to the welcome page
<%= link_to 'Back', controller: 'welcome' %>

```

To the new article page (`views/articles/new.html.erb`), add the option to return to the article list.<br/>
```
<%= link_to 'Back', articles_path %>
```

To the show articles view (`views/articles/show.html.erb`), add the option to return to the article list.<br/>
```
<%= link_to 'Back', articles_path %>
```

## Addings some validation
Rails includes methods to help you validate the data that you send to models. See `app/models/article.rb`<br/>
First, set the validates command in the model:<br/>

```
# Validation, must have title, and title must be 5 chars minimum
validates :title, presence: true, length: {minimum: 5}
```

Amend the article controller to respond to a failure to save, based on validation result:<br/>

```
def new
  @article = Article.new
end


def create
  @article = Article.new(article_params)
 
  if @article.save
    redirect_to @article
  else
    render 'new'
  end  
end

```

The `new` action defines a new instance of Article.new<br/>
The create option now returns a `bool` if save is successful, based on the validation.<br/>
A successful save redirects the user to the `@article` object.<br/>
A failed save renders the `@article` object is passed back to the new template when it is rendered.<br/>
So, you the users input is not lost, whereas `redirect_to` would issue another request.<br/>

To report validation errors to the user, amend the `app/views/articles/new.html.erb` to:<br/>

```
<% if @article.errors.any? %>
    <div id="error_explanation">
      <h2>
        <%= pluralize(@article.errors.count, "error") %> prohibited
        this article from being saved:
      </h2>
      <ul>
        <% @article.errors.full_messages.each do |msg| %>
          <li><%= msg %></li>
        <% end %>
      </ul>
    </div>
<% end %>
```

- `@article.errors.any?` Bool, checks for existence of errors
- `pluralize(@article.errors.count, "error")` Rails helper that pluralizes words if first arg > 1
- `@article.errors.full_messages.` Returns a list of errors 

The reason `@article = Article.new` was initialised in the controller is because otherwise `@article` would be nil in the view.<br/>

## Updating articles *(crUd)*
Add an *edit* action to the article controller:<br/>

```
def edit
  @article = Article.find(params[:id])
end
```

Add an `edit` view, which is almost a copy of the `new` view. The only difference is the header and the call to `form_with`<br/>

```
<h1>Edit article</h1>
<%= form_with(model: @article, local: true) do |form| %>
```
By passing the `@article` object to the `model` parameter, we are telling rails to fill the form fields<br/>
with the fields from the `@article` object. Passing in a symbol scope (`scope: :article`), creates unfilled fields <br/>

Define the update action in the article controller:<br/>

```
def update
  @article = Article.find(params[:id])
 
  if @article.update(article_params)
    redirect_to @article
  else
    render 'edit'
  end
end
```
Operates similar to the `create` action, but finds and updates a form based on the ID.

Finally, create an edit link on the articles list index page:<br/>

```
<td><%= link_to 'Edit', edit_article_path(article) %></td>
```
and also add one to the `Show` view.

```
<%= link_to 'Edit', edit_article_path(@article) %> |
```
## Using partials to reduce duplication
The `new` and `edit` views for displaying forms are almost identical.<br/>
Using partial view files (prefixed with `'_'`) remove this duplication.<br/>

You can use the code from the `edit` view for a partial, named `_form.html.erb`.<br/>
Remember to remove headers from the form template.<br/>
Then, change the `new` and `edit` view files to render the form view. E.g., <br/>

```
<h1>New article</h1>

<%= render 'form' %>

<%= link_to, 'Back', article_path %>
```

## Deleting articles (cruD)
Following the REST convention, the route for deleting articles is<br/>:

`DELETE /articles/:id(.:format)      articles#destroy`

To delete articles, we define a *destroy* action in the article controller<br/>

```
def destroy
  @article = Article.find(params[:id])
  @article.destroy

  redirect_to articles_path
end
```
Note, we do not need to define a destroy view, since we redirect to the articles index(list) page<br/>
Finally, add the `Destroy` links:

```
<td><%= link_to 'Destroy', article_path(article),
              method: :delete,
              data: { confirm: 'Are you sure?' } %></td>
```

## Adding some security measures
Currently, the blog allows anyone to create, update and destroy articles.<br/>
To add some basic security, you can require that a username and password is required for <br/>
your *CRUD* operations, except when for read/show actions. Add the following to your articles controller <br/>

```
http_basic_authenticate_with name: "admin", password: "secret", except: [:index, :show]

def index
  # etc...
end

```

Congrations, you can now Create, Read, Update and Destroy articles, and have some basic<br/>
security measures in place to protect your articles from unauthorised modification.

