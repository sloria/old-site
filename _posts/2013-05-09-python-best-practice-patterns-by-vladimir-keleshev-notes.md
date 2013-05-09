---
layout: post
title: "Python Best Practice Patterns by Vladimir Keleshev (Notes)"
description: ""
category: programming
tags: [programming, python]
---
{% include JB/setup %}

These are my notes from Vladimir Keleshev's talk entitled "Python Best Practice Patterns", given on May 2, 2013 at the Python Meetup in Denmark. The original video is [here](http://youtu.be/GZNUfkVIHAY) (about 38 minutes long).

## Composed method
    
- Divide program into methods that perform one task
    - Keep *all* operation in a method at the same level of abstraction
- Use many methods only a few lines long

<pre><code class="python"># Bad
class Boiler:
    def safety_check(self):
        # Convert fixed-point floating point
        temperature = self.modbus.read_holding()
        pressure_psi = self.abb_f100.register / F100_FACTOR
        if psi_to_pascal(pressure_psi) > MAX_PRESSURE:
            if temperature > MAX_TEMPERATURE:
                # Shutdown!
                self.pnoz.relay[15] &= MASK_POWER_COIL
                self.pnoz.port.write('$PL,15\0')
                sleep(RELAY_RESPONSE_DELAY)
                # Successful shutdown?
                if self.pnoz.relay[16] & MASK_POWER_OFF:
                    # Play alarm
                    with open(BUZZER_MP3_FILE) as f:
                        play_sound(f.read())
</code></pre>

- Different levels of abstraction: bitmaps, filesystem operations, playing sounds...

<pre><code class="python"># Better
class Boiler:
    def safety_check(self):
        if any([self.temperature > MAX_TEMPERATURE,
                self.pressure > MAX_PRESSURE]):
            if not self.shutdown():
                self.alarm()

    def alarm(self):
        with open(BUZZER_MP3_FILE) as f:
            play_sound(f.read())

    @property
    def pressure(self):
        pressure_psi = abb_f100.reguster / F100_FACTOR
        return psi_to_pascal(pressure_psi)
    ...
</code></pre>

- `safety_check` deals with temp and pressure
- `alarm` deals with files and sound
- `pressure` deals with bits and converting them

## Constructor method

- provide constructors that create well-formed instances
    - Pass all required parameters to them

<pre><code class="python"># Bad
point = Point()
point.x = 12
point.y = 5
</code></pre>

- At initiation `point` is not well-formed

<pre><code class="python"># Better
point = Point(x=12, y=5)
</code></pre>

- Can use class methods to make multiple constructors
    - Example: Using Cartesian or polar coordinates

<pre><code class="python">class Point:
    def __init_(self, x, y):
        self.x, self.y = x, y

    @classmethod
    def polar(class_, r, theta):
        return class_(r * cos(theta),
                        r * sin(theta))

point = Point.polar(r=13, theta-22.6)
</code></pre>

## Method objects
- How do you code a method where many lines of code share many arguments and temporary variables?

<pre><code class="python">def send_task(self, task, job, obligation):
    ...
    processed = ...
    ...
    copied = ...
    ...
    executed = ...
    100 more lines
</code></pre>

- Can't be solved by making many small methods (would use more code)

<pre><code class="python"># Solution
class TaskSender:
    def __init__(self, task, job obligation):
        self.task = task
        self.job = job
        self.obligation = obligation
        self.processed = []
        self.copied = []
        self.executed = []

    def __call__(self):
        self.prepare()
        self.process()
        self.execute()
    ...
</code></pre>

## Execute around method (in Python: Context manager)
- How do you represent pairs of actions that should be taken together?

<pre><code class="python">f = open('file.txt', 'w')
f.write('hi')
f.close()

# Better
with open('file.txt', 'w') as f:
    f.write('hi')   

with pytest.raises(ValueError):
    int('hi')

with SomeProtocol(host, port) as protocol:
    protocol.send(['get', signal])
    result = protocol.receive() 

class SomeProtocol:
    def __init__(self, host, port):
        self.host, self.port = host, port

    def __enter__(self):
        self._client = socket()
        self._client.connect((self.host,
                                self.port))

    def __exit__(self, exception, value, traceback):
        self._client.close()

    def send(self, payload): ...

    def receive(self): ...
</code></pre>

## Debug printing method
- `__str__` for users
    - e.g. `print(point)`
- `__repr__` for debugging/interactive mode

## Method comment
- small methods can be more effective than comments

<pre><code class="python"># Meh
if self.flags &amp; 0b1000:  # Am I visible?
    ...

# Better
...
@property
def is_visible(self):
    return self.flags &amp; 0b1000

if self.is_visible:
    ...
</code></pre>

## Choosing message

<pre><code class="python"># Bad
if type(entry) is Film:
    responsible = entry.producer
else:
    responsible = entry.author
</code></pre>
- Shouldn't use type() or instanceof() in conditional --> smelly

<pre><code class="python"># Better
class Film:
    ...
    @property
    def responsible(self):
        return self.producer

