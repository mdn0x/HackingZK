1. Start building your `todo` list app. This first step focuses on crafting the basic structure and adding tasks. 

```python
# Create the list
todo_list = ["Task1", "Task2"]

while True:
  
  # Create new tasks
  new_task = input("Enter the task: ")
  todo_list.append(new_task)
  print(f"{new_task} added")
  
  # Showing
  if len(todo_list) == 0:
	print("Your ToDo list is empty")
  else:
	index = 1
	for task in todo_list:
	  print(f"{index}. {task}")
	  index += 1
```

2. Add an interactive menu to your `todo` list and enhance it by allowing deleting tasks again.

```python
# Create the list
todo_list = []

while True:
  # Showing
  if len(todo_list) == 0:
	print("Your ToDo list is empty")
  else:
  index = 1
  for task in todo_list:
	print(f"{index}. {task}")
	index += 1

  # Options
  print("Options:")
  print("1) Add Task")
  print("2) Remove Task")
  print("3) Quit")
  choice = input("Choose a number")
  if choice == "1":
	print("Adding task")
	new_task = input("Enter the new task: ")
	todo_list.append(new_task)
	print("Task added successfully")
  
  elif choice == "2":
	print("Removing task")
	if len(todo_list) == 0:
	  print("Can't remove from empty list")
	elif len(todo_list) > 0:
	  todo_list.pop()
	print("Last task removed successfully")
  
  elif choice == "3":
	print("Quitting")
	break
  else:
	print("Please enter a number from 1 to 3")
```