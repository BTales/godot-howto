Introduction
============

The current godot documentation can be found here: https://docs.godotengine.org/en/stable/.

Execution model
---------------
Godot uses an eventloop, see https://en.wikipedia.org/wiki/Event_loop. This means, that every bit of user-code is 
generally triggered by an event - such as keypress, mouse movement, timer events or rendering events. The general idea
is, that the eventloop will run once per generated frame of the resulting video stream.

All code needs to register itself for execution. This is hidden to the user when using one of Godot's integrated classes,
but for a good understanding of what happens, the programmer needs to be aware of this.

When the engine starts, there is an initialization phase where some basic events are registered and some internal threads might be started. 
After this, the engine starts it eventloop, which is an endless-loop and everything that happens after this point is event-based.

The eventloop is basically single-threaded, so the programmer has full control of the CPU when her code runs - there is no need
for locking and nothing can change while the code is executing. It also means, that the code should never block or run too long - 
otherwise the engine cannot update the generated picture in time and the frame rate will break down.

As a consequence, if the programmer needs to execute a task which needs some time to finish (e.g. some time-consuming calculations or
some complex IO), some more advanced programming techniques are required to perform these tasks in the background without disturbing
the main event loop. For example, Godot offers the ResourceLoader class, which implements background loading of resources while the
loop is running undisturbed.

Syntax
------
Godot's native programming language is GDScript, which has been developed for the Godot engine. It is 
a dynamically-typed language whose syntax resembles Python - see https://docs.godotengine.org/en/stable/tutorials/scripting/gdscript/gdscript_basics.html 
for an introduction.

Important: With Godot's version 4 also GDscript has received a major upgrade (from 1.0 to 2.0). A very helpful summary
of changes can be found here: https://gdscript.com/articles/godot-4-gdscript/. It is important to know, that there 
have been incompatible changes from 1.0 to 2.0 - be careful with online tutorials and documentation, as it may still
refer GDScript 1.0! Even the official documentation in non-english languages might not be up-to-date! 

One easy was to find out the GDScript version is the presence of so-called annotations, which start with a @-character. So, if 
you find a line starting with @export it is Godot 2.0, if you find export (without @) it is Godot 1.0.

Object Orientation
------------------
Godot is build around an object-oriented approach *without* multiple inheritance. Thus, to access any of Godot's complex
API, the programmer needs to extend one of Godot's classes.

Each .gd script is automatically an unnamed class (unless the keyword 'class_name' is used) and automatically inherits from
the class 'RefCounted' (https://docs.godotengine.org/en/stable/classes/class_refcounted.html#class-refcounted), unless a different 
parent class is referenced using the keyword 'extends'. The root class in Godot's class  hierarchy is called 'Object' 
(https://docs.godotengine.org/en/stable/classes/class_object.html#class-object).

Godot's main building blocks are inheriting from the 'Node' class, see: https://docs.godotengine.org/en/stable/classes/class_node.html. 
The 'Node' class offers a _processing() method, which is automatically registered in the eventloop and executed in each processing
step.

Memory Management
-----------------
Most of Godot's classes inherit from the RefCounted class, which implements an automatic memory management using reference 
counting https://en.wikipedia.org/wiki/Reference_counting. This basically means, that every reference to an object is 
counted and if the count drops to zero, the object will be freed from memory. 

Although this approach has its limitations when reference loops occur, it seems good enough in most practical cases. 
If needed, the programmer can avoid reference loops using weakref(Object) to create a weak reference that does not
contribute to the reference count. 

If reference counting is not desired, the programmer may inherit from 'Object' and implement manual memory management.

Disc Layout
-----------
A Godot game is completely self-contained and resides in one folder on your disk. Inside this folder, the structure is 
completely user-defined; Godot will track metadata in some additional files. The editor will also add a .godot folder,
which contains cache only and can be safely ignored e.g. when using some version control.

As a consequence, sharing code in-between games is nothing Godot supports - there are no external libraries to do so. 
A good work-around here is the use of the git version control tool and it's submodule feature. 
(UPDATE: The editor offers so-called Asset-Libs, which may be added to the current project. Unsure if it is possible
to create a private library.)

Game Layout
-----------
A Godot game consists of one or more so-calles 'Scenes', whose metadata is stored in .tcsn files. Every scene references 
excactly one root-Node, which must be one of Godot's built-in classes that inherit from the 'Node' class. A node may again
contain other Nodes, so that the scene basically consists of a tree of nodes with a single root.

A scene itself also inherits from Node, so that a scene may contain another scene (one or multiple times) as a child-scene.

When a scene is loaded and activated, each node of its node tree will be instantiated.

One or many .gd scripts may be attached to a node. Each script describes a class of its own and one instance will be
automatically created as soon as the node is instantiated. It lives as a property of the containing node.

The programmer may also create scripts (classes) without attaching them to any node. An instance of any of these classes
must be created using ClassName.new(). (This implies that the class has a name using the class_name keyword - unsure if an
unnamed class may be instatiated manually.)

Editor
------
The programmer's interface is the Godot Editor, which offers live syntax checking and debugging of your code. 
(Fun fact: the editor itself is basically using the Godot engine!)


Best Practices
--------------
See https://docs.godotengine.org/en/stable/tutorials/best_practices/index.html

As always in programming, it is good practice to break down the code into small pieces, which can be run and 
tested independently. Deriving from this, it is desirable to design the game in a way that *all* basic building blocks - 
the scenes - can run independently. To achieve this, the programmer must avoid that scenes need to find any environment
by themselves but instead allow external configuration (i.e. dependency injection) where needed.

Another helpful item to access global state may be so-called autoloads: 
https://docs.godotengine.org/en/stable/tutorials/best_practices/autoloads_versus_internal_nodes.html
