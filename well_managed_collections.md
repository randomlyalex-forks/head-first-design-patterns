# Well-Managed Collections

There are lots of ways to stuff objects into a collection. Put them in an Array, a List, a Hashtable, take your pick. Each has its own advantages and tradeoffs.
How you can allow your clients to iterate through your objects without ever getting a peek at how store your objects? How to create some super collections of objects that can leap over some impressive data structures in a single bound?

## Objectville Diner and Pancake House Merge

The Diner menu has lots of lunch items, while the Pancake House consists of breakfast items. Every menu item has a name, a description, and a price.

Both have lots of time and code invested in the way they store their menu items in a menu, and lots of other code that depends on it.

On the Pancake House they use an ArrayList that makes the menu easily to expand

On the Diner they use an Array so they control the maximum size of their menu and get their MenuItems without having to use a cast

Having two different menu representations complicates things.

We're going to have the job of implementing a common Waitress that is going to be hard to maintain and extend.
It would really be nice if we could find a way to allow them to implement the **same interface** for their menus.

We can encapsulate the iteration caused by different collections of objects being returned from the menus:

```java
// 1) to iterate through the breakfast items we use the size() and get() methods on the ArrayList

for (int i = 0; i < breakfastItems.size(); i++) {
    MenuItem menuItem = (MenuItem)breakfastItems.get(i);
}

// 2) and to iterate through the lunch items we use the Array length field and the array subscript notation on the MenuItem Array

for (int i = 0; i < lunchItems.length; i++) {
    MenuItem menuItem = lunchItems[i];
}

// 3) what if we create an object, let's call it an Iterator that encapsulates the way we iterate through a collection of objects?

// try on the ArrayList
Iterator iterator = breakfastMenu.createIterator();

while (iterator.hasNext()) {
    MenuItem menuItem = (MenuItem)iterator.next();
}

// try on the Array
Iterator iterator = lunchMenu.createIterator();

while (iterator.hasNext()) {
    // this code is exactly the same
    MenuItem menuItem = (MenuItem)iterator.next();
}
```

### The Iterator Pattern

This pattern relies on an interface called Iterator

```java
// the interface class diagram
interface Iterator {
    // tells us if there are more elements in the aggregate to iterate through
    hasNext()
    // returns the next object in the aggregate
    next()
}
```

We can implement Iterators for any kind of collection of objects: arrays, lists, hashtables,...

- Implementation Code [Diner and Pancake House Iterator](11_iterator_pattern/DinerAndPancakeHouseIterator)

Once we gave a PancakeHouseMenuIterator and a DinerMenuIterator, all they had to do was add a getIterator() method and they were finished.
The Waitress will be much easier to maintain and extend down the road.

```java
// the current design

// we're now using a common Iterator interface and we've implemented two concrete classes
interface Iterator {
    hasNext()
    next()
}

// the Iterator allows the Waitress to be decoupled from the actual implementation of the concrete classes
class Waitress {
    printMenu()
}

// these two menus implement the same exact set of methods, but they aren't implementing the same Interface. We're going to fix this and free the Waitress from any dependencies on concrete Menus
class PancakeHouseMenu {
    menuItems
    createIterator()
}

class DinerMenu {
    menuItems
    createIterator()
}

// implement the new createIterator() method; they are responsible for creating the iterator for their respective menu items implementations
class PancakeHouseMenuIterator implements Iterator {
    hasNext()
    next()
}

class DinerMenuIterator implements Iterator {
    hasNext()
    next()
}
```

