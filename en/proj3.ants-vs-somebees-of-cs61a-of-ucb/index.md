# Solution of Proj3.Ants vs SomeBees of CS61A (2021-Fall)


## Intro

---

I have finished the first two projects - [Hog](https://github.com/MartinLwx/CS61A-Fall-2021-UCB/tree/main/Projects/hog) and [Cats](https://github.com/MartinLwx/CS61A-Fall-2021-UCB/tree/main/Projects/cats). The first two projects are relatively simple and uncomplicated. But today, the difficulty of the third project has indeed increased (you can see how complicated this is by looking at the rules of the game). It feels like *Plants vs. Zombies*



So I'm going to write a blog to sort out the ideas when writing code. :hugs:

## Phase 1: Basic gameplay

---

### Problem 1 (1 pt)

>   **Part A**: Currently, there is no cost for placing any type of `Ant`, and so there is no challenge to the game. The base class `Ant` has a `food_cost` of zero. Override this class attribute for `HarvesterAnt` and `ThrowerAnt` according to the "Food Cost" column in the table below.
>
>   
>
>   **Part B**: Now that placing an `Ant` costs food, we need to be able to gather more food! To fix this issue, implement the `HarvesterAnt` class. A `HarvesterAnt` is a type of `Ant` that adds one food to the `gamestate.food` total as its `action`.

Set the properties of `HarversterAnt` and `ThrowerAnt` according to the requirements of the description, and implement the `action` method of `HarvesterAnt`, so that it can give `food` + 1 each time it acts

```python
class HarvesterAnt(Ant):
    """HarvesterAnt produces 1 additional food per turn for the colony."""

    name = 'Harvester'
    implemented = True
    food_cost = 2

    def action(self, gamestate):
        """Produce 1 additional food for the colony.
        
        gamestate -- The GameState, used to access game state information.
        """
        gamestate.food += 1


class ThrowerAnt(Ant):
    """ThrowerAnt throws a leaf each turn at the nearest Bee in its range."""

    name = 'Thrower'
    implemented = True
    damage = 1
    food_cost = 3
```

### Problem 2 (1 pt)

>   In this problem, you'll complete `Place.__init__` by adding code that tracks entrances. Right now, a `Place` keeps track only of its `exit`. We would like a `Place` to keep track of its entrance as well. A `Place` needs to track only one `entrance`. Tracking entrances will be useful when an `Ant` needs to see what `Bee`s are in front of it in the tunnel.
>
>   
>
>   However, simply passing an entrance to a `Place` constructor will be problematic; we would need to have both the exit and the entrance before creating a `Place`! (It's a [chicken or the egg](https://en.wikipedia.org/wiki/Chicken_or_the_egg) problem.) To get around this problem, we will keep track of entrances in the following way instead. `Place.__init__` should use this logic:
>
>   -   A newly created `Place` always starts with its `entrance` as `None`.
>   -   If the `Place` has an `exit`, then the `exit`'s `entrance` is set to that `Place`.

In fact, this colony is a bit like the structure of a doubly linked list in the data structure. Use `.exit` to go to the left and `.entrance` method to go to the right.

```python
class Place:
    """A Place holds insects and has an exit to another Place."""
    is_hive = False

    def __init__(self, name, exit=None):
        """Create a Place with the given NAME and EXIT.

        name -- A string; the name of this Place.
        exit -- The Place reached by exiting this Place (may be None).
        """
        self.name = name
        self.exit = exit
        self.bees = []        # A list of Bees
        self.ant = None       # An Ant
        self.entrance = None  # A Place
        # Phase 1: Add an entrance to the exit
        if exit is not None:
            self.exit.entrance = self
```

### Problem 3 (1 pt)

>   In order for a `ThrowerAnt` to throw a leaf, it must know which bee to hit. The provided implementation of the `nearest_bee` method in the `ThrowerAnt` class only allows them to hit bees in the same `Place`. Your job is to fix it so that a `ThrowerAnt` will `throw_at` the nearest bee in front of it **that is not still in the `Hive`.** This includes bees that are in the same `Place` as a `ThrowerAnt`
>
>   >   *Hint:* All `Place`s have an `is_hive` attribute which is `True` when that place is the `Hive`.
>
>   Change `nearest_bee` so that it returns a random `Bee` from the nearest place that contains bees. Your implementation should follow this logic:
>
>   -   Start from the current `Place` of the `ThrowerAnt`.
>   -   For each place, return a random bee if there is any, and if not, inspect the place in front of it (stored as the current place's `entrance`).
>   -   If there is no bee to attack, return `None`.

Now we have to add a function to `ThrowerAnt`, so that it can attack the closest bee🐝. **Note that if the bee is in the same place as it, it can also attack the 🐝**.



Our job requirement is to traverse each grid (**just like you traverse the linked list**) to find the first place that contains at least a bee, and return a random bee

```python
def nearest_bee(self):
    """Return the nearest Bee in a Place that is not the HIVE, connected to
    the ThrowerAnt's Place by following entrances.

    This method returns None if there is no such Bee (or none in range).
    """
    pos = self.place
    while pos.entrance is not None:
        if not pos.is_hive:
            if len(pos.bees) > 0:
                return random_bee(pos.bees)
        pos = pos.entrance
    return None
```

## Phase 2: Ants!

---

### Problem 4 (2 pt)

>   A `ThrowerAnt` is a powerful threat to the bees, but it has a high food cost. In this problem, you'll implement two subclasses of `ThrowerAnt` that are less costly but have constraints on the distance they can throw:
>
>   -   The `LongThrower` can only `throw_at` a `Bee` that is found after following at least 5 `entrance` transitions. It cannot hit `Bee`s that are in the same `Place` as it or the first 4 `Place`s in front of it. If there are two `Bees`, one too close to the `LongThrower` and the other within its range, the `LongThrower` should only throw at the farther `Bee`, which is within its range, instead of trying to hit the closer `Bee`.
>   -   The `ShortThrower` can only `throw_at` a `Bee` that is found after following at most 3 `entrance` transitions. It cannot throw at any bees further than 3 `Place`s in front of it.
>
>   Neither of these specialized throwers can `throw_at` a `Bee` that is exactly 4 `Place`s away.

Now we have to implement two classes, `LongThrower` and `ShortThrower`. Both of them are subclasses of `ThrowererAnt` . In fact, it can be seen from their names that the differences are the attack range. 



**How do we express the concept of attack range?** In fact, it is very simple. In problem 3, when we find the nearest bee, we move forward one place at a time, and we can calculate the number of steps at the same time, then we will get the distance. Then we may check whether the distance is fall between the `min_range` and `max_range` (class variables, indicating the attack range of the ants corresponding to this class)



Also, note that we cannot affect the results in problem 3. Just need to do some simple modifications. We let `min_range=-1`, `max_range=float('inf')`, which is equivalent to no limit ~! Because of The advantages of OOP, we save a lot of code.

```python
# In problem 3
class ThrowerAnt(Ant):
    """ThrowerAnt throws a leaf each turn at the nearest Bee in its range."""

    name = 'Thrower'
    implemented = True
    damage = 1
    food_cost = 3
    min_range = -1
    max_range = float('inf')

    def nearest_bee(self):
        """Return the nearest Bee in a Place that is not the HIVE, connected to
        the ThrowerAnt's Place by following entrances.

        This method returns None if there is no such Bee (or none in range).
        """
        steps_cnt = 0
        pos = self.place
        while pos.entrance is not None:
            if steps_cnt > self.max_range:
                return None
            if not pos.is_hive:
                if len(pos.bees) > 0 and steps_cnt >= self.min_range:
                    return random_bee(pos.bees)
            pos = pos.entrance
            steps_cnt += 1
        return None
    
class ShortThrower(ThrowerAnt):
    """A ThrowerAnt that only throws leaves at Bees at most 3 places away."""

    name = 'Short'
    food_cost = 2
    # OVERRIDE CLASS ATTRIBUTES HERE
    implemented = True   # Change to True to view in the GUI
    max_range = 3


class LongThrower(ThrowerAnt):
    """A ThrowerAnt that only throws leaves at Bees at least 5 places away."""

    name = 'Long'
    food_cost = 2
    # OVERRIDE CLASS ATTRIBUTES HERE
    implemented = True   # Change to True to view in the GUI
    min_range = 5
```

### Problem 5 (3 pt)

>   Implement the `FireAnt`, which does damage when it receives damage. Specifically, if it is damaged by `amount` health units, it does a damage of `amount` to all bees in its place (this is called *reflected damage*). If it dies, it does an additional amount of damage, as specified by its `damage` attribute, which has a default value of `3` as defined in the `FireAnt` class.
>
>   
>
>   To implement this, override `FireAnt`'s `reduce_health` method. Your overriden method should call the `reduce_health` method inherited from the superclass (`Ant`) to reduce the current `FireAnt` instance's health. Calling the *inherited* `reduce_health` method on a `FireAnt` instance reduces the insect's `health` by the given `amount` and removes the insect from its place if its `health` reaches zero or lower.

When the `FireAnt` receives damage, it will reflect the damage it has received to all bees in the current place, and if it dies because of the bee's attack, it can also deal its damage to these bees again (depending on the damage of the `FireAnt`)


Details: All the bees in the current place are stored as a `list`. As a result,  **we may mutate the `list` while we are iterating it, so we need to traverse its copy(slice)


The final code is as follows:

```python
class FireAnt(Ant):
    """FireAnt cooks any Bee in its Place when it expires."""

    name = 'Fire'
    damage = 3
    food_cost = 5
    implemented = True   # Change to True to view in the GUI

    def __init__(self, health=3):
        """Create an Ant with a HEALTH quantity."""
        super().__init__(health)

    def reduce_health(self, amount):
        """Reduce health by AMOUNT, and remove the FireAnt from its place if it
        has no health remaining.

        Make sure to reduce the health of each bee in the current place, and apply
        the additional damage if the fire ant dies.
        """
        # FireAnt attack bees
        for bee in self.place.bees[:]:
            bee.reduce_health(amount)

        # FireAnt will be dead
        if self.health <= amount:
            for bee in self.place.bees[:]:
                bee.reduce_health(self.damage)
            super().reduce_health(amount)
        else:
            super().reduce_health(amount)
```

## Phase 3: More Ants!

---

### Problem 6 (1 pt)

>   We are going to add some protection to our glorious home base by implementing the `WallAnt`, an ant that does nothing each turn. A `WallAnt` is useful because it has a large `health` value.
>
>   
>
>   Unlike with previous ants, we have not provided you with a class header. Implement the `WallAnt` class from scratch. Give it a class attribute `name` with the value `'Wall'` (so that the graphics work) and a class attribute`implemented` with the value `True` (so that you can use it in a game).

Implement a `WallAnt` from scratch. This one is easy !

```python
class WallAnt(Ant):
    """WallAnt has a large health value"""
    name = 'Wall'
    damage = 0
    food_cost = 4
    implemented = True

    def __init__(self, health=4):
        super().__init__(health)
```

### Problem 7 (3 pt)

>   Implement the `HungryAnt`, which will select a random `Bee` from its `place` and eat it whole. After eating a `Bee`, a `HungryAnt` must spend 3 turns chewing before eating again. If there is no bee available to eat, `HungryAnt` will do nothing.
>
>   
>
>   Give `HungryAnt` a `chew_duration` **class** attribute that stores the number of turns that it will take a `HungryAnt` to chew (set to 3). Also, give each `HungryAnt` an **instance** attribute `chew_countdown` that counts the number of turns it has left to chew (initialized to 0, since it hasn't eaten anything at the beginning. You can also think of `chew_countdown` as the number of turns until a `HungryAnt` can eat another `Bee`).
>
>   
>
>   Implement the `action` method of the `HungryAnt`: First, check if it is chewing; if so, decrement its `chew_countdown`. Otherwise, eat a random `Bee` in its `place` by reducing the `Bee`'s health to 0. Make sure to set the `chew_countdown`when a Bee is eaten!

Implementing a `HungryAnt` from scratch that can swallow a whole bee at random!!!! But it takes `chew_duration` to chew before the next attack. Isn't this the piranha in Plants vs. Zombies!!!



We only need to judge whether it is currently in the chewing state. Note that the value of `chew_duration` may be modified in the runtime.

```python
class HungryAnt(Ant):
    """HungryAnt will select a random bee from its place and eat it whole"""
    name = 'Hungry'
    damage = 0
    food_cost = 4
    implemented = True
    chew_duration = 3

    def __init__(self, health=1):
        super().__init__(health)
        self.chew_countdown = 0

    def action(self, gamestate):
        # it is chewing
        if self.chew_countdown != 0:
            self.chew_countdown -= 1
        # it is not chewing
        else:
            if len(self.place.bees) > 0:
                # WARNING: the test cases may change the chew_duration variable in runtime
                self.chew_countdown = self.chew_duration
                bee = random_bee(self.place.bees)
                bee.reduce_health(bee.health)
```

### Problem 8 (3 pt)

>   A `BodyguardAnt` differs from a normal ant because it is a `ContainerAnt`; it can contain another ant and protect it, all in one `Place`. When a `Bee` stings the ant in a `Place` where one ant contains another, only the container is damaged. The ant inside the container can still perform its original action. If the container perishes, the contained ant still remains in the place (and can then be damaged).
>
>   
>
>   Each `ContainerAnt` has an instance attribute `ant_contained` that stores the ant it contains. This ant, `ant_contained`, initially starts off as `None` to indicate that there is no ant being stored yet. Implement the `store_ant` method so that it sets the `ContainerAnt`'s `ant_contained` instance attribute to the passed in `ant`argument. Also implement the `ContainerAnt`'s `action` method to perform its `ant_contained`'s action if it is currently containing an ant.

The ant to be implemented here is also very interesting, it can protect an ant. It can even stay in the same place with the protected ant inside.



Note a few details here:

1. `BodyguardAnt` cannot protect `BodyguardAnt`!
2. When `BodyguardAnt` and the ant inside are in the same place, make `place.ant` always point to `BodyguardAnt`



There are actually a lot of things to be changed here (some code changes may be missed below, see my [repo](https://github.com/MartinLwx/CS61A-Fall-2021-UCB/tree/main/Projects/ants))

```python
class Ant(Insect):
    """An Ant occupies a place and does work for the colony."""

    implemented = False  # Only implemented Ant classes should be instantiated
    food_cost = 0
    is_container = False
	
    ...
    
    def add_to(self, place):
        if place.ant is None:
            place.ant = self
        else:
            assert (
                (place.ant is None)
                or self.can_contain(place.ant)
                or place.ant.can_contain(self)
            ), 'Two ants in {0}'.format(place)
            if place.ant.is_container and place.ant.can_contain(self):
                place.ant.store_ant(self)
            elif self.is_container and self.can_contain(place.ant):
                self.store_ant(place.ant)
                # the place.ant should refer to the container ant
                place.ant = self

        Insect.add_to(self, place)

        
class ContainerAnt(Ant):
    """
    ContainerAnt can share a space with other ants by containing them.
    """
    is_container = True

    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.ant_contained = None

    def can_contain(self, other):
        # we can't have two BodyguardAnt in the same place
        if self.ant_contained is None and not other.is_container:
            return True

    def store_ant(self, ant):
        self.ant_contained = ant

    def remove_ant(self, ant):
        if self.ant_contained is not ant:
            assert False, "{} does not contain {}".format(self, ant)
        self.ant_contained = None

    def remove_from(self, place):
        # Special handling for container ants (this is optional)
        if place.ant is self:
            # Container was removed. Contained ant should remain in the game
            place.ant = place.ant.ant_contained
            Insect.remove_from(self, place)
        else:
            # default to normal behavior
            Ant.remove_from(self, place)

    def action(self, gamestate):
        if self.ant_contained is not None:
            return self.ant_contained.action(gamestate)


class BodyguardAnt(ContainerAnt):
    """BodyguardAnt provides protection to other Ants."""

    name = 'Bodyguard'
    food_cost = 4
    implemented = True   # Change to True to view in the GUI

    def __init__(self, health=2):
        super().__init__(health)
```

### Problem 9 (1 pt)

>   The `BodyguardAnt` provides great defense, but they say the best defense is a good offense. The `TankAnt` is a container that protects an ant in its place and also deals 1 damage to all bees in its place each turn.
>
>   
>
>   We have not provided you with a class header. Implement the `TankAnt` class from scratch. Give it a class attribute `name` with the value `'Tank'` (so that the graphics work) and a class attribute `implemented` with the value `True` (so that you can use it in a game).
>
>   You should not need to modify any code outside of the `TankAnt` class. If you find yourself needing to make changes elsewhere, look for a way to write your code for the previous question such that it applies not just to `BodyguardAnt` and `TankAnt` objects, but to container ants in general.

According to the description, we can know that `TankAnt` is a special kind of `ContainerAnt`I its attack method is quite special: the attack method of the ant it protects + deals its damage to all the bees in the same place

```python
class TankAnt(ContainerAnt):
    name = 'Tank'
    damage = 1
    food_cost = 6
    implemented = True

    def __init__(self, health=2):
        super().__init__(health)

    def action(self, gamestate):
        if self.ant_contained is not None:
            self.ant_contained.action(gamestate)

        # 1 damage for all the bees
        for bee in self.place.bees[:]:
            bee.reduce_health(self.damage)
```

## Phase 4: Water and Might

---

### Problem 10 (1 pt)

>   Let's add water to the colony! Currently there are only two types of places, the `Hive` and a basic `Place`. To make things more interesting, we're going to create a new type of `Place` called `Water`.
>
>   
>
>   Only an insect that is waterproof can be placed in `Water`. In order to determine whether an `Insect` is waterproof, add a new class attribute to the `Insect` class named `is_waterproof` that is set to `False`. Since bees can fly, set their `is_waterproof` attribute to `True`, overriding the inherited value.
>
>   
>
>   Now, implement the `add_insect` method for `Water`. First, add the insect to the place regardless of whether it is waterproof. Then, if the insect is not waterproof, reduce the insect's health to 0. *Do not repeat code from elsewhere in the program.* Instead, use methods that have already been defined.

In order to make the game more interesting, we will add a new kind of Place - `Water`, only creatures with `is_waterproof = True` can be placed in(Of course, bees can fly, ants can't)



Add the class attribute `is_waterproof` in many classes, I will only put the code of the `Water` class below

```python
class Water(Place):
    """Water is a place that can only hold waterproof insects."""

    def add_insect(self, insect):
        """Add an Insect to this place. If the insect is not waterproof, reduce
        its health to 0."""
        super().add_insect(insect)
        if not insect.is_waterproof:
            insect.reduce_health(insect.health)
```

### Problem 11 (1 pt)

>   Currently there are no ants that can be placed on `Water`. Implement the `ScubaThrower`, which is a subclass of `ThrowerAnt` that is more costly and waterproof, *but otherwise identical to its base class*. A `ScubaThrower` should not lose its health when placed in `Water`.
>
>   
>
>   We have not provided you with a class header. Implement the `ScubaThrower` class from scratch. Give it a class attribute `name` with the value `'Scuba'` (so that the graphics work) and remember to set the class attribute`implemented` with the value `True` (so that you can use it in a game).

Implementing a `ScubaThrower` from scratch, it can be seen from the name that it should be a special `ThrowerAnt`.  It can be placed in `Water` !

```python
class ScubaThrower(ThrowerAnt):
    name = 'Scuba'
    food_cost = 6
    is_waterproof = True
    implemented = True

    def __init__(self, health=1):
        super().__init__(health)
```

### Problem 12 (3 pt)

>   Finally, implement the `QueenAnt`. The queen is a waterproof `ScubaThrower` that inspires her fellow ants through her bravery. In addition to the standard `ScubaThrower` action, the `QueenAnt` doubles the damage of all the ants behind her each time she performs an action. Once an ant's damage has been doubled, it is *not* doubled again for subsequent turns.
>
>   
>
>   However, with great power comes great responsibility. The `QueenAnt` is governed by three special rules:
>
>   1.  If the queen ever has its health reduced to 0, the ants lose. You will need to override `Ant.reduce_health` in `QueenAnt` and call `ants_lose()` in that case in order to signal to the simulator that the game is over. (The ants also still lose if any bee reaches the end of a tunnel.)
>   2.  There can be only one queen. A second queen cannot be constructed. To check if an Ant can be constructed, we use the `Ant.construct()` class method to either construct an Ant if possible, or return `None` if not. You will need to override `Ant.construct` as a class method of `QueenAnt` in order to add this check. To keep track of whether a queen has already been created, you can use an instance variable added to the current `GameState`.
>   3.  The queen cannot be removed. Attempts to remove the queen should have no effect (but should not cause an error). You will need to override `Ant.remove_from` in `QueenAnt` to enforce this condition.

Finally, we came to the last question (except the extra questions), we have to implement a queen ant🐜. It has the following features:

1.   It can be placed in `water`

     -   Thought: The description also said that it is a kind of `ScrubaThrower`

2.   After it moves, it will double the attack power of the ants behind it, **but not multiple times!!!**

     -   Thought: How to express the relationship of "behind"? From the previous topic, we know the colony is a double linked list. The right side is the positive direction, so "behind" is actually corresponding to the left side. We can access the `.exit` method of `Place` to get all the ants behind.

     - Thought: How to indicate that it cannot be doubled multiple times? It is easy to think that we need to set a flag to indicate whether the current ant has doubled its attack power. So we can directly add an instance variable to the `Ant` class
     - Thought: Pay attention to how to deal with the GuardAnt here, because **the ant it contains may be replaced by a new one**. At this time we have to double the attack power of this new ant inside.

3.   There can only be one queen ant🐜

     -   Thought: How to make it possible that even if we call the constructor of queen ants 🐜 many times, there will be exactly only one queen ant🐜? This depends on a class called `GameState`. We can add a `has_queen` variable to the `GameState` class to indicate whether the queen ant🐜 has been created.

4.   Queen Ant 🐜 cannot be removed

     -   That's easy.



The final code is probably as follows:

```python
class QueenAnt(ScubaThrower): 
    """The Queen of the colony. The game is over if a bee enters her place."""

    name = 'Queen'
    food_cost = 7
    implemented = True   # Change to True to view in the GUI

    @classmethod
    def construct(cls, gamestate):
        """
        Returns a new instance of the Ant class if it is possible to construct, or
        returns None otherwise. Remember to call the construct() method of the superclass!
        """
        if cls.food_cost > gamestate.food:
            print('Not enough food remains to place ' + cls.__name__)
            return
        # I add a class variable to indict if we have created a QueenAnt()
        if not gamestate.has_queen:
            gamestate.has_queen = True
            return super().construct(gamestate)
        else:
            return None


    def action(self, gamestate):
        """A queen ant throws a leaf, but also doubles the damage of ants
        in her tunnel.
        """
        super().action(gamestate)
        pos = self.place.exit
        while pos:
            if pos.ant is not None:
                if not pos.ant.is_doubled:
                    pos.ant.is_doubled = True
                    pos.ant.buff()
                if pos.ant.is_container and pos.ant.ant_contained is not None:
                    # the pos.ant.ant_contained may change
                    if not pos.ant.ant_contained.is_doubled:
                        pos.ant.ant_contained.buff()
                        pos.ant.ant_contained.is_doubled = True
            pos = pos.exit

    def reduce_health(self, amount):
        """Reduce health by AMOUNT, and if the QueenAnt has no health
        remaining, signal the end of the game.
        """
        if self.health <= amount:
            ants_lose()

    def remove_from(self, place):
        return None
```

### Extra Credit (2 pt)

>   Implement two final thrower ants that do zero damage, but instead apply a temporary "status" on the `action`method of a `Bee` instance that they `throw_at`. This "status" lasts for a certain number of turns, after which it ceases to take effect.
>
>   We will be implementing two new ants that inherit from `ThrowerAnt`.
>
>   -   `SlowThrower` throws sticky syrup at a bee, slowing it for 3 turns. When a bee is slowed, it can only move on turns when `gamestate.time` is even, and can do nothing otherwise. If a bee is hit by syrup while it is already slowed, it is slowed for an additional 3 turns.
>   -   `ScaryThrower` intimidates a nearby bee, causing it to back away instead of advancing. (If the bee is already right next to the Hive and cannot go back further, it should not move. To check if a bee is next to the Hive, you might find the `is_hive` instance attribute of `Place`s useful). Bees remain scared until they have tried to back away twice. Bees cannot try to back away if they are slowed and `gamestate.time` is odd. *Once a bee has been scared once, it can't be scared ever again.*

Implement two special types of ants, which do no damage themselves, but add debuffs to bees.

- `SlowThrower` can slow down the bees, so that they can only move forward when the current time is even. This effect can last for 3 turns, but **we can slow the bees as many times as we want.
- `ScaryThrower` will make the bee move back(if you can't move back, just keep still). This effect **lasts for 2 turns**. **But if it is slowed, it will continue to stay still**. This kind of  debuff only can last time

This question, really, is completely difficult. I debug the code for a long time before I succeeded. Let me talk about the design ideas:

`SlowThrower`

- Set the `is_slow` variable to indicate whether the current bee is being slowed down, and set another variable called `slow_turns` to remember how many turns left to cancel this state
- Every turn, if the current bee is slowed down, it needs to check if the current gametime is an even number, if it is, it can move forward, otherwise stay in place, **but no matter if you are moving, `slow_turns -= 1 ` forever established**

`ScaryThrower`

- `is_scared` and `scared_turns` are set like `is_slow` and `slow_turns`

- Firstly, we don't consider whether the current bee is slowed down for now (it's easier to think about it this way). Obviously, what we need to do every turn is to let `scared_turns -= 1`, and **the `is_scared` state actually determines the bee's destination**. Now, we will add more complexity to this. It's problematic that we didn't consider whether we were being slowed down. The description says that if it is slowed down while it is in scared state, it will keep still in its place. However, we let `scared_turns -= 1` anyway, so we need to add one more judgment here, that is, in the case of **being decelerated + being scared, if we do not move successfully, then we need to undo our changes to `scared_turns`**



The code is as follows:

```python
class Bee(Insect):
    """A Bee moves from place to place, following exits and stinging ants."""

    name = 'Bee'
    damage = 1
    is_waterproof = True
    # 2 flags
    is_slow = False
    is_scared = False
    # turns remained
    slow_turns = 0
    scared_turns = 0
    # we can't scare a bee twice
    has_been_scared = False

    def action(self, gamestate):
        """A Bee's action stings the Ant that blocks its exit if it is blocked,
        or moves to the exit of its current place otherwise.

        gamestate -- The GameState, used to access game state information.
        """
        if self.is_scared:
            destination = self.place.entrance
            self.scared_turns -= 1
        else:
            destination = self.place.exit

        if self.is_slow:
            self.slow_turns -= 1
            if self.slow_turns == 0:
                self.is_slow = False
            if gamestate.time % 2 == 0 and self.health > 0 and destination is not None:
                self.move_to(destination)
            elif self.is_scared:
                # is_slow + is_scared + gamestate.time is odd, we need to cancel `self.scared_turns -= 1`  \
                # if we didn't move
                self.scared_turns += 1
        else:
            if self.blocked():
                self.sting(self.place.ant)
            elif self.health > 0 and destination is not None:
                self.move_to(destination)

        # we can't put this in side `if self.is_scared`, why?
        # because only when we run if self.is_slow we can know
        # should we cancel it or not
        if self.scared_turns == 0:
            self.is_scared = False

        # Extra credit: Special handling for bee direction

    def slow(self, length):
        """Slow the bee for a further LENGTH turns."""
        self.is_slow = True
        self.slow_turns += length

    def scare(self, length):
        """
        If this Bee has not been scared before, cause it to attempt to
        go backwards LENGTH times.
        """
        # a bee can't be scared twice
        if self.has_been_scared:
            return
        else:
            self.is_scared = True
            self.scared_turns += length
            self.has_been_scared = True
```

## Optional Problems

---

### Optional Problem 1

>   Implement the `NinjaAnt`, which damages all `Bee`s that pass by, but can never be stung.
>
>   
>
>   A `NinjaAnt` does not block the path of a `Bee` that flies by. To implement this behavior, first modify the `Ant` class to include a new class attribute `blocks_path` that is set to `True`, then override the value of `blocks_path` to `False`in the `NinjaAnt` class.
>
>   
>
>   Second, modify the `Bee`'s method `blocked` to return `False` if either there is no `Ant` in the `Bee`'s `place` or if there is an `Ant`, but its `blocks_path` attribute is `False`. Now `Bee`s will just fly past `NinjaAnt`s.
>
>   
>
>   Finally, we want to make the `NinjaAnt` damage all `Bee`s that fly past. Implement the `action` method in `NinjaAnt`to reduce the health of all `Bee`s in the same `place` as the `NinjaAnt` by its `damage` attribute. Similar to the `FireAnt`, you must iterate over a potentially changing list of bees.

Ninja Ant🥷🐜, pay attention to a few details:

1. Cannot be attacked by bees
2. It will not block the bees, but will cause harm to the passing bees



This problem is relatively simple, and the solutions are indicated by the description.

```python
class Bee(Insect):
    """A Bee moves from place to place, following exits and stinging ants."""
    
    def blocked(self):
        """Return True if this Bee cannot advance to the next Place."""
        if self.place.ant is None: 
            return False
        if not self.place.ant.blocks_path:
            return False
        return True


class NinjaAnt(Ant):
    """NinjaAnt does not block the path and damages all bees in its place.
    This class is optional.
    """

    name = 'Ninja'
    damage = 1
    food_cost = 5
    blocks_path = False
    implemented = True   # Change to True to view in the GUI

    def action(self, gamestate):
        for bee in self.place.bees[:]:
            bee.reduce_health(self.damage)
```

### Optional Problem 2

>   The `LaserAnt` shoots out a powerful laser, damaging all that dare to stand in its path. Both `Bee`s and `Ant`s, of all types, are at risk of being damaged by `LaserAnt`. When a `LaserAnt` takes its action, it will damage all `Insect`s in its place (excluding itself, but including its container if it has one) and the `Place`s in front of it, excluding the `Hive`.
>
>   
>
>   If that were it, `LaserAnt` would be too powerful for us to contain. The `LaserAnt` has a base damage of `2`. But, `LaserAnt`'s laser comes with some quirks. The laser is weakened by `0.25` each place it travels away from`LaserAnt`'s place. Additionally, `LaserAnt` has limited battery. Each time `LaserAnt` actually damages an `Insect` its laser's total damage goes down by `0.0625` (1/16). If `LaserAnt`'s damage becomes negative due to these restrictions, it simply does 0 damage instead.

Laser 🐜, pay attention to several features:

1. Damage all creatures in your own place, including all creatures in the entire colony
2. But each time it deals damage to other creatures, the damage will decrease, minus `0.0625` each time
3. The power of the laser is also related to its distance from the laser ants🐜. For each additional place, the distance will be subtracted by `0.25`



Just handle two functions

- `calculate_damage` : Note that if the calculated damage is < 0, then you need to return 0 instead.
- `insects_in_front` : This returns a `dict` indicating the distance of each creature from the laser 🐜. I divided it into the current place and the remaining places to process, and I calculate the distance while traversing all places.

```python
class LaserAnt(ThrowerAnt):
    name = 'Laser'
    food_cost = 10
    implemented = True   # Change to True to view in the GUI
    damage = 2


    def __init__(self, health=1):
        super().__init__(health)
        self.insects_shot = 0
        self.current_damage = LaserAnt.damage

    def insects_in_front(self):
        """Return a dict contains every Insect"""
        dis = {}
        
        for bee in self.place.bees:
            dis[bee] = 0
        # take care of the ContainerAnt
        if self.place.ant is not self:
            dis[self.place.ant] = 0
        pos = self.place.entrance
        distance = 1
        while pos.entrance is not None:
            if not pos.is_hive:
                for bee in pos.bees:
                    dis[bee] = distance
                if pos.ant is not None:
                    dis[pos.ant] = distance
                    # take care of the ContainerAnt
                    if pos.ant.is_container and pos.ant.ant_contained is not None:
                        dis[pos.ant.ant_contained] = distance
            distance += 1
            pos = pos.entrance
        return dis

    def calculate_damage(self, distance):
        damage_result = self.damage - 0.0625 * self.insects_shot - 0.25 * distance
        return damage_result if damage_result > 0 else 0

    def action(self, gamestate):
        insects_and_distances = self.insects_in_front()
        for insect, distance in insects_and_distances.items():
            damage = self.calculate_damage(distance)
            insect.reduce_health(damage)
            if damage:
                self.insects_shot += 1
```


