1. In the Draw a Card project you'll create a virtual deck of cards,. Shuffle them and then display them one by one using ASCII art.
```python
import random

# Create deck function
def create_deck():
  suits = ["♥", "♦", "♣", "♠"]
  ranks = ["2", "3", "4", "5", "6", "7", "8", "9", "10", "J", "Q", "K", "A"]

  deck = []
  for i in suits:
    for b in ranks:
    deck.append((i, b))
  return deck

  # Shuffle the deck
  random.shuffle(deck)
  return deck

# Drawing function
def draw_card(deck):
  hand = deck.pop()
  return [hand], deck

# Create deck
deck = create_deck()

# Draw cards
while len(deck) > 0:
  input("Press Enter to draw the next card")
  hand, deck = draw_card(deck)
  print(hand[0])
print('We are out of cards')
```

2. Take control by choosing how many cards to draw, making the card game even more engaging.
```python
import random

# Create deck function
def create_deck():
  suits = ["♥", "♦", "♣", "♠"]
  ranks = ["2", "3", "4", "5", "6", "7", "8", "9", "10", "J", "Q", "K", "A"]

  deck = []
  for i in suits:
    for b in ranks:
    deck.append((i, b))
  return deck

  # Shuffle the deck
  random.shuffle(deck)
  return deck

# Drawing function
def draw_card(deck, num_cards):

  hand = []

  for _ in range(num_cards):

    if deck:

      hand.append(deck.pop())

    else:

      break

  return hand, deck

# Create deck
deck = create_deck()

# Show card function
def show_card(card):
 space = " "
 if len(card[1]) == 2:
   space = ""
   print (f"""
  +-------+
  |{card[1]} {space}|
  | |
  | {card[0]} |
  | |
  |{space} {card[1]}|
  +-------+
  """)
  
# Draw cards
while len(deck) > 0:
  num_cards = int(input("How many cards do you want to draw? "))
  hand, deck = draw_card(deck, num_cards)
  for card in hand:
    show_card(card)
print("We are out of cards")
```