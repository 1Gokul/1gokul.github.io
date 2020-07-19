---
layout: posts
title: "Character Movement and HUD"
---


## MedievalCombatProject

### HUD
HUD for the Player. Has a Health and Stamina bar (Magicka will be added soon) and a Coin count(Will be moved to Inventory Menu).

The Health bar will will show changes when the Character takes damage or heals themselves.

The Stamina bar shows changes when the Character sprints or successively blocks an Enemy's attack.

<img src = "/postassets/Basic HUD.png"  style="border:5px solid black">

<!-- <blockquote class="imgur-embed-pub" lang="en" data-id="SFPNkGe"><a href="https://imgur.com/SFPNkGe">View post on imgur.com</a></blockquote><script async src="//s.imgur.com/min/embed.js" charset="utf-8"></script> -->

### Movement

Movement of the Character uses a 1D BlendSpace for gradually changing from Idle to Walk to Run Animations, 
    depending on the speed of the Character.

A function `bCanMove` determines if the Character is allowed to move or not. The `MoveForward` and `MoveRight` functions execute
their contents only when `bCanMove` returns `true`.

```cpp 
bool AMain::bCanMove(float Value)
{
	if (MainPlayerController)
	{
		return (
			(Value != 0.0f)		// If Value to move is 0
			&& (!bAttacking)	// If Attacking
			&& (MovementStatus != EMovementStatus::EMS_Dead)	// If not Dead
			&& (!MainPlayerController->bPauseMenuVisible)		// If Pause Menu Visible
			&& (!MainPlayerController->bInventoryMenuVisible)	// If Inventory Menu Visible
		);
	}
	return false;
}
```

#### In Action 

- While Unarmed and not in Combat Mode
<iframe src="https://www.youtube.com/embed/vVXSP_3Dkjk" width="560" height="315" frameborder="0"> </iframe> 

- While Unarmed and in Combat Mode
<iframe src="https://www.youtube.com/embed/DEbCmu_ghQM" width="560" height="315" frameborder="0"> </iframe> 

- While Armed with a One-Handed Weapon
<iframe src="https://www.youtube.com/embed/pOGELCzg5UA" width="560" height="315" frameborder="0"> </iframe> 

- While Armed with a Two-Handed Weapon (Bug: Character rises above the ground. Will be patched soon.)
<iframe src="https://www.youtube.com/embed/jpWaXgGB_vg" width="560" height="315" frameborder="0"> </iframe> 


### Sprinting

When the Player presses the Shift key while the Character is moving, the Character will start sprinting. This sprinting 
lets the Character move faster at the cost of using up Stamina by an amount `StaminaDrainRate` every second. 

Once the Stamina gets depleted, the Stamina Bar will turn red and the Character will not be able to sprint till the Stamina regenerates to a value greater than
`MinSprintStamina`, ater which the Stamina bar turns green again. I plan to add a pulsing effect soon.

#### In Action

- Sprinting while Unarmed
<iframe src="https://www.youtube.com/embed/P1oTZ67NyiY" width="560" height="315" frameborder="0"> </iframe> 

- Sprinting while Armed
<iframe src="https://www.youtube.com/embed/gPQChIK7Wo0" width="560" height="315" frameborder="0"> </iframe> 

