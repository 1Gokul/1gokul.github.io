---
layout: posts
title: "Weapon Sheathing and Drawing"
categories: "MedievalCombat"
---

## MedievalCombatProject

When the Player presses the **R** key, the Character changes into *Combat Mode*, which makes them ready for combat.

If the Character has a Weapon equipped, pressing **R** would toggle Sheathing and Drawing the Weapon.
If no Weapon was equipped, the Character would be ready for melee combat.

Sheathing and Drawing is done in two parts: the first being setting up the Character for combat and the second
being the attachment of the Weapon to the Hand/Sheath.

#### Sheathing

```cpp
void AMain::SheatheWeapon()
{
	bInCombatMode = false;
	bAttacking = false;


	if (bIsWeaponDrawn && CurrentWeapon)
	{
		bIsWeaponDrawn = false;

		// Play the Sheath sound
		if (CurrentWeapon->OnSheathSound)
		{
			UGameplayStatics::PlaySound2D(this, CurrentWeapon->OnSheathSound);
		}

		// Play the Sheath Animation
		UAnimInstance* AnimInstance = GetMesh()->GetAnimInstance();

		if (AnimInstance && UpperBodyMontage)
		{
			AnimInstance->Montage_Play(UpperBodyMontage, 1.0f);

			if (CurrentWeapon->bIsTwoHanded)
				AnimInstance->Montage_JumpToSection(FName("SheatheWeapon_TwoHanded"), UpperBodyMontage);

			else AnimInstance->Montage_JumpToSection(FName("SheatheWeapon_OneHanded"), UpperBodyMontage);
		}
	}
}
```

The above function sets up the Character for Sheathing the Weapon. Attaching the Weapon to the Sheath is done 
by a *BlueprintCallable* function, `TimedSheathe`:

```cpp
void AMain::TimedSheathe()
{
	if (CurrentWeapon)
	{
		CurrentWeapon->DeactivateCollision();

		CurrentWeapon->SetInstigator(nullptr);

		const USkeletalMeshSocket* SheathSocket = GetMesh()->GetSocketByName(CurrentWeapon->SheathSocketName);

		if (SheathSocket)
		{
			SheathSocket->AttachActor(CurrentWeapon, GetMesh());
		}
	}
}	
```

This function is called as an Anim Notify in the Sheathing Animation, at the moment where the Character is about to release the Weapon.

<img src = "/postassets/TimedSheathe.png"  style="border:5px solid black" alt="Sheathing One-Handed">

#### Drawing Setup

```cpp
void AMain::DrawWeapon()
{
	bInCombatMode = true;

	if (!bIsWeaponDrawn && CurrentWeapon)
	{
		bIsWeaponDrawn = true;

		// Play the Draw sound
		if (CurrentWeapon->OnEquipSound)
		{
			UGameplayStatics::PlaySound2D(this, CurrentWeapon->OnEquipSound);
		}

		// Play the Draw Animation
		UAnimInstance* AnimInstance = GetMesh()->GetAnimInstance();

		if (AnimInstance && UpperBodyMontage)
		{
			AnimInstance->Montage_Play(UpperBodyMontage, 1.0f);

			if (CurrentWeapon->bIsTwoHanded)
				AnimInstance->Montage_JumpToSection(FName("DrawWeapon_TwoHanded"), UpperBodyMontage);
			else AnimInstance->Montage_JumpToSection(FName("DrawWeapon_OneHanded"), UpperBodyMontage);
		}
	}
}
```
The above function sets up the Character for Drawing the Weapon. Attaching the Weapon to the Hand is done 
by a *BlueprintCallable* function, `TimedDraw`:

```cpp 
void AMain::TimedDraw()
{
	if (CurrentWeapon)
	{
		CurrentWeapon->SetInstigator(nullptr);

		const USkeletalMeshSocket* RightHandSocket = GetMesh()->GetSocketByName(CurrentWeapon->HandSocketName);

		if (RightHandSocket)
		{
			RightHandSocket->AttachActor(CurrentWeapon, GetMesh());
		}
	}
}
```
Similarly, this function is called as an Anim Notify in the Sheathing Animation, at the moment where the Character is about to grab the Weapon.

<img src = "/postassets/TimedDraw.png"  style="border:5px solid black" alt="Drawing One-Handed">

#### In Action 

- One-Handed Weapon
<iframe src="https://www.youtube.com/embed/U46rnurwsqs" width="560" height="315" frameborder="0"> </iframe> 

- Two-Handed Weapon
<iframe src="https://www.youtube.com/embed/Yf2b-jehppk" width="560" height="315" frameborder="0"> </iframe> 



