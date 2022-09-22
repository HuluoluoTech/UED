How to use `AActor` or it's sub-class to `GetOverlappingComponents`
```c
TArray<CustomComponent*> components;
this->GetOverlappingComponents(components);
```
---

Component's Inheritance hierarchy:
`UActorComponent -> USceneComponent -> UPrimitiveComponent`

---

There're two methods that would get overlap component results in `AActor`'s class:
* `void GetOverlappingComponents(TArray<UPrimitiveComponent*>& OverlappingComponents) const;`
* `void GetOverlappingComponents(TSet<UPrimitiveComponent*>& OverlappingComponents) const;`
And we'll see that below method's definitions.

```c
// AActor:

/**
 * All ActorComponents owned by this Actor. Stored as a Set as actors may have a large number of components
 * @see GetComponents()
 */
// Use TSet !!!
TSet<UActorComponent*> OwnedComponents;

// Capture TArray as parameter
void AActor::GetOverlappingComponents(TArray<UPrimitiveComponent*>& OutOverlappingComponents) const
{
    // construct a TSet to call second method
	TSet<UPrimitiveComponent*> OverlappingComponents;
	GetOverlappingComponents(OverlappingComponents);

	OutOverlappingComponents.Reset(OverlappingComponents.Num());

	for (UPrimitiveComponent* OverlappingComponent : OverlappingComponents)
	{
		OutOverlappingComponents.Add(OverlappingComponent);
	}
}

void AActor::GetOverlappingComponents(TSet<UPrimitiveComponent*>& OutOverlappingComponents) const
{
	OutOverlappingComponents.Reset();
	TArray<UPrimitiveComponent*> OverlappingComponentsForCurrentComponent;

	for (UActorComponent* OwnedComp : OwnedComponents)
	{
        // cast ParentClass to SubClass
		if (UPrimitiveComponent* PrimComp = Cast<UPrimitiveComponent>(OwnedComp))
		{
			// get list of components from the component
			PrimComp->GetOverlappingComponents(OverlappingComponentsForCurrentComponent);

			OutOverlappingComponents.Reserve(OutOverlappingComponents.Num() + OverlappingComponentsForCurrentComponent.Num());

			// then merge it into our final list
			for (UPrimitiveComponent* OverlappingComponent : OverlappingComponentsForCurrentComponent)
			{
				OutOverlappingComponents.Add(OverlappingComponent);
			}
		}
	}
}

// UPrimitiveComponent:

/** Set of components that this component is currently overlapping. */
TArray<FOverlapInfo> OverlappingComponents;

void UPrimitiveComponent::GetOverlappingComponents(TArray<UPrimitiveComponent*>& OutOverlappingComponents) const
{
    // iff OverlappingComponents > 12, using Set
	if (OverlappingComponents.Num() <= 12)
	{
		// TArray with fewer elements is faster than using a set (and having to allocate it).
		OutOverlappingComponents.Reset(OverlappingComponents.Num());
		for (const FOverlapInfo& OtherOverlap : OverlappingComponents)
		{
			UPrimitiveComponent* const OtherComp = OtherOverlap.OverlapInfo.Component.Get();
			if (OtherComp)
			{
				OutOverlappingComponents.AddUnique(OtherComp);
			}
		}
	}
	else
	{
		// Fill set (unique)
		TSet<UPrimitiveComponent*> OverlapSet;
		GetOverlappingComponents(OverlapSet);
		
		// Copy to array
		OutOverlappingComponents.Reset(OverlapSet.Num());
		for (UPrimitiveComponent* OtherOverlap : OverlapSet)
		{
			OutOverlappingComponents.Add(OtherOverlap);
		}
	}
}

void UPrimitiveComponent::GetOverlappingComponents(TSet<UPrimitiveComponent*>& OutOverlappingComponents) const
{
	OutOverlappingComponents.Reset();
	OutOverlappingComponents.Reserve(OverlappingComponents.Num());

	for (const FOverlapInfo& OtherOverlap : OverlappingComponents)
	{
		UPrimitiveComponent* const OtherComp = OtherOverlap.OverlapInfo.Component.Get();
		if (OtherComp)
		{
			OutOverlappingComponents.Add(OtherComp);
		}
	}
}
```
