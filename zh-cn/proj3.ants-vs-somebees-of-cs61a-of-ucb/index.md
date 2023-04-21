# CS61A 的项目三之 Ants vs SomeBees 实现 (2021-Fall)


## Intro

---

前两个项目还算简单, 比较不复杂. 但是今天这个第三个项目难度确实是上升了(看游戏规则就知道这个有多复杂了). 感觉像是植物大战僵尸



所以我打算为他写一篇博客来整理一下写代码时候的思路. 话不多说, 让我们进入正题吧 !

## Phase 1: Basic gameplay

---

### Problem 1 (1 pt)

>   **Part A**: Currently, there is no cost for placing any type of `Ant`, and so there is no challenge to the game. The base class `Ant` has a `food_cost` of zero. Override this class attribute for `HarvesterAnt` and `ThrowerAnt` according to the "Food Cost" column in the table below.
>
>   
>
>   **Part B**: Now that placing an `Ant` costs food, we need to be able to gather more food! To fix this issue, implement the `HarvesterAnt` class. A `HarvesterAnt` is a type of `Ant` that adds one food to the `gamestate.food` total as its `action`.

根据题目的要求设置 `HarversterAnt` 和 `ThrowerAnt` 的属性, 同时实现 `HarvesterAnt` 的 `action` 方法, 让它可以在每次行动的时候给 `food` + 1

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

其实这个赛道有点像数据结构中的双向链表的结构, 往左边用 `.exit`, 往右边用 `.entrance` 方法. 要求已经在上面给出, 没什么难度

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

现在我们要给 `ThrowerAnt` 加上功能, 这样才能让它攻击距离它最近的蜜蜂🐝. **注意如果蜜蜂和他在同一个格子里, 也是可以攻击的**.



我们的工作要求是遍历每个格子(**就跟你遍历链表一样**)找到第一个有蜜蜂的格子, 随机返回一个蜜蜂

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

现在我们要实现两个类, `LongThrower` 和 `ShortThrower`. 两个都是 `ThrowererAnt` 的子类, 其实从他们的名字可以看出来他们的区别在于**攻击范围的不同**.



思路: 我们如何表示攻击范围这个概念呢 ? 其实很简单, 在 problem 3 中我们找到最近的蜜蜂的时候就是一个格子一个格子前进的, 我们可以同时计算步数, 那么我们就这道这个距离, 再配合 `min_range` 和 `max_range` 这两个参数(类变量, 表示这个类对应的蚂蚁的攻击范围, 只有落在这个区间的才行)



同时要注意我们不能影响 problem 3 中的结果, 这也容易想到, 我们让 `min_range=-1`, `max_range=float('inf')`, 这样就相当于没有限制了 ~! 而且因为面向对象程序设计的优势, 我们省去了不少代码量.

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

这个 `FireAnt` 有点意思的, 当他收到伤害的时候会反弹自己受到的伤害到当前格子所有的蜜蜂上, 同时如果它因此死了还能对这些蜜蜂再一次造成伤害(这一次取决于自己的攻击力)



这个比较 tricky 的地方是：当前格子的所有蜜蜂是一个 `list`, 也就是**我们可能要在迭代访问 `list` 的时候修改这个 `list`**, ==这个我们遍历它的拷贝即可==



最后代码如下:

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

从零实现一个 `WallAnt`, 这种蚂蚁生命值高, 其他倒是没什么

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

从零实现一个 `HungryAnt`, 可以随机吞下一整只蜜蜂!!!!但是它要花费 `chew_duration` 来咀嚼才能进行下一次攻击. 这个不就是植物大战僵尸里面的食人花嘛!!!





我们只要判断当前它是否处于咀嚼状态即可. 这里题目是有挖坑的, 测试样例里面可能在运行的时候会修改 `chew_duration` 的值, 注意别被坑了!

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

这里要实现的蚂蚁也很有意思, 它可以把其他蚂蚁保护起来. 甚至可以和被保护的蚂蚁一起呆在同一个格子里面. 



这里注意几个细节:

1.   `BodyguardAnt` 不能保护 `BodyguardAnt` !(🈲️止套娃)
2.   当 `BodyguardAnt` 和被保护的蚂蚁在同一个格子的时候, 要让 `place.ant` 始终指向 `BodyguardAnt`



