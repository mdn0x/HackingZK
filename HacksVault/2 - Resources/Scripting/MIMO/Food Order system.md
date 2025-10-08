1.  Embark on a culinary journey: an italian food ordering system. Dive into the delicious world of pizzas and pasta as you build a delivery service.

```python
italian_food = ["Pasta Bolognese","Pepperoni pizza", "Margherita pizza", "Lasagna"]

def find_meal(name, menu):
  return name if name in menu else None

def select_meal(name):
  return find_meal(name, italian_food)

def display_available_meals():
  print(f"Available Italian Meals:")

for meal in italian_food:
  print(meal)

def create_summary(name, amount):
  order = select_meal(name)
  if order:
    return f"You ordered {amount} {name}"
  else:
    return "Meal not found"

print("Welcome to the Food Order System")
display_available_meals()
name_input = input("Choose a Meal: ")
amount_input = input("Choose the amount: ")
result = create_summary(name_input, amount_input)
print(result)
```

2. Expand your food order system to include a world of flavors! In this part, we add diverse cuisines, making your system a global food destination.

```python
italian_food = ["Pasta Bolognese","Pepperoni pizza", "Margherita pizza", "Lasagna"]
indian_food = ["Curry","Chutney", "Samosa", "Naan"]
  
def find_meal(name, menu):
  return name if name in menu else None

def select_meal(name, food_type):
  if food_type == "Italian":
	return find_meal(name, italian_food)
  elif food_type == "Indian":
	return find_meal(name, indian_food)
  else:
	return None

def display_available_meals(food_type):
  if food_type == "Italian":
	print(f"Available Italian Meals:")
	for meal in italian_food:
		print(meal)
  elif food_type == "Indian":
	print(f"Available Indian Meals:")
	  for meal in indian_food:
		print(meal)
  else:
	print("Invalid food type")

def create_summary(name, amount, food_type):
  order = select_meal(name, type_input)
  if order:
	return f"You ordered {amount} {name}"
  else:
	return "Meal not found"

print("Welcome to the Food Order System")
type_input = input("Choose the type of food: ")
display_available_meals(type_input)
name_input = input("Choose a Meal: ")
amount_input = input("Choose the amount: ")
result = create_summary(name_input, amount_input, type_input)
print(result)
```

