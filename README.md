# Cheat-Sheets-Rails

## Installation

Commencer par créer une nouvelle appli rails :

```
$ rails new nom_de_mon_app
```
🚀 ALERTE BONNE ASTUCE
Un petit alias pour que $ rails new ajoute le -d postgresql me semble être une bonne idée 😉
Ex :
```
alias rn="rails new -d postgresql"
```
Donnera
```
$ rails new -d postgresql nom_de_mon_app
```


Modifier son Gemfile. Voici celui de Félix : 
https://github.com/felhix/cheat_sheets/blob/master/Ruby/Gemfile.rb

Ajouter les Gems suivantes si besoin :

```
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

Initialiser la base de donnée
```
$ rails db:create
```

### First commit
En théorie, installer une app rails initialize un git, donc pas besoin de git commit. On va donc faire le premier commit. Si l'on veut lier son application Rails à un remote, c'est possible avec git remote add origin blabla_nom_origine.
```
git add .
git commit -m "premier commit"
```


### Github
Push an existing repository from the command line
```
git remote add origin https://github.com/Laspargus/testtest.git
git push -u origin master
```


### Git push heroku master
Maintenant il ne reste plus qu'à commiter, et pousser ça chez heroku.

```
$ git add .
$ git commit -m "Gemfile OK"
$ heroku create
$ git push heroku master
```

## Rails


### Migration
Créer sa première migration
```
$ rails generate migration NomDeTaMigration
```

Si tu exécutes cette commande, il devrait créer un fichier de migration qui ressemble à quelque chose comme ça :
```python
class NomDeTaMigration < ActiveRecord::Migration[5.2]
  def change
    # ici tu mettras les modifications que tu aimerais apporter à ta base de données
  end
end
```

Pour effectuer la migration, il te suffira de saisir la commande :
```
$ rails db:migrate
```











