## A Simple Contact Book

status: Ready for review - should publish the code

In this chapter we develop a simple model for a contact book.
Then we define a user interface.
This example will be used later in the book as an example to explain concepts such as commands, applications, windows.

Now it is more a replay of the concept previously mentioned.
We start by implementing classes modeling the domain and then we will add a basic graphical user interface to obtain a little application as shown in Figure *@overview@*.


![A rudimentary contact book application.](figures/withmenuExtension.png width=60&label=overview)

### Contact book model

The model for the domain of our example is composed of two classes: Contact and ContactBook as shown in Figure *@contactmodel@*.

![A simple model for the contact book.](figures/contactBook.pdf width=60&label=contactmodel)

#### Contact

The class modeling a contact is defined as follow.

```
Object << #EgContact
    slots: {#name . #phone};
    package: 'EgContactBook'
```


It just defines a `printOn:` method and a couple of accessors (not shown in the text).

```
EgContact >> printOn: aStream

        super printOn: aStream.
        aStream nextPut: $(.
        aStream nextPutAll: name.
        aStream nextPut: $).
```

```
EgContact >> hasMatchingText: aString
    ^ name includesSubstring: aString caseSensitive: false
```


```
EgContact class >> name: aNameString phone: aPhoneString

    ^ self new
        name: aNameString;
        phone: aPhoneString;
        yourself
```


#### ContactBook

Now we define the class modeling the contact book.
As for the contact class, it is simple and quite straighforward.

```
Object << #EgContactBook
    slots: { #contacts };
    package: 'EgContactBook'
```

```
EgContactBook >> initialize

    super initialize.
    contacts := OrderedCollection new
```

We add the possibility to add and remove a contact

```
EgContactBook >> addContact: aContact
    contacts add: aContact
```


```
EgContactBook >> removeContact: aContact
    contacts remove: aContact
```


```
EgContactBook >> addContact: newContact after: contactAfter
    contacts add: newContact after: contactAfter
```


We add a simple testing method in case one want to write some tests \(which we urge you to do\).

```
EgContactBook >> includesContact: aContact
    ^ contacts includes: aContact
```


And now we add a method to create a contact and add it to the contact book.

```
EgContactBook >> add: contactName phone: phone
    | contact |
    contact := EgContact new name: contactName; phone: phone.
    self addContact: contact.
    ^ contact
```


Finally some facilities to query the contact book.

```
EgContactBook >> findContactsWithText: aText
    ^ contacts select: [ :e | e hasMatchingText: aText ]
```


```
EgContactBook >> size
    ^ contacts size
```


#### Pre-filling up the contact book


Since we want to have some contacts and we way to keep them without resorting
to a database or file we set some class instance variables.

We defined two class instance variables: `family` and `coworkers` and define
some class method accessors as follows:

```
EgContactBook class >> family
    ^family ifNil: [
        family := self new
            add: 'John' phone: '342 345';
            add: 'Bill' phone: '123 678';
            add: 'Marry' phone: '789 567';
            yourself]
```


```
EgContactBook class >> coworkers
    ^coworkers ifNil: [
        coworkers := self new
            add: 'Stef' phone: '112 378';
            add: 'Pavel' phone: '898 678';
            add: 'Marcus' phone: '444 888';
            yourself]
```


We add one method to be able to reset them if necessary.
The `<script>` pragma tells the system browser to add a small button to execute `reset` method easily.

```
EgContactBook class >> reset
    <script>
    coworkers := nil.
    family := nil
```



### A simple graphical user interface


Now we define the graphical user interface \(GUI\) to expose the model to the user.
The targeted GUI is shown in Figure *@firstFullUI@*.

![A rudimentary contact book application.](figures/firstFullUI.png width=60&label=firstFullUI)

We define the class `EgContactBookPresenter`.
It holds a reference to a contact book and it is structured around a table.

```
SpPresenter << #EgContactBookPresenter
    slots: { #table . #contactBook};
    package: 'EgContactBook'
```


We define an accessor for the contact book and the table.

```
EgContactBookPresenter >> contactBook
    ^ contactBook
```


```
EgContactBookPresenter >> table: anObject
    table := anObject
```


```
EgContactBookPresenter >> table
    ^ table
```


#### Initializing the model


We specialize the method `setModelBeforeInitialization:` that is invoked by the framework to assign the `contactBook` instance variable to the object passed during the execution of the expression `(EgContactBookPresenter on: EgContactBook coworkers) open`.

```
EgContactBookPresenter >> setModelBeforeInitialization: aContactBook
    super setModelBeforeInitialization: aContactBook.
    contactBook := aContactBook
```


#### Layout


```
EgContactBookPresenter class >> defaultSpec

    ^ SpBoxLayout newVertical add: #table; yourself
```


#### Widget initialization


We initialize the table to display two columns for the name and the phone.
The respective accessor messages will be sent to the elements to fill up the columns.
Finally the table contents is set using the contact book contents.

```
EgContactBookPresenter >> initializePresenters
    table := self newTable.
    table
        addColumn: (StringTableColumn title: 'Name' evaluated: #name);
        addColumn: (StringTableColumn title: 'Phone' evaluated: #phone).
    table items: contactBook contents.
```


Now we can start opening the UI by executing the following snippet
`(EgContactBookPresenter on: EgContactBook coworkers) open`

We define a class method to be able to easily re-execute the set up.
```
EgContactBookPresenter class >> coworkersExample
    <example>
    ^ (self on: EgContactBook coworkers) open
```


You should obtain the following GUI as shown in Figure *@first@*.

![First version of the GUI without menus and toolbar.](figures/firstVersion.png width=60&label=first)

#### Interacting with user


We now implement the method that will open a window to ask the user to create
a new contact for the contact book.

```
EgContactBookPresenter >> newContact
    | rawData splitted |
    rawData := self
        request: 'Enter new contact name and phone (split by comma)'
        initialAnswer: ''
        title: 'Create new contact'.
    splitted := rawData splitOn: $,.
    (splitted size = 2 and: [ splitted allSatisfy: #isNotEmpty ])
        ifFalse: [ SpInvalidUserInput signal: 'Please enter contact name and phone (split by comma)'  ].

    ^ EgContact new
        name: splitted first;
        phone: splitted second;
        yourself
```


To test it, we can get access to the presenter as follows
```
(EgContactBookPresenter on: EgContactBook coworkers)
    open presenter inspect
```


and you can send the `newContact` message as shown in Figure *@inspector@*.

![Playing inside the inspector.](figures/inspector.png width=80&label=inspector)


#### Some extra methods


We will also define the methods `isContactSelected` and `selectedContact` to know if a contact is currently selected and to return it.
It will help us later to add contact just after the currently selected contact.

```
EgContactBookPresenter >> isContactSelected
    ^ self table selectedItems isNotEmpty
```


```
EgContactBookPresenter >> selectedContact
    ^ table selection selectedItem
```


### Conclusion


We have a little contact book manager now that we can use to explain other topics.
