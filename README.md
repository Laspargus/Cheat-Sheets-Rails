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

To remove a repository :

```
git remote rm origin
```

Pour visualiser les repo

```
git remote -v
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

La home page 
```ruby
root 'controller#action'
```


Au final, une route dynamique contient de l'information (la variable) que tu peux récupérer ensuite dans le controller via le hash params. Dans l'exemple ci-dessus, si on tape l'URL "/users/123" on aurait params[:id] qui serait égal à "123".

#### Les liens

##### Le helper
Un helper est une méthode spécifique à Rails qui va t'aider d'une façon où d'une autre. Ici, le helper link_to est une méthode que tu vas pouvoir mettre dans une view sous le format :
```ruby 
<%= link_to "clique ici", le_fameux_path %>
```
Quand il va la lire, elle va se transformer ainsi avant d'être envoyée au navigateur :
```ruby
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


### Les controllers

On peut créer les méthodes et les vues correspondantes directement avec la commande
```ruby
$ rails g controller users index show new create edit update destroy 
```
Voici le controller généré :

```ruby
class UsersController < ApplicationController

  def index
    # Méthode qui récupère tous les potins et les envoie à la view index (index.html.erb) pour affichage
  end

  def show
    # Méthode qui récupère le potin concerné et l'envoie à la view show (show.html.erb) pour affichage
  end

  def new
    # Méthode qui crée un potin vide et l'envoie une view qui affiche le formulaire pour 'le remplir' (new.html.erb)
  end

  def create
    # Méthode qui créé un potin à partir du contenu du formulaire de new.html.erb, soumis par l'utilisateur
    # pour info, le contenu de ce formulaire sera accessible dans le hash params (ton meilleur pote)
    # Une fois la création faite, on redirige généralement vers la méthode show (pour afficher le potin créé)
  end

  def edit
    # Méthode qui récupère le potin concerné et l'envoie à la view edit (edit.html.erb) pour affichage dans un formulaire d'édition
  end

  def update
    # Méthode qui met à jour le potin à partir du contenu du formulaire de edit.html.erb, soumis par l'utilisateur
    # pour info, le contenu de ce formulaire sera accessible dans le hash params
    # Une fois la modification faite, on redirige généralement vers la méthode show (pour afficher le potin modifié)
  end

  def destroy
    # Méthode qui récupère le potin concerné et le détruit en base
    # Une fois la suppression faite, on redirige généralement vers la méthode index (pour afficher la liste à jour)
  end
end
```

### Les formulaires

Example de formualire simple
```ruby
<form action= <%= gossips_path %> method="POST">
<input type="hidden" name="authenticity_token" value="<%= form_authenticity_token %>
  <input type="text" name="gossip_text">
  <input type="submit" value="Valider">
</form>
```

Les champs du formulaire sont ensuite disponibles dans le controller via le hash params


### Formulaire avec Formtag et Bootstrap

```ruby
<%= form_tag attendnace_path(@event) do %>

 	<div class="form-group">
		<label>Title</label>
		<%= f.text_field :title , class:'form-control'%>
    	</div>
        
	<div class="form-group">
	       <label>Description</label>
		<%= f.text_field :description , class:'form-control'%>
	</div>
<% end %>
```
#### Les Helpers 


Nous allons apprendre à changer un ```User.find_by(id: session[:user_id])``` en un plus simple ```current_user``` avec deux raisons en tête :

En Rails, c'est le mal de faire un appel au model (= une requête SQL) dans une vue. C'est le rôle du controller ! Le moindre find dans la view et c'est -30 points en test technique.

En Rails on aime le code bien DRY et bien lisible. current_user c'est très lisible et plus court que l'autre pâté.
Pour refactorer (= condenser) de cette façon notre code, nous allons passer par un helper. Les helpers se rangent dans le dossier ```app/helpers/```  : ils ont pour mission de créer des méthodes pour remplacer les bouts de code qu'on utilise fréquemment. 
Tout ce que tu as à faire, c'est de mettre dans ton controller :

```ruby
class TonController < ApplicationController
  include TonHelper
end
```

TonHelper correspond au fichier '''app/helpers/ton_helper.rb''' qui devrait ressembler à ceci :
```ruby
module TonHelper
  def some_method
    # une méthode et son code
  end
end
```


Si tu es perspicace, tu auras noté que chaque controller commence par la ligne ```class TonController < ApplicationController``` : en fait ils héritent tous de ApplicationController ! Au final, c'est un peu le controller ultime : tout ce qui est écrit dedans, tu peux considérer qu'il est écrit dans chaque controller.

Donc une fois la ligne rajoutée, il faut créer un fichier ```app/helpers/sessions_helper.rb``` et y mettre les lignes suivantes :

```ruby
module SessionsHelper
   def current_user
    User.find_by(id: session[:user_id])
  end

   def log_in(user)
    session[:user_id] = user.id
  end
end
```

Et voilà ! Maintenant tu peux appeler current_user dans n'importe quel controller ou view, et cette méthode te retournera l'instance de User contenant les infos de ton utilisateur connecté.


#### Les callbacks

La bonne façon de le présenter, c'est de faire ça dans une méthode privée authenticate_user, et de l'appeler EN AMONT de la méthode du controller, grâce à un callback before_action. Voici le résultat :

```ruby
class TonController < ApplicationController
  before_action :authenticate_user, only: [:index]

  def index
    # on code quelque chose qui permet d'afficher le dashboard de l'utilisateur
  end
  ```
  On crée une méthode private :
  
  ```ruby
  private

  def authenticate_user
    unless current_user
      flash[:danger] = "Please log in."
      redirect_to new_session_path
    end
  end
```


Grâce à ce magnifique callback, à chaque fois que la méthode index de ton controller est appelée, la méthode authenticate_user va être exécutée en amont. Elle filtrera les utilisateurs non connectés pour les rediriger vers la page login : en quelques lignes tu viens de sécuriser ton application !

/!\
Tu peux stocker des informations au sein du hash
```session```.Son contenu est conservé jusqu'à fermeture du navigateur.

Pour gérer la connexion / déconnexion des utilisateurs à leur compte, il faut créer un controller ```sessions_controller``` contenant les méthodes ```#new```, ```#create``` et ```#destroy```.

Ces méthodes vont permettre de stocker l'id de l'utilisateur connecté dans ```session[:user_id]```

Les helpers se rangent dans le dossier ```app/helpers/``` : ils ont pour mission de créer des méthodes pour remplacer les bouts de code qu'on utilise fréquemment. Par exemple, le fait de récupérer l'identité de l'utilisateur connecté avec ```current_user```.

