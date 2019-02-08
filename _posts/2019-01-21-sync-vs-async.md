---
layout: post
title: "Sync vs. Async example"
date: 2019-01-21
---

Chess master Judit Polgár hosts a chess exhibition in which she plays multiple amateur players. She has two ways of conducting the exhibition: synchronously and asynchronously.

Assumptions:

  * 24 opponents
  * Judit makes each chess move in 5 seconds
  * Opponents each take 55 seconds to make a move
  * Games average 30 pair-moves (60 moves total)

**Synchronous version**: Judit plays one game at a time, never two at the same time, until the game is complete. Each game takes (55 + 5) * 30 == 1800 seconds, or 30 minutes. The entire exhibition takes 24 * 30 == 720 minutes, or 12 hours.

**Asynchronous version**: Judit moves from table to table, making one move at each table. She leaves the table and lets the opponent make their next move during the wait time. One move on all 24 games takes Judit 24 * 5 == 120 seconds, or 2 minutes. The entire exhibition is now cut down to 120 * 30 == 3600 seconds, or just 1 hour. ([Source](https://www.youtube.com/watch?v=iG6fr81xHKA&feature=youtu.be&t=4m29s))