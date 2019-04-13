## Tornado Web API Example

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