Les helpers utilisés dans de nombreux controllers peuvent être rajoutés à ApplicationController via un ```include TonHelper```.

Le callback before_action ```:authenticate_user, only: [:index] ``` permet d'exécuter la méthode authenticate_user AVANT une méthode du controller (ici la méthode index).

### ActionMailer

L'Action Mailer est organisé en plusieurs éléments au sein d'une app Rails :

Des mailers, qui sont ni plus ni moins que des sortes de controllers appliqués aux e-mails. Tout comme les controllers "normaux", ils contiennent des méthodes qui vont faire des appels à la BDD (via les models) et ensuite envoyer des e-mails (au lieu d'envoyer des pages web à des navigateurs).

Des views, qui sont des sortes de templates des e-mails à envoyer. Tout comme les views de ton site, elles sont personnalisées grâce à du code Ruby inclus dedans (pour rajouter un nom, un e-mail, le contenu d'un objet récupéré en BDD, etc.). Il existe deux types de views : les .text.erb et les .html.erb car on peut envoyer des e-mails au format HTML comme au format text.

#### Action Mailer : Le premier mailer

On va générer un mailer avec ```$ rails g mailer UserMailer```. On l'a appelé UserMailer dans l'idée qu'à terme, il pourrait gérer tous les e-mails à destination des utilisateurs. On pourrait aussi avoir un AdminMailer qui enverrait les e-mails aux gestionnaires du site.

Maintenant, jette un œil au mailer que tu viens de générer dans app/mailers/user_mailer.rb : il est vide mais hérite de ApplicationMailer que tu pourras retrouver à app/mailers/application_mailer.rb.

On va éditer le mailer pour rajouter une méthode dont le rôle sera simple : envoyer un e-mail de bienvenue à tous les utilisateurs s'inscrivant sur notre site. Rajoute donc les lignes suivantes :

```ruby
class UserMailer < ApplicationMailer
  default from: 'no-reply@monsite.fr'
 
  def welcome_email(user)
    #on récupère l'instance user pour ensuite pouvoir la passer à la view en @user
    @user = user 

    #on définit une variable @url qu'on utilisera dans la view d’e-mail
    @url  = 'http://monsite.fr/login' 

    # c'est cet appel à mail() qui permet d'envoyer l’e-mail en définissant destinataire et sujet.
    mail(to: @user.email, subject: 'Bienvenue chez nous !') 
  end
end
```

La première ligne permet de définir la valeur de default[:from]. Le hash default permet de définir tout un ensemble de valeurs par défaut : celles-ci sont écrasées si la méthode d'envoi d’e-mail définit une valeur autre. Ici, l'objectif est que nos e-mails affichent toujours une adresse d’e-mail d'envoi : soit celle définie par la méthode du mailer, soit, à défaut, no-reply@monsite.fr.


#### Action Mailer : View

On va créer le template de notre e-mail de bienvenue. 
Pour ça, crée un fichier welcome_email.html.erb dans app/views/user_mailer/. Bien évidemment le nom est extrêmement important : il doit être identique à celui de la méthode welcome_email et placé dans le dossier views/user_mailer/ qui contient tous les templates e-mails relatifs au mailer UserMailer. Le contenu du template sera le suivant :
```ruby
<!DOCTYPE html>
<html>
  <head>
    <meta content='text/html; charset=UTF-8' http-equiv='Content-Type' />
  </head>
  <body>
    <h1>Salut <%= @user.name %> et bienvenue chez nous !</h1>
    <p>
      Tu t'es inscrit sur monsite.fr en utilisant l'e-mail suivant : <%= @user.email %>.
    </p>
    <p>
      Pour accéder à ton espace client, connecte-toi via : <%= @url %>.
    </p>
    <p> À très vite sur monsite.fr !
  </body>
</html>
```

####  Action Mailer : envoie automatique


- Si tu veux envoyer un email à la création d'un utilisateur, c'est un callback ```after_create``` dans le model ```User```
- Si tu veux envoyer un email quand un utilisateur vient de prendre un RDV sur Doctolib, c'est un callback ```after_create``` à la création d'un ```Appointment```
- Si tu veux envoyer une newsletter hebdomadaire, c'est un Service qui tourne de manière hebdomadaire (on verra comment faire des services cette semaine 😉)

Dans notre cas, on veut envoyer un e-mail juste après la création d'un utilisateur : il serait logique que ce travail revienne au model car 1) c'est lui qui crée l'utilisateur donc autant qu'il fasse les 2 actions et 2) Fat model Skinny controller, duuuuuude ! 🕺

Du coup, va dans ton model User et rajoute la ligne suivante :

```ruby
class User < ApplicationRecord
  after_create :welcome_send

  def welcome_send
    UserMailer.welcome_email(self).deliver_now
  end

end
```


On a utilisé un callback qui permet juste après l'inscription en base d'un nouvel utilisateur, d'appeler la méthode d'instance welcome_send. Celle-ci ne fait qu'appeler le mailer UserMailer en lui faisant exécuter welcome_email avec, pour seule entrée, l'instance créée (d'où le self).

À noter qu'on rajoute ensuite un deliver_now pour envoyer immédiatement l’e-mail. Il est possible d'utiliser un deliver_later mais son fonctionnement en production est moins évident : il faut savoir gérer les tâches asynchrones avec Active Job…


#### Action Mailer : config en dev

Pour ça on va utiliser une gem assez cool qui s'appelle [Letter Opener](https://github.com/ryanb/letter_opener). Son fonctionnement ? Dès qu'un e-mail doit être envoyé par ton app Rails, celui-ci est automatiquement ouvert dans ton navigateur web.

Mets letter_opener dans le groupe de développement de ton Gemfile puis ```bundle install```
Maintenant va dans config/environments/development.rb (fichier contenant les paramètres de ton environnement de développement) et colle les lignes ```config.action_mailer.delivery_method = :letter_opener``` et ```config.action_mailer.perform_deliveries = true```

#### Action Mailer : config en prod

Nous allons utiliser [Sendgrid](https://app.sendgrid.com/guide/integrate/langs/smtp).
Un fois la clefs API récupérée :

- Crée un fichier ```.env``` à la racine de ton application.
- Ouvre-le et écris dedans les informations suivantes : ```SENDGRID_LOGIN='apikey'``` et ```SENDGRID_PWD='ta_clef_API'``` en remplaçant bien sûr ta_clef_API par la clef que tu viens de générer. Elle est au format ```SG.sXPeH0BMT6qwwwQ23W_ag.wyhNkzoQhNuGIwMrtaizQGYAbKN6vea99wc8```. N'oublie pas les guillemets !
Rajoute ```gem 'dotenv-rails'``` à ton Gemfile et fait le ```$ bundle install```
Et l'étape cruciale qu'on oublie trop souvent : ouvre le fichier ```.gitignore``` à la racine de ton app Rails et écris ```.env``` dedans.

 Il ne te reste qu'à entrer les configurations SMTP de SendGrid dans ton app. Va dans /config/environment.rb et rajoute les lignes suivantes :
```ruby
ActionMailer::Base.smtp_settings = {
  :user_name => ENV['SENDGRID_LOGIN'],
  :password => ENV['SENDGRID_PWD'],
  :domain => 'monsite.fr',
  :address => 'smtp.sendgrid.net',
  :port => 587,
  :authentication => :plain,
  :enable_starttls_auto => true
}
```

### Devise

Pour installer Devise, il faut d'abord commencer par mettre la gem dans le Gemfile, puis de faire bundle install.

Ensuite, on va devoir installer Devise avec la commande suivante :

```$ rails generate devise:install```

Ce qui a pour effet de créer deux fichiers :

```config/initializers/devise.rb``` : le fichier de configuration de Devise. On s'en servira par exemple pour paramétrer le service que l'on va utiliser pour les envois d'emails
```config/locales/devise.en.yml``` : un fichier contenant les messages d'erreur de Devise. Tu pourras utiliser ses version françaises quand tu seras plus à l'aise

#### Devise - Mail en dévelopement

Puis après avoir créé les fichiers, il faut préciser à Devise comment envoyer les mails en développement. Donc dans le fichier ```config/environments/development.rb``` mets-donc la ligne suivante :

```ruby 
config.action_mailer.default_url_options = { host: 'localhost', port: 3000 }
```

#### Devise - Modele

Devise utilise comme Rails un générateur qui va nous mâcher le travail. Pour dire à Devise que tel model va être en mode Devise (avec la notion de session, login, etc), on fera rails g devise model. La majorité du temps cela concerne les utilisateurs (tout site), avec quelques exceptions quand cela concerne plutôt les admins (un blog sans gestion d'utilisateur). Ainsi, pour une app normale dans laquelle nous voulons avoir Devise branchée sur les utilisateurs, tu feras :

```$ rails g devise User```

Cela va créer et modifier quelques fichiers, mais trois nous intéressent beaucoup : une fichier de migration, un fichier de model, et une modification des routes.

###### Devise : Le fichier de migration
Voici le fichier de migration que Devise va te générer :

```ruby
# frozen_string_literal: true

class DeviseCreateUsers < ActiveRecord::Migration[5.2]
  def change
    create_table :users do |t|
      ## Database authenticatable
      t.string :email,              null: false, default: ""
      t.string :encrypted_password, null: false, default: ""

      ## Recoverable
      t.string   :reset_password_token
      t.datetime :reset_password_sent_at

      ## Rememberable
      t.datetime :remember_created_at

      ## Trackable
      # t.integer  :sign_in_count, default: 0, null: false
      # t.datetime :current_sign_in_at
      # t.datetime :last_sign_in_at
      # t.inet     :current_sign_in_ip
      # t.inet     :last_sign_in_ip

      ## Confirmable
      # t.string   :confirmation_token
      # t.datetime :confirmed_at
      # t.datetime :confirmation_sent_at
      # t.string   :unconfirmed_email # Only if using reconfirmable

      ## Lockable
      # t.integer  :failed_attempts, default: 0, null: false # Only if lock strategy is :failed_attempts
      # t.string   :unlock_token # Only if unlock strategy is :email or :both
      # t.datetime :locked_at


      t.timestamps null: false
    end

    add_index :users, :email,                unique: true
    add_index :users, :reset_password_token, unique: true
    # add_index :users, :confirmation_token,   unique: true
    # add_index :users, :unlock_token,         unique: true
  end
end
```

C'est une migration pour créer une table users, puis lui ajouter les attributs qui vont permettre à Devise de paramétrer. que Devise va gérer. De base, Devise ne gère que Database Authenticatable, Registerable, Recoverable, Rememberable, et Validatable. Les autres attributs sont en gris et tu n'as qu'à décommenter les lignes qui t'intéressent. 

---
Mais dis-donc Jamy, comment on fait si l'on a déjà créé le model User ?
Devise est plutôt intelligente, puisque elle va changer ta migration de create_table à change_table. Il y aura deux détails à gérer : 
Si jamais tu as déjà dans ta table users une colonne que Devise utilise (email, encrypted_password, etc), la migration plantera puisque Devise les ajoute aussi (c'est comme si tu avais deux colonnes email, et ça ne marche pas). En l'état il n'est pas possible de rollback ta migration. Il te faudra ajouter à la main la partie self.down, avec des : remove_column ou des remove_index pour que tout aille bien.

---

###### Devise : Le fichier de model

Voici ton fichier de model :
```ruby
class User < ApplicationRecord
  # Include default devise modules. Others available are:
  # :confirmable, :lockable, :timeoutable, :trackable and :omniauthable
  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :validatable
end
```

Ces quelques lignes vont juste dire à ton model, "hey, c'est possible de faire de database_authenticatable, registerable, recoverable, rememberable, et validatable". Si tu veux faire l'un des autres, il te faudra le mettre dans la liste des options non commentées.

##### Devise : Le fichier des routes

Normalement les routes devraient t'afficher devise_for :users. Inspectons ceci avec rails routes :
```ruby
                   Prefix Verb   URI Pattern                                                                              Controller#Action
         new_user_session GET    /users/sign_in(.:format)                                                                 devise/sessions#new
             user_session POST   /users/sign_in(.:format)                                                                 devise/sessions#create
     destroy_user_session DELETE /users/sign_out(.:format)                                                                devise/sessions#destroy
        new_user_password GET    /users/password/new(.:format)                                                            devise/passwords#new
       edit_user_password GET    /users/password/edit(.:format)                                                           devise/passwords#edit
            user_password PATCH  /users/password(.:format)                                                                devise/passwords#update
                          PUT    /users/password(.:format)                                                                devise/passwords#update
                          POST   /users/password(.:format)                                                                devise/passwords#create
 cancel_user_registration GET    /users/cancel(.:format)                                                                  devise/registrations#cancel
    new_user_registration GET    /users/sign_up(.:format)                                                                 devise/registrations#new
   edit_user_registration GET    /users/edit(.:format)                                                                    devise/registrations#edit
        user_registration PATCH  /users(.:format)                                                                         devise/registrations#update
                          PUT    /users(.:format)                                                                         devise/registrations#update
                          DELETE /users(.:format)                                                                         devise/registrations#destroy
                          POST   /users(.:format)                                                                         devise/registrations#create
			  
```

##### Devise : les vues

Avec Devise, il est facile de générer les views que la gem va gérer, il suffit de rentrer la ligne suivante :

```$ rails generate devise:views```

Par défaut, les formulaires sont un peu moches (c'est peu dire). 

Tu peux remplacer le formulaire registration#new par celui-ci :

```ruby
<div class="container">
  <div class="row">
    <div class="col-md-6 offset-md-3">
      <br><br><br>
      <%= form_for resource, as: resource_name, url: registration_path(resource_name), html: { class: "form-signin mt-3" } do |f| %>
        <h1 class="h3 mb-3 font-weight-normal text-center">Sign up</h1>
        <%= devise_error_messages! %>
        <div class="form-group">
          <%= f.label :email, "Email" %><br />
          <%= f.email_field :email, autofocus: true, autocomplete: "email", class: "form-control" %>
        </div>
        <div class="form-group">
          <%= f.label :password %>
          <% if @minimum_password_length %>
          <em>(<%= @minimum_password_length %> characters minimum)</em>
          <% end %><br />
          <%= f.password_field :password, autocomplete: "new-password", class: "form-control" %>
        </div>
        <div class="form-group">
          <%= f.label :password_confirmation %><br />
          <%= f.password_field :password_confirmation, autocomplete: "new-password", class: "form-control" %>
        </div>
        <div class="actions mt-5">
          <%= f.submit "Sign up", class: "btn btn-lg btn-primary btn-block" %>
        </div>
      <% end %>
      <%= render "devise/shared/links" %>
    </div>
  </div>
</div>
```



Gràce à la méthode ```authenticate_user!```, il est aisé de restreindre une page pour les utilisateurs connectés. Va dans le ```home_controller```, puis ajoute en ligne 2 la ligne suivante :

```ruby before_action :authenticate_user!, only: [:secret]``` 
Maintenant tente d'aller sur ```/home/index``` en étant déconnecté : super, Devise t'a redirigé vers l'écran de login. Petit bonus : si tu te connectes à partir de cet écran, Devise va savoir où tu voulais aller et va t'envoyer comme convenu sur /home/index. 

##### Devise : le mailer

quand tu vas recevoir un email pour récupérer ton mot de passe, l'email va te donner un lien à cliquer qui va rediriger vers ton application sur un formulaire pour changer le mot de passe. Il faut donc dire à Rails "pour la production, l'URL de mon app est : monapp.herokuapp.com". Insère donc la ligne suivante dans ton fichier ```config/environments/production.rb``` :

```config.action_mailer.default_url_options = { :host => 'YOURAPPNAME.herokuapp.com' }```


##### Devise : modifier les formulaires

###### Autoriser un attribut en plus dans le signup / edit

Aller dans ```app/controllers/application_controller.rb``` et avoir les lignes de code suivantes :

```ruby
class ApplicationController < ActionController::Base
  protect_from_forgery with: :exception

  before_action :sanitize_devise_params, if: :devise_controller?

  def sanitize_devise_params
    devise_parameter_sanitizer.permit(:sign_up, keys: [:truc_qui_nous_intéresse])
  end
end
```

Puis ajouter dans les views du dossier registration le field qui nous intéresse.

###### Changer les url des méthodes de Devise

On peut changer les urls des méthodes de Devise, en mettant dans le fichier Routes devise_scope.

```ruby
devise_for :users, skip: [:registrations, :sessions]
devise_scope :user do
	get 'inscription', to: 'devise/registrations#new', as: :new_user_registration
	post 'inscription', to: 'devise/registrations#create', as: :user_registration
	get 'profil/editer', to: 'devise/registrations#edit', as: :edit_user_registration
	post 'profil/editer', to: 'devise/registrations#update', as: :users_edit
	get 'login', to: 'devise/sessions#new', as: :new_user_session
	post 'login', to: 'devise/sessions#create', as: :user_session
	delete 'logout', to: 'devise/sessions#destroy', as: :destroy_user_session
end
```

Ainsi, là j'ai dit à Devise de skip les routes pour registrations (#new et #edit les users), et les sessions. Puis dans le devise_scope, je lui paremètre les méthodes importantes. Pour s'inscrire ce ne sera plus /users/new, mais inscription.

###### Changer de controller

Pour changer de controller dans Devise pour certaines fonctions, rien de plus simple. Il faut mettre dans le fichier routes.rb :
```ruby
devise_for :users, controllers: { registrations: 'registrations' }
```

Ainsi, le controller ```app/controllers/registrations_controller.rb``` sera le controller de Devise pour tout ce qui concerne les registrations.


### Active Storage

permet d'ajouter un avatar à l'utilisateur ou une image à nos évènements. 

Pour installer Active Storage sur ton app, lance ```$ rails active_storage:install``` Cette commande a pour effet de générer une migration permettant de créer 2 tables dans ta BDD :

```active_storage_blobs``` : qui contient toutes les métadonnées des fichiers uploadés (taille, nom, type, etc.).
```active_storage_attachments``` : une table jointe entre tes modèles et les uploads.

Active Storage est installé : il faut à présent être capable de lier un fichier qu'on upload avec un objet de notre BDD.
```ruby
class User < ApplicationRecord
  has_one_attached :avatar
end
```


Pour afficher ton avatar sur le site ou dans ta navbar, tu peux utiliser le code suivant :

```ruby
<%if user_signed_in?%>
	<%if current_user.avatar.attached? %>
             <%= image_tag current_user.avatar, alt: 'avatar', size: '30x30' %>
 	<%end%>
 <%end%>
 ```
               

##### Active Storage & Devise

Si tu souhaites ajouter le boutton d'upload d'image au sein d'un formulaire Devise, il faut réaliser deux chose : 
1. Ajouter cette ligne dans le formulaire de la vue :
```ruby 
<%= form.file_field :avatar %>
```

2. Ajouter le paramètre ```:avatar``` dans la liste des attribut autorisés par Device depuis le fichier ```application_controller.rb````

Voic le code avec les paramètre username, first_name, last_name et avatar :

```ruby 
class ApplicationController < ActionController::Base
	  protect_from_forgery with: :exception

  before_action :sanitize_devise_params, if: :devise_controller?

  def sanitize_devise_params
    devise_parameter_sanitizer.permit(:sign_up, keys: [:first_name, :last_name, :username, :avatar])
  end
end
```

Si tu ne souhaites pas utiliser les formulaires Devise pour ajouter une image (ajout d'illustration d'un évènement par exemple), voii la marche à suivre. 

1. Commence par créer un controller (il sera lié à ton formualaire)
```ruby
$ rails g controller avatars new create
 ```

Pour rappel, cette commande va créer un controller 'avatars' avec deux méthodes new & create ainsi que les vues associées. 
La méthode create aura pour rôle d'ajouter un avatar à un user donné.
```ruby
class AvatarsController < ApplicationController
  def create
    @user = User.find(params[:user_id])
    @user.avatar.attach(params[:avatar])
    redirect_to(user_path(@user))
  end
end
```

Explications du code : la ligne ```ruby @user = User.find(params[:user_id])``` nous permet d'identifier l'utilisateur concerné. 
Ensuite nous lui attribuons l'avatar dont la référence est contenue dans ```ruby params[:avatar]``` avec la commande ```ruby @user.avatar.attach(params[:avatar])```. 
Une fois cette association faite, on redirige vers la page show de cet utilisateur en suivant la route dynamique user_path(@user).

Évidemment, pour que cette méthode create fonctionne (notamment la partie avec params[:user_id]), il faut une route adéquate. 
Cette route sera, sans surprise ```ruby resources :avatars, only: [:create]```. 

Mais par contre, elle doit contenir l'information "sur quel utilisateur travaille-t-on ?", autrement dit, elle doit contenir l'id d'un user : on va donc imbriquer la route dans le resources :users pour lui rajouter un /users/user_id/ en amont. 
Ça donne :

```ruby 
Rails.application.routes.draw do
  resources :users, only: [:show] do
    resources :avatars, only: [:create]
  end
end
```

Si tu tapes maintenant un $rails routes, tu vois apparaître les 2 routes qui vont te servir : la page show des users en GET et la page create des avatars en POST. Et cette dernière est bien imbriquée avec un /users/user_id/. Illustration :

```ruby 

             Prefix Verb URI Pattern                                                                              Controller#Action
             user_avatars POST /users/:user_id/avatars(.:format)                                                        avatars#create
                     user GET  /users/:id(.:format)                                                                     users#show
```

Au top ! Il ne reste qu'à finaliser notre formulaire dans la view show.html.erb afin qu'il pointe vers la méthode !



#### Active Storage en production

Si tu vas dans les fichiers de configuration des environnements de développement ```config/environments/development.rb``` et production ```config/environments/production.rb```, tu verras qu'ils contiennent tous les deux cette ligne :

```ruby
config.active_storage.service = :local
```

C'est elle qui indique que, par défaut, Active Storage stockera les fichiers directement sur l'ordinateur qui fait tourner l'app Rails. En développement, pas de souci : ça stocke tout sur ton ordi et tu peux tester que tout marche. 

Par contre en production, ça a ses limites : de nombreux services refuseront de faire fonctionner Active Storage de cette façon, car ils ne veulent pas se retrouver à stocker des fichiers (ça peut vite peser lourd les milliers de photos de profil de tes futurs utilisateurs).

Chez Heroku, ils sont plutôt cools car ils mettent à ta disposition un disque dur éphémère : en gros, sans rien changer dans la configuration d'Active Storage (on reste en :local), ils acceptent les uploads de fichiers pour les sauver en local mais ceux-ci disparaîtront au bout de 24 h. C'est suffisant pour tester ton app… mais pas pour faire un produit pro.

La solution la plus courante consiste à utiliser le service Amazon S3. 

Une fois ta clefs API récupérée, va dans le fichier ```storage.yml``` : tu verras qu'il contient des configurations de bases pour les 3 prestataires recommandés par Rails. Dé-commente la partie qui concerne Amazon :

```ruby
amazon:
	service: S3
	access_key_id: <%ENV['AMAZON_ACCESS_KEY_ID']%>
	secret_access_key: <%ENV['AMAZON_SECRET_ACCESS_KEY']%>
	region: us-east-1 #a priori, si tu es en Europe, il faut mettre ici : eu-west-3
	bucket: your_own_bucket #Mets ici le nom de ton bucket. Par exemple : prisme

```
Maintenant ajoute la ```gem 'dotenv-rails'```, puis ``` bundle install```


Maintenant, crée un fichier .env, rajoute-le à ton .gitignore puis saisis dedans tes clefs d'API récupérée sur Amazon. Exemple :
```ruby
AMAZON_ACCESS_KEY_ID= 'AKIAKOB5SEYHW8APSIYQ'
AMAZON_SECRET_ACCESS_KEY= 'vCxbJWzolEJCLqZ4zXcSBgT5i9mAQCYMSw1zXyu'
```

À présent, pour indiquer à Active Storage de bosser via Amazon (et non plus en local), va dans ```ruby config/environments/production.rb ``` et change la ligne ```config.active_storage.service = :local``` en : ```config.active_storage.service = :amazon```

Dernière étape indispensable pour que Amazon S3 fonctionne : rajoute à ton Gemfile ```gem "aws-sdk-s3",require: false``` puis ```bundle install```.


Ensuite, il faut également saisir tes clefs sur Heroku. 
Tout est expliqué [ici](https://devcenter.heroku.com/articles/config-vars)

### Les Services

Les services sont une feature de Rails (pour changer) qui permettent d'y mettre du PORO (Plain Old Ruby Objects), c'est à dire du code Ruby, que tu pourras appeler où tu veux dans Rails. 

Chaque fichier du dossier services invoque une seule class que tu pourras appeler librement dans ton application rails. 
Ex avec une Class SayHello
```ruby
class SayHello
  def perform
    p "bonjour"
  end
end
```

En cas d'erreur, relancer spring avec ```$ bin/spring stop```


Le fichier s'apelle  say_hello.rb et la classe SayHello. C'est (encore) une convention de Rails. Le nom du fichier doit être le snake_case du nom de la classe.
Les services se rangent dans le dossier ```app/services/.``` Pour les appeler tu peux faire (quasiment partout dans ton app) : ```TonService.new.perform.```

###Dotenv et clefs API

####Komment que ça marche ?
En gros on va mettre toutes les clés dans un fichier .env, puis on pourra les appeler en faisant ```ENV['NOM_DE_LA_CLÉ']``` dans notre programme Ruby. Puis afin d'éviter de pousser le bousin sur Github, on mettra le fichier ```.env``` dans notre ```.gitignore```

Exemple avec un dossier Ruby simple
Il faut commencer par créer un dossier avec nos fichiers, puis le lier à Github. Ne pas oublier d'installer la gem :

```$ gem install dotenv```
Créer un fichier .env qui contiendra nos clés d'APIs au format suivant :
```ruby
TWITTER_API_KEY="my-twitter-api-key"
TWITTER_API_SECRET="shh-so-secret"
```

En appelant la gem dotenv, tout fichier Ruby dans le dossier pourra accéder à ces données. Voici un exemple de programme qui s'appelle kikou.rb et qui appelle les variables :

#### Appelle la gem dotenv
```require 'dotenv'```

Ceci appelle le fichier ```.env ```grâce à la gem dotenv, et enregistre toutes les données enregistrées dans une hash ```ENV
Dotenv.load```

Il est très facile d'appeler les données sensibles du fichier ```.env```, par exemple là je vais afficher ```TWITTER_API_SECRET
puts ENV['TWITTER_API_SECRET']```

Et avant de faire un commit et de push sur Github, nous allons mettre le fichier .env dans le .gitignore. Ainsi, ce fichier ne sera pas push sur Github et il sera très facile d'y gérer nos clés d'API secrètes. Mettre la ligne suivante dans un fichier qui .gitignore :
```
.env
```

Et voilà, notre dossier devrait ressembler à ceci :
```ruby
└── TON_DOSSIER
    ├── .env
    ├── .gitignore
    └── kikou.rb
```
    
Exemple avec une app Rails
Hyper simple, mettre dans les gems de development et tests :

```gem 'dotenv-rails'```
Puis créer un fichier .env dans lequel on mettra les variables. Et mettre .env dans le fichier .gitignore

Et puis c'est tout ! Les variables sont appelées pendant le before_configuration, donc on peut faire notre ```ENV['TA_VARIABLE']``` quand on veut. 

###STRIPE

Rien de mieux que le petit tuto de Stripe
https://stripe.com/docs/checkout/rails

### HEROKU SCHEDULER
Permet de 

Dans ce tuto, Félix nous montre comment lancer des programmes en serveur via Ruby et Rails, et comment les programmer pour qu'ils tournent régulièrement.

https://github.com/felhix/cheat_sheets/blob/master/Ruby/Scheduling_Tasks_Online.md



###Intégrer un template Bootstrap


Tous se passe dans l'Asset Pipeline, intégrer les templates dans le bon dossier etc... Tout un tas de choses un peu durs à comprendre au début, mais c'est pour ça qu'on est là ! Il y a plusieurs dossier assets dans votre app rails :

- app/assets: La partie permettant de stocker le css qu'on écrit à la main. c'est à dire si l'on veut un partial pour écrire du css spécifique à une view, c'est ici qu'on le créera.
- lib/assets: Ici on met nos librairie perso.
- vendor/assets: Ici on intègre nos librairie externes, comme Bootstrap par exemple, c'est donc de ce dossier qu'on aura besoin pour notre template.

On commence par afficher le code source de la homepage du template choisi.
Ensuite récupère les librairies de ton templates dans le dossier assets/vendor de l'archive du template pour les copier dans vendor/assets sur ton app rails. Fais de même avec les répertoire css et js dans le dossier lib/assets.

```ruby
  ├── app
  ├── bin
  ├── config
  ├── db
  ├── lib
  │   └── assets
  │       ├── css [venant de l'archive du template]
  │       └── js [venant de l'archive du template]
  ├── log
  ├── public
  ├── storage
  ├── test
  ├── tmp
  └── vendor
      └── assets
          └── [Intégralité des dossiers assets/vendor du template]
	  
```


De base ton app rails va s'initialiser en checkant uniquement le app/assets, il faut donc lui faire comprendre que tu veux qu'elle recherche aussi dans vendor et lib. Pour cela il faut aller dans ```config/initializers/assets.rb``` et y ajouter ces lignes:
```ruby
Rails.application.config.assets.paths << Rails.root.join('lib')
Rails.application.config.assets.paths << Rails.root.join('vendor')
```

En gros, c'est ici que tu indiqueras au pipeline dans quel répertoire il devra aller chercher tout ce dont il aura besoin. Pour faire appel aux fichiers il faut procéder comme ceci:

Dans application.css:
```
*= require Dossier_de_ta_librairie/Ton_fichier
```

Ton fichier application.css devrait ressembler à quelque chose comme ceci:
```ruby
*= require_tree .
*= require_self
*= require css/bootstrap.min

*= require css/bootstrap.min
*= require css/swiper.min
*= require hamburgers.min
*= require animate.min
```

Dans application.js:
```
//= require Dossier_de_ta_librairie/Ton_fichier
```

ton fichier application.js devrait ressembler à quelque chose comme ceci:
```
//= require rails-ujs
//= require activestorage
//= require turbolinks
//= require_tree .

//= require jquery.min
//= require popper.min
//= require js/bootstrap.min
//= require slidebar/slidebar
```
Lorsque tu appelles des ressources se trouvant dans les sous répertoires du dossier lib de ton app, il ne faut pas ajouter le sous repertoire, mais uniquement le nom du fichier que tu souhaites appeler.

### Interface Admin

Pour cette ressource, nous allons voir trois façons de faire une interface admin :

- Brancher sa base de données à une SaaS qui fait tout le travail pour toi
- Installer une gem qui va générer des scaffold administrateurs
- Coder à la main son interface administrateur

#### 1. Une SaaS
Il existe des services où tu as juste à brancher des clé d'APIs, qui auront un accès à ta BDD, et qui te vendent une interface toute faite customisable à sa guise. ForestAdmin fait exactement cela. ForestAdmin est une entreprise française originaire de l'un des startup studios les plus célèbre au monde : eFounders.

Tu as juste à installer la gem, mettre les clés d'APIs, pousser en prod, créer ton compte, puis à toi la gloire.

#### 2. Une gem
Si jamais tu trouves ForestAdmin trop cher / pas assez permissif, voici une sélection de gems qui te permettent d'avoir une interface admin :

- administrate de nos amis Thoughbot (que l'on recommande)
- rails_admin de sferik (le créateur de la gem Twitter)

#### 3. Du code
La technique que l'on utilise personnellement : coder son dashboard Admin à la main. Cela a plusieurs avantages : déjà, cela permet une maîtrise totale de ce que tu veux pouvoir y modifier. Ensuite, comme les views sont souvent des scaffolds purs et durs, et bien cela va assez vite au final. Voici quelques conseils pour faire son dashboard admin à la main :


Comment ranger proprement son code ? Avec la notion de namespace, tu peux facilement mettre tout ce qui concerne le dashboard admin dans un dossier bien rangé. Puis comme cela tu peux avoir deux controllers pour les users : un pour l'interface normale, puis un autre dans le namespace admin. Le premier va juste faire des opérations limitées, puis l'autre c'est open bar pour faire ce que tu veux.

Accès aux admins ?
C'est simple : tu fais une méthode ```check_if_admin``` en ```before_action``` dans tous tes controllers Admin. Puis tu ajoutes un attribut ```is_admin``` booléen à tes utilisateurs. Si le ```current_user``` n'est pas ```admin```, tu l'envoies chier quand il arrive sur n'importe quelle action de ton dashboard admin.

🤓 QUESTION RÉCURRENTE
Mais dis-donc Jamy, vaut-il mieux faire un attribut is_admin aux utilisateurs, ou bien ajouter un model Admin indépendant du model User ?
Très bonne question, et il y a deux écoles qui se valent. Voici ce qui peut faire pencher la balance vers un côté ou l'autre :

L'attribut is_admin est bien plus rapide à coder que la génération d'un model via Devise pour les admins
Si tes administrateurs ne peuvent pas être utilisateurs de ton site (trombinoscope des élèves d'une école par exemple), c'est mieux d'avoir un model Admin
C'est légèrement plus sécurisé d'avoir le model Admin, car tu auras moins de chance de créer un model que de faire user.update(is_admin: true)


### RAILS - LES PARTIALS


Les partials permettent de mettre des bouts de code de front dans un fichier à part. 
- Le premier est que tu restes trop dans l'état d'esprit page par page (#so_2005) alors que le web moderne se pense composant par composant
Prenons l'exemple de l'organisme d'un formulaire de contact. Ce dernier sera appelé principalement dans le controller contacts, sur la méthode new. Ainsi dans ta
```view app/views/contacts/new.html.erb```
 tu mettrais :

- Le premier est que tu restes trop dans l'état d'esprit page par page (#so_2005) alors que le web moderne se pense composant par composant
```ruby
<%= render "form" %>
```

Puis tu vas créer un fichier ```app/views/contacts/_form.html.erb``` dans lequel tu mettrais ton organisme du formulaire de contact. Et voilà ! Ton application va appeler ton formulaire très facilement. Grâce à ce système de partials, tu peux penser ton application Rails comme une application moderne : chaque page est une succession d'organismes. Et chaque organisme va être appelé par une ou plusieurs pages. 

⚠️ ALERTE ERREUR COMMUNE
Comme tu peux le voir, la partial s'apelle en faisant ```<%= render "form" %>``` et le fichier s'appelle ``` _form.html.erb ``` et commence par un _. C'est une convention de rails qui permet de distinguer facilement une view (sans _) d'une partial (avec _).

Donc, tu appelles les partials sans _, mais les fichiers se nomment avec un _.

#### Où mettre ses partials
Tu peux mettre les partials dans n'importe quel dossier de views et les appeler depuis n'importe quelle view. Voici comment faire :

- Si la partial est dans le même dossier que la view, tu l'appelles en faisant ```<%= render "ta_partial" %>```
- Si la partial n'est pas dans le même dossier que la view, tu précises le dossier. 
Ainsi si je voulais appeler la partial ```app/views/contacts/_form.html.erb``` dans la view edit de la ressource users, tu écrirais dans ```<%= render "contacts/form" %>```

Une partial que l'on appelle depuis le fichier ```application.html.erb``` (header, footer, alert) se met dans le dossier ```app/views/layouts/```
Une partial que l'on appelle dans plusieurs pages de l'application (une bannière) se mettra dans un dossier ```app/views/shared/```
Une partial qui concerne une ressource précise (une liste, un formulaire, un affichage précis) se met dans le dossier de la ressource. Ainsi, le formulaire pour créer un événement serait un fichier ```app/views/events/_forms.html.erb```


Tu peux faire plein d'opérations sur les partials, comme par exemple passer des variables locales (pratique quand tu veux changer la photo de ta bannière). 

#### application.html.erb
Il existe un fichier qui s'appelle ```app/views/layouts/application.html.erb``` qui contient les fondements du fichier html de ton application. Le head, et le body. Tu peux y mettre des CDNs facilement, et notamment le fondement d'une page web.

Certaines parties de ton application se retrouvent sur toutes les pages :

- Le header
- Le footer
- Les alertes
Ainsi, nous te conseillons de faire une partial pour chacun de ces éléments (que tu mettras dans ```app/views/layouts/```), et les mettre dans le body du fichier ```application.html.erb```. Cela devrait ressembler à ceci :
```ruby
<%= render 'layouts/header' %>
<%= render 'layouts/flash'%>
<%= yield %>
<%= render 'layouts/footer'%>
```
🤓 QUESTION RÉCURRENTE
Dis donc Jamy, comme les partials sont dans le même dossier que le fichier application.html.erb, ne faut-il pas faire simplement ```<%= render 'ma_partial' %>``` pour les appeler et non pas ```<%= render 'layouts/ma_partial' %>``` ? 🤔

Oui pour tous les autres dossiers de views, mais pas celui-là 😉

Ainsi, chaque page de ton application contiendra le header, le footer, et tu n'auras qu'à mettre les dernières molécules concernant la page 🤠

## LES ALERTES

#### Le Flash

Tu te souviens du hash session qui te servait à stocker ce que tu voulais pendant une session d'utilisateur ? Figure-toi qu'il existe un autre hash dans Rails nommé flash et qui ne stocke les données que pour une page (session reste pour toute la session, mais flash disparait au bout d'une page).

Ce système est utilisé principalement par Rails pour annoncer des informations aux utilisateurs. Ainsi, quand on crée un utilisateur, on peut faire ```flash[:success] = "Utilisateur bien enregistré"```, ce qui aura pour effet de le stocker dans un hash que l'on n'aura qu'à appeler dans notre view à la prochaine page.

Tu peux créer facilement un hash flash qui tiendra sur une page de ton application. Ce hash sera modifié dans les controller et affiché dans les views. Avec Bootstrap, c'est facile de transformer ce hash en jolies alertes. On va commencer par mettre dans son fichier application.html.erb les lignes suivantes dans le body :

```ruby
<%= render 'layouts/header' %>
<%= render 'layouts/flash'%>
<%= yield %>
<%= render 'layouts/footer'%>
```

Puis dans le fichier ```app/views/layouts/_flash.html.erb``` :

```ruby
<% flash.each do |type, msg| %>
  <div class="alert <%= bootstrap_class_for_flash(type) %> alert-dismissable fade show" role="alert">
    <%= msg %>
  <button type="button" class="close" data-dismiss="alert" aria-label="Close">
    <span aria-hidden="true">×</span>
  </button>
  </div>
<% end %>
```

Enfin, dans le fichier ```application_helper.rb``` :

```ruby
def bootstrap_class_for_flash(type)
  case type
    when 'notice' then "alert-info"
    when 'success' then "alert-success"
    when 'error' then "alert-danger"
    when 'alert' then "alert-warning"
  end
end
```

Rails utilise par convention 4 types de flash :

```notice```, pour les informations
```success```, pour les opérations fructueuses
```alert```, pour annoncer une information importante
```error```, pour annoncer une erreur

Ainsi, si tu veux créer un flash pour annoncer que le post que tu voulais créer a bien été créé en base, tu auras un controller qui ressemble à ceci :
```ruby 
def create
  @post = Post.new(post_params)
  if @post.save
    flash[:success] = "Post was successfully created."
    redirect_to @post
  else
    flash.now[:error] = @post.errors.full_messages.to_sentence
    render :new
  end
end
```

#### Flash.now ou flash ?
Lorsque tu n'arrives pas à faire ```.save```, tu fais ```render :la_view``` alors que tu fais un ```redirect_to``` lorsque tu arrives à faire ```.save.``` Pourquoi ? Car en faisant render tu gardes la variable d'instance ```@user```, ce qui est bien pour pouvoir afficher ses erreurs ou encore préremplir le formulaire avec ce que l'utilisateur a mis auparavant. Le soucis est que ton flash va s'afficher pour deux pages puisque tu ne fais pas de redirection. Il te faudra faire un ```flash.now[:ton_type] = "Ton message".```

Donc pour rappel :

Si tu fais un ```render :some_view```, tu fais un ```flash.now[:ton_type] = "Ton message"```
Si tu fais ```redirect_to``` , tu fais un ```flash[:ton_type] = "Ton message"```



#### Ajouter des images à ma home en javascript

Tableau d'image stcokées en ligne :
```let imagesArray = ["https://img.icons8.com/color/480/000000/html-5.png", "https://img.icons8.com/color/480/000000/css3.png", "https://img.icons8.com/color/480/000000/javascript.png", "https://img.icons8.com/color/480/000000/ruby-programming-language.png", "https://img.icons8.com/color/480/000000/bootstrap.png", "https://img.icons8.com/color/480/000000/github.png", "http://jeudisdulibre.be/wp-content/uploads/2014/02/Ruby_on_Rails-logo.png", "https://avatars2.githubusercontent.com/u/25484553?s=200&v=4", "https://img.icons8.com/color/480/000000/heroku.png"];```

Fonction js qui ajoute une image pour chaque card bootsrap
```
function populateImages() {
var collection = document.querySelectorAll(".card-img-top");
console.log(collection);

for  (var i = 0; i < collection.length; i++) {
   collection[i].setAttribute("src", imagesArray[i]);
   console.log(document.querySelector(".card-img-top"));
}};
```

## ASSET PIPELINE
L'Asset Pipeline permet de gérer les assets d'une application Rails (CSS, JavaScript, images) pour la production. Ce concept va disparaître d'ici quelques semaines mais est pour l'heure une partie importante de Rails.

Tu peux commencer à regarder Weback, qui remplacera Rails dans quelques temps.
https://github.com/rails/webpacker





 ## AJAX FOR RAILS
 
 ### Formulaire Ajax
 Ajouter l'atttribut ```remote: true``` dans un formulaire ou un boutton permet de soumettre de manière asynchrone via AJAX.
 
 Un exemple de code pour le controller :
 ```ruby
 def create
  @book = Book.new(book_params)
    if @book.save
      respond_to do |format|
        format.html { redirect_to books_path }
        format.js
      end
    end
  end

  private

  def book_params
    params.permit(:title, :author)
  end
  ```
  
  Je veux que lorsque j'ajoute un titre, il s'ajoute à ma liste sur la page index sans recharger la page. Je vais donc créer un partial ```_book.html.erb``` dans mon dossier views:

```ruby
<%= book.title %> by <%= book.author  %>
```
Maintenant, je vais écrire mon script ```create.js.erb``` dans le dossier ```app>views>books``` qui s'occupera de rajouter mon élément dans le ul de ma page index
```javascript 
document.getElementById("book").querySelectorAll('ul')[0].insertAdjacentHTML("beforeend", "<%= j render 'book', book: @book %>")
```


Dans ce script, je sélectionne l'élément ul puis j'ajoute l'HTML de mon partial à la fin de mon ul. Ce script est à affiner pour par exemple vider le contenu des inputs quand il est soumis, etc...




Pour éviter les problème de double quote, on utilise la fonction escape_javascript. Elle assure que vos caractères spéciaux ne casse pas votre script ! Ex :
```javascript
document.getElementById("books").appendChild("<%= escape_javascript(render "book", book: @book) %>");
alert("book created");
```

Mais que fait ce script ? C'est très simple : il va sélectionner l'élément du DOM qui t'intéresse et va lui rajouter le partial book et lui passer la variable locale @book.


Ce script peut également être en Jquery. Voici un exemple d'utilisation avec du jQuery. Voici un exemple d'tuilisation pour afficher le contenu d'un mail. j remplace escape_javascript.

```javascript
<%# $('show-mail').html("<%= j (render 'email_show')") %>
$( document ).ready(function() {

$('#show-mail').html("<%= j (render 'email_show') %>");
$("#show-mail").show();
$('#mail-list').html("<%= j (render 'emails_list') %>");

  $(document).on("click", "#close-show", function(){
    $("#show-mail").hide();
  });
});
```
Un exemple de controller qui affiche le contenu du mail et modifie son statut (read:true||false)
```ruby
  def show
    @email = Email.find(params[:id])
    @emails=Email.all # needed for sidebar, probably better to use a cell for this
    @email.toggle!(:read)
    # @email.update_attribute(:read, [true])
        respond_to do |format|
        format.html # show.html.erb
        format.js {render layout: false}
    end
  end
```

 
 http://thehackingproject.herokuapp.com/dashboard/weeks/8/days/3?locale=fr
 
 # A FAIRE
 
### FORM WITH
 https://m.patrikonrails.com/rails-5-1s-form-with-vs-old-form-helpers-3a5f72a8c78a
 
 ###A faire 
explication sur appel en backgroudn CSS des images du pipeline
background-image: url("home.jpg");

###A faire
cheatsheet Javascript
