# Package de paramètres persistants pour Laravel

Ce package vous permet de sauvegarder des paramètres de manière plus persistante. Vous pouvez utiliser la base de données et/ou le fichier json pour sauvegarder vos paramètres. Vous pouvez également remplacer la configuration de Laravel.

* Support du driver
* Fonction helper
* Directive Blade
* Remplacer les valeurs de configuration
* Cryptage
* Fichier, table et colonnes personnalisés
* Sauvegarde automatique
* Colonnes supplémentaires
* Prise en charge du cache

## Démarrer

### 1. Installation

Exécutez la commande suivante :

```bash
composer require likewares/laravel-setting
```

### 2. Enregistrer (pour Laravel < 5.5)

Enregistrer le fournisseur de services dans `config/app.php`.

```php
Likewares\Setting\Provider::class,
```

Ajoutez un alias si vous souhaitez utiliser la facade.

```php
'Setting' => Likewares\Setting\Facade::class,
```

### 3. Publication

Publication du fichier de configuration.

```bash
php artisan vendor:publish --tag=setting
```

### 4. Base de données

Créer une table pour le driver de base de données

```bash
php artisan migrate
```

### 5. Configuration

Vous pouvez modifier les options de votre application à partir du fichier `config/setting.php`.

## UtilisationVous pouvez soit utiliser la méthode d'aide comme `setting('foo')` ou la façade `Setting::get('foo')`

Vous pouvez soit utiliser la méthode d'aide comme `setting('foo')` ou la façade `Setting::get('foo')`

### Facade

```php
Setting::get('foo', 'default');
Setting::get('nested.element');
Setting::set('foo', 'bar');
Setting::forget('foo');
$settings = Setting::all();
```

### Helper

```php
setting('foo', 'default');
setting('nested.element');
setting(['foo' => 'bar']);
setting()->forget('foo');
$settings = setting()->all();
```

Vous pouvez appeler la méthode `save()` pour enregistrer les modifications.

### Enregistrement automatique

Si vous activez l'option `auto_save` dans le fichier de configuration, les paramètres seront sauvegardés automatiquement à chaque arrêt de l'application si quelque chose a été modifié.

### Blade Directive

Vous pouvez obtenir les paramètres directement dans vos modèles de lames à l'aide de la méthode helper ou de la directive de lames, comme suit `@setting('foo')`

### Overrider Valeurs Config

Vous pouvez facilement remplacer les valeurs de configuration par défaut en les ajoutant à l'option `override` dans `config/setting.php`, éliminant ainsi le besoin de modifier les fichiers de configuration par défaut et vous permettant également de changer ces valeurs pendant la production. Ex :

```php
'override' => [
    "app.name" => "app_name",
    "app.env" => "app_env",
    "mail.driver" => "app_mail_driver",
    "mail.host" => "app_mail_host",
],
```

La valeur de gauche correspond à la valeur de configuration respective (Ex: config('app.name')) et la valeur de droite est le nom de la `key` dans votre fichier settings table/json.

### Encryption

Si vous souhaitez crypter les valeurs pour une clé donnée, vous pouvez passer la clé à l'option `encrypted_keys` dans `config/setting.php` et le reste est automatiquement géré en utilisant les facilités de cryptage intégrées de Laravel. Ex :
```php
'encrypted_keys' => [
    "payment.key",
],
```

### Stockage JSON

Vous pouvez modifier le chemin utilisé à l'exécution en utilisant `setting()->setPath($path)`.

### Stockage de la base de données

Si vous voulez utiliser la base de données comme stockage des paramètres, vous devez exécuter `php artisan migrate`. Vous pouvez modifier les champs de la table à partir du fichier `create_settings_table` dans le répertoire migrations.

#### Colonnes supplémentaires

Si vous souhaitez stocker les paramètres de plusieurs users/clients dans la même base de données, vous pouvez le faire en spécifiant des colonnes supplémentaires :

```php
setting()->setExtraColumns(['user_id' => Auth::user()->id]);
```

où `user_id = x` sera désormais ajouté à la requête de la base de données lorsque les paramètres seront récupérés, et lorsque de nouveaux paramètres seront sauvegardés, le `user_id` sera renseigné.

Si vous avez besoin d'un contrôle plus fin sur les données à interroger, vous pouvez utiliser la méthode `setConstraint` qui prend une fermeture avec deux arguments :

- `$query` est l'instance du constructeur de la requête
- `$insert` est un booléen qui indique si la requête est une insertion ou non. S'il s'agit d'une insertion, vous n'avez généralement rien à faire à `$query`.

```php
setting()->setConstraint(function($query, $insert) {
	if ($insert) return;
	$query->where(/* ... */);
});
```

### Drivers personnalisés

Ce paquet utilise la classe Laravel `Manager` sous le capot, il est donc facile d'ajouter votre propre driver de stockage. Tout ce que vous avez à faire est d'étendre la classe abstraite `Driver`, d'implémenter les méthodes abstraites et d'appeler `setting()->extend`.
```php
class MyDriver extends Likewares\Setting\Contracts\Driver
{
	// ...
}

app('setting.manager')->extend('mydriver', function($app) {
	return $app->make('MyDriver');
});
```

## License

La licence MIT (MIT). Veuillez consulter [LICENSE](LICENSE.md) pour plus d'informations.
