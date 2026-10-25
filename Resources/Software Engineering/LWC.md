# LWC

# Events Up, Properties Down

In a complex component (one that contains several parent and child components), the components can communicate up and down.

![Parent and child components passing information up and down.](https://res.cloudinary.com/hy4kyit2a/f_auto,fl_lossy,q_70/learn/modules/lightning-web-components-basics/handle-events-in-lightning-web-components/images/dd71192f1a4684f933a4f7a872c55df0_kix.sij0ly3q7km1.jpg)

## **Passing Information Up**

The c-todo-item child component dispatches an event to the parent c-todo-app component. The parent can handle the event and change the current page.

```java
// todoItem.js
import { LightningElement } from 'lwc';
  ...
  nextHandler() {
    this.dispatchEvent(new CustomEvent('next'));
  }
}
```

The parent component listens for the event with the inline event handler prefixed with 'on'(onnext).

```java
<!-- todoApp.html -->
<template>
  <c-todo-item onnext={nextHandler}></c-todo-item>
</template>
```

And passes the event object to an event handler.

```java
// todoApp.js
import { LightningElement } from 'lwc';
export default class TodoApp extends LightningElement {
  ...
  nextHandler(){
    this.page = this.page + 1;
  }
}
```

## **Passing Information Down**

The c-todo-app parent component passes a property or invokes a method in the child component. For example, the parent can set a text value in a child component, or invoke a method in the child component.

You can make a component property public by prefacing it with the `@api` decorator. Then, set the public property by an external component.

For example (don't create these components), if the c-todo-item child component has the following:

```java
// todoItem.js
import { LightningElement, api } from 'lwc';
export default class TodoItem extends LightningElement {
  @api itemName;
}
```

Set the value from the parent with the following:

```java
<!-- todoApp.html -->
<template>
  <c-todo-item item-name="Milk"></c-todo-item>
</template>
```

Similarly, you can create public methods that are callable from a parent component. Create a public method in the child component by defining it with the `@apiCopy` decorator, then call it from the parent component.

Let's say that we have a child component like this one (don't create these components).

```java
// videoPlayer.js
import { LightningElement, api } from 'lwc';
export default class VideoPlayer extends LightningElement {
  @api play() {
    // Play music!
  }
}
```

When the c-video-player component is included in a parent component, we can invoke the method from the parent component like this:

```java
// methodCaller.js
import { LightningElement } from 'lwc';
export default class MethodCaller extends LightningElement {
  handlePlay() {
    this.template.querySelector('c-video-player').play();
  }
}
```

## Using HTML Standard Global Event Handlers

Each tile component listens for the user click because the tile component's HTML (tile.html) contains an `onclick` event listener.

```java
<template>
  <div class="container">
    <a onclick={tileClick}>
      <div class="title">{product.fields.Name.value}</div>
      <img class="product-img" src={product.fields.Picture_URL__c.value} alt={product.fields.Name.value}/>
    </a>
  </div>
</template>
```

When a user clicks one of the tile instances in the UI, the `onclick` listener calls the handler function `tileClick` in the tile.js JavaScript file.

```java
import { LightningElement, api } from 'lwc';
export default class Tile extends LightningElement {
  @api product; tileClick() {
    const event = new CustomEvent('tileclick', {
      // detail contains only primitives
      detail: this.product.fields.Id.value
    });
    // Fire the event from c-tile
    this.dispatchEvent(event);
  }
}
```

## **Wire Service To Bring Data to Your App**

The wire service is part of our platform and delivers a stream of data. The `@wireCopy` decorator, which we mentioned briefly before, implements the wire service for your app. To use the wire service, do the following.

1. Import a wire adapter in the JavaScript file.
2. Decorate a property or function with the `@wireCopy` decorator.

Here's the syntax.

import { adapterId } from 'adapter-module'; @wire(adapterId, adapterConfig) propertyOrFunction;

Copy to clipboard

```java
import { adapterId } from 'adapter-module';
@wire(adapterId, adapterConfig) propertyOrFunction;
```

- `adapterIdCopy` (Identifier)—The identifier of the wire adapter.
- `adapter-moduleCopy` (String)—The identifier of the module that contains the wire adapter function.
- `adapterConfigCopy` (Object)—A configuration object specific to the wire adapter.
- `propertyOrFunctionCopy`—A private property or function that receives the stream of data from the wire service. If a property is decorated with `@wireCopy`, the results are returned to the property's data property or error property. If a function is decorated with `@wireCopy`, the results are returned in an object with a data property and an error property.