
1. Dive into OOP by crafting book objects. This first part of the project Introduces you to creating and managing your own digital bookshelf. 

```python
class Book:
  def __init__(self, title, author):
	self.title = title
	self.author = author
	self.available = True

  def checkout(self):
	if self.available:
	  self.available = False
	  return True
	else:
	  return False

  def return_book(self):
	self.available = True

  def display_info(self):
	print(f"Title: {self.title}\nAuthor: {self.author}\nAvailable: {'Yes' if self.available else 'No'}")

  
book1 = Book("Dracula", "Bram Stoker")
book2 = Book("1984", "George Orwell")
book3 = Book("Frankenstein", "Mary Shelley")
books = [book1, book2, book3]
for book in books:
  book.display_info()
```

2.  Expand your project into a fully-fledged library system that organizes and holds all your books.

```python
class Book:
  def __init__(self, title, author):
	self.title = title
	self.author = author
	self.available = True

  def checkout(self):
	if self.available:
	  self.available = False
	  return True
	else:
	  return False

  def return_book(self):
    self.available = True

  def display_info(self):
    print(f"Title: {self.title}\nAuthor: {self.author}\nAvailable: {'Yes' if self.available else 'No'}")

book1 = Book("Dracula", "Bram Stoker")
book2 = Book("1984", "George Orwell")
book3 = Book("Frankenstein", "Mary Shelley")

class Library:
  def __init__(self):
	self.books = []

  def add_book(self, book):
    self.books.append(book)

  def display_books(self):
    for book in self.books:
	  book.display_info()

  def get_book_by_title(self, title):
    for book in self.books:
	  if book.title.lower() == title.lower():
	    return book
    return None

library = Library()

library.add_book(book1)
library.add_book(book2)
library.add_book(book3)

library.display_books()
```