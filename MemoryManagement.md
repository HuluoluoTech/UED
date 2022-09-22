# Memory management systems in Unreal Engine
* Garbage Collection, 
* Smart Pointers, 
* Standard C++ Memory Management.

## Garbage Collection
Garbage collection (GC) tracks UObject(s) and its sub-classes, which include AActor and UActorComponent.
When creating a new UObject, UE4 will automatically add them to its internal objects list, so even with improper use, it's not easy to have memory leaks, but it is easy to cause crashes.
**UObjects should never be created with new**, but only with the default creation methods (NewObject, SpawnActor, CreateDefaultSubobject)
### Objects are primarily kept alive in 3 ways
* By having a strong reference (**UPROPERTY**) to them (from objects that are also referenced)
* By calling UObject::AddReferencedObjects (from objects that are also referenced)
* By adding them to the root set with UObject::AddToRoot (typically unnecessary)
#### destroyed Time
When objects do not fulfill any of the above conditions, on **the next GC cycle** they will be marked as unreachable and garbage collected (destroyed).
Passing an object as an Outer to another object does not automatically mean the other object will be kept alive, the same goes for default subobjects.

To force the destruction of objects that are still reachable, you can call **MarkPendingKill** or **MarkAsGarbage** on them, and it will force their destruction on the next GC cycle (you generally want to avoid doing this, as that is what garbage collection is for, and some classes, like AActor and UActorComponent do not directly support it).
The destruction of an object doesn't necessarily all happen in the same frame, when garbage collection starts on it, it will first call **BeginDestroy** (do not call this yourself), then, when ready **FinishDestroy**.

**The GC runs in the game thread** so you can trust that it won't clean up an object within the lifetime of a function.

While the most common way of keeping objects alive is through a UPROPERTY, actors and components work differently:
**Levels reference their actors and actors reference their components.** Both work by overriding the **UObject::AddReferencedObjects** implementation and collecting what they do not want to be garbage collected. This means that even if there are no strong references to level actors and components, they won't be garbage collected until manually destroyed, or their level is unloaded.
```c
// AActor
TSet<UActorComponent*> OwnedComponents;
```
### The garbage collector will automatically clear the following references to garbage collected objects:
* Raw pointers declared with UPROPERTY - will be set to nullptr
* Raw pointers in UObject compatible containers declared with UPROPERTY (such as TArray, TSet or TMap) - the affected elements will be set as nullptr but not removed

Whenever code references an AActor or a UActorComponent, it has to deal with the possibility that AActor::Destroy could be called on the actor or UActorComponent::DestroyComponent could be called on the component. These functions will mark them for pending kill, thus triggering their garbage collection at the first opportunity (note that destroying an actor also destroys all its components). Since the garbage collector automatically nulls out UPROPERTY pointers when it actually gets to destroy them, null-checking an actor or component pointer is sufficient to know it's safe to use, though you might also want to check IsPendingKill on them (through IsValid) to avoid accessing them after they have been marked for destruction (TWeakObjectPtr already checks for this when retrieving the raw pointer).

```c
FORCEINLINE_DEBUGGABLE bool UKismetSystemLibrary::IsValid(const UObject* Object)
{
	return ::IsValid(Object);
}

/**
* Test validity of object
*
* @param	Test			The object to test
* @return	Return true if the object is usable: non-null and not pending kill or garbage
*/
FORCEINLINE bool IsValid(const UObject *Test) 
{
	return Test && FInternalUObjectBaseUtilityIsValidFlagsChecker::CheckObjectValidBasedOnItsFlags(Test);
}

struct FInternalUObjectBaseUtilityIsValidFlagsChecker
{
	FORCEINLINE static bool CheckObjectValidBasedOnItsFlags(const UObject* Test)
	{
		// Here we don't really check if the flags match but if the end result is the same
		PRAGMA_DISABLE_DEPRECATION_WARNINGS
		checkSlow(GUObjectArray.IndexToObject(Test->InternalIndex)->HasAnyFlags(EInternalObjectFlags::PendingKill | EInternalObjectFlags::Garbage) == Test->HasAnyFlags(RF_InternalPendingKill | RF_InternalGarbage));
		PRAGMA_ENABLE_DEPRECATION_WARNINGS
		return !Test->HasAnyFlags(RF_InternalPendingKill | RF_InternalGarbage);
	}
};

```
Should not be used for Garbage Collection checks, as on UPROPERTY pointers it will always return true, while on raw pointer it will return true or false depending whether the object had already been destroyed, but in the latter case it's also likely to also crash the application as the pointed memory could have been overwritten.

If you write your own non-garbage classes that references garbage collected objects, you may want to sub-class **FGCObject**.

## Unreal Smart Pointer Library
The Unreal Smart Pointer Library (**TUniquePtr**, **TSharedPtr**, **TWeakPtr**) is for code that is not based on the UObject system. It is similar in function to the C++11 standard library smart pointers. **Unreal Smart Pointers cannot be used to reference UObjects, because the garbage collector isn't aware of smart pointers**.

## Further Reading

* [Objects](https://docs.unrealengine.com/en-US/Programming/UnrealArchitecture/Objects/index.html) - Explanations of the basic gameplay elements, Actors and Objects

* [Unreal Object Handling](https://docs.unrealengine.com/en-US/Programming/UnrealArchitecture/Objects/Optimizations/index.html) - Overview of the features of the UObject system

* [Unreal Smart Pointer Library](https://docs.unrealengine.com/en-US/Programming/UnrealArchitecture/SmartPointerLibrary/index.html) - UE4 implementation of smart pointers, including weak pointers and non-nullable shared references
`
## Reference
* [Memory Management
](https://unrealcommunity.wiki/memory-management-6rlf3v4i)
