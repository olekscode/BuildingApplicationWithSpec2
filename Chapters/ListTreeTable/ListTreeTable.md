## Lists, tables and trees

status: definitively missing

An important part of user interfaces is about displaying lists of data. 
Such lists can be structured as tables, plain lists but also trees supporting nesting of data.

Spec proposes three main presenters: `SpListPresenter`, `SpTreePresenter` and `SpTablePresenter`. 
In addition it offers `SpComponentListPresenter` which allows one to embedd any presenter in a list. 
In this chapter we present some of the functionality of such presenters.

### Lists

Creating a list is a simple as instantiating a `SpListPresenter` and specifying a list of items that the list should display.
The following script illustrates this and the result is shown in Figure *@figSimpleList@*.

```
SpListPresenter new
	items: Smalltalk allClasses;
	open
```

![A simple list showing class names.](figures/List1Simple.png width=60&label=figSimpleList)

We can change the header title of the list using the message `headerTitle:`. 
The header title can be hidden using the message `hideHeaderTitle`.


###### Controlling item display.
By default a list item is displayed using the result of the `displayString` message sent to the item.
We can configure a list to apply a block to control the display of each item using the message `display:`.
The following script configures a list presenter to display the name of the methods of the class `Point` instead of showing the result of `printString`. See Figure *@figSimpleList2@*.

```
SpListPresenter new
	items: Point methods;
	display: [ :item | item selector ];
	open
```

![A simple list controlling the way items are displayed.](figures/List1SimpleDisplay.png width=60&label=figSimpleList2)

We can sort the items using the message `sortingBlock:`.

```
SpListPresenter new
	items: Point methods;
	display: [ :item | item selector ];
	sortingBlock: [ :a :b | a selector < b selector];
	open
```

### Decorating elements

We can configure the way items are displayed in a more finer grained way. The following example illustrates it: we can control the icons are associated with the item using the message `displayIcon:`, the item color using the mssage `displayColor:`. The format (bold, italic, underline) can the controlled by the corresponding messages `displayItalic:`, `displayBold:` and `displayUnderline:` (See Figure *@figSimpleListDecorated@*).



```
SpListPresenter new
	items: self environment allClasses;
	displayIcon: [ :aClass | self iconNamed: aClass systemIconName ];
	displayColor: [ :aClass | 
		(aClass name endsWith: 'Test')
			  ifTrue: [ Color green ]
			  ifFalse: [ Smalltalk ui theme textColor ] ];
	displayItalic: [ :aClass | 
		aClass name includesSubstring: 'abstract' caseSensitive: false ];
	displayBold: [ :aClass | aClass hasSubclasses ];
	displayUnderline: [ :aClass | aClass numberOfMethods > 10 ];
	open
```

![A decorated list: icons, text styling and color.](figures/List1Decorated.png width=60&label=figSimpleListDecorated)


### About multiple/single selection

Lists can support multiple selection or not.
The message `beMultipleSelection` controls such aspect.

```
SpListPresenter new
	items: Smalltalk allClasses;
	beMultipleSelection;
	open
```



Since selection can hold multiple item, there is an impact on the protocol to react to selection changes.
Indeed, lists, filtering list, trees, and tables offers the `whenSelectionChangedDo:`API and not `whenSelectedItemDo:`.
The argument of the block is then a selection instance of `SingleSelectionMode` or `MultipleSelectionMode` (`SpSingleSelectionMode`, `SpMultipleSelectionMode`, or `SpTreeMultipleSelectionMode` and `SpTreeSingleSelectionMode`).


Here is a typical use case of the method `whenSelectionChangedDo:`.

```
connectPresenters

	changesTree whenSelectionChangedDo: [ :selection | 
		selection selectedItem
			ifNil: [ textArea text: '' ]
			ifNotNil: [ :item | textArea text: (self buildDiffFor: item) ] ]
```


### Drag and drop

Lists and other container structures supports drag and drop.
The following script shows how to configure two lists to support drag from one and dropping in another.