- the iterator give us a way to step through the elements of an aggregate without forcing the aggregate to clutter its own interface with a bunch of methods to support traversal of its elements
- it also allows the implementation of the iterator to live outside of the aggregate (we've encapsulated the interaction)

### Cleaning with java.util.Iterator

#### PancakeHouseMenu class

We just:

- delete the PancakeHouseMenuIterator class
- add an import java.util.Iterator to the top of PancakeHouseMenu
- change one line of the PancakeHouseMenu

```java
public Iterator createIterator() {
    // instead of creating our own iterator now
    // we just call the iterator() method on the menuItems ArrayList
    return menuItems.iterator();
}
```

#### DinerMenuIterator

We change the DinerMenuIterator class:

- add an import java.util.Iterator to the top
- add the remove() method

```java
import java.util.Iterator;

public class DinerMenuIterator implements Iterator {
    // none of our current implementation changes

    // we do need to implement remove()
    public void remove() {
        if (position <= 0) {
            throw new IllegalStateException ("You can't remove an item you've done at least one next()");
        }
        // we just shift all the elements up one when remove() is called
        if (list[position-1] != null) {
            for (int i = position-1; i < (list.length-1); i++) {
                list[i] = list[i+1];
            }
            list[list.length-1] = null;
        }
    }
}
```

#### Waitress

We need to give the Menus a common interface and rework the Waitress.

```java
public interface Menu {
    // lets clients get an iterator for the items in the menu
    public Iterator createIterator();
}
```

We need to add an *implements Menu* to both the PancakeHouseMenu and the DinerMenu class definitions.

```java
import java.util.Iterator;

public class Waitress {
    Menu pancakeHouseMenu;
    Menu dinerMenu;

    // we need to replace the concrete Menu classes with the Menu Interface
    public Waitress(Menu pancakeHouseMenu, Menu dinerMenu) {
        this.pancakeHouseMenu = pancakeHouseMenu;
        this.dinerMenu = dinerMenu;
    }

    // nothing changes here
    public void printMenu() {}
}
```

- Definition of the Iterator Pattern

The effect of using iterators in your design is just as important:

1. once you have a uniform way of accessing the elements of all your aggregate objects, you can write polymorphic code that works with any of these aggregates
2. the Iterator Pattern takes the responsibility of traversing elements and gives that responsibility to the iterator object, not the aggregate object.

```java
// class diagram

// having a common interface is handy for your client
// it decouples your client from the implementation of your collection of objects
interface Aggregate {
    createIterator()
}

// has a collection of objects
// each ConcreteAggregate is responsible for instantiating a ConcreteIterator that can iterate over its collection of objects
class ConcreteAggregate implements Aggregate {
    createIterator()
}

// the interface that all iterators must implement
interface Iterator {
    hasNext()
    next()
    remove()
}

// is responsible for managing the current position of the iteration
class ConcreteIterator implements Iterator {
    hasNext()
    next()
    remove()
}

// interact with the interfaces
class Client {}
```

## The Responsibility Principle

Every responsibility of a class is an area of potential change.
This principle guides us to keep each class to a single responsibility.

- Definition of the Single Responsibility Principle

Separating responsibility in design is one of the most difficult things to do.

- Implementation Code [Cafe](11_iterator_pattern/Cafe)

## Iterators and Collections

Java gives you a lot of collection classes that allow you to store and retrieve groups of objects (es. Vector and LinkedList).
Most have different interfaces.
But almost all of them support a way to obtain an iterator.
And if they don't support Iterator, that's ok, because now you know how to build your own.

Each of these classes implements the *java.util.Collection interface*, which contains a bunch of useful methods for manipulating groups of objects:

- add(), addAll(), clear(), remove(), removeAll() -> add and remove elements from your collection without even knowing how it's implemented
- iterator() get an Iterator for any class that implements the Collection interface
- size() to get the number of elements
- toArray() to turn your collection into an array

Java 5 includes a new form of the **for** statement, called for/in, that lets you iterate over a collection or an array without create explicitly an iterator.

```java
for (Object obj: collection) {
    // ...
}
```

The Waitress implementation needs one call to printMenu() for each menu.
This is violation of the Open Closed Principle.
We need a way to manage the menu objects together.

```java
// solution: package the menus up into an ArrayList and then get its iterator to iterate through each Menu
public class Waitress {
    ArrayList menus;

    // now we just take an ArrayList of menus
    public Waitress(ArrayList menus) {
        this.menus = menus;
    }

    public void printMenu() {
        Iterator menuIterator = menus.iterator();
        // we iterate through the menus
        // passing each menu's iterator to the overloaded printMenu() method
        while (menuIterator.hasNext()) {
            Menu menu = (Menu)menuIterator.next();
            printMenu(menu.createIterator());
        }
    }

    public void printMenu(Iterator iterator) {
        // no code changes here
    }
}
// this looks pretty good, although we've lost the names of the menus
```

## Add a sub-menu

We can't assign a sub-menu to a MenuItem array.
Time for a change.

We need out of our new design:

- some kind of a three shaped structure that will accomodate menus, sub-menus and menu items
- to make sure we maintain a way to traverse the items in each menu that is at least as convenient as what we are doing now with iterators
- to be able traverse the items in a more flexible manner

## The Composite Pattern

- Definition of the Composite Pattern

### A tree structure

Elements with child elements are called nodes.
Elements without children are called leaves.

This pattern gives us a way to create a tree structure that can handle a nested group of menus and menu items in the same structure.
The Menus are nodes and MenuItems are leaves.
On this hierarchy any menu is a *composition* because it can contain both other menus and menu items. The individual objects are just the menu items.
Using a design that follows the Composite Pattern is going to allow us to write some simple code that can apply the same operation over the entire menu structure.

### The Composite Pattern class diagram

```java
// defines an interface for all objects in the composition
class Component {
    // may implement a default behavior for add(), remove(), getChild()
    // and its operations
    operation()
    add(Component)
    remove(Component)
    getChild(int)
}

// defines the behavior for the elements in the composition
// by implementing the operations the Composite supports
class Leaf extends Component {
    // also inherits methods like add(), remove() and getChild()
    // which don't necessarily make a lot of sense for a leaf node
    operation()

    // a leaf has no children
}

// defines behavior of the components having children and stores child components
class Composite extends Component {
    // also implements the leaf-related operations
    // some of these may not make sense on a Composite
    operation()
    add(Component)
    remove(Component)
    getChild(int)
}

// uses the Component interface to manipulate the objects in the composition
class Client {}
```

### Menu with Composite class diagram

We need to create a component interface; this acts as the common interface for both menus and menu items and allows us to treat them uniformly.

```java
// represents the interface for both MenuItem and Menu
// we've used an abstract class here because we want to provide
// default implementations for these methods
abstract class MenuComponent {
    // we have some of the same methods from our previous versions of MenuItem and Menu
    getName()
    getDescription()
    getPrice()
    isVegetarian()
    // we have added print(), add(), remove() and getChild()
    print()
    // methods for manipulating the components
    add(Component)
    remove(Component)
    getChild(int)
}

// uses the MenuComponent interface to access both Menus and MenuItems
class Waitress {}

class MenuItem extends MenuComponent {
    // overrides the methods that make sense
    getName()
    getDescription()
    getPrice()
    isVegetarian()
    // overrides print
    print()
}

class Menu extends MenuComponent {
    // overrides the methods that make sense
    getName()
    getDescription()
    print()
    add(Component)
    remove(Component)
    getChild(int)
}
```

- Implementation Code [Menu Composite](11_iterator_pattern/MenuComposite)
- Implementation Code [Menu Composite with Iterator](11_iterator_pattern/MenuCompositeWithIterator)
