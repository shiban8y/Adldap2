# Upgrade Guide

## Upgrading to v6 from v5.2

Due to the `v6` being a new major release, large changes have occurred. You will need to modify your code accordingly.

##### Adldap\Adldap Class

Adldap now supports multiple LDAP connections at once. The `Adldap\Adldap` class is now a "Gateway" to multiple connections.

You now construct an Adldap instance, and then attach connection providers (`Adldap\Connections\Provider`) to it.

For example:

```php
$provider = new \Adldap\Connections\Provider($config, $connection, $schema);

$ad = new Adldap();

$ad->addProvider('provider-name', $provider);

$ad->connect('provider-name');
```

###### Adldap\Schemas\ActiveDirectory

The `ActiveDirectory` schema has now been removed in favor of a Schema object.

You can utilize the schema object to manage which Schema you'd like to use, for example:

```php
$schema = new \Adldap\Schemas\ActiveDirectory();

\Adldap\Schemas\Schema::set($schema);
```

Then you can use the `\Adldap\Schemas\Schema` object to retrieve your current schema:

```php
$schema = \Adldap\Schemas\Schema::get();

$user = $provider->search()->where($schema->commonName(), '=', 'John Doe')->first();
```

##### Search Classes Removed / Replaced with Search Factory

All search classes have been removed and replaced with query 'scopes' utilized in `Adldap\Search\Factory`.

For example, you used to call:

```php
// v5.2
$ad = new Adldap\Adldap($config);

$ad->users()->all();
```

In `v6` you would call:

```php
// v6.0
$provider = $ad->connect('provider-name');

$provider->search()->users()->get();
```

A `Adldap\Search\Factory` instance is returned when calling the `search()` method on your connection provider.
Inside this factory, you can utilize the many scopes for only retrieving certain records (such as Computers or Users).

Please take a look at the [Query Builder documentation](docs/query-builder.md#scopes) for all of the methods.

##### Adldap\Adldap::authenticate() method replaced with Adldap\Auth\Guard

To check a users credentials using your AD server, you used to be able to perform:

```php
// v5.2
$ad = new \Adldap\Adldap($config);

$ad->authenticate($username, $password, $bindAsUser = false);
```

Now you need to utilize the `Adldap\Auth\Guard` object of checking user credentials.
This object is returned when calling the `auth()` method on your connection provider. For example:

```php
// v6.0
if ($provider->auth()->attempt($username, $password, $bindAsUser = false)) {
    // Credentials were valid!
}
```