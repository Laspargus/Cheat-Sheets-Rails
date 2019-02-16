# Cheat-Sheets-Rails
## Installation


### Rails

Commencer par créer une nouvelle appli rails :

```
$ rails new nom_de_mon_app
```
Cela va créer un nouveau dossier Rails nommé my_first_app. Tu n'as plus qu'a te déplacer dans ce répertoire et lancer la fameuse commande :

Modifier son Gemfile. Voici celui de Félix : 
https://github.com/felhix/cheat_sheets/blob/master/Ruby/Gemfile.rb

Ajouter les Gems suivantes si besoin :

````
gem 'table_print'
#Pour afficher les tables dans la console

gem 'faker'
gem 'rb-readline'
#En cas de bug avec la console

gem 'devise'
gem 'stripe'
gem 'dotenv-rails'
gem "aws-sdk-s3", require: false
#Pour héberger les images sur Amazon

gem 'rails_db', '2.0.4'
#Pour accéder à sa base de données via http://localhost:3000/rails/db

```

Installer les Gems avec un : 

```
$ bundle install
```

Puis se mettre dedans (sinon dur de travailler dessus) :

```javascript
var s = "JavaScript syntax highlighting";
alert(s);
```
```ruby
rails new appname --skip-sprockets
@event=Event.new
`````


```command
$rails g controller

