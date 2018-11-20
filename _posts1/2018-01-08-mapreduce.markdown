---
layout: post
title:  "Preventing Leakage in MapReduce"
date:   2018-01-08 10:00:28 -0500
categories: jekyll update
---

Let's say Mark and Morgan are the mappers, Rich is the reducer, Smith is the data shuffling node. Diamond is the poker card Mark holds, Heart is the poker card Morgan holds. 

Without shuffle-in-the-middle (SITM), the attacker sees a card handed from Mark to Rich. Because the attacker knows Rich handle all the Diamond cards, the attacker knows Mark has a Diamond card, which is bad.  

With SITM, the attacker sees both cards go to Smith, Smith then shuffles the card in an unobservable way and handed in one of them to Rich. Now, the attacker knows that card is a Diamond, but the attacker cannot know it comes from Mark or Morgan.

The left problem is how Smith shuffles the card in an unnoticed way. The paper uses Melbourne shuffle. And implement Melbourne shuffle using MapReduce. In particular, distribute is a map, and cleanup is a reduce. 

The left problem is then why MapReduce-version Melbourne shuffle is secure. One intuition is that to label the message, the key distribution needs some variation. For example, apple:orange = 4:1.  If the intermediate keys distribution is even, then the label will not be a success. 
