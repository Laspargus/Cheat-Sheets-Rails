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
```ruby
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
```ruby
def change
  create_table :users do |t|
    t.string :name
    t.boolean :is_admin
    t.timestamps
  end
end
````

Voici une migration qui dit "dans la table users qui existe déjà, ajoute une colonne email, qui sera un string" :
```ruby
def change
  add_column :users, :email, :string
end
```

Voici une migration qui dit "dans la table users qui existe déjà, enlève la colonne is_admin" :

```ruby
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

#### Ajout de liens entre 2 tables déjà existantes
```
$ rails generate migration AddIndexToBooks
```

Ensuite on rentre les lignes suivantes dans le fichier de migration, avant de la passer :
```Ruby
def change
  add_reference :books, :author, foreign_key: true
end
```

Reste alors à modifier tes modèles pour prendre en compte les associations :
```Ruby
class Book < ApplicationRecord
  belongs_to :author
end
```
```Ruby
class Author < ApplicationRecord
  has_many :books
end
```

Si l'on souhaite lier une base à sa création

```
$ rails generate model Book
```

Rajoute la ligne t.belongs_to :author, index: true dans la migration :
```Ruby
class CreateBooks < ActiveRecord::Migration[5.1]
  def change
    create_table :books do |t|
      t.belongs_to :author, index: true
      t.timestamps
    end
  end
end
```
#### La console

Il suffit de taper la ligne de commande ``` $ rails console```
Tu peux lancer la console en mode sandbox grâce à l'option ```$ rails console --sandbox``` Cela fera perdre toutes les données que tu as enregistrées lorsque tu quitteras la console. Pratique pour tester des models à la volée.


Si on a installé la gem table_print. On peut l'utiliser pour visualier les Tables dans la console

```$ tp User.all```


#### Seed

Au lieu de remplir sa BDD de dummy data à la main, ce qui est particulièrement fastidieux, on peut la seed. C'est à dire la remplir de données random en exécutant la commande :

```$ rails db:seed```

Au préalable, tu devras préciser ce que tu vas seeder dans le fichier db/seeds.rb.

```ruby
require 'faker'
100.times do
  user = User.create!(name: Faker::Company.name, email: Faker::Internet.email)
end
```

Lien Faker https://github.com/stympy/faker

🚀 ALERTE BONNE ASTUCE
Avant de commencer un seed, il est généralement bien vu de remettre sa base de données à 0, pour éviter l'armée d'utilisateurs ayant le même email. Tu peux commencer ton seed par :
```ruby
User.destroy_all
Event.destroy_all
```

### Les relations avancées NN et de type admin >> user

#### Les tables NN
Pour les migrations, c'est plutôt simple : il faut créer trois tables et mettre les foreign keys dans la table intermédiaire
```ruby
def change
  create_table :doctors do |t|
    t.timestamps
  end

  create_table :patients do |t|
    t.timestamps
  end

  create_table :appointments do |t|
    t.belongs_to :doctor, index: true
    t.belongs_to :patient, index: true
    t.timestamps
  end
end
```

Si tes 3 tables existent déjà et que tu as juste besoin d'ajouter les foreign keys à la table des rendez-vous, voici ce que cela donnerait :
```ruby
def change
  add_reference :appointments, :doctor, foreign_key: true
  add_reference :appointments, :patient, foreign_key: true
end
```

Les docteurs, dans le fichier app/models/doctor.rb :
```ruby
class Doctor < ApplicationRecord
  has_many :appointments
  has_many :patients, through: :appointments
end
```
Les patients, dans le fichier app/models/patient.rb
```ruby
class Patient < ApplicationRecord
  has_many :appointments
  has_many :doctors, through: :appointments
end
```
Les rendez-vous, dans le fichier app/models/appointment.rb
```ruby
class Appointment < ApplicationRecord
  belongs_to :doctor
  belongs_to :patient
end
```

⚠️ ALERTE ERREUR

```has_many :trucs``` = "A plusieurs trucs", plusieurs donc au pluriel. 
```belongs_to :truc``` = "Appartient à un truc", donc singulier.


#### Les class_name



Imaginons que la table users existe déjà avec son model User. Pour rajouter les messages, on crée un model PrivateMessage avec une migration pour créer la table private_messages :

```ruby
class CreatePrivateMessages < ActiveRecord::Migration[5.2]
  def change
    create_table :private_messages do |t|
      t.references :recipient, index: true
      t.references :sender, index: true

      t.timestamps
    end
  end
end
```

Tu vois que, directement, on annonce que la table contient des foreign key de type recipient et sender… Seul petit problème : ces tables n'existent pas, car les destinataires et expéditeurs sont des utilisateurs avant tout ! C'est là que la méthode class_name intervient. Pour l'implémenter, on va dans le model (fichier private_message.rb) et on fait :

```ruby
class PrivateMessage < ApplicationRecord
  belongs_to :sender, class_name: "User"
  belongs_to :recipient, class_name: "User"
end
```

Et dans la classe User, tu mettras :

```ruby
class User < ApplicationRecord
  has_many :sent_messages, foreign_key: 'sender_id', class_name: "PrivateMessage"
  has_many :received_messages, foreign_key: 'recipient_id', class_name: "PrivateMessage"
end
```

En gros, voici ce que tu as fait (si on se concentre sur le côté sender / sent_messages) :

Tu as fait une migration pour créer une table private_messages avec une référence sender. 
Comme la table senders n'existe pas, tu ne mets pas foreign_key: true, mais index: true. 
La table private_messages a donc une colonne sender_id.
Dans le model PrivateMessage, tu dis que ce dernier appartient à un sender, qui est en fait de la classe User
Dans le model User, tu dis que ce dernier has_many sent_messages. Ces messages envoyés correspondent à la colonne sender_id de la classe PrivateMessage




