# Livewire Bootstrap Modal

Livewire Bootstrap Modal is a component that provides you with a modal that supports multiple child modals while maintaining state.

## Livewire directive

Add the Livewire directive `@livewire('livewire-bootstrap-modal')` and also the Javascript to your template.

```html
<html>
<head>
    ...
    @livewireStyles
</head>
<body>
    ...
    <script src="/vendor/livewire-bootstrap-modal/scripts.js"></script>
    @livewire('livewire-bootstrap-modal')
    @livewireScripts
</body>
```

## Alpine

Livewire Bootstrap Modal requires [Alpine](https://github.com/alpinejs/alpine) and the plugin [Focus](https://alpinejs.dev/plugins/focus). You can use the official CDN to quickly include Alpine:

```html
<!-- Alpine v3 -->
<script defer src="https://unpkg.com/alpinejs@3.x.x/dist/cdn.min.js"></script>

<!-- Focus plugin -->
<script defer src="https://unpkg.com/@alpinejs/focus@3.x.x/dist/cdn.min.js"></script>
```

## Creating a modal

You can run `php artisan make:livewire EditUser` to make the initial Livewire component. Open your component class and make sure it extends the `ModalComponent` class:

```php
<?php

namespace App\Http\Livewire;

use VenusSoft\Livewire\BootstrapModal\ModalComponent;

class EditUser extends ModalComponent
{
    public function render()
    {
        return view('livewire.edit-user');
    }
}
```

## Opening a modal

To open a modal you will need to emit an event. To open the `EditUser` modal for example:

```html
<!-- Outside of any Livewire component -->
<button onclick="Livewire.emit('openModal', 'edit-user')">Edit User</button>

<!-- Inside existing Livewire component -->
<button wire:click="$emit('openModal', 'edit-user')">Edit User</button>

<!-- Taking namespace into account for component Admin/Actions/EditUser -->
<button wire:click="$emit('openModal', 'admin.actions.edit-user')">Edit User</button>
```

## Passing parameters

To open the `EditUser` modal for a specific user we can pass the user id (notice the single quotes):

```html
<!-- Outside of any Livewire component -->
<button onclick='Livewire.emit("openModal", "edit-user", {{ json_encode(["user" => $user->id]) }})'>Edit User</button>

<!-- Inside existing Livewire component -->
<button wire:click='$emit("openModal", "edit-user", {{ json_encode(["user" => $user->id]) }})'>Edit User</button>

<!-- Example of passing multiple parameters -->
<button wire:click='$emit("openModal", "edit-user", {{ json_encode([$user->id, $isAdmin]) }})'>Edit User</button>
```

The parameters are passed to the `mount` method on the modal component:

```php
<?php

namespace App\Http\Livewire;

use App\Models\User;
use VenusSoft\Livewire\BootstrapModal\ModalComponent;

class EditUser extends ModalComponent
{
    public User $user;

    public function mount(User $user)
    {
        Gate::authorize('update', $user);
        
        $this->user = $user;
    }

    public function render()
    {
        return view('livewire.edit-user');
    }
}
```

## Opening a child modal

From an existing modal you can use the exact same event and a child modal will be created:

```html
<!-- Edit User Modal -->

<!-- Edit Form -->

<button wire:click='$emit("openModal", "delete-user", {{ json_encode(["user" => $user->id]) }})'>Delete User</button>
```

## Closing a (child) modal

If for example a user clicks the 'Delete' button which will open a confirm dialog, you can cancel the deletion and return to the edit user modal by emitting the `closeModal` event. This will open the previous modal. If there is no previous modal the entire modal component is closed and the state will be reset.

```html
<button wire:click="$emit('closeModal')">No, do not delete</button>
```

You can also close a modal from within your modal component class:

```php
<?php

namespace App\Http\Livewire;

use App\Models\User;
use VenusSoft\Livewire\BootstrapModal\ModalComponent;

class EditUser extends ModalComponent
{
    public User $user;

    public function mount(User $user)   
    {
        Gate::authorize('update', $user);
        
        $this->user = $user;
    }

    public function update()
    {
        Gate::authorize('update', $user);
            
        $this->user->update($data);

        $this->closeModal();
    }

    public function render()
    {
        return view('livewire.edit-user');
    }
}
```

If you don't want to go to the previous modal but close the entire modal component you can use the `forceClose` method:

```php
public function update()
{
    Gate::authorize('update', $user);
    
    $this->user->update($data);

    $this->forceClose()->closeModal();
}
```

Often you will want to update other Livewire components when changes have been made. For example, the user overview when a user is updated. You can use the `closeModalWithEvents` method to achieve this.

```php
public function update()
{
    Gate::authorize('update', $user);
    
    $this->user->update($data);

    $this->closeModalWithEvents([
        UserOverview::getName() => 'userModified',
    ]);
}
```

It's also possible to add parameters to your events:

```php
public function update()
{
    $this->user->update($data);

    $this->closeModalWithEvents([
        UserOverview::getName() => ['userModified', [$this->user->id],
    ]);
}
```

## Changing modal properties

You can change the classes of the modal by overriding the static `modalClasses` method in your modal component class:

```php
public static function modalClasses(): string
{
    return 'your-css-class-here';
}
```

By default, the modal will close when you hit the `escape` key. If you want to disable this behavior to, for example, prevent accidentally closing the modal you can overwrite the static `closeModalOnEscape` method and have it return `false`.

```php
public static function closeModalOnEscape(): bool
{
    return false;
}
```

By default, the modal will close when you click outside the modal. If you want to disable this behavior to, for example, prevent accidentally closing the modal you can overwrite the static `closeModalOnClickAway` method and have it return `false`.

 ```php
 public static function closeModalOnClickAway(): bool
 {
     return false;
 }
 ```

By default, closing a modal by pressing the escape key will force close all modals. If you want to disable this behavior to, for example, allow pressing escape to show a previous modal, you can overwrite the static `closeModalOnEscapeIsForceful` method and have it return `false`.

 ```php
 public static function closeModalOnEscapeIsForceful(): bool
 {
     return false;
 }
 ```

When a modal is closed, you can optionally enable a `modalClosed` event to be fired. This event will be fired on a call to `closeModal`, when the escape button is pressed, or when you click outside the modal. The name of the closed component will be provided as a parameter;

 ```php
 public static function dispatchCloseEvent(): bool
 {
     return true;
 }
 ```

## Skipping previous modals

In some cases you might want to skip previous modals. For example:

1. Team overview modal
2. -> Edit Team
3. -> Delete Team

In this case, when a team is deleted, you don't want to go back to step 2 but go back to the overview.
You can use the `skipPreviousModal` method to achieve this. By default it will skip the previous modal. If you want to skip more you can pass the number of modals to skip `skipPreviousModals(2)`.

```php
<?php

namespace App\Http\Livewire;

use App\Models\Team;
use VenusSoft\Livewire\BootstrapModal\ModalComponent;

class DeleteTeam extends ModalComponent
{
    public Team $team;

    public function mount(Team $team)
    {
        $this->team = $team;
    }

    public function delete()
    {
        Gate::authorize('delete', $this->team);
        
        $this->team->delete();

        $this->skipPreviousModal()->closeModalWithEvents([
            TeamOverview::getName() => 'teamDeleted'
        ]);
    }

    public function render()
    {
        return view('livewire.delete-team');
    }
}
```

## Security

If you are new to Livewire I recommend to take a look at the [security details](https://laravel-livewire.com/docs/2.x/security). In short, it's **very important** to validate all information given Livewire stores this information on the client-side, or in other words, this data can be manipulated. Like shown in the examples above, use the `Guard` facade to authorize actions.

## Credits

- [Huy Duong](https://github.com/truonghuytq)
- [All Contributors](../../contributors)
