---
layout: post
title:  "Byzantine Generals Problem"
date:   2018-01-12 10:00:28 -0500
categories: jekyll update
---

The Byzantine Generals Problem asks how a commanding general sends an order to his n-1 lieutenant generals such that,
1, All loyal lieutenants obey the same order
2, If the commanding general is loyal, then every loyal lieutenant obeys the order he sends.

Surprisingly, there's no solution to this problem unless two thirds of the generals are loyal. In paritcular, if there's one traitor in three generals, there's no solution.

The paper then discussed the solutions to the BGP in two settings, oral message (unauthenticated) setting and written message setting (authenticated message). 

In OM setting, it assumes that A1, every message that is sent is delivered correctly (no forgery), A2, The receiver of a message knows who sent it and A3, the absence of a message can be detected.
