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



### Modele : Les attributs optionnels 

Imaginons que tu veuilles une relation 1-N entre un author et un book mais avec la possibilité qu'un book soit créé sans author. En temps normal, impossible : ActiveRecords refuserait et te renverrait des gros ROLLBACK. 
Pour remédier à ceci, optional est là pour toi 
```ruby
class Book < ApplicationRecord
  belongs_to :author, optional: true
end
```

### Modele :  Dependent
Dans certains cas, les associations entre tables de ta BDD sont très forts. Par là, je veux dire que si un objet disparaît, l'existence des objets qui lui sont liés n'a peut-être plus de sens et il serait préférable qu'ils disparaissent aussi pour ne pas polluer ta BDD. Un exemple ? Les RDV entre patients et docteurs. Si un docteur vient à demander à sortir de ta BDD, il faut probablement supprimer tous les appointments où il apparaît (et bien sûr prévenir les patients). C'est le rôle de dependent: :destroy d'effectuer cette suppression automatiquement.
```ruby
class Doctors < ApplicationRecord
  has_many :appointments, dependent: :destroy
end
```


### Modele : les validates

Prenons l'exemple de l’e-mail : il faut qu'il soit impossible de créer un utilisateur en base sans e-mail.  Eh bien avec les lignes suivantes, en haut du model User, c'est possible de l'imposer :
```ruby
class User < ApplicationRecord
  validates :email, presence: true
end
```
#### Valider l'unicité d'un attribut

Restons sur les e-mails : tu n'as pas envie que deux utilisateurs puissent s'inscrire avec le même email (sinon tu vas les confondre, ils vont reset leurs mots de passe entre eux, etc.). Pour ceci, rien de plus simple :

```ruby
class User < ApplicationRecord
  validates :email, uniqueness: true
end
```

#### Le format

Encore un exemple applicable aux e-mails ! Pour le moment, si l'on fait validates :email, uniqueness: true, presence: true, il est possible pour un utilisateur de s'enregistrer avec l'email "coucou123", ce qui n'est pas un e-mail valide. On peut rajouter une validation de format à cet attribut :

```ruby
class User < ApplicationRecord
  validates :email,
    presence: true,
    uniqueness: true,
    format: { with: /\A[^@\s]+@([^@\s]+\.)+[^@\s]+\z/, message: "email adress please" }
end
```

#### Longueur

Des fois, tu veux imposer un certain nombre de caractères. Par exemple tu veux que ton mot de passe fasse 6 caractères minimum, ou que le title de ton article n'en fasse pas plus de 140. Tu peux valider la length de tes attributs :
```ruby
class Person < ApplicationRecord
  validates :name, length: { minimum: 2 }
  validates :bio, length: { maximum: 500 }
  validates :password, length: { in: 6..20 }
  validates :registration_number, length: { is: 6 }
end
```

#### Les méthodes d'instance

Pour vérifier que la date de rendez-vous n'est pas encore passé par exemple
```ruby
class Appointment < ApplicationRecord
 validate : is_not_past
end
```
/!\ Pour vérifier une méthode. Validate est au singulier.

La méthode ```ìs_not_past``` vérifie que l'évènement est dans le futur :

```ruby
def is_not_past
		if self.start_date < Time.now
			self.errors.add(:start_date, "Your event is already past")
		end
	end
  ```
  
####  Les callbacks
 
Un callback est une action que tu souhaites réaliser à un instant précis lorsque tu appelles ton model. Voici un exemple de moment où tu peux appeler les callbacks :

Juste AVANT la sauvegarde en BDD d'une instance de ton model ;
Juste APRES la sauvegarde en BDD d'une instance de ton model ;
Juste APRES la mise à jour en BDD d'une instance de ton model ;


Prenons l'exemple d'un e-mail de bienvenue que tu aimerais bien envoyer à la création de ton utilisateur. Tu pourrais mettre dans ton controller :
```ruby 
def create_user
  user = User.create(form_params)
  user.send_welcome_email
end 
```

Encore une fois, ce n'est pas au controller de faire ceci. Pourquoi ? Car l'action est intimement liée à la sauvegarde en BDD et donc au taff du model : autant qu'il le prenne en charge. Sans compter que plusieurs controllers pourraient avoir à créer des User : il faudrait alors à chaque fois rajouter cette ligne d'envoi d'e-mail et là, c'est plus DRY du tout. 
On va donc implémenter un callback after_create pour envoyer un email de bienvenue à chaque sauvegarde en BDD d'un utilisateur :

```ruby
class User < ActiveRecord
  after_create :send_welcome_email

  def send_welcome_email
    # le code qui envoit l'email
  end

end
```

#### Les routes 
--------------------------------------

