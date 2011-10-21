This project provides widgets for lists of data, including simple sets of scrolling rows,
grids of data, on-demand lazy-loaded data, and various plugins for additional functionality. 
This project also provides touch scrolling for mobile devices with native style
momentum, bouncing, and scrollbars. To use this package, install with
[CPM](https://github.com/kriszyp/cpm):

    cpm install dgrid

Or download it along with its dependencies, which are:

* [xstyle](https://github.com/kriszyp/xstyle)
* [put-selector](https://github.com/kriszyp/put-selector)
* [The Dojo Toolkit](http://dojotoolkit.org) version 1.7
    * Out of the DTK components, Dojo core is the only hard dependency for dgrid;
      however, some of the test pages also use components from Dijit, and
      Dojox (namely grid for a comparison test, and mobile for a mobile page).

The dgrid project is available under the same dual BSD/AFLv2 license as the Dojo Toolkit.

# Components

dgrid's primary components are broken down into the following top-level modules:

## List

This provides the basic facilities for taking an array of objects and rendering as rows
of HTML in a scrollable area. This will automatically include touch scrolling capabilities
(via the `TouchScroll` module) on mobile devices.

The List can be used to render an array of data. For example:

    define(["dgrid/List"], function(List){
        // attach to a DOM id
        var list = new List({}, "list");
        // render some data
        list.renderArray(arrayOfData);
        ...
    });

### More List APIs

The base List class (inherited by all other classes) also has the following methods:

* `row(value)`: This will look up the requested row and return a Row object.
  The single parameter may be a DOM event, DOM node, data object id, or data object.
  The Row object has the following properties:
    * `id`: The data object id
    * `element`: The row's DOM element
    * `data`: The data object represented by the row
* `on(event, listener)`: Basic event listener functionality;
  simply delegates to the DOM element using standard `dojo/on` behavior.
* `renderArray(array, beforeNode)`: This can be called to render an array.
  The `beforeNode` parameter can be used to render at a specific place in the List.
* `renderRow(value, options)`: This can be overridden to provide
  custom rendering logic for rows
* `sort(property, descending)`: This can be called to sort the List by a given
  property; if the second parameter is passed `true`, the sort will be in descending order.
  The Grid and OnDemandList modules further extend this functionality.

See the [API viewer](doc/api.html?dgrid/doc/List)
for a list of methods available on the List component.
(Make sure to download the project to view this link; it is not available from GitHub.)

## Grid

This extends List to provide tabular display of data in columns. The Grid component will
provide a header for each column that corresponds to columns within the scrollable
grid of data. The columns of the grid are defined by using the `columns` property.
The columns property should be a hash (object) or array, containing column definition objects.
When `columns` is an object, each property's key is used as the id of the column, and
each value is the column definition object. When `columns` is an array,
the numeric indices become the column IDs. The column definition object
may have the following properties (all are optional):

* `field`: This is the property from the object in the list to display in the body of the grid. 
  Defaults to the id of the column (if no `get()` function is provided).
* `label`: This is the label to show in the header of the grid.
  Defaults to the id of the column.
* `className` - A DOM/CSS class to assign to the cells in the column. 
* `id` - This is the id of the column; normally this is determined automatically
  from the keys or indices in the `columns` object or array.
* `sortable`: This indicates whether or not you can sort on this column/field.
  Defaults to `true`.
    * Note that you can always programmatically sort a Grid by a given column using the
      `sort(property, descending)` method, regardless of whether that column's `sortable` status
      or even presence in the Grid altogether.
* `get`: This can be a function that will retrieve the value to render from the object in the list.
* `formatter`: This can be a function that will convert the value to an HTML string for rendering.
* `renderCell`: This can be a function that will be called to render the value into the target &lt;td> for each cell.
  (If `formatter` is specified, `renderCell` is ignored.)
* `renderHeaderCell` - This can be a function that will be called to render the value into the target &lt;th> for the columns header.

For example, we could create a grid with columns like:

    define(["dgrid/Grid"], function(List){
        var grid = new Grid({
            columns: {
                first: {
                    label: "First Name"
                },
                last: {
                    label: "Last Name"
                },
                age: {
                    get: function(object){
                        return (new Date().getTime() - object.birthDate.getTime()) / 31536000000;
                    }
                }
            }
        }, "grid"); // attach to a DOM id 
        // render some data
        grid.renderArray(arrayOfData);
        ...
    });

The column definition may alternately simply be a string, in which case the value of the string is
interpreted as the name of the column. Thus, we can more succinctly write simple columns:

    var grid = new Grid({
        columns: {
            first: "First Name",
            last: "Last Name",
            ...
        },
        ...
    }, ...);

Note that the Grid component also supports structures with multiple "sub-rows";
that is, it supports the idea of rendering multiple rows for each item.
Specification of multiple subrows is very much like specifying columns, except
that one uses the `subRows` property instead of `columns`, and it receives an
array of columns objects/arrays.

### Specifying Columns via HTML: GridFromHtml

Some developers prefer specifying column layouts in an actual table structure
because it is more convenient or semantically clearer.  dgrid supports this
via the GridFromHtml module.  When using this module, a `table` element should
be specified as the source node for the grid instance; it then scans for `th`
nodes within rows (typically placed inside the `thead`) to determine columns.

Column properties are specified within the `th`, primarily via the
`data-dgrid-column` attribute, which should contain a JavaScript object.
Properties which coincide with standard HTML attributes can also be specified
as such, e.g. `class`, `rowspan`, and `colspan`.  The innerHTML of the `th` is
interpreted as the column's `label` by default.

Note that *unlike* `data-dojo-props`, `data-dgrid-column` requires that you
include the surrounding curly braces around the object - this allows
alternatively specifying a column plugin instead of just a straight-up object.
(Note, however, that referencing column plugins requires that they be exposed
in the global scope, perhaps under a namespace.)

Examples of creating grids from HTML can be found in the
`GridFromHtml.html` and `complex_columns.html` test pages.

It is also possible to specify columnsets (for the `ColumnSet` module) via
HTML tables by using the `GridWithColumnSetsFromHtml` module.  ColumnSets are
expressed in HTML via `colgroup` tags.  See the `complex_columns.html` test
page for an example of this as well.

### Grid Styling

dgrid components are designed to be highly CSS-driven for optimal performance and organization,
so visual styling should be controlled through CSS. The Grid creates classes
based on the column ids and field names with the convention of
`column-<column-id>` and `field-<field-name>`.
(If you specify a `className` in the column definition, it is used in place of
`field-<field-name>`.) For example, you could define a grid and CSS like so:

    <style>
    .column-age {
        width: 80px;
    }
    .column-first {
        font-weight: bold;
    }
    </style>
    <script>
    define(["dgrid/Grid"], function(Grid){
        grid = new Grid({
                columns: [ // define the columns
                    age: "Age",
                    first: "First Name",
                    ...
                ]});
        grid.renderArray(someData);
        ...
    });
    </script>

The Grid class also provides a `styleColumn(colId, css)` method to programmatically
add styles to a column, by injecting a rule into a stylesheet in the document.
This method returns a handle with a `remove` function, which can be called to
remove the style rule that was added.

See the [API viewer](doc/api.html?dgrid/doc/Grid) for a list of methods available
on the Grid component.
(Make sure to download the project to view this link, it is not available from GitHub.)

## OnDemandList

This extends List to provide on-demand lazy loading or paging of data as the user
scrolls through the list, and interacts with a
[Dojo object store](http://dojotoolkit.org/reference-guide/dojo/store.html) for
querying of data. This provides a seamless, intuitive interface for viewing
large sets of data in scalable manner. This also provides sorting delegation to the store.

The OnDemandList requires that a store be specified via the `store` property,
and will call the `query()` method on the store to retrieve the data to be rendered.
OnDemandList can `query()` with start and count properties so as to only retrieve
the necessary objects needed to render the visible rows. As the list or grid is
scrolled, more `query()` calls will be made to retrieve additional rows,
and previous rows will be pruned from the DOM as they are scrolled well out of view.

For best results, the specified store should return query results with an `observe`
method, which enables the list to keep its display up to date with any changes
that occur in the store after the items are rendered.
The [`dojo/store/Observable`](http://dojotoolkit.org/reference-guide/dojo/store/Observable.html)
module can prove useful for this.

The given store should also support the `get` and `getIdentity` functions.

OnDemandList provides the following properties/methods:

* `renderQuery(query)`: Renders the given query into the list.
* `sort(property, descending)`: OnDemandList defers sorting to the store.
* `sortOrder`: Initially managed by List's `sort()` method,
  this stores the current sort order.
* `setQuery`: Allows specifying a new `query` object (and optionally, `queryOptions`)
  that the list will use when issuing queries to the store.
* `setStore`: Allows specifying a new store (and optionally, also `query` and `queryOptions`)
  for the list to reference.

## OnDemandGrid

This module is simply the composition of Grid and OnDemandList.
For example:

    define(["dgrid/OnDemandGrid"], function(Grid){
        grid = new Grid({
                store: myStore, // a Dojo object store
                columns: [ // define the columns
                    {label: 'Column 1', field: 'col1', editable: true, sortable: true},
                    {label: 'Column 2', field: 'col2'},
                    ...
                ]
            }, "grid");
        ...
    });

# Plugins

The following modules can be used as plugins to add extra functionality to a Grid.
To use these, simply add the module as a mixin in a `dojo.declare` inheritance chain.
For example, to create a grid based on OnDemandGrid with the
Selection and Keyboard handling plugins, we could do the following:

    define(["dojo", "dgrid/OnDemandGrid", "dgrid/Selection", "dgrid/Keyboard"], function(dojo, Grid, Selection, Keyboard){
        // create a grid based on plugins
        var MyGrid = dojo.declare([Grid, Selection, Keyboard]);
        // instantiate it
        grid = new MyGrid({
            store: myStore,
            ...
        }, "grid");

You can also perform inline mixin and instantiation:

    grid = dojo.declare([Grid, Selection, Keyboard])({
        store: myStore,
        ...
    }, "grid");

Below is a synopsis of available plugins.

## ColumnSet

TODOC

## Selection

Adds selection capability to a List/Grid. The list instance will include a
`selection` property representing the selected items.  This plugin will also
fire `dgrid-select` and `dgrid-deselect` events. For example:

    grid = dojo.declare([Grid, Selection])({
        selectionMode: "single",
        ...});
    grid.on("dgrid-select", function(event){
        // get the row that was just selected
        var row = grid.row(event);
        for(var id in grid.selection){
            if(grid.selection[id] === true){
                // iterate through all selected items
            }
        }
    });
    grid.on("dgrid-deselect", function(event){
        var row = grid.row(event);
        // row was just deselected 
    });

The following properties and methods are added by the Selection plugin:

* `selection`: The object containing the ids of the selected objects.
* `selectionMode`: A string indicating the mode of selection.
  The following values are acceptable:
    * `extended`: The default setting; follows common ctrl and shift key practices for selection
    * `single`: Only allows one row to be selected at a time
    * `multiple`: Similar to `extended`,
      but normal clicks add selection without removing previous selections
    * `none`: Nothing can be selected by user interaction;
      only programmatic selection (or selection via selectors) is allowed
</ul>
* `select(id)`: Programmatically select a row
* `deselect(id)`: Programmatically deselect a row

The `select` and `deselect` methods can be passed an object id, or anything else
acceptable by List's `row` method.

### CellSelection

The CellSelection plugin extends upon the functionality of the Selection plugin
to provide celection at the cell level.

Whereas Selection's `select` and `deselect` methods look up the passed argument
via List's `row` method, CellSelection looks it up via Grid's `cell` method.

## Keyboard

This plugin adds keyboard handling functionality.
The cursor keys can be used to navigate the focus across cells and rows,
providing accessibility and ease of use.

When used with grids, this plugin references the `cellNavigation` property of
the grid instance, to determine whether keyboard navigation and focus should
operate at the individual cell level (`true`, the default) or at the row level
(`false`).

# Column Plugins

The following modules are plugins designed for specific columns of cells.
These plugins are used by creating an instance and specifying it as a column in
`columns` (or `subRows`).

For example, to create a column structure where the first column has a
tree expander and the second column has a checkbox, we could do this:

    define(["dgrid/OnDemandGrid", "dgrid/Tree", "dgrid/Editor"], function(Grid, Tree, Editor){
        grid = new Grid({
                store: myHierarchicalStore, // a Dojo object store
                columns: [ // define the columns
                    // first column will have a tree expander:
                    Tree({label:'Name', field:'name'}),
                    // second column will render with a checkbox: 
                    Editor({label: 'A CheckBox', field: 'bool'}, "checkbox"),
                    // just a normal column:
                    {label:'Type', field:'type'},
                    ...
                ]
            }, "grid");
        ...
    });

## Tree

The Tree plugin enables expansion of rows to display children.
It expects to operate on an OnDemandGrid, whose store is expected to provide
a `getChildren(object, options)` method to return the children for each object.
Note that for best results, `getChildren` should return results with an `observe`
function as well, so that changes to children can also be reflected as they occur.

The store may also (optionally) provide a `mayHaveChildren(object)` method
which returns a boolean indicating whether or not the row can be expanded.

## Editor

The Editor plugin provides the ability to render editor widgets within cells
for a column, directly correlating them with the dirty state of an OnDemandGrid.
Changes will then be saved back to the store based on edits performed in the grid.

(TODO: flesh out more)

# Extensions

The following are additional plugins which dwell outside dgrid's core feature set.
Extensions live in the `extensions` subdirectory; their tests and
css/image resources also live under respective `css/extensions` and
`test/extensions` subdirectories.

## ColumnResizer

The ColumnResizer plugin, originally based on
[the gridx ColumnResizer module](https://github.com/evanhw/gridx/blob/master/gridx/modules/ColumnResizer.js)
but further developed to better integrate with dgrid,
can be used to add column resizing functionality (accessible via mouse drag).

## DnD

The DnD plugin can be used to add row drag'n'drop functionality.

### Requirements

The DnD module assumes usage of the OnDemandList or OnDemandGrid module; thus, it
expects a store to be in use.

The store should be order-aware, supporting the `options.before` parameter
on `add()` and `put()` calls to properly respond to DnD operations.

Additionally, if the store supports a `copy` method, it will be called for
DnD copy operations within the same list/grid (since a `put` would normally
relocate the item).

# Themes/Skins

The dgrid automatically loads the necessary structural CSS to work properly. However, you can
also use one of the the included skins/themes. There are claro.css, tundra.css, soria.css, and nihilo.css theme
files in the css/skins directory that can be used to skin the dgrid to a particular
look and feel.

## Grid Structure for custom CSS Styling

dgrid's appearance is designed to be styled and customized via CSS.
Many of the classes involved can be discovered by simply looking at elements in
your browser developer tools of choice.
Perhaps the most important classes are the `column-<id>` and `field-<fieldname>`
classes assigned to each cell in grids, which allow for per-column styling.

The following class names are used by dgrid and can be referenced from CSS:

* `dgrid`: Applied to each dgrid list or grid at the top-level element
* `dgrid-header`: Applied to the element which contains the header area
* `dgrid-scroller`: Applied to the element responsible for scrolling the data content
* `dgrid-content`: Applied to the element inside of the scroller area,
  which holds all the data contents
* `dgrid-row`: Applied to each row element
* `dgrid-row-even`: Applied to each even row element
* `dgrid-row-odd`: Applied to each odd row element
  Applying a different color to alternating rows can help visually distinguish individual items.
* `dgrid-selected`: Applied to selected rows or cells
* `dgrid-cell`: Applied to each cell element
* `dgrid-cell-padding`: Applied to each cell element, or to an
  inner element within the cell in older versions of non-quirks mode IE to
  properly apply padding to keep the padding within the box measurements
  (box-sizing is preferred by the grid).
* `dgrid-focus`: Applied to the element (cell or row) with the focus (for keyboard based navigation)
* `dgrid-expando-icon`: Applied to the expando icon on tree nodes
* `dgrid-header-scroll`: Applied to the node in the top right corner, above the vertical scrollbar

The following generic class names are also available for generic skinning
(following the jQuery ThemeRoller convention):

* `ui-widget-content`: Applied to each dgrid list or grid at the top element
* `ui-widget-header`: Applied to the element that contains the header rendering
* `ui-state-default`: Applied to each row element
* `ui-state-active`: Applied to selected rows or cells
* `ui-state-highlight`: Applied to a row for a short time when the contents are change (or it is newly created)
