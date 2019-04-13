# Python REST API Example (With Microservices) 

## Writing REST APIs with Python is an important skill for the microservices world. We'll start learning the steps with a basic class in this tutorial. 

This is instructions on how to write REST APIs with Python for microservices.

With the transition to microservices, it becomes necessary to know how to write simple REST APIs using Python.

In this post, I give a Python REST API example using Tornado. This is the first post in the series where we will design the microservice and code the sample class that the microservice will manage.

## Design

To begin with, let's go ahead and define what out microservice is going to do. For this example, I want to track books. It can add books, remove books and give us a list of all the books. We could add more functionality but I want to keep this simple. This gives us a couple of endpoints:

We are not going to use a database, so we will have no persistence if we kill the web service. Again, I really want to keep this Python REST API example simple.
Our Python REST API Example Code

Now let's get to the code. Let's start out by writing our Book class that we will use to track our books.
```
import json

class Book:

    def __init__(self):
        self.books = []

    def add_book(self, title, author):
        new_book = {}
        new_book["Title"] = title
        new_book["Author"] = author
        self.books.append(new_book)
        print("Book: {0}".format(new_book))
        return json.dumps(new_book)

    def del_book(self, title):
        found = False
        for idx, book in enumerate(self.books):
            if book["Title"] == title:
                index = idx
                found = True
                del self.books[idx]
        print("books: {0}".format(json.dumps(self.books)))
        return found

    def get_all_books(self):
        return self.books

    def json_list(self):
        return json.dumps(self.books)
```

This is just a very simple class to let us add, delete and show all books which is a simple list of dicts which contains the book title and author. Notice that we have a method that returns JSON back. We will need this later for our web service to send output back.

## Learn the next steps of writing a Python REST API with an API example using Tornado. 
With the transition to microservices, it has become necessary to know how to write simple REST APIs using Python. In this post, I will give a Python REST API example using Tornado.

This is the second part of the series, where we will code the API.
Our Python REST API Example Code

Let's get right into it. Here is the main API code:
api.py
```
import tornado.ioloop
import tornado.web

from book import Book
from addhandler import AddHandler
from delhandler import DelHandler
from gethandler import GetHandler

books = Book()

class MainHandler(tornado.web.RequestHandler):
    def get(self):
       self.write("Book Microservice v1")

def make_app():
    return tornado.web.Application([
        (r"/v1", MainHandler),
        (r"/v1/addbook", AddHandler, dict(books = books)),
        (r"/v1/delbook", DelHandler, dict(books = books)),
        (r"/v1/getbooks", GetHandler, dict(books = books)),
        ])

if __name__ == "__main__":
    app = make_app()
    app.listen(8888)
    tornado.ioloop.IOLoop.current().start()
```
We import the Book class we defined in the last part so that we can work with books. Next, we import three handlers that we will define in the next part of this series.

Next, we define the MainHandler class, which is derived from tornado.web.RequestHandler. This is a very basic version of a RequestHandler that will just return "Book Microservice v1" to the browser.

In the make_app method, we define the main Tornado application.
```
def make_app():

    return tornado.web.Application([
        (r"/v1", MainHandler),
        (r"/v1/addbook", AddHandler, dict(books = books)),
        (r"/v1/delbook", DelHandler, dict(books = books)),
        (r"/v1/getbooks", GetHandler, dict(books = books)),

])
```
This returns a new Tornado application with several endpoints. These correlate to the endpoints defined in part one: addbook, delbook, and getbooks. The first part is what Tornado uses to match with the URL. For example:

http://192.0.2.98:8888/v1/addbook?title=test

Then the first handler would match "/v1/addbook" and AddHandler would be called. The last parameter defined is how we persist the books object between the handlers. We defined the books variable in line 8 of the api.py code. We then pass it to each of the handlers when called, which performs the desired action on the object.

The last bit of the code sets up a loop and runs the application on port 8888.



## This is the third part of this series, where we will code the API endpoints.
Our Python Rest API Example Code

As stated earlier, this post is where we code the endpoint handlers for our microservice. The first handler we will code is the AddHandler class which we will use to add a book.

addhandler.py
```
import tornado.web
import book
import json

class AddHandler(tornado.web.RequestHandler):
    def initialize(self, books):
        self.books = books

    def get(self):
        title = self.get_argument('title')
        author = self.get_argument('author')
        result = self.books.add_book(title, author)
        self.write(result)
```
Notice the initialize method of our class. This is how we pass the instantiated books object to this class so that we can actually do something with it. If we defined a new one here it would instantiate a new Book object for each request. This way we persist the data across the calls as long as the Tornado application is running. A better way to do this is to use a database, but I wanted to keep it simple for this post. In a future post, we will convert this microservice to use a database backend so that the data can persist between application runs.

We use the tornado.web.RequestHandler.get_argument method to get the URL parameters we need to add a book (title and author).

Here is the DelHandler code:

delhandler.py
```
import tornado.web
import book
import json

class DelHandler(tornado.web.RequestHandler):

    def initialize(self, books):
        self.books = books

    def get(self):
        title = self.get_argument('title')
        result = self.books.del_book(title)
        if result:
            self.write("Deleted book title: {0} succsessfully".format(title))
            self.set_status(200)
        else:
            self.write("Book '{0}' not found".format(title))
            self.set_status(404)
```
This is very similar to our AddHandler above. We pass it the books object in the initialize method so we can work with it. We also define a parameter "title" which we will use to call the books.del_book(title) method to delete the book. This will return true or false based on whether the book was found and deleted. If True then the book was deleted and we can return an HTTP status of 200 with the book title deletion status. If the book was not found then we return an HTTP status of 404 and write a status that we can't find the book with the given title.

The last handler is the GetHandler which will list out all our books as JSON. This will make it handy to use from other microservices or applications.

gethandler.py
```
import tornado.web
import book
import json

class GetHandler(tornado.web.RequestHandler):

    def initialize(self, books):
        self.books = books

    def get(self):
        self.write(self.books.json_list())
```
This our easiest handler because it doesn't take any arguments. We simply call books.json_list() and write the output back to the Request.

Now all you have to do is run the API with the following command:
```
$ python api.py
```
Add a book:
```
http://yourserver:8888/v1/addbook?title="How to Write Python programs for microservice"&author="Rongbing Miao"
```
List all the books:
```
http://192.0.2.98:8888/v1/getbooks
```
Delete the same book:
```
http://192.0.2.98:8888/v1/delbook?title="How to Write Python programs for microservice"
```
There you go. A simple python Rest API example. In the future, I hope to expand on this example by adding a database backend, token security, and more. Subscribe to my newsletter to be notified of all my latest articles.

All of this code is available on [Github](https://github.com/rbmiao/python_restapi).

