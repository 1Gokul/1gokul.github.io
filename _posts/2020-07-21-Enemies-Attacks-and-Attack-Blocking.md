---
title: "Enemies, Enemy Attacks and Attack Blocking"
categories: "MedievalCombat"
---

## MedievalCombatProject

For now, the Player will face Enemies of only one type - Minion. These Minion assets were sourced from the **Paragon: Minions** Asset Pack
from the Epic Games Store. I will be adding more Enemies very soon from the **Infinity Blade: Adversaries** Asset Pack. 

An Enemy's Collision Volumes are as below:

<img src = "/postassets/MinionSpheres.png"  style="border:5px solid black">

All such Collision Volumes use the `OnComponentBeginOverlap` and `OnComponentEndOverlap` functions to perform actions if an overlap occurs.

- AgroSphere - If the Character enters this sphere, the Enemy will get agressive and chase the Character. 
Chasing is done with UE4's inbuilt MoveToTarget() function, using which the Enemy will follow a path to reach the Character's location. This
functionality only works inside NavMeshBounds Volumes.

- CombatSphere - When the Enemy reaches close enough to the Character to make it overlap with the CombatSphere, the Enemy will begin attacking.

Additional functionality added is the ability for the Enemy to turn so that attacks hit the Character i.e. if the Character moves when 
it is attacking, it will turn so that it can look directly at the Character.

<img src = "/postassets/MinionCombatCollisions.png"  style="border:5px solid black">

- LeftCombatCollision & RightCombatCollision - Used to check if the Enemy's attack has hit the Character or not. They are box collisions covering
the swords of the Enemies completely. By default, they do not respond to any types of overlaps. They are only activated to register overlaps when the
Enemy performs an attack. 

Attack Animations for Enemies are kept together in an Animation Montage. Out of the set of Animations, one is chosen at random.

#### Attack()
```cpp
void AEnemy::Attack()
{
	if (Alive() && bHasValidTarget)
	{
		if (AIController)
		{
			// As Character is overlapping CombatSphere, there is no need to move
			AIController->StopMovement();
			SetEnemyMovementStatus(EEnemyMovementStatus::EMS_Attacking);
		}
		if (!bAttacking)
		{
			// Turn towards Character
			SetInterpToEnemy(true);

			bAttacking = true;
			UAnimInstance* AnimInstance = GetMesh()->GetAnimInstance();

			if (AnimInstance)
			{
				//Randomly choose between the 4 attack animations
				AttackSection = FMath::RandRange(1, 4);

				// Append the AttackSection to the FString below
				FString AttackName("Attack_");
				AttackName.AppendInt(AttackSection);

				AnimInstance->Montage_Play(CombatMontage, 1.0f);
				AnimInstance->Montage_JumpToSection(FName(*AttackName), CombatMontage);
			}
		}
	}
}
```

If the CombatCollision overlaps with the Character, a check will be performed to see if the Character was blocking.

### If not blocking

If the Character was not blocking, a function `InflictDamageOnMain` is called. This function uses UE4's ApplyDamage() function to decrease the 
Character's health, along with calling functions to emit blood particles and playing hit sounds.

The hit reaction animations are not picked at random, but are related to the `AttackSection` shown above AND if the Character is facing the
attacking Enemy. As a result, the appropriate animation plays in reaction to the Enemy's attack.

#### InflictDamageOnMain()
```cpp
void AEnemy::InflictDamageOnMain(AMain* Main, bool bHitFromBehind)
{
	// If Character was hit from behind
	if (bHitFromBehind)
	{
		// Play HitFromBehind Animation
		Main->Impact(1);

		/**
		 * First 2 Attack Sockets in Main's SocketNames are the names of the Sockets that will be used
		 * if the Player gets hit by the Enemy facing them. By adding 4, we get the Sockets that will be used if
		 * the Player gets hit by the Enemy facing their back. Used below to use the appropriate Socket to emit the blood.
		 */
		AttackSection += 4;
	}

	// If Character was hit while facing the attacking Enemy
	else
	{
		// Play appropriate Hit Reaction Animation
		Main->Impact(AttackSection + 1);
	}

	// Spawn Blood particles
	if (Main->HitParticles)
	{
		Main->SpawnHitParticles(this);
	}

	// Play Hit Sound
	if (Main->HitSound)
	{
		UGameplayStatics::PlaySound2D(this, Main->HitSound);
	}

	// Apply Damage to Main
	if (DamageTypeClass)
	{
		UGameplayStatics::ApplyDamage(Main, Damage, AIController, this, DamageTypeClass);
	}

	//In case the Player's attack was interrupted
	Main->bAttacking = false;

	Main->bInterpToEnemy = false;
}
```

