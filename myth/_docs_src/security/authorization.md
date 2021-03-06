# User Authorization

A simple Flat RBAC authorization library is included with Sprint. It provides a simple method to group users into one or more roles, create and assign permissions to the roles, and restrict users from gaining access to content they're not allowed to.

This guide describes the libraries and methods involved with group and permissions management, as well as actions to use with users for low-level security. While the class can easily be used on it's own, you are encouraged to use the [Auth Trait](security/auth_trait) on your controllers to provide several Authentication and Authorization convenience methods.

## Configuration
There is only one configuration option: which Authorization library to use. Sprint only ships one, FlatAuthorization, which implements a simple Flat RBAC authentication system. You can easily create new libraries by extending the `Myth\Auth\AuthorizeInterface`. 

The config value sits in your `application/config/auth.php` configuration file. 

	$config['auth.authorize_lib'] = '\Myth\Auth\FlatAuthorization';

# The Library API
The following methods describe the library API itself. These methods provide finer-grained abilities to work with the system than what the Trait provides, as well as the abilities to work with the groups and permissions themselves. 

## Actions
These first methods are the methods that you will use the most throughout your application. They are the primary actions on users and  permissions handling.

### inGroup()
To check if a user belongs to a group, you would use the `inGroup()` method. The first parameter is the group(s) to check. The second parameter is the ID of the user in question.

The `group` parameter is very flexible. It can be either a single group or an array of groups. The group itself can be expressed as either the group's ID or the name of the group.

	// Use a single group id
	inGroup(12, $user_id);

	// Use multiple group ids
	inGroup([12,14], $user_id);

	// Use a group name
	inGroup('admins', $user_id);

	// Use multiple group names
	inGroup(['admin', 'moderators'], $user_id);

When checking multiple groups it only checks if the user belongs to one of the groups, not all of them. In other words, it's an OR query, not and AND query.

### hasPermission()
If you need to check if a user belongs to any group that has a permission, than you can use the `hasPermision()` method. The first parameter is the permission. It can be either its ID or its name. The second parameter is the user_id to check against. 

	// Use a permission ID
	hasPermission(12, $user_id);

	// Use a permission name
	hasPermission('manageUsers', $user_id);

### addUserToGroup()
Adds a user to a group. The first parameter is the users ID. The second parameter is the group. The group can be either the group ID or the name of the group. 

	addUserToGroup($user_id, $group_id);
	addUserToGroup($user_id, 'moderators');


### removeUserFromGroup()
Removes a user from a single group. The first parameter is the user id. The second parameter is the group. The group can be either the group ID or the group's name. 

	removeUserFromGroup($user_id, $group_id);
	removeUserFromGroup($user_id, 'moderators');

### addPermissionToGroup()
Adds a permission to a single group. The permission must already exist. The first parameter is the permission. The second parameter is the group to add it to. Both of the parameters can be either the ID or the name. 

	addPermissionToGroup($permission_id, $group_id);
	addPermissionToGroup('permission name', $group_id);
	addPermissionToGroup('permission name', 'group name');
	addPermissionToGroup($permission_id, 'group_name');

### removePermissionFromGroup()
Removes a single permission from a group. Does not delete the permission. The first parameter is the permission. The second parameter is the group. Both of the parameters can be either the ID or the name. 

	removePermissionFromGroup($permission_id, $group_id);
	removePermissionFromGroup('permission name', $group_id);
	removePermissionFromGroup('permission name', 'group name');
	removePermissionFromGroup($permission_id, 'group_name');
	
### addPermissionToUser()
Adds a private permission to a single user. This is in addition to any permissions that might be granted by the user's groups. 

The permission must already exist. The first parameter is the permission. The second parameter is the user's ID. The permission may be either the permission ID or the name assigned to it.

    addPermissionToUser($permission_id, $user_id);
    addPermissionToUser('permission name', $user_id);

### removePermissionFromUser()
Removes a single private permission from a user. Does not delete the user. The first parameter is the permission. The second parameter is the user ID. The permission may be either the permission ID or the name assigned to it.

    removePermissionFromUser($permission_id, $user_id);
    removePermissionFromUser('permission name', $user_id);
    
### doesUserHavePermission()
Checks a user to see if they have a private permission. The first parameter is the user_id. The second parameter is the permission. The permission may be either the permission ID or the name of the permission. Returns either true or false.

This is called automatically by the `hasPermission()` method, so you don't need to call it again if you are using that method, or the `restrictWithPermissions()` method in the Auth Trait.

## Groups
We refer to our Roles as Groups because we do not enforce the concept of a role. Groups are simply collections of users that have a set of permissions assigned to them. While it makes natural sense to treat them like roles, you could use them for any other purpose that you needed to. You could group beta testers into a `beta` group, for example. 

### group()
Grabs the details about a single group. The only parameter is the group, which can be either the ID or it's name. It returns an object straight from the database. 

	$auth = new Myth\Auth\FlatAuthorization();
	$group = $auth->group($group_id);
		or
	$group = $auth->group('moderators');

	// Returns a standard object with the `id`, `name` and `description`.

### groups()
Grabs the details about all groups in the system. Returns an array of group objects. 

	$groups = $auth->groups();

### createGroup()
Creates a new group. The first parameter is the name of the group. The second (optional) parameter is the description. 

Returns the ID of the newly created group, or FALSE if it ran into errors.

	$id = $auth->createGroup('admins', 'Site Administrators with god-like powers.');

### deleteGroup()
Deletes a single group. Unlike most of the other methods, this first parameter must be the ID of the group to delete. 

	$auth->deleteGroup($id);

### updateGroup()
Updates a single group. The first parameter is the group id. The second parameter is the name of the group. The third (optional) parameter is the group's description. 

Returns either TRUE or FALSE on success/failure.

	$auth->updateGroup($id, 'modders', 'No description today.');

## Permissions
Permissions are simple strings that represent some permission your user might need to be restricted from, like 'viewUsers' or 'manageUsers'. To ensure you don't hit any issues with any other modules, you should consider adopting a pattern that helps to protect against collisions. This is easily done by using your name or initials, and the name of your module or library that this is for. If I was writing a blog module, I might use something like "myth.blog.managePosts". Or "blog.posts.manage". 

### permission()
Returns the details about a single permission as an object straight from the database. The only parameter is the permission itself. It can be either the ID or the name.

	$auth = new Myth\Auth\FlatAuthorization();
	$permission = $auth->permission('blog.posts.manage');
	$permissions = $auth->permission(12);
	
### permissions()
Returns an array of all of the permissions in the system. There are no parameters.

### createPermission()
Creates a new permissions without assigning it to any group. The first parameter is the name of the permission. The second parameter is the description. Returns the ID of the new permission, or FALSE on failure. 

	$id = $auth->createPermission('blog.posts.manage', 'Allows a user to create, edit, and delete blog posts.');

### deletePermission()
Deletes a single permission and removes it from all groups it is assigned to. The only parameter is the **permission id**. Returns TRUE or FALSE on success or failure. 

	$auth->deletePermission('blog.posts.manage');

###updatePermission()
Updates the details of a single permission. The first parameter is the ID. The second parameter is the name. The third (optional) parameter is the description. Returns TRUE or FALSE on success or failure.

	$auth->updatePermission(12, 'new.perm.name', 'A better description goes here.');
	



