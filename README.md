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

### Creational Pattern
- Used to create different types: Maze game, create different mazes 
- Singleton can ensure there's only one maze per game and that all game objectshave ready access to it—without resorting to global variables or functions
- Singleton also makes it easy to extend or replace the maze without touching existing code