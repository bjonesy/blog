---
template: post
title: Event Bubbling In Angular
slug: /posts/event-bubbling-in-angular
draft: true
date: 2019-06-23T15:09:24.470Z
description: >-
  In this post I will share an example of event bubbling in Angular using parent
  and child component communication. This example is a common use case that I
  come across daily in the current application I work on.
category: Angular
tags:
  - Angular
---
In this post I will share an example of event bubbling in Angular using parent and child component communication. This example is a common use case that is used in the current applications I work on.  

In this example I will display a list of items that will contain an index number, label and a call to action button.  When the call to action button is clicked it will display a confirmation dialog with actions allowing the user to remove the list item from the list or cancel the current action. 

The parent component will be responsible for displaying an array of list items and will show a confirmation dialog when the child component list item call to action is clicked. The confirmation dialog will ask the user if they would like to remove the current list item or cancel this action. Once the user clicks on an action in the confirmation dialog, the list item will either be filtered out of the list or the dialog will just close.

**Example: Parent Component**

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

I've created an array `listItems` that is just a simple array of items that contains an `id` which is a number and a `label` which is a string. I've also created a function `onRemoveListItem` that takes one parameter which is an `id` that is a number. This function will fire when an event from the child component is emitted. This event is when the user clicks on the call to action button in the child component. The parent component will then show a confirmation dialog asking the user to proceed with another set of actions. Based on the user action from the dialog, the list item will be filtered out or the dialog will just close.

```
// When the child component emits and event. User has clicked call to action in the child component
onRemoveListItem(id: number): void {
    // Message
    this.dialogConfirmation(of('Remove list item?'))
      .pipe(
        takeUntil(this.dispose),
        // Action - removing item from the list
        switchMap(() => (this.listItems = this.listItems.filter(x => x.id !== id)))
      )
      .subscribe();
  }
```
Let's take a look at the child component. The child component will mainly be dumb and will let the parent component dictate what happens based on the user's action. This child component will take in some data from each list item such as the list item index and the list item itself. When the call to action button on this child component is clicked it will emit an event to the parent component. In this example we are emitting the list item id.  

**Example: Child Component**

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
There are two inputs `listItem` which is the list item that is passed down from the parent and the `index` which is the index of the list item that is passed down. There is one output `onRemoveListItem` which is a `EventEmitter` that will emit an event which in this case will be a number.  

Let's look at the template for the parent component.

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
I've created a list `<ul>` and using the structural directive NgForOf shorthand `ngFor` I iterate through each list item setting a variable for the index. I use my child component as the template for each list item and pass in the data I need. I pass in the index and the list item to the child component. I also attach the `onRemoveListItem` function to the `EventEmitter` in the child component which is `onRemoveListItem`.

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
``
