---
layout: post
title:  "Messaging in Azure"
date:   2020-02-17 20:56:00 +0100
categories: azure messaging
---

## Messages or Events

Azure offers the following services for handling messages and events:

- Event Grid
- Event Hubs
- Service Bus

At first sight it can be difficult to choose the right service for your scenario. To better determine what service to use, you must first know what the difference is between messages and events.

### Messages

When we look at distributed applications, messages have the following characteristics:

- A message contains raw data, produced by one component, that will be consumed by another component.
- A message contains the data itself, not just a reference to that data.
- The sending component expects the message content to be processed in a certain way by the destination component.

An example of a messaging scenario is a customer ordering something from a webshop, and the webshop sending an invoice and a shipment message. 

The order contains the items the customer wants to buy and where he or she wants it delivered. The invoice contains information about the amount of items, the total price and VAT. The shipment message contains information about the expected delivery date and tracking information.

```mermaid
sequenceDiagram
    participant C as Customer
    participant S as Webshop
    C->>S: Order
    S->>C: Invoice
    S->>C: Shipment
```

### Events

Compared to messages which can seem bulky, events are lightweight. That also makes them suitable in broadcast situations. The component sending the event is the publisher. The receivers are known as subscribers.

Events have the following characteristics:

- An event is a lightweight notification, i.e. something happened.
- An event can be sent to multiple receivers, or none at all.
- Events are intended to have multiple subscribers so they can "fan out".
- The publisher of the event has no expectations of any subscriber.
- Events can be discrete and self contained.
- Events can also be related to other events.

An example scenario using events is an app of a sportsclub that notifies its users when a goal has been made in a match.

```mermaid
graph LR
    F[FC Azure]-.GOAL!.->A[Alice] & B[Bob] & D[Dick]
```

The `goal!` event describes what has taken place and is "fanned out" to the subscribers Alice, Bob and Dick. The sportsclub "FC Azure" does not expect anything in return.
