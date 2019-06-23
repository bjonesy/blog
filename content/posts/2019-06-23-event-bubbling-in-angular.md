---
template: post
title: Event Bubbling In Angular
slug: /posts/event-bubbling-in-angular
draft: false
date: 2019-06-23T15:09:24.470Z
description: >-
  In this post I will share an example of event bubbling in Angular using parent
  and child component communication. This example is a common use case that is
  used on the current applications I work on.
category: Angular
tags:
  - angular
  - javascript
---
In this post I will share an example of event bubbling in Angular using parent and child component communication. This example is a common use case that is used on the current applications I work on.  

In this example I will display a list of items that will contain an index number, label and a call to action button.  When the call to action button is clicked it will display a confirmation dialog with actions allowing the user to remove the list item from the list or cancel the current action. 

The parent component will be responsible for displaying an array of list items and will show a confirmation dialog when the child component list item call to action button is clicked. The confirmation dialog will ask the user if they would like to remove the current list item or cancel this action. Once the user clicks on an action in the confirmation dialog, the list item will either be filtered out of the list or the dialog will just close.

**Example: Parent Component TS**

```
import { Component, HostBinding, OnDestroy } from '@angular/core';

import { Observable, of, Subject } from 'rxjs';
import { switchMap, takeUntil } from 'rxjs/operators';

import { DialogService } from '@app/shared/services';

@Component({
  selector: 'app-list',
  templateUrl: './list.component.html',
  styleUrls: ['./list.component.scss']
})
export class ListComponent implements OnDestroy {
  listItems = [
    {
      id: 1,
      label: 'List item 1'
    },
    {
      id: 2,
      label: 'List item 2'
    },
    {
      id: 3,
      label: 'List item 3'
    },
    {
      id: 4,
      label: 'List item 4'
    }
  ];

  @HostBinding('class.list')
  addComponentClass = true;

  private dispose = new Subject<void>();

  constructor(private dialogService: DialogService) {}

  ngOnDestroy(): void {
    this.dispose.next();
    this.dispose.complete();
  }

  dialogConfirmation(confirmMessage: Observable<string>): Observable<any> {
    return this.dialogService.confirm(confirmMessage, of(undefined), of('Yes'), of('Cancel'));
  }

  onRemoveListItem(id: number): void {
    this.dialogConfirmation(of('Remove list item?'))
      .pipe(
        takeUntil(this.dispose),
        switchMap(() => (this.listItems = this.listItems.filter(x => x.id !== id)))
      )
      .subscribe();
  }
}
```

I've created an array `listItems` that is just a simple array of items that contains an `id` which is a number and a `label` which is a string. I've also created a function `onRemoveListItem` that takes one parameter which is an `id` that is a number. This function will fire when an event from the child component is emitted. This event is triggered from a click event when the user clicks on the call to action button in the child component. The parent component will then show a confirmation dialog asking the user to proceed with another set of actions. Based on the user's action from the dialog, the list item will be filtered out or the dialog will just close.

Let's take a look at the child component. The child component will be a dumb component and will let the parent component dictate what happens based on the user's action. This child component will take in some data from each list item such as the list item index and the list item itself. When the call to action button on this child component is clicked it will emit an event to the parent component. In this example we are emitting the list item id.  

**Example: Child Component TS**

```
import { Component, EventEmitter, Input, Output } from '@angular/core';

import { ListItem } from '@app/pages/models';

@Component({
  selector: 'app-list-item',
  templateUrl: './list-item.component.html',
  styleUrls: ['./list-item.component.scss']
})
export class ListItemComponent {
  @Input()
  listItem: ListItem;

  @Input()
  index: number;

  @Output()
  readonly onRemoveListItem = new EventEmitter<number>();
}
```

There are two inputs `listItem` which are the list item that is passed down from the parent and the `index` which is the index number of the list item that is passed down. There is one output `onRemoveListItem` which is a `EventEmitter` that will emit an event which in this case will be a number. Let's look at the template for the child component.

**Example: Child Component HTML**

```
<div class="list-item-content">
  <div class="list-item-id">
    {{ index }}
  </div>
  <div class="list-item-label">
    {{ listItem.label }}
  </div>
  <div class="list-item-commands">
    <button
      mat-icon-button
      aria-label="Remove list item"
      (click)="onRemoveListItem.emit(listItem.id)"
    >
      <mat-icon>close</mat-icon>
    </button>
  </div>
</div>
```

We are using the `index` to display the list item number, using the `listItem.label` to display the text for the list item label and setting a click event on the call to action button `(click)="onRemoveListItem.emit(listItem.id)"` which will emit the `listItem.id`.

Now let's look at the template for the parent component.

**Example: Parent Component HTML**

```
<div class="page">
  <div class="page-header">
    <h2>Event Bubbling Example</h2>
  </div>
  <div class="page-content">
    <ul class="list">
      <li class="list-item" *ngFor="let item of listItems; let i = index">
        // Child component
        // @Input [listItem] is the list item
        // @Input [index] is the index which was set to the variable i
        // Child component EventEmitter is onRemoveListItem which is set the the function onRemoveListItem which captures the $event that is emitted which is the parameter we set in that function 
        <app-list-item
          [listItem]="item"
          [index]="i + 1"
          (onRemoveListItem)="onRemoveListItem($event)"
        ></app-list-item>
      </li>
    </ul>
  </div>
</div>
```

I've created a list `<ul>` and using the structural directive NgForOf shorthand `ngFor` I iterate through each list item setting a variable `i` for the index. I use my child component as the template for each list item and pass in the data I need. I pass in the index and the list item to the child component. I also attach the `onRemoveListItem` function to the `EventEmitter` in the child component which is `onRemoveListItem`.

```
<ul class="list">
      <li class="list-item" *ngFor="let item of listItems; let i = index">
        <app-list-item
          [listItem]="item"
          [index]="i + 1"
          (onRemoveListItem)="onRemoveListItem($event)"
        ></app-list-item>
     </li>
</ul>
```

This will display each list item with a number, a text label and a call to action button as mentioned before. When a user clicks the call to action button the `EventEmitter` from the child component will fire which will invoke the `onRemoveListItem` function in the parent. The event will bubble up the `listItem.id` to the parent and will trigger the confirmation dialog to open. 

![A Set of List Items](/media/screen-shot-2019-06-23-at-3.08.20-pm.png "A Set of List Items")

```
<app-list-item
  [listItem]="item"
  [index]="i + 1"
  // Output / EventEmitter from child attached to onRemoveListItem function in parent. The $event is the list item id that is bubbled up.
  (onRemoveListItem)="onRemoveListItem($event)"
></app-list-item>
```

In the confirmation dialog there is a message and there are two buttons. A message asking the user if they would like to remove the list item. The two buttons are labeled cancel and yes. If the user clicks cancel the dialog will close and nothing happens the list. If the user clicks yes the dialog will close and remove the list item from the list. This is all done in the parent component function `onRemoveListItem`.

![Confirmation Dialog Example](/media/screen-shot-2019-06-23-at-3.08.32-pm.png "Confirmation Dialog Example")

```
// When the child component emits and event. User has clicked call to action in the child component
onRemoveListItem(id: number): void {
    // Message
    this.dialogConfirmation(of('Remove list item?'))
      .pipe(
        takeUntil(this.dispose),
        // Action in dialog confirm user clicks Yes removing item from the list
        switchMap(() => (this.listItems = this.listItems.filter(x => x.id !== id)))
      )
      .subscribe();
  }
```
This is just one of the many ways you can use events in Angular. This happens to be one of the most common use cases I come across from day to day when developing.    
