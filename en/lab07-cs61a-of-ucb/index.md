# Lab07 CS61A of UCB(2021-Fall)


## Accounts

---

### Q2: Retirement

>   Add a `time_to_retire` method to the `Account` class. This method takes in an `amount` and returns how many years the holder would need to wait in order for the current `balance` to grow to at least `amount`, assuming that the bank adds `balance` times the `interest` rate to the total balance at the end of every year.

The description tells us that: We will add our `balance` every year, so when may we retire? Both the math and code are simple.

```python
def time_to_retire(self, amount):
    """Return the number of years until balance would grow to amount."""
    assert self.balance > 0 and amount > 0 and self.interest > 0
    year, curAmount = 0, self.balance
    while True:
        year += 1
        curAmount *= (1 + self.interest)
        if curAmount > amount:
            return year
```

### Q3: FreeChecking

>   Implement the `FreeChecking` class, which is like the `Account` class from lecture except that it charges a withdraw fee after 2 withdrawals. If a withdrawal is unsuccessful, it still counts towards the number of free withdrawals remaining, but no fee for the withdrawal will be charged.

We need to check an additional parameter(`free_withdrawals`) compare with `withdraw`, which indicates the number of times we can withdraw without being charged the fee. There are 2 points to note here:

1.   The number of `free_wighdrawals` will be reduced even if we fail to withdraw.
2.   The sum of the fee and the `amount` should be bigger than the `balance` so that we can withdraw successfully when we need to pay the fee.

```python
def withdraw(self, amount):
    if self.free_withdrawals > 0:
        if amount > self.balance:
            self.free_withdrawals -= 1
            return "Insufficient funds"
        if amount > self.max_withdrawal:
            self.free_withdrawals -= 1
            return "Can't withdraw that amount"
        self.free_withdrawals -= 1
        self.balance = self.balance - amount
    else:
        if amount + self.withdraw_fee > self.balance:
            self.free_withdrawals -= 1
            return "Insufficient funds"
        if amount + self.withdraw_fee > self.max_withdrawal:
            self.free_withdrawals -= 1
            return "Can't withdraw that amount"
        self.balance = self.balance - amount - self.withdraw_fee
```

## Magic: the Lambda-ing

---

### Description

