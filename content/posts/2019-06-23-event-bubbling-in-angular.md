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
In this post I will share an example of event bubbling in Angular using parent and child component communication. This example is a common use case that is used in the current application I work on.  

In this example I will display a list of items that will contain an index number, label and a call to action button, that when clicked, will display a confirmation dialog allowing the user to remove the list item from the list. 

The parent component will be responsible for displaying an array of list items and will show a confirmation dialog when the list item call to action is clicked.  When the call to action is click it will ask the user if they would like to remove the current list item or cancel this action. Once the user clicks on an action from the confirmation dialog the list item will either be filtered out of the list or the dialog will just close. 

The child component will take in some data from each list item such as the list item index and the list item.  It will emit an event which will emit  to the parent when the call to action is clicked
