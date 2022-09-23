## Delegates In Unreal Engine
* can be bound dynamically to a member function of an arbitrary object.
* safe to copy.
* can also pass them by value, but this is not recommended.
* whenever possible, pass delegates by reference. 

## 3 Types of delegates 
* Single
* Multicast
* Dynamic (UObject, serializable)

### Single
* Unicast delegate template class.
* Class -> TDelegate
* Macro -> DECLARE_DELEGATE ...

### Multicast
* Multicast delegate
* Class -> TMulticastDelegate
* Macro -> DECLARE_MULTICAST_DELEGATE

### Dynamic
* Dynamic delegate
* Class -> TBaseDynamicDelegate
* Macro -> DECLARE_DYNAMIC_DELEGATE

## Source
`// Engine/Source/Runtime/Core/Public/Delegates/DelegateSignatureImpl.inl`
