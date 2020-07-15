---
layout: posts
title: "Character Idle Animations"
---

## MedievalCombatProject

If the Player does not perform any action on the Character for an amount of time(taken as 10 seconds), 
a C++ function "IdleEnd" gets called.

This function chooses one random number from [0 -> NumberOfAnimations - 1] and assigns this value to 
a variable called IdleAnimSlot. NumberOfAnimations is unique for every Player State (i.e UnarmedNormal, Armed-OneHanded).

```cpp 

/**
 * Every time after an Idle Animation plays, the base Idle Animation should be played.
 * After the base Idle Animation is complete, one of the other Idle Animations will be randomly chosen.
 */
void AMain::IdleEnd(int32 PlayerStatusNo)
{
  // If Character was in Original Idle state, play an Idle Animation.
	if (IdleAnimSlot == 0)
	{                               // [0 -> NumberOfIdleAnims - 1]
		IdleAnimSlot = FMath::RandRange(1, NumberOfIdleAnims[PlayerStatusNo] - 1);
	}

  // else return back to the Original Idle state.
	else IdleAnimSlot = 0;
}

```

In the Character's Anim Blueprint,  out of a set of Idle Animations, one will be chosen at random and will play on the Character.

Once that Idle Animation finishes playing, it will return to the original Idle State and will wait again 
for the same amount of time (10 seconds), playing another Idle Animation if the Player does not do anything even then.

### In Action 

- While Unarmed and not in Combat Mode
<iframe src="https://www.youtube.com/embed/tc39nbjUEmc" width="560" height="315" frameborder="0"> </iframe> 

- While Unarmed and in Combat Mode
<iframe src="https://www.youtube.com/embed/hr8tECcHG1k" width="560" height="315" frameborder="0"> </iframe> 

- While Armed with a One-Handed Weapon
<iframe src="https://www.youtube.com/embed/KqrbYdRlxcw" width="560" height="315" frameborder="0"> </iframe> 

- While Armed with a Two-Handed Weapon (Bug: Character rises above the ground. Will be patched soon.)
<iframe src="https://www.youtube.com/embed/nVskVluF3to" width="560" height="315" frameborder="0"> </iframe> 



