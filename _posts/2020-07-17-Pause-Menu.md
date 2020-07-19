---
layout: posts
title: "Pause Menu"
---

## MedievalCombatProject

The Pause menu is toggled by the **ESC** key and shows options to `Resume`, `Save`, `Load` and `Quit`.

The Animation in the Pause menu was done by making the Render Opacity of all items (buttons, text, background blur) in the menu
increase from `0.0` to `1.0` in the Widget Blueprint in 0.25 seconds.

<iframe src="https://www.youtube.com/embed/zvekPGgqofU" width="560" height="315" frameborder="0"> </iframe> 

**Resume** performs the same action as the ESC key when the Pause menu is open i.e. playing an Animation and removing the Pause menu 
widget from the screen.

**Save** Saves the game. Uses an `Actor` called `ItemStorage` to save the Character's current position, rotation, Weapon or Shield (if any),
Inventory(soon) and Stats. 

The game saves automatically once the Player enters a new map/level.

**Load** Uses the same `ItemStorage` to load the last save. I plan to add a list of the player's last 5 saves to the menu in case
the Player wishes to make multiple saves.

#### Saving and Loading

- Game saves on changing level
<iframe src="https://www.youtube.com/embed/gR0BdHFScvQ" width="560" height="315" frameborder="0"> </iframe> 

- Saving and Loading
<iframe src="https://www.youtube.com/embed/rdjxNS-oRhQ" width="560" height="315" frameborder="0"> </iframe> 