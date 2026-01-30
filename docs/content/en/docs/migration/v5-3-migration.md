---
title: Migrating from v5.2 to v5.3
description: Migrating from v5.2 to v5.3
---

## ResourceOperations API replaces ReconcileUtils and PrimaryUpdateAndCacheUtils

The `ReconcileUtils` and `PrimaryUpdateAndCacheUtils` utility classes have been replaced with a new instance-based `ResourceOperations` API that provides a more intuitive and flexible way to interact with Kubernetes resources.

### What's Changed

The new `ResourceOperations<P>` class is instantiated with a `Context<P>` and provides instance methods instead of static utility methods. This change provides:

- **Better encapsulation**: The context is stored in the instance, eliminating the need to pass it to every method call
- **Cleaner API**: Instance methods are more discoverable and easier to use
- **Consistent patterns**: All resource operation methods follow the same pattern

### Migration Guide

#### Creating a ResourceOperations Instance

Create an instance of `ResourceOperations` from your reconciler's context:

```java
@Override
public UpdateControl<MyCustomResource> reconcile(
    MyCustomResource resource, Context<MyCustomResource> context) {

    var resourceOps = new ResourceOperations<>(context);
    // use resourceOps for resource operations
}
```

#### Finalizer Operations

**Before (v5.2 and earlier):**
```java
// Adding finalizer
ReconcileUtils.addFinalizer(context, finalizerName);

// Adding finalizer with SSA
ReconcileUtils.addFinalizerWithSSA(context, finalizerName);

// Removing finalizer
ReconcileUtils.removeFinalizer(context, finalizerName);
```

**After (v5.3):**
```java
var resourceOps = new ResourceOperations<>(context);

// Adding finalizer
resourceOps.addFinalizer(finalizerName);

// Adding finalizer with SSA
resourceOps.addFinalizerWithSSA(finalizerName);

// Removing finalizer
resourceOps.removeFinalizer(finalizerName);
```

#### Resource Patch Operations

**Before (v5.2 and earlier):**
```java
// Generic resource patch
ReconcileUtils.resourcePatch(context, resource, updateOperation);
```

**After (v5.3):**
```java
var resourceOps = new ResourceOperations<>(context);

// Generic resource patch
resourceOps.resourcePatch(context, resource, updateOperation);
```

#### Status Updates (replacing PrimaryUpdateAndCacheUtils)

The `PrimaryUpdateAndCacheUtils` class is deprecated in favor of `ResourceOperations`:

**Before (v5.2 and earlier):**
```java
// SSA patch status
var updatedResource = PrimaryUpdateAndCacheUtils.ssaPatchStatusAndCacheResource(
    resource, freshCopy, context);

// Non-SSA patch status
var updatedResource = PrimaryUpdateAndCacheUtils.patchStatusAndCacheResource(
    resource, statusUpdater, context);
```

**After (v5.3):**
```java
var resourceOps = new ResourceOperations<>(context);

// Server-side apply for primary resource status
var updatedResource = resourceOps.serverSideApplyPrimaryStatus(freshCopy);

// Update primary resource status with optimistic locking
resource.getStatus().setValue(newValue);
var updatedResource = resourceOps.updatePrimaryStatus(resource);

// JSON patch for primary resource status
var updatedResource = resourceOps.jsonPatchPrimaryStatus(resource, r -> {
    r.getStatus().setValue(newValue);
    return r;
});
```

### New ResourceOperations Methods

The `ResourceOperations` class provides comprehensive methods for all resource operation patterns:

**Server-Side Apply (SSA):**
- `serverSideApply(resource)` - SSA for any resource
- `serverSideApplyStatus(resource)` - SSA for resource status
- `serverSideApplyPrimary(resource)` - SSA for primary resource
- `serverSideApplyPrimaryStatus(resource)` - SSA for primary resource status

**Update with Optimistic Locking:**
- `update(resource)` - Update with optimistic locking
- `updateStatus(resource)` - Update status with optimistic locking
- `updatePrimary(resource)` - Update primary resource
- `updatePrimaryStatus(resource)` - Update primary resource status

**JSON Patch:**
- `jsonPatch(resource, unaryOperator)` - Apply JSON Patch
- `jsonPatchStatus(resource, unaryOperator)` - Apply JSON Patch to status
- `jsonPatchPrimary(resource, unaryOperator)` - Apply JSON Patch to primary
- `jsonPatchPrimaryStatus(resource, unaryOperator)` - Apply JSON Patch to primary status

**JSON Merge Patch:**
- `jsonMergePatch(resource)` - Apply JSON Merge Patch
- `jsonMergePatchStatus(resource)` - Apply JSON Merge Patch to status
- `jsonMergePatchPrimary(resource)` - Apply JSON Merge Patch to primary
- `jsonMergePatchPrimaryStatus(resource)` - Apply JSON Merge Patch to primary status

**Conflict Retrying Patch:**
- `conflictRetryingPatch(resourceChangesOperator, preCondition)` - Retry on conflict with precondition

All these methods automatically handle caching to ensure the next reconciliation sees the updated resource.

### Deprecation Notice

The following classes are deprecated and will be removed in a future version:
- `ReconcileUtils`
- `PrimaryUpdateAndCacheUtils`

Please migrate to `ResourceOperations` as soon as possible.

## Renamed JUnit Module

If you use JUnit extension in your test just rename it from:

```
<dependency>
      <groupId>io.javaoperatorsdk</groupId>
      <artifactId>operator-framework-junit-5</artifactId>
      <version>5.2.x<version>
      <scope>test</scope>
</dependency>
```

to

```
<dependency>
      <groupId>io.javaoperatorsdk</groupId>
      <artifactId>operator-framework-junit</artifactId>
      <version>5.3.0<version>
      <scope>test</scope>
</dependency>
```