Un utilisateur navigue et interagit avec un site web en effectuant des requêtes. Une requête est définie par un verbe GET/POST/PUT/etc. et une URL. Le rôle du routeur (dans Rails, il s'agit du fichier routes.rb) est de rediriger une requête vers la méthode d'un controller de l'application.

Faire une route c'est comme ça :
```ruby
get '/static_pages/contact', to: 'controller#method'
```
```$ rails routes``` te servira beaucoup, notamment pour récupérer le prefix de tes routes.

Les routes dynamiques peuvent être faites en ajoutant une variable dans la route avec un :, par exemple :
```ruby
get '/users/:id', to: 'controller#method'
```
Au final, une route dynamique contient de l'information (la variable) que tu peux récupérer ensuite dans le controller via le hash params. Dans l'exemple ci-dessus, si on tape l'URL "/users/123" on aurait params[:id] qui serait égal à "123".

#### Les liens

##### Le helper
Un helper est une méthode spécifique à Rails qui va t'aider d'une façon où d'une autre. Ici, le helper link_to est une méthode que tu vas pouvoir mettre dans une view sous le format :
```ruby 
<%= link_to "clique ici", le_fameux_path %>
```
Quand il va la lire, elle va se transformer ainsi avant d'être envoyée au navigateur :
```html
<a href="url_liée_au_path_saisi_dans_ton_link_to">clique ici</a>
```

Pour pointer vers des URLs dynamiques, passe les paramètres dynamiques dans ton path. year_month_day_path(@year.id, @month.id, @day.id) pointera vers l'URL : /years/:year_id/months/:month_id/days/:id

Pour créer un boutton avec une classe bootstrap
```ruby
<%= link_to "Create New Event", new_event_path, class:"btn btn-success" %>
```

Pour créer un lien sur une image ou un div avec le helper 
```ruby 
<%= link_to 'Mon lien',  users_path do %>
    <div>YOUR CONTENT</div>
<% end %>
```

### Heroku


Avant de push, tu vas créer ton application sur Heroku :

```
$ heroku create
```


🚀 ALERTE BONNE ASTUCE
De base, ton application aura un nom de genre pacific-turkey-4983849. Mais tu peux la changer en un truc plus sympa (à condition que ce ne soit pas pris) du genre nom-de-ton-app avec la ligne suivante :
``` 
$ heroku create nom-de-ton-app
```

Et tu peux changer le nom de ton app existante avec :

```
heroku apps:rename nouveau-nom-de-ton-app
```

#### Mettre ton app en prod

Et voilà, LA partie importante. Avant de faire le push, on va voir un truc ensemble. Fais donc :

```
$ git remote --v
```

Il devrait te renvoyer un truc du genre :

```
heroku https://git.heroku.com/nom-de-ton-app.git (fetch)
heroku  https://git.heroku.com/nom-de-ton-app.git (push)
origin  git@github.com:username/repo.git (fetch)
origin  git@github.com:username/repo.git (push)
```

Si tu n'es pas trop en galère sur git, tu devrais comprendre qu'en créant ton application, tu as branché une nouvelle remote à git. En plus de pouvoir push ton joli travail sur Origin (donc GitHub), tu peux le push sur la remote Heroku. Et bien testons cela :

```
$ git add .
$ git commit -m "First commit and pushing to Heroku"
$ git push heroku master
```

Pour migrer sa base de donnée sur heroku 
```
$ heroku run rails db:migrate
```

En cas d'erreur de type : ```You must use Bundler 2 or greater with this lockfile```
You might fix this with a command :

```
heroku buildpacks:set https://github.com/bundler/heroku-buildpack-bundler2
```


### Le CRUD

S'il y a une chose à retenir, c'est que toute route sur Rails correspond à une requête du REST, et que derrière, cette route va pointer vers une action du CRUD, pour la ressource qui la concerne. Cette action s'effectuera au sein d'une méthode de controller puis renverra vers une view correspondante. 
En d'autres termes, les routes suivent la convention REST (7 requêtes possibles) et pointent vers des controllers qui établissent une action du CRUD avec des méthodes ayant des noms fixés par convention : #new, #create, #show, #index, #edit, #update et #destroy.


#### Les routes automatises avec resources

il existe un moyen d'automatiser l'écriture des routes qui suivent la convention. 
Pour cela, il suffit d'une seule ligne dans le fichier routes.rb :
```ruby
resources :gossips
```


Une fois cette ligne écrite, tu peux voir son résultat en faisant un petit $ rails routes :
```ruby
                   Prefix Verb   URI Pattern                                                                              Controller#Action
                  gossips GET    /gossips(.:format)                                                                       gossips#index
                          POST   /gossips(.:format)                                                                       gossips#create
               new_gossip GET    /gossips/new(.:format)                                                                   gossips#new
              edit_gossip GET    /gossips/:id/edit(.:format)                                                              gossips#edit
                   gossip GET    /gossips/:id(.:format)                                                                   gossips#show
                          PATCH  /gossips/:id(.:format)                                                                   gossips#update
                          PUT    /gossips/:id(.:format)                                                                   gossips#update
                          DELETE /gossips/:id(.:format)                                                                   gossips#destroy

```

Sache aussi que tu peux imbriquer les routes entre elles et faire ce que l'on appelle des nested resources.

```ruby
resources :gossips do
  resources :comments
end
```


Et voilà ce que cela ajoute dans $ rails routes :
```ruby
                   Prefix Verb   URI Pattern                                                                              Controller#Action
                          GET    /message/:user_entry(.:format)                                                           message#show
          gossip_comments GET    /gossips/:gossip_id/comments(.:format)                                                   comments#index
                          POST   /gossips/:gossip_id/comments(.:format)                                                   comments#create
       new_gossip_comment GET    /gossips/:gossip_id/comments/new(.:format)                                               comments#new
      edit_gossip_comment GET    /gossips/:gossip_id/comments/:id/edit(.:format)                                          comments#edit
           gossip_comment GET    /gossips/:gossip_id/comments/:id(.:format)                                               comments#show
                          PATCH  /gossips/:gossip_id/comments/:id(.:format)                                               comments#update
                          PUT    /gossips/:gossip_id/comments/:id(.:format)                                               comments#update
                          DELETE /gossips/:gossip_id/comments/:id(.:format)                                               comments#destroy
			  
			  ```

Tu peux supprimer une route créée par resources avec un except: comme cela :
```ruby
resources :gossips, except: [:destroy] 
```

Ou carrément restreindre les routes en faisant un petit only:
```ruby
resources :comments, only: [:new, :create, :index, :destroy]

```
