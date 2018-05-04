# AlloyPerms
A new approach to permissions. Permissions are no longer pure! They are now alloys!

## How does this work?
There are different layers to determine the permission for a permissible. The layers are applied from top to bottom. If a layer reports a non-null value, the lower layers will not be evaluated.

The effective value type `T` of the permission can be boolean, int, float or string, or a set of one of the above (A set refers to an unordered collection of values without duplication). In PHP, the set is represented by an associative array, where the set entries are in the keys and the all values are set to `true`.

- Particular permission predicates
  - Plugins can register permission predicates (`interface function test(Permissible) : ?T`) for a particular permission. If a non-null value is returned, later predicates are not evaluated.
  - Permission predicates should be registered once the plugin is enabled, not per permissible.
  - Due to performance concerns, permission attachments should be preferred to permission predicates if possible, because permission attachments can be pre-calculated.
- Particular permission attachments
  - Permission attachments are applied according to a descending priority index. The attachment carries a constant `t` (`T | null`)  with an inheritance mode:
    - `default`: Reports the result from previous attachments if it is not null. Reports `t` otherwise. It does not make sense to report `t=null` with `default` inheritance mode, because the permission attachment is functionally useless.
    - `force`: Reports `t` regardless the result from previous attachments. If `t` is null, this is equivalent to cancelling the effect of all previous inheritance.
  - Permission attachments are bound to a specific permissible. The result of this layer is recalculated lazily when attachments are changed and a new value is requested. This means plugins should bind permission attachments when:
    - a Permissible is constructed (PermissibleCreationEvent). This may not be equivalent to PlayerLoginEvent for players, because permissions may also be requested for offline players.
    - a new Permission is registered (PermissionRegisterEvent). Permission manager plugin should bind attachments to the new permission for all existing Permissibles.
    - PermissibleCreationEvent is fired asynchronously, while PermissionRegisterEvent is not. Therefore, in the initial PermissibleCreationEvent, permission manager plugins should asynchronously prepare for **all** possible permissions, including both existing and potential permissions, then apply them synchronously in PermissionRegisterEvent.
- Group permission predicates
  - Plugins can register permission predicates for permissions under a particular subtree. If a non-null value is returned, later predicates are not evaluated.
- Parent permission attachments
  - If the permission has a parent, and the permission for the parent is not null, the parent value will be reported.

If all layers report null, the permission default (e.g. `true`, `op`) will be used. This default will **not** affect child permissions.

## How is this different?
### Permissions have multiple types.
Permissions are no longer limited to booleans. They can also be an integer, a float or a string, or a set of integers, or a set of floats, or a set of strings. (It does not make sense to create a set of booleans, because there are only two boolean values)

### ~~Permission parent default values will not affect children.~~
## The PocketMine permission system is so broken that I don't know what it was intended to do!
