react-designer-core
=======================

Warning - it is prototype and work in progress.

React designer is WYSIWYG editor for **easy content creation** (legal contracts, business forms, marketing leaflets, technical guides, visual reports, rich dashboards, tutorials and other content, etc.).

[Live demo](http://rsamec.github.io/react-designer-core/)

It is based on separation between

+   logical component tree - JSON simple document description - [Page Transform Tree (PTT)](#PTT) - it is framework agnostic definition
+   visual component tree - [React virtual DOM](http://facebook.github.io/react) React virtual DOM - rendering to DOM so that it maps each component (terminal node) from logical tree to react component and its properties

## Main goals 

+   it is aimed for developers - to use it in their own solutions  
+   it is free and open source
+   it is simple by design - minimal JSON framework agnostic definition maps directly to React components       

## Features

+   directly manipulate the layout of a document without having to type or remember names of components, elements, properties or other layout commands.
+   high-quality on-screen output and on-printer output
+   precise visual layout that corresponds to an existing paper version
    +   support for various output formats - html, pdf.
+   comfortable user experience - basic WYSIWYG features
    +   support drag nad drop - resize object length, move object positions
    +   support manipulating objects -> copy, move, up, down objects in object schema hierarchy
    +   highlighting currently selected object and its parent
+   build-in html content publishing (preview of html dynamic document)
+   binding support using [react-binding](https://github.com/rsamec/react-binding) - experimental
+   props inheritance - when rendering occurs -> the props value is resolved by using a value resolution strategy (Binding Value -> Local Value -> Style Value -> Default Value)
+   usable for big documents - careful designed to use react performance
    +   we won't render the component if it doesn't need it
    +   simple comparison is fast because of using immutable data structure
+   undo/redo functionality


It comes with core components typical for WYSIWYG designers

+   Workplace - main working area for drawing documents
+   PropertyGrid - component properties editors (html editors, color pickers, json editors, etc.)
+   ObjectTree - logical component tree - enables to search and move components between nodes


## <a name="PTT">Page Transform Tree</a>

The PTT is essentially a set of semantic assumptions laid on top of the JSON syntax. The PPT document is a plain text JSON describing visual content on the page.
The typical description of the visual content consists of two parts:

+   logical component tree - it enables composition of elements - it describes the logical hierarchical layout of components
+   visual component tree - it describes the visual appearance - it describes the visual hierarchical layout of components

The PPT format is __framework agnostic__ and the is why the PTT document prescribes __only the logical component__ tree. The visual component description is not the part of the PPT format

### The structure of PTT document

It is a simple component tree that consists of these two nodes

+   **containers** - nodes that are invisible components - usable for logical grouping of reactive parts of document (sections)
+   **boxes** - terminal nodes (leaf) that are visible components - (components, boxes, widgets) - it maps to props of component

There is an minimal 'Hello world' example. The logical tree consists of one container and one box with TextBox element.

```json
{
 "name": "Hello World Example",
 "elementName": "PTTv1",
 "containers": [
    {
     "name": "container",
     "elementName": "Container",
     "style": { "top": 0, "left": 0, "height": 200, "width": 740, "position": "relative" }
     "boxes": [{
        "name": "TextBox",
        "elementName": "TextBox",
        "style": { "top": 0, "left": 0 },
        "props":{
             "content": "Hello world"
            }
        }],
    }]
}
```

### PPT Node

+   **containers** node - collection of children
+   **boxes** node - collection of widgets

The component schema tree is composed using __containers__ property as collection of children.
The boxes on the other hand is a leaf collection that can not have other children.

### PTT Node Properties

Each node can have these object properties

+   **name** - optional element identifier (has no impact on page tree rendering)
+   **elementName** - required component name - type of element to render 
+   **style** - required element positions and dimensions
    +   top, left - element position - if not specified the default value is 0
    +   width, height - element dimensions - if not specified the default value is the same as its parent
    +   position - support for various position schemas -> absolute or relative position of elements (normal flow, flex or grid position schemas is not yet implemented)
    +   zIndex - optional - it defines stacking context, if not defined - stacking context is based on the order in document
    +   transform - optional - it enables to translate, rotate, scale, move transform origin component
+   **props** - component props as component's options - specific properties of element to render

### PTT positioning schemas and units

PPT format distinguishes this positioning schemas (the meaning is the same as CSS positions schemas)  
 
+   __absolute positioning__ - position is assigned with respect to its parent (container)
+   __relative positioning__ - normal flow with support for offset relative to this position - for containers (sections) 

There are two rendering modes

+   __mixed positioning__ - absolute positioning for boxes (widgets) and relative for containers (sections)
+   __normal flow__ - relative positioning for both - boxes and containers 

The units are pixes and be careful is DPI dependend.
 

### PTT pages rendering

This section specifies how the page rendering occurs.

#### Page options

+   page height and width
+   page orientation ('portrait', 'landscape') is optional and defaults to 'portrait'.
+   margin can provided as an object containing one or more of the following properties: 'top', 'left', 'bottom', 'right'

#### Page algorithm

Example of algorithm how to render pages from PTT document definition. 

+   apply data binding for containers (sections) (visibility, ranges for repeatable containers)
+   remove invisible containers
+   expand repeatable containers - cloning row templates
+   transform relative positions to absolute positions
+   reduce to boxes (terminal nodes) - using containers absolute positions (top,height) and its dimensions (with, height)
+   group to pages - create pages and add boxes to them
+   apply data binding for boxes (widgets)


The proprietary implementation of the algorithm can be found [here](https://github.com/rsamec/react-page-renderer/blob/master/src/utilities/transformToPages.js).

### Visual componenet tree - using react

To render react components specifies in react is really simple

```js
    createComponent: function (box) {
        var widget =widgets[box.elementName];
        if (widget === undefined) return React.DOM.span(null,'Component ' + box.elementName + ' is not register among widgets.');

        return React.createElement(widget,box, box.content!== undefined?React.DOM.span(null, box.content):undefined);
    },
    render:function(){
       {this.props.boxes.map(function (box, i) {
                var component = this.createComponent(box);
                return (
                       <div style={box.style}>
                            {component}
                       </div>
                       );
       }, this)}
    }
```

## Demo & Examples

[Live demo](http://rsamec.github.io/react-designer-core/)

To build the examples locally, run:

```
npm install
gulp dev
```

Then open [`localhost:8000`](http://localhost:8000) in a browser.


## Installation

The easiest way to use this component is to install it from NPM and include it in your own React build process (using [Browserify](http://browserify.org), [Webpack](http://webpack.github.io/), etc).

You can also use the standalone build by including `dist/react-shapes.js` in your page. If you use this, make sure you have already included React, and it is available as a global variable.

```
npm install react-designer-core --save
```

## Usage

import {Workplace,ObjectBrowser,ObjectPropertyGrid} from 'react-designer-core';

See the example folder.

## Roadmap

+   support for typography (vertical rhythm, modular scale, web fonts, etc.)
+   improve designer experience
    +   move objects in object browser
    +   disabled add widget when box is selected
    +   improve property editor
+   performance issues
    +   recheck - should component update
    +   parse property values (parseInt,etc.) - to many places - remove defensive programming favor contract by design
+   data binding refactoring    
    +   support for binding to remote stores (consider falcor)
    +   data watchers - if some data changes, it changes on the other site
+   support for more positioning schemas - flex, grid, ...
+   better support for PDF
    +   better support html fragments -> to pdf (using html parser) - consider using pdfmake

### License

MIT. Copyright (c) 2015 Roman Samec

