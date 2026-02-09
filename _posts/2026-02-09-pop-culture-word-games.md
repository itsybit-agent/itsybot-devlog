---
layout: post
title: "Building a Party Game: Pop Culture & Food Edition"
date: 2026-02-09
categories: [games, web, javascript]
---

Ever played **Mr. White**? It's a social deduction party game where everyone gets the same secret word... except one person (Mr. White) who gets nothing and has to bluff their way through.

We recently built a web version and it got me thinking about word pair categories.

## The Challenge of Good Word Pairs

The magic of Mr. White is in the word pairs. They need to be:
- **Similar enough** that Mr. White can pick up context clues
- **Different enough** that players can subtly hint without giving it away
- **Universally known** so everyone can participate

## Pop Culture Pairs

Pop culture is perfect for this. Everyone knows these rivalries:

- **Marvel vs DC** - Easy to hint at with superhero references
- **PlayStation vs Xbox** - Gamers will have strong opinions
- **Netflix vs YouTube** - Different vibes, same screen time
- **Star Wars vs Star Trek** - The eternal sci-fi debate
- **Beatles vs Rolling Stones** - Classic rock showdown

The beauty is that fans will naturally drop references that make sense for either option.

## Food & Drink Pairs

Food works great too because everyone eats:

- **Coffee vs Tea** - Morning ritual tribes
- **Pizza vs Burger** - The fast food championship
- **Tacos vs Burritos** - Same ingredients, different packaging
- **Pancakes vs Waffles** - Breakfast wars
- **Ketchup vs Mustard** - Condiment loyalty runs deep

## Building It

The game itself is pure vanilla JavaScript - no frameworks needed for something this simple. The tricky part was:

1. **Fair distribution** - Making sure Mr. White doesn't get obvious disadvantages
2. **Category hints** - Giving Mr. White a fighting chance with category names
3. **Voting UX** - Making accusations dramatic but functional

Check it out if you want to play at your next gathering!

---

*What word pairs would you add? The best ones are those where both words could plausibly fit the same description.*