We will be implementing a card game in the lab. You can check the rules [here](https://inst.eecs.berkeley.edu/~cs61a/fa21/lab/lab07/#magic-the-lambda-ing)

### Q4: Making Cards

>   To play a card game, we're going to need to have cards, so let's make some! We're gonna implement the basics of the `Card` class first.
>
>   First, implement the `Card` class constructor in `classes.py`. This constructor takes three arguments:
>
>   -   a string as the `name` of the card
>   -   an integer as the `attack` value of the card
>   -   an integer as the `defense` value of the card
>
>   Each `Card` instance should keep track of these values using instance attributes called `name`, `attack`, and `defense`.
>
>   You should also implement the `power` method in `Card`, which takes in another card as an input and calculates the current card's power. Refer to the [Rules of the Game](https://inst.eecs.berkeley.edu/~cs61a/fa21/lab/lab07/#rules-of-the-game) if you'd like a refresher on how power is calculated.

We need to implement the constructor and the `power` functions. 

```python
class Card:
    cardtype = 'Staff'

    def __init__(self, name, attack, defense):
        """
        Create a Card object with a name, attack,
        and defense.
        """
        self.name = name
        self.attack = attack
        self.defense = defense

    def power(self, opponent_card):
        """
        Calculate power as:
        (player card's attack) - (opponent card's defense)/2
        """
        return self.attack - opponent_card.defense / 2
```

### Q5: Making a Player

>   Now that we have cards, we can make a deck, but we still need players to actually use them. We'll now fill in the implementation of the `Player` class.
>
>   A `Player` instance has three instance attributes:
>
>   -   `name` is the player's name. When you play the game, you can enter your name, which will be converted into a string to be passed to the constructor.
>   -   `deck` is an instance of the `Deck` class. You can draw from it using its `.draw()` method.
>   -   `hand` is a list of `Card` instances. Each player should start with 5 cards in their hand, drawn from their `deck`. Each card in the hand can be selected by its index in the list during the game. When a player draws a new card from the deck, it is added to the end of this list.
>
>   Complete the implementation of the constructor for `Player` so that `self.hand` is set to a list of 5 cards drawn from the player's `deck`.
>
>   Next, implement the `draw` and `play` methods in the `Player` class. The `draw` method draws a card from the deck and adds it to the player's hand. The `play` method removes and returns a card from the player's hand at the given index.
>
>   >   Call `deck.draw()` when implementing `Player.__init__` and `Player.draw`. Don't worry about how this function works - leave it all to the abstraction!

The tasks are as follow:

We have 5 cards at the beginning of the game. We need to implement these:

1.   the constructor. We need to draw 5 cards from the `deck`. For the simplicity, we can use list comprehension here.
2.   the `draw` function. Use `deck.draw()` is enough.
3.   the `play` function. We need to play card at the given index. You should distinguish between `.remove()` and `.pop()`.

```python
def __init__(self, deck, name):
    """Initialize a Player object.
    """
    self.deck = deck
    self.name = name
    self.hand = [deck.draw() for i in range(5)]

def draw(self):
    """Draw a card from the player's deck and add it to their hand.
    """
    assert not self.deck.is_empty(), 'Deck is empty!'
    self.hand.append(self.deck.draw())

def play(self, card_index):
    """Remove and return a card from the player's hand at the given index.
    """
    card = self.hand[card_index]
    self.hand.pop(card_index)
    return card
```

## Optional Questions

### Q6: AIs: Defenders

>   Implement the `effect` method for AIs, which reduces the opponent card's attack by the opponent card's defense, and then doubles the opponent card's defense.
>
>   >   *Note:* The opponent card's resulting attack value cannot be negative.

Note that if the attack is less than zero, we need to set it to zero.

```python
def effect(self, opponent_card, player, opponent):
    """
    Reduce the opponent's card's attack by its defense,
    then double its defense.
    """
    opponent_card.attack -= opponent_card.defense
    if opponent_card.attack < 0:
        opponent_card.attack = 0
    opponent_card.defense *= 2
```

### Q7: Tutors: Flummox

>   Implement the `effect` method for TAs, which swaps the attack and defense of the opponent's card.

We should discard our cards **without breaking the abstraction**. So we can use `.play` and `.draw` methods here.

```python
def effect(self, opponent_card, player, opponent):
    """
    Discard the first 3 cards in the opponent's hand and have
    them draw the same number of cards from their deck.
    """
    # discard 3 cards
    for i in range(3):
        opponent.play(i)
    # draw 3 cards
    for i in range(3):
        opponent.draw()
    # You should add your implementation above this.
    print('{} discarded and re-drew 3 cards!'.format(opponent.name))
```

### Q8: TAs: Shift

>   Implement the `effect` method for TAs, which swaps the attack and defense of the opponent's card.

Just swap the attack and the defense

```python
def effect(self, opponent_card, player, opponent):
    """
    Swap the attack and defense of an opponent's card.
    """
    opponent_card.attack, opponent_card.defense = opponent_card.defense, opponent_card.attack
```

### Q9: The Instructor Arrives

>   A new challenger has appeared! Implement the `effect` method for the Instructors, who add the opponent card's attack and defense to all cards in the player's deck and then removes *all* cards in the opponent's deck that have the same attack or defense as the opponent's card.
>
>   >   *Note:* If you mutate a list while iterating through it, you may run into trouble. Try iterating through a copy of the list instead. You can use slicing to make a copy of a list:
>   >
>   >   ```
>   >     >>> original = [1, 2, 3, 4]
>   >     >>> copy = original[:]
>   >     >>> copy
>   >     [1, 2, 3, 4]
>   >     >>> copy is original
>   >     False
>   >   ```

This one is a little more difficult compared with the other ones. The key of this question is how can we mutate the list while we iterate it. You may check [this](https://stackoverflow.com/questions/1207406/how-to-remove-items-from-a-list-while-iterating)

```python
def effect(self, opponent_card, player, opponent):
    """
    Adds the attack and defense of the opponent's card to
    all cards in the player's deck, then removes all cards
    in the opponent's deck that share an attack or defense
    stat with the opponent's card.
    """
    orig_opponent_deck_length = len(opponent.deck.cards)

    # add the attack and defense of the opponent's card ...
    for card in player.deck.cards:
        card.attack += opponent_card.attack
        card.defense += opponent_card.defense

    # remove all cards in the opponent's deck that share ...
    for card in opponent.deck.cards[:]:
        if card.attack == opponent_card.attack and card.defense == opponent_card.defense:
            opponent.deck.cards.remove(card)


    # You should add your implementation above this.
    discarded = orig_opponent_deck_length - len(opponent.deck.cards)
    if discarded:
        print('{} cards were discarded from {}\'s deck!'.format(discarded, opponent.name))
        return
```