```
| list1 list2 |
list1 := SpListPresenter new
list	
	items: #( 'abc' 'def' 'xyz' );
	dragEnabled: true.

list2 := SpListPresenter new.
list2	dropEnabled: true;
	wantsDrop: [ :transfer | transfer passenger allSatisfy: #isString ];
	acceptDrop: [ :transfer | list2 items: list2 items , transfer passenger ].

SpPresenter new
	layout: (SpBoxLayout newLeftToRight
		 add: list1;
		 add: list2;
		 yourself);
	open
```

The following script illustrates the API.
- `dragEnabled:` configures the receiver to be dragged.
- `dropEnabled:` configures the receiver to accept dropped items.
- `wantsDrop: [ :transfer | transfer passenger allSatisfy: #isString ]`. With the message `wantsDrop:` we can specify a predicate to accept a dropped element. 
- `acceptDrop: [ :transfer | list2 items: list2 items , transfer passenger ]`. The message `acceptDrop:` specifies the treatment performed once the dropped item is accepted.

### Activation clicks

An element on a list can be _'activated'_, meaning it will trigger an event to execute an action on it. 
Note that an activation is different than a selection: one can _select_ an element without activating it.
The messages `activateOnDoubleClick` configures the list to react to double click, while its counterpart `activateOnSingleClick`.


### Filtering lists

Lists can also be filtered as shown in Figure *@figFiltering@*.
The following script shows the use of the `SpFilteringListPresenter`.



```
SpFilteringListPresenter new
	items: Smalltalk allClasses ;
	open; 
	withWindowDo: [ :window | 
		window title: ' SpFilteringListPresenter example' ]
```

![A filtering list with bottom filter.](figures/FilteringList.png width=60&label=figFiltering)

The following script shows that the filter can be placed on the top. 

```
SpFilteringListPresenter new
	items: Smalltalk allClasses ;
	openWithLayout: SpFilteringListPresenter topLayout ; 
	open; 
	withWindowDo: [ :window | 
		window title: ' SpFilteringListPresenter example' ]
```

Note that a filter can be declared upfront using the message `applyFilter:`.

```
SpFilteringListPresenter new
	items: Smalltalk allClasses ;
	openWithLayout: SpFilteringListPresenter topLayout ; 
	applyFilter: 'ZZ';
	open ; 
	
	withWindowDo: [ :window | 
		window title: 'SpFilteringListPresenter prefiltered example' ]
```

### Selectable filtering lists

Often lists are used to select items. This is what the class `SpFilteringSelectableListPresenter` offers. 
In addition to be able to filter items, it lets the user select items by ticking them as shown by Figure *@figSelectable@*.

![A selectable filtering list with bottom filter.](figures/SelectableList.png width=60&label=figSelectable)

The following script produces the situation describes by 
```
(SpFilteringSelectableListPresenter new
	items: Smalltalk allClasses ;
	layout: SpFilteringListPresenter topLayout ; 
	asWindow)
		title: 'SpFilteringSelectableListPresenter example';
		open

```


### Component List

While the lists we saw until now are homogeneous in the sense that they all display strings, Spec offers the possibility to display list of presenters. It means that elements in the list do not have the same size and can contains other presenters. 

This lets developers produce advanced user interface such as the one of the report builder of the ModMoose toolsuite shown in Figure *@figModMoose@*.

![An example of a component list from the ModMoose platform.](figures/mooseQDScreenshot.png width=80&label=figModMoose)

The following script shows how to define a `SpComponentListPresenter` as its result is shown in Figure *@figCompo@*

```
| list |
list := { 
        (SpLabelPresenter new
	         label: 'Test 1';
	         yourself).
        (SpImagePresenter new
	         image: (self iconNamed: #smallOk);
	         yourself).
        (SpButtonPresenter new
	         label: 'A button';
	         yourself).
        (SpImagePresenter new
	         image: PolymorphSystemSettings pharoLogo asForm;
	         yourself) }.

SpComponentListPresenter new
	presenters: list;
	open
```

![A component list with several different presenter: a label, an image, a button, and an image.](figures/ComponentList.png width=50&label=figCompo)

### Tables

### Trees

### Conclusion