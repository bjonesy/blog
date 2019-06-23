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

I've created an array `listItems` that is just a simple array of items that contains an `id` which is a number and a `label` which is a string. I've also created a function `onRemoveListItem` that takes one parameter which is an `id` that is a number. This function will fire when an event from the child component is emitted. This event is when the user clicks on the call to action button in the child component. The parent component will then show a confirmation dialog asking the user to proceed with another set of actions.

The child component will mainly be dumb and will let the parent component dictate what happens based on the user's action. This child component will take in some data from each list item such as the list item index and the list item itself. When the call to action button on this child component is clicked it will emit an event to the parent component.  

**Example: Child Component** 
