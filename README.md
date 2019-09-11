# Notes-from-Design-Patterns-Elements-of-Reusable-Object-Oriented-Software

# Index

1. [Intro](#intro)
2. [Case Study: Designing a Document Editor](#document-editor)

# <a name="intro">1. Intro </a>

### Inheritance vs Composition
- **Inheritance:** define implementation of one class in terms of another
- It achieves reusibility, and its reffered to **white-box reuse**, which refers to visibility; the internal parent class are visible to its subclass
- Disadvantages: Can break encapsulation 
- **Composition:** new functionality is obtained by composing objects, and requires objects to have well-defined interfaces. This style of reuse is called ** black-box reuse**, meaning no internal details of objects are visible

### Delegation
- In delegation, two objects are involved in handling a request: a receiving object delegates operations to its delegate
- For example, instead of making class Window a subclass of Rectangle (because windows happen to be rectangular), the Window classmight reuse the behavior of Rectangle by keeping a Rectangle instance variable and delegating Rectangle-specific behavior to it
- So, instead of a Window **being** a Rectangle, it would **have** a Rectangle
- The main advantage of delegation is that it makes it easy to compose behaviors at run-time and to change the way they're composed

### Common cause for system redisgn:
- Creating an object by specifying a class explicitly. To avoid it, create objects indirectly
- Dependence on specific operations
- Dependence on hardware and software platform
- Algorithmic dependency

### How to Select a Design Pattern
- Consider howdesign patterns solve design problems: help you find appropriate objects and interfaces 
- intent

![design-aspects](/img/design-aspects.PNG)

# <a name="document-editor">2. Case Study: Designing a Document Editor</a>

### Recursive Composition
- A common way to represent hierarchically structured information is through a technique called recursive composition, which entails building increasingly complex elements out of simpler ones
- we can tile a set of characters and graphics from left to right to form a line in the document. Then multiple lines can be arranged to form a column, multiple columns can form a page, and so on

![document-graph](/img/document-graph.PNG)
- We can represent this physical structure by devoting an object to each important element. That includes not just the visible elements like the characters and graphics but the invisible, structural elements as well—the lines and the column

### Bridge Pattern

![bridge](/img/bridge.PNG)
- Windowlmp class hierarchy in which to hide different window system implementations. Windowlmp is an abstract class for objects that encapsulate window system-dependent code
- By hiding the implementations in Windowlmp classes, we avoid polluting the Window classes with window system dependencies, which keeps the Window class hierarchy comparatively small and stable
- The intent behind Bridge isto allow separate class hierarchies to work together even as they evolve independently
- Separating windowing functionality into Window and Windowlmp hierarchies lets us implement and specialize these interfaces independently
- Benifits: Can develop two different interfaces for two different purposes
- Benifits: work with various database servers
### User Operations
- we don't want to associate a particular user operation with a particular user interface, because we may want multiple user interfaces to the same operation
- We could define a subclass of Menultem for every user operation and then hard-code each subclass to carry out the request. But that's not really right
- we should parameterize Menultems with an object, not a function

### Command class

![command-class](/img/command-class.PNG)
- Menultem can store a Command object that encapsulates a request
- To the requester, a Command object is a Command object—they are treated uniformly
- When a user chooses a particular menu item, the Menultem simply calls Execute on its Command object to carry out the request, as shown below

![menu-comman](/img/menu-comman.PNG)

### Undo/redo
- To undo and redo commands, we add an **Unexecute** operation to Command's interface

### Spelling Checking and Hyphenation
- The last design problem involves textualanalysis, specifically checking for misspellings and introducing hyphenation points where needed for good formatting
- there's more than one way to check spelling and compute hyphenation points. So, here too we want to support multiple algorithms. A diverse set of algorithms can provide a choice of space/time/quality trade-offs
- There are two pieces to this puzzle: (1) accessing the information to be analyzed, which we have scattered over the glyphs in the document structure, and (2)doing the analysis

### Accessing Scattered Information
- Tests will be scattered all over the pages, so we need an access mechanism that has knowledge about the data structures in which objects are stored
- access mechanism need to indetify objects that are: linked list, arrays etc
- our access mechanism must accommodate differing data structures, and we must support different kinds of traversals, such as preorder, postorder, and inorder
- We may add this operation:
```java
void First(Traversal kind)
void Next()
bool IsDone()
Glyph* GetCurrent()
void Insert(Glyph*)
```

- Since we need to be flexible with different data structures for traversal, we could implement a `Iterator` class so that we don't need to modify the Glyph class:
![Iterator](/img/Iterator.PNG)

- We'll use an abstract class called Iterator to define a general interface for access and traversal
- Concrete subclasses like Arraylterator and Listlterator implement the interface to provide access to arrays and lists, while Preorderlterator, Postorderlterator, and the like implement different traversals on specific structures

### Traversal versus Traversal Actions
- Now that we have away oftraversing the glyph structure, we need to checkthe spelling and do the hyphenation
- Both analyses involve accumulating information during the traversal
- We could add the spell-checkign into the taversal functionaloity, But we get more flexibility and potential for reuse ifwe distinguish between the traversal and the actions performed during traversal

### Encapsulating the Analysis
- We don't want a SpellingChecker class to include (pseudo)code like:
```java
void SpellingChecker::Check (Glyph* glyph) {
Character* c;
Row* r;
Image* i;
if (c = dynamic_cast<Character*>(glyph)) {
// analyze the character
  } else if (r = dynamic_cast<Row*>(glyph)) {
// prepare to analyze r's children
 } else if (i = dynamic_cast<Image*>(glyph)) {
// do nothing
 }
}
```

- This code is pretty ugly. It relies on fairly esoteric capabilities like type-safe casts. It's hard to extend as well
- We'll have to remember to change the body of this function whenever we change the Glyph class hierarchy. In fact, this is the kind of code that object-oriented languages were intended to eliminate
- The right solution is to add an abstract class, like this:

```java
class SpellingChecker {
public:
SpellingChecker();
virtual void CheckCharacter(Character*);
virtual void CheckRow(Row*);
virtual void Checklmage(Image*);
// ... and so forth
List<char*>& GetMisspellings();
protected:
virtual bool IsMisspelled(const char*);
private:
char _currentWord[MAX_WORD_SIZE];
List<char*> _misspellings;
};
```
- `SpellingChecker` checking operation for Character glyphs might look something like this:
```java
void SpellingChecker::CheckCharacter (Character* c) {
const char ch = c->GetCharCode();
if (isalpha(ch)) {
// append alphabetic character to _currentWord
} else {
//we hit a nonalphabetic character
if (IsMisspelled(_currentWord)) {
// add _currentWord to _misspellings
_misspellings.Append(strdup(_currentWord));
}
_currentWord[0] = '\0 ' ;
// reset _currentWord to check next word
}
}
```
- `CheckCharacter` accumulates alphabetic characters into the `_currentWord` buffer
- When it encounters a nonalphabetic character, such as an underscore, it uses the `IsMisspelled` operation to check the spelling of the word
- Then it must clear out the `_currentWord` buffer to ready it for the next word
- When the traversal is over, you can retrieve the list of misspelled words with the `GetMisspellings` operation

## Design Pattern Catalog: Creational - Structural - Behavioral Patterns

## Creational Pattern: Abstract Factory - Factory hod - Prototype - Singleton
- Used to create different types: Maze game, create different mazes 
- Singleton can ensure there's only one maze per game and that all game objectshave ready access to it—without resorting to global variables or functions
- Singleton also makes it easy to extend or replace the maze without touching existing code

## ABSTRACT FACTORY
- clients depend only upon interfaces rather than the concrete classes required to instantiate objects
- Clients call these operations to obtain widget instances, but clients aren't aware of the concrete classes they're using. Thus clients stay independent of the prevailing look and feel, clients only have to commit to an interface defined by an abstract class,not a particular concrete class
- Use the Abstract Factorypattern when:
  * a system should be independent of how its products are created, composed, and represented
  * a system should be configured with one of multiple families of products
  * a family of related product objects is designed to be used together, and you need to enforce this constraint
  * you want to provide a class library of products, and you want to reveal just their interfaces, not their implementations
- Consequences:
1) It isolates concrete classes: encapsulation
2) It makes exchanging product families easy
3) Supporting new kinds of products is difficult: adding new concrete classes may require additional changes in the interface

