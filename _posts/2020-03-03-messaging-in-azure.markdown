---
layout: post
title:  "Messaging in Azure"
date:   2020-03-03 11:16:00 +0100
categories: azure messaging events
---

## Messages or Events

Azure offers the following services for handling messages and events:

- Event Grid
- Event Hubs
- Service Bus

At first sight it can be difficult to choose the right service for your scenario. To better determine what service to use, you must first know what the difference is between messages and events.
<!-- more -->
### Messages

When we look at distributed applications, messages have the following characteristics:

- A message contains raw data, produced by one component, that will be consumed by another component.
- A message contains the data itself, not just a reference to that data.
- The sending component expects the message content to be processed in a certain way by the destination component. 

### Events

Compared to messages which can have a considerable payload, events are lightweight. That also makes them suitable in broadcast situations. The component sending the event is the publisher. The receivers are known as subscribers.

Events have the following characteristics:

- An event is a lightweight notification, i.e. something happened.
- An event can be sent to multiple receivers, or none at all.
- Events are intended to have multiple subscribers so they can "fan out".
- The publisher of the event has no expectations of any subscriber.
- Events can be discrete and self contained.
- Events can also be related to other events.

## Choosing between messages and events

It's important to analyze the architecture and use cases of your application. For each communication you identify you should ask the following question: **Does the sender expect the communication to be processed in a particular way by the receiver?**

If the answer is *yes*, then choose messages. If the answer is *no*, you should be able to use events.