#### In Action

- Character hit reaction when not blocking
<iframe src="https://www.youtube.com/embed/M6Bvj2696RA" width="560" height="315" frameborder="0"> </iframe>

### If blocking

Performed using the Right Mouse Button(RMB). The Character can block attacks using a One-Handed Weapon AND a Shield OR with a Two-Handed Weapon. 
One-Handed Weapons alone cannot be used for blocking.

The Character can walk while blocking. This is done by using UE4's `Layered Blend per bone` node, done before in the
[Crouching](https://1gokul.github.io/2020-07-15/Character-Crouching/) post.

#### Walking while Blocking: 

- Blocking with Shield
<iframe src="https://www.youtube.com/embed/k9Bv3js7mxU" width="560" height="315" frameborder="0"> </iframe>

- Blocking with Two-Handed Weapon
<iframe src="https://www.youtube.com/embed/P1oTZ67NyiY" width="560" height="315" frameborder="0"> </iframe>

To successfully block an Enemy's attack, the Character has to be blocking AND has to be facing the attacking Enemy,
while also having sufficient Stamina available.

Facing an Enemy means that the Enemy should be inside the Character's Field of View (taken as 120°).

<img src = "/postassets/CharacterFOV.png"  style="border:5px solid black">

The code snippet below shows how this is done: (Note: The Character's name is "Main")

#### CombatOnOverlapBegin()
```cpp
void AEnemy::CombatOnOverlapBegin(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor,
                                  UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep,
                                  const FHitResult& SweepResult)
{
	// If OtherActor is valid
	if (OtherActor)
	{
		AMain* Main = Cast<AMain>(OtherActor);

		// If the OtherActor is the Character
		if (Main)
		{
			//Check if Character is facing the Enemy
			FVector MainLocation = Main->GetActorLocation();
			FVector EnemyLocation = GetActorLocation();

			FRotator LookAtRotation = UKismetMathLibrary::FindLookAtRotation(MainLocation, EnemyLocation);

			FRotator MainRotation = Main->GetActorRotation();

			FRotator DeltaRotator = UKismetMathLibrary::NormalizedDeltaRotator(MainRotation, LookAtRotation);

			// Acceptable range of rotation = -180 -> -60 degrees OR 60 -> 180 degrees.
			// i.e. Character's field of view = 120 degrees
			bool bAngleCheck1 = UKismetMathLibrary::InRange_FloatFloat(DeltaRotator.Yaw, -180, -60);
			bool bAngleCheck2 = UKismetMathLibrary::InRange_FloatFloat(DeltaRotator.Yaw, 60, 180);

			// If Character is Blocking
			if (Main->bBlocking)
			{
				//If facing the Enemy
				if (!(bAngleCheck1 || bAngleCheck2))
				{
					//If Character is blocking with shield and has enough stamina to successfully block an attack  
					if (Main->EquippedShield && Main->Stamina - Main->EquippedShield->BlockStaminaCost >= 0)
					{
						// Add Blocking particle effects and Sound
						// Decrease Stamina by BlockStaminaCost
					}

						//If Character is blocking with weapon and has enough stamina to successfully block an attack
					else if (Main->bIsWeaponDrawn && Main->Stamina - Main->EquippedWeapon->BlockStaminaCost >= 0)
					{
						//Weapons don't block attacks completely, so decrease Health by DamageIfBlocked
						// Add Blocking particle effects and Sound
						// Decrease Stamina by BlockStaminaCost
					}

					//If Character does not have enough stamina to successfully block an attack
					else
					{
						InflictDamageOnMain(Main, false);
					}
				}
				else
				{
					//If not facing the Enemy
					InflictDamageOnMain(Main, true);
				}
			}

			else
			{	//If not blocking
				InflictDamageOnMain(Main, (bAngleCheck1 || bAngleCheck2));
			}
		}
	}
}

```

If the attack was blocked, the Character's Stamina will reduce by `BlockStaminaCost`(declared in the Weapon and Shield classes).

Additionally, if a Two-Handed Weapon was used for blocking, the Character's Health will decrease by an amount `DamageIfBlocked`, 
as Weapons cannot block as effectively as Shields.

You can view the code of the project [here](https://github.com/1Gokul/MedievalCombatProject)!

### Blocking in Action 

- One-Handed Weapon + Shield
<iframe src="https://www.youtube.com/embed/J_KpgKHG55c" width="560" height="315" frameborder="0"> </iframe> 

- Two-Handed Weapon
<iframe src="https://www.youtube.com/embed/Lrt6pQSxiIQ" width="560" height="315" frameborder="0"> </iframe> 