这里其实还涉及到挺多要改的地方的(可能会漏放某些代码, 完整的建议看我的[仓库](https://github.com/MartinLwx/CS61A-Fall-2021-UCB/tree/main/Projects/ants))

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

根据题目的描述可以知道 `TankAnt` 是一种特殊的 `ContainerAnt`, 我们要从零实现它的功能, 它的攻击方式比较特殊, 是自己保护的蚂蚁的攻击方式 + 对当前格子的蜜蜂造成自己攻击力的伤害



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

为了让地形更有趣, 我们要增加一种地形 - `Water`, 只有能在水里的活动的生物才能被放在这种地形中(蜜蜂会飞当然都可以, 蚂蚁目前还没有)



记得还要修改很多类, 增加类属性 `is_waterproof`, 下面我就放 `Water` 类的代码

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

从零实现一个 `ScubaThrower`, 听名字可以看出来应该是一种特殊的 `ThrowerAnt`. 特殊在: 能放在 `Water` 里

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

终于来到了最后一题(除了额外的题目以外), 我们要实现一个女王蚁🐜. 它有以下几个特性:

1.   可以在水中行走
     -   思路: 题目也说了它**是一种** `ScrubaThrower`, 根据这个描述其实就抽象概括出了它是 `ScrubaThrower` 的子类.
2.   它在行动后会把在它后面的蚂蚁们的攻击力都翻倍, **但是不可以多次翻倍**
     -   思路: 如何表示 “在后面” 这个关系 ? 根据前面的题目我们可以知道. 右边为正方向, 所以 “在后面” 实际上就是在左边, 我们可以通过访问 `Place` 的 `.exit` 方法不断获取到它左边(后面的)的
     -   思路: 如何表示不能多次翻倍 ? 很容易想到, 我们需要通过设置一个标记来表示当前的蚂蚁是否已经攻击力翻倍过, 所以我们直接在 `Ant` 类里加一个实例变量即可
     -   思路: 这里还要注意如何处理 GuardAnt, 因为**实际上它守护的蚂蚁是可能被替换为新的蚂蚁**, 此时我们就要让这个新的被守护的蚂蚁的攻击力翻倍. ==注意细细体会这里代码是怎么写的==
3.   只能有一个女王蚁🐜
     -   思路: 如何做到即使我们多次调用女王蚁🐜的构造函数也不会有多的女王蚁🐜 ? 这个依赖于一个游戏变量叫做 `gamestate`, 我们仍然是通过加标记的方式, 只不过这次我们是在 `GameState` 这个类里加一个 `has_queen` 表示当前是否已经有女王蚁🐜
4.   女王蚁🐜不能被移除
     -   思路: 这个简单, 我们什么都不做就行了



最后代码大概如下(结合上面的解释细细体会~~)

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

实现两种特殊的蚂蚁类, 本身没有伤害, 但是能给蜜蜂加上 debuff.

-   `SlowThrower` 可以让蜜蜂减速, 让他们只能在当前时间为偶数的时候前进否则什么事情也干不了. 这个效果可以维持三个回合, 但是**这个 debuff 可以无限叠加**
-   `ScaryThrower` 会让蜜蜂后退, 注意如果不能再后退的话, 就要保持不动. 该效果**维持两回合**. **但是如果被减速就会继续保持不动**, 这个 debuff 只能上一次

这一题, 真的, 难度完全是上来了. 我调了挺久的 bug 才成功. 下面我来讲一下设计思路:

`SlowThrower`

-   设置 `is_slow` 变量表示当前的蜜蜂是否被减速, 同时设置 `slow_turns` 来记住剩余几回合可以解除这个状态
-   每个回合, 如果当前的蜜蜂被减速了, 它只能看当前的游戏时间是否为偶数, 如果是的话就可以前进, 否则在原地不动, **但不论你动不动, `slow_turns -= 1` 永远都成立

`ScaryThrower`

-   类似 `is_slow` 和 `slow_turns` 设置了 `is_scared` 和 `scared_turns`

-   我们暂时先不考虑当前蜜蜂是否被减速了(这样思考问题会比较简单). 显然, 我们每回合要做的事情是让 `scared_turns -= 1`, 然后**是否为 scared 其实决定着蜜蜂的前进方向**. 有了这个基础之后我们再来思考被减速带情况下又该如何, 显然我们前面这样是有问题的, 题目说了如果被减速会原地保持不动, 但是我们却让 `scared_turns -= 1`, 所以我们需要多加一个判断, 也就是**被减速 + 被 scared 的情况下如果我们没有成功移动, 那么我们需要撤销我们对 `scared_turns` 的更改**



知道了上面的设计思路, 理解下面的代码就不难了(**这里删去了无关的代码**):

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
            else:
                # is_slow + is_scared, we need to cancel `self.scared_turns -= 1`  \
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

忍者蚁🥷🐜哈哈哈哈, 注意几个细节:

1.   无法被蜜蜂攻击
2.   不会堵住蜜蜂的路, 但是会对经过的蜜蜂造成伤害



这个问题比较简单, 解法也几乎都写在了问题描述里面

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

激光🐜, 注意几个特性:

1.   伤害自己格子所在的所有生物, 甚至包括整条路径上的所有生物
2.   但是每次对其他生物造成伤害的时候伤害会衰减, 每次减去 `0.0625`
3.   激光的威力跟它离激光蚁🐜的距离也有关系, 距离每多一个格子, 就会减去 `0.25`

只要处理好两个函数即可

-   `calculate_damage` : 这个要注意的地方是, 如果算出来的伤害 < 0, 那么你就需要返回 0
-   `insects_in_front` : 这个要返回一个 `dict` 表示每个生物距离激光🐜的距离. 我这了是分成当前格子和剩下的格子这样来处理, 一边遍历所有格子一边计算距离和把生物加到我们的 `dict`里.

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


