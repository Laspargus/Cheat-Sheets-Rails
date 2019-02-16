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
#### Créer sa première migration
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


Voici une migration qui dit "crée une table users et donne-lui une colonne name, de type string, et une colonne is_admin, de type booléen" :
```python
def change
  create_table :users do |t|
    t.string :name
    t.boolean :is_admin
    t.timestamps
  end
end
````

Voici une migration qui dit "dans la table users qui existe déjà, ajoute une colonne email, qui sera un string" :
```python
def change
  add_column :users, :email, :string
end
```

Voici une migration qui dit "dans la table users qui existe déjà, enlève la colonne is_admin" :

```python
def change
  remove_column :users, :is_admin, :boolean
end
```

#### Les commandes de migration
```
$ rails db:migrate
```
Cette commande exécute TOUTES les migrations du dossier db/migrate qui sont en statut down. Si tu te rends dans le dossier db après l'avoir exécutée, et ouvre le fichier development.sqlite3, tu verras que la base a été crée (et beaucoup plus facilement qu'hier en SQL).
```
$ rails db:migrate:status
```
te sort un joli tableau pour voir où tu en es dans tes migrations entre les up et les down.
```
$ rails db:rollback
```
Revient en arrière sur la dernière migration. Très pratique quand on a fait une coquille qu'on détecte de suite.
```
$ rails db:migrate VERSION=20180905201547
```
Annule toutes les migrations postérieures à celle portant le nom "20180905201547".

Attention à ne JAMAIS changer ni supprimer un fichier de migration à partir du moment où elle a été passée. Sans le fichier de migration, Rails ne saura pas quoi faire avec ta base de données donc il sera perdu

### Les modèles

Le nom d'une table (définie dans une migration) sera toujours en pluriel snake_case. On parlera par exemple de la table users.
Le nom de la classe de ton model sera toujours au singulier en CamelCase. On parlera par exemple du model User.
Le nom de fichier de ton model sera toujours au singulier en snake_case. On parlera par exemple du fichier user.rb

```
$ rails generate model NomDeTonModel
```

Cette commande aura pour effet de créer une migration de création de table en plus du model correspondant. Un petit 2-en-1 digne des meilleurs détergents. 


La commande de génération d'un modèle te permet même d'ajouter direct les colonnes de ta table DANS la migration. Par exemple si tu veux créer un model user avec un email qui est un string et un is_admin qui est un booléen, tu écrirais la commande :
```
$ rails generate model User email:string is_admin:boolean
```



