---
type: posts
title: Object Oriented Adventure Game, part 1
category: code
---

Project: Object Oriented Adventure Game from free online course:
[MIT 6.001 Structure and Interpretation of Computer Programs](https://ocw.mit.edu/courses/electrical-engineering-and-computer-science/6-001-structure-and-interpretation-of-computer-programs-spring-2005/projects/)

I have been reading _Structure and Interpretation of Computer Programs_ and
have been working through the exercises in the book. The online course from
MIT mentioned above has some great resources to accompany this book. There
are lectures, in the form of presentation slides with notes, 1986 video lectures
from the book's authors: Hal Abelson and Gerald Jay Sussman, and also 5 projects
with the supporting code provided. I chose to do the object oriented adventure
game, project 4 from this course because it looked fun.

I took an object oriented course in college that mostly centered on learning
Java and also a course on "programming paradigms" that included learning C++
and Python, and so I feel comfortable and familiar with the concepts of object
oriented programming. However, reading some of the sections in chapters 2 and 3
of SICP, in which an object oriented programming structure is constructed in
scheme, has taught me a lot about programming paradigms.

Using procedures, with local state, and message passing as the building blocks,
the essence and the utility of programming with an object oriented structure
becomes clear. Essential concepts of object oriented programming are:
message passing, local state, and inheritance.

I would say that the main theme in SICP is how to use programming structures to
manage increasing levels of complexity in programming applications. To handle
the complexity of having multiple representations for one type of data (e.g.
complex numbers), each representation with its own operations (+, -, etc), the
book introduces the use of message passing and data directed programming. The
structure of the program centers around the data. Messages are passed to the
data objects, and objects "respond" by dispatching a method (messages and
responses are metaphors used for mentally organizing the model, implemented by
a simple dispatch function). General operations can handle different types (or
representations) of data by dispatching the specific procedure based on the data
type (denoted by a type tag).

Local state is where we firmly step away from the functional programming
paradigm. Instead of using functions as mathematical functions we can
alternately model objects that have a state of their own, the value or result of
procedures called on these objects depend on the (changing and changeable) state
of the object. Inheritance in an object oriented model helps organize reusable
code. Through the examples in the book, one can see the bare structure that
makes up object oriented programming.

This project, in particular, is an exercise in taking a considerable amount
of code written by someone else and reading, understanding, extending, and
as it turns out: debugging it. I didn't find information on what specific version
of mit-scheme was used for the code provided in the above link. However there
were a couple of bugs possibly due to the code being written for a different
[version of scheme than I am using (9.2)](https://www.gnu.org/software/mit-scheme/documentation/mit-scheme-ref/).
More about this below.

<h3>debugging</h3>

```
1 ]=> (setup 'ana)

;I don't know how to make a handler from #f
;To continue, call RESTART with an option number:
; (RESTART 1) => Return to read-eval-print level 1.

2 error>
```
With the code as given, I tried to start the game only to get this error. To debug
this I ran each line of code in the setup procedure one by one and determined
that it was the populate-spells procedure that caused the error. I created a
place and then tried to create a spell in that place. The spell had its location
set, but the place did not have any spells in it. It seems the spells were
not being installed. The "install" method in the things class uses the "add-thing"
method in the container class.  The "add-thing" method uses the "have-thing?"
method. Checking the "have-thing?" method on the place returned true even though
the "things" list was empty.

```
'HAVE-THING? (lambda (thing)
                (not (null? (memq thing things))))
```

The way this is implemented it expects memq to return an empt list, or null,
when the expected item is not found. However, in [mit scheme 9.2](https://www.gnu.org/software/mit-scheme/documentation/mit-scheme-ref/Searching-Lists.html) memq returns #f when the item is not found, and (null? #f) returns #f, where we
expect #t. So I changed "HAVE-THING?" like this:
```
'HAVE-THING? (lambda (thing)
		            (if (memq thing things)
			              #t
			              #f))
```

Now I was able to complete set up and play the game. After some gameplay
I started getting the same kind of error again. "I don't know how to make a
handler from #f"

```
1 ]=> (ask me 'go 'up)

ana moves from lobby-10 to 10-250
At 10-250 ana says -- Hi grendel
--- the-clock Tick 9 ---
grendel moves from 10-250 to barker-library
;I don't know how to make a handler from #f
```

```
1 ]=> (ask me 'go 'up)

ana moves from 10-250 to barker-library
At barker-library ana says -- Hi grendel
--- the-clock Tick 9 ---
grendel moves from barker-library to 10-250
;I don't know how to make a handler from #f
```

Every call to ask takes the method name and object name and gets the method
through the get-method function which gets the object's handler by calling the
->handler helper function. If the object passed to ->handler is not an object it
returns "I don't know how to make a handler from" x, whatever was actually passed
in.

The error seemed to happen randomly around the grendel or troll character's
actions, and since the EAT-PEOPLE method uses a random hunger value, this should
be a clue to test different possible values of hunger. But I didn't think of
that at first. I ran a number of repeated calls to all the methods of the troll
object to try to get the error to occur. The result was that the error occurred
when the troll had a non zero hunger value but there were no people around to
eat.

So again, a change in implementation ([#f and '() are no longer treated as the same value](https://groups.csail.mit.edu/mac/ftpdir/scheme-7.4/doc-html/scheme_2.html#SEC12))
caused this error.

```
'EAT-PEOPLE
(lambda ()
  (if (= (random hunger) 0)
      (let ((people (ask self 'PEOPLE-AROUND)))
        (if people
            (let ((victim (pick-random people)))
              (ask self 'EMIT
                (list (ask self 'NAME) "takes a bite out of"
                      (ask victim 'NAME)))
              (ask victim 'SUFFER (random-number 3) self)
            'tasty)
            (ask self 'EMIT
              (list (ask self 'NAME) "'s belly rumbles"))))
      'not-hungry-now))
```

When there are no people around, the value of people is the empty list '()
So the "(if people..." statement should be changed to "(if (not (null? people)) ...)"

```
'EAT-PEOPLE
(lambda ()
  (if (= (random hunger) 0)
      (let ((people (ask self 'PEOPLE-AROUND)))
        (if (not (null? people))
            (let ((victim (pick-random people)))
              (ask self 'EMIT
                (list (ask self 'NAME) "takes a bite out of"
                      (ask victim 'NAME)))
              (ask victim 'SUFFER (random-number 3) self)
            'tasty)
            (ask self 'EMIT
              (list (ask self 'NAME) "'s belly rumbles"))))
      'not-hungry-now))
```


----

Here is a class diagram of the classes in the given code:

![class diagram](assets/images/OOAdventure_objects.png)

Here is a diagram of the rooms and exits:

![rooms](assets/images/OOAdventure_rooms.png)

During setup the following characters are created in a randomly chosen
room:
  Students:
    ben-bitdiddle
    alyssa-hacker
    course-6-frosh
    lambda-man
  Hall Monitors:
    dr-evil
    mr-bigglesworth
  Trolls:
    grendel
    registrar