## Factory method

![factory-method](/img/factory-method.jpg)

![creator-product](/img/creator-product.jpg)
- Define an interface for creating an object, but let subclasses decide which class to instantiate. Factory Method lets a class defer instantiation to subclasses
- To use me, just subclass me and implement my factory method
- Factories handle the details of object creation
- This pattern is usuaully the first and easiest pattern to implement
- Factory Method makes a design more customizable and only a little more complicated
- Other design patternsrequire new classes, whereas FactoryMethod only requires a new operation
- its used when you dont anticipate the type of object it creates in the superclass
- To create a drawing application, for example,we define the classes DrawingApplication and DrawingDocument
- The Application class is responsible for managing Documents and will create them as required—when the user selects Open or New from a menu, for example
- the Application class only knows **when** a new document should be created, not **what** kind of Document to create
- Factory Method: creation through inheritance. Prototype: creation through delegation
- Benefits: decouple - encapsulate
- Benefits: decoupling the implementation of the product from its use
- Benefits: avoid duplication - one place to perform maintinance
- Benefits: clients depend only upon interfaces rather than the concrete classes required to instantiate objects
- Drawback: require creating a new subclass justto change the class ofthe product
- Drawback: limitation though: subclasses may return different types of products only if these products have a common base class or interface
- Drawback: The factory method in the base class should have its return type declared as this interface
- What’s the advantage of this? It looks like we are just pushing the problem off to another object?
- The pizzs factory may have many clients, so the solution is by encapsulating the pizza creating in one class, we now have only one place to make modifications when the implementation changes.
- Simple Factory VS Factory Method?
- SimpleFactory doesn’t give you the flexibility of the Factory Method because there is no way to vary the products you’re creating.

