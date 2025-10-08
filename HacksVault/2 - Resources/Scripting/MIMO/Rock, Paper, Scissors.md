1. Build your own game! We'll set up the stage in this first part by crafting the basic game logic. We'll let the player and the computer choose an option and determine the winner.

```python
import random

print("Let's play rock, papers or scissors")

player_choice = input("Rock, paper or scissors?: ").lower()
choices = ["rock", "paper", "scissors"]
computer_choice = random.choice(choices)

print(f"Computer chose: {computer_choice}")

if (player_choice == "rock" and computer_choice == "scissors") or (player_choice == "scissors" and computer_choice == "paper") or (player_choice == "paper" and computer_choice == "rock"):
  winner = "Player"
elif computer_choice == player_choice:
  winner = "Tie"
else:
  winner = "Computer"

if winner == "Player":
  print("You won")
elif winner == "Computer":
  print("Computer won")
else:
  print("It's a tie")
```

2.  Here we'll extend the project to play a best of three and, with that, make it a real game.

```python
import random

# Wins Tracker
player_wins = 0
computer_wins = 0

# Play
print("Let's play rock, papers or scissors")

while player_wins < 2 and computer_wins < 2:
  player_choice = input("Rock, paper or scissors?: ").lower()
  choices = ["rock", "paper", "scissors"]
  computer_choice = random.choice(choices)
  print(f"Computer chose: {computer_choice}")

	if (player_choice == "rock" and computer_choice == "scissors") or (player_choice == "scissors" and computer_choice == "paper") or (player_choice == "paper" and computer_choice == "rock"):
		winner = "Player"
	elif computer_choice == player_choice:
		winner = "Tie"
	else:
		winner = "Computer"

  if winner == "Player":
    player_wins += 1
    print("You won")
  elif winner == "Computer":
    computer_wins += 1
	print("Computer won")
  else:
	print("It's a tie")
  print(f"Current Score - Player: {player_wins}, Computer: {computer_wins}")

if player_wins > computer_wins:
  print("Congratulations! You won.")
else:
  print("Computer won!")
```