entry.responsible
</code></pre>

## Intention revealing message
- How do you communicate your intent when implementation is simple?
- Use for methods that do the same thing (for readability)

<pre><code class="python"># Meh
class ParagraphEditor:
    ...
    def highlight(self, rectangle):
        self.reverse(rectangle)

# Better
class ParagraphEditor:
    ...
    highlight = reverse  # More readable, more composable
</code></pre>

## Constant method (constant class variable)

<pre><code class="python"># Bad
_DEFAULT_PORT = 1234

class SomeProtocol:
    ...
    def __enter__(self):
        self._client = socket()
        self._client.connect(
            (self.host,
            self.port or _DEFAULT_PORT)
        )
        return self  
</code></pre>

- If you want to subclass SomeProtocol, you would have to overwrite very method!

<pre><code class="python"># Better
class SomeProtocol:
    _default_port = 1234
    ...
    def __enter__(self):
        self._client = socket()
        self._client.connect(
            (self.host,
            self.port or self._default_port))
</code></pre>
- Depends if you are designing to make your class subclassable

## Direct and indirect variable access
- Direct
    - no need for getters and setters

<pre><code class="python">
class Point:
    def __init__(self, x, y):
        self.x, self.y = x, y
</code></pre>

- Sometimes need more flexibility --> use properties

<pre><code class="python">
class Point:
    def __init__(self, x, y):
        self._x, self._y = x, y

    @property
    def x(self):
        return self._x

    @x.setter
    def x(self, value):
        self._x = value
</code></pre>

## Enumeration (iteration) method

<pre><code class="python"># Bad
class Department:
    def __init__(self, *employees):
        self.employees = employees

for employee in department.employees:
    ...
</code></pre>
- Can't change type of collection
    - e.g. can't change employees from a list to a set

<pre><code class="python"># Better
class Department:
    def __init__(self, *employees):
        self._employees = employees

    def __iter__(self):
        return iter(self._employees)

for employee in department:  # More readable, more composable
    ...
</code></pre>

## Temporary variable

<pre><code class="python"># Meh
class Rectangle:
    def bottom_right(self):
        return Point(self.left + self.width,
                    self.top + self.height)

# Better to use temporary variables for readability
class Rectangle:
    ...
    def bottom_right(self):
        right = self.left + self.width
        bottom = self.top + self.height
        return Point(right, bottom)
</code></pre>

Sets
- Can often use sets instead of combination of for loops

<pre><code>Set
---

item in a_set
item not in a_set

a_set &lt;= other
a_set.is_subset(other)

a_set | other
a_set.union(other)

a_set &amp; other
a_set.intersection(other)

a_set - other
a_set.difference(other)
</code></pre>

## Equality method
<pre><code class="python">
obj == obj2
obj1 is obj2

class Book:
    ...
    def __eq__(self, other):
        if not isinstance(other, self.__class__):
            return False
        return (self.author == other.author and 
                self.title == other.title)
</code></pre>
- Probably the only case to check `isinstance()`

## Hashing method
<pre><code class="python">class Book:
    ...
    def __hash__(self):
        return hash(self.author) ^ hash(self.other)
</code></pre>

## Sorted collection
<pre><code class="python">class Book:
    ...
    def __lt__(self):
        return (self.author &lt; self.author and
                self.title &lt; self.title)    
</code></pre>

## Concatenation
<pre><code class="python">class Config:
    def __init__(self, **entries):
        self.entries = entries

    def __add__(self, other):
        entries = (self.entries.items() + 
                    other.entries.items())
        return Config(**entries)
default_config = Config(color=False, port=8080)
config = default_config + Config(color=True)
</code></pre>

## Simple enumeration parameter
- When you can't come up with an iteration param that makes sense, just use `each`

<pre><code class="python"># Awkward
for options_shortcut in self.options_shortcuts:
    options_shortcut.this()
    options_shortcut.that()

# Better
for each in self.options_shortcuts:
    each.this()
    each.that()    
</code></pre>

## Cascades
- Instead of writing methods without return values, make them return self
    - allows cascading of methods

<pre><code class="python"># Instead of this
self.release_water()
self.shutdown()
self.alarm()

class Reactor:
    ...
    def release_water(self):
        self.valve.open()
        return self

self.release_water().shutdown().alarm()    
</code></pre>

## Interesting return value

<pre><code class="python"># Meh
def match(self, items):
    for each in enumerate(items):
        if each.match(self):
            return each
</code></pre>
- Is this supposed to reach the end? Is this a bug?

<pre><code class="python"># Better
def match(self, items):
    for each in enumerate(items):
        if each.match(self):
            reutnr each
    return None
</code></pre>
- Explicit better than implicit
- Include return value if it's interesting (even if it's `None`)

### Further reading
- [Smalltalk Best Practice Patterns](http://www.amazon.com/Smalltalk-Best-Practice-Patterns-ebook/dp/B00BBDLIME/ref=dp_kinw_strp_1)
    - Not for Smalltalk: applicable to Python, Ruby, and many other languages