## Factory Method VS Abstract FactoryMethod

![factory-vs-abstract](/img/factory-vs-abstract.jpg)

![factory-vs-abstract2](/img/factory-vs-abstract2.jpg)
- **Similarities:**
  * decoupling applications from specific implementations
  * Both create objects
  * both encapsulate object creation to keep applications loosely coupled and less dependent on implementations
- **Differences:**
- **Factory Method:**
  * create objects thru inheritance (is a " relationship): object creation is delegated to subclasses, which implement the factory method to create objects
  * extend a class and provide an implementation for a factory method, which the factory method is used to create objects and lets the subclasses do their own creation
  * Create interface for one product
- **Abstract Factory Method:**
  * create objects thru composition (parts that make up the whole - has a" relationship): object creation is implemented in methods exposed in the factory interface
  * Does the same as pt2 but in a dif way. Provide abstract type for creating family of products, basically to group together a set of related products
  * Create interface for entire families of products
  * Concrete factories often implement a factory method to create their products. they are used purely to create products

## BUILDER
- Separate the construction of a complex object from its representation so that the same construction process can create different representations
- Solution to open ended objects
- Used for related products, and create different interfaces
-Structure:

![builder](/img/builder.PNG)

- **Why no abstract classfor products?** because in many cases, concrete builders differ so greatly in their representation that there is little to gain from giving different products a common parent class, ands unlikely to have a common interface

### Related Patterns
- Abstract Factory is different from the Builder is that the Builder pattern focuseson constructing a complex object step by step
- Abstract Factory's emphasis is on families of product objects
- Builder returns the product as a final step, whreas the Abstract Factory pattern , the product gets returned immediately



## Prototype
- Used to copy objects, and useful to subtitue when new objects of smal variation are created
- reduce new subclasses
- Disadvantage: needing to add clone method to each subclass

## SINGLETON
- There should be only one file system and one window manager. A digital filter will have one A/D converter. An accounting system will be dedicated to serving one company
-  A global variable makes an object accessible, but it doesn't keep you from instantiating multiple objects
- Examples: we only need one of: thread pools, caches, dialog boxes, objects that handle preferences and registry settings, objects used for logging, and objects that act as device drivers to devices like printers and graphics cards

![logging](/img/logging.PNG)
- Example Logging (since using the pattern here doesnt affect program execution), but still singleton should be last option as u might need two log files in the future, so make log accessible everywhere instead and create static field
- Example: obtain a handle to a service locator
- Example: reading configuration file
- if we were to instantiate more than one we’d run into all sorts of problems like incorrect program behavior, overuse of resources, or inconsistent results
- The signleton pattern work like global vars, but the downside of the latter: if you assign an object to a global variable, then that object might be created when your application begins. Right? What if this object is resource intensive and your application never ends up using it? As you will see, with the Singleton Pattern, we can create our objects only when they are needed
- Drawback: THe singleton pattern violates the SRP in that it does: provide one object - responsbile for whatever role its tasked for
- Drawback: Hard to test: hidden coupling
- Drawback: Hard to subclass; since contructor is private 
- Only use singleton when: every class uses it exactly the same way - every application need one instance only - classes that do not store any state on it’s internal variables;
- Alternative: seperate 'need of one' and 'need of singleton
- Alternative: since singleton can be thread unsafe, create an interface and build default implementation. This will make the dependencies clearer

## Structural Patterns
- Concerned with how classes and objects are composed to form larger structures

## Facade
- It doesnt encapsulate the system, it provides a simplified interface and exposes full functionality interfaces. 
- The intent of the Adapter Pattern is to alter an interface so that it matches one a client is expecting.
- The intent of the Facade Pattern is to provide a simplified interface to a subsystem.
- Benifit: avoid tight coupling between clients and subsystems

## Proxy Pattern
- proxy pretends it’s the real object, but it’s really just communicating over the net to the real object.

## Command Pattern

![command](/img/command.PNG)
- The Command Pattern allows you to decouple the requester of an action from the object that actually performs the action
- all command objects implement the same interface, and two method called execute() and undo()
- Invoker: stores command... Receiver: invokes the command action
- How does the remote know the living room from the kitchen light? IT DOESNT
- Benifit: helps with undo, define an order/queue for commands

## Observer Pattern
- Benifit: loosely coupled: the subjest doesn’t need to know the concrete class of the observer
- Benefit: we dont need to modify the subject if an observer is added