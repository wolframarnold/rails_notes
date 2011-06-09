!SLIDE

## MVC

![MVC](images/mvc.png)

# Saving for posterity

    git init

Create `.gitignore` file

    git status
    git add .
    git status
    git commit -m "Initial commit"

# Deploy

    heroku create

    git push heroku master

# Onto a Real (Demo) App

## We're following Michael Hartl's Rails Tutorial

<http://ruby.railstutorial.org/>

# Create App

    rails new demo_app

    echo "rvm use 1.9.2@rails3" > .rvmrc
    
# Create a Scaffold for User

A scaffold is a vertical slice of application, including a database migration, a model, as well as controller, and views for basic CRUD operations.

    rails generate scaffold User name:string email:string

# A note on Rails conventions

* Tables
    * Lower case and plurals, e.g. `users`
    * They represent many users

* Models
    * CamelCase and singular, e.g. `User`
    * Models are Ruby Classes
    * An instance of a model represents a single database record

# Migrating

    rake db:migrate

# A look at the model

    rails console

    > ActiveRecord::Base.logger = Logger.new(STDOUT)
    > User.create!(:name => 'Joe', :email => 'joe@example.com')
    > User.find(1)


# Restful Routes

    map.resources :users (config/routes.rb)

    rake routes

    users_path, users_url "named route methods"
    GET		/users			→ “index” action
    POST	/users			→ “create” action

    new_user_path, new_user_url
    GET		/users/new		→ “new” action

    edit_user_path(id), edit_user_url(id)
    GET		/users/:id/edit	→ “edit” action with ID

    user_path(id), user_url(id)
    GET		/users/:id		→ “show” action with ID
    PUT 	/users/:id		→ “update” action with ID
    DELETE  /users/:id		→ “destroy” action with ID


!SLIDE

![MVC detailed](images/mvc_detailed.png)

Source: (c) 2011, Michael Hartl, railstutorial.com

# Generated controllers

* Take a look at `app/controllers/users_controller`
* Note that each restful route correspond to a method (an "action")
* There are 7 of them:
    * `new`, `edit` -- serve forms to create a new or edit existing records
    * `index`, `show` -- list all or a single record
    * `create`, `update` -- process a form post for creating or updating a record
    * `delete` -- removing a record

# Problems with Scaffold

* No data validations
* No authentication or access protection
* Both HTML and XML responses -- often not warranted
* Tests are boilderplate, and not very robust
* No real understanding

# Models

# Rails Console

    rails c[onsole]

    ActiveRecord::Base.logger = Logger.new(STDOUT)

    u = User.new
    u = User.new(:name => "Joe Smith", :email => "joe@example.com")
    u.save
    u.save!
    u = User.create(:name => "Joe Smith", :email => "joe@example.com")
    u = User.create!(:name => "Joe Smith", :email => "joe@example.com")

    u = User.find(1)
    u = User.where(:email => "joe@example.com").all

# Attribute Methods

Each column in the database auto-magically turns into a method on the model object, both setters and getters.

    @@@ruby
    u = User.new
    u.name = "Joe"
    u.name  # => "Joe"

Behind the veil of magic: `method_missing`

# Instantiating vs. Creating

## In-memory operations
* `User.new({...})`
* `u.attributes = {...}`
* `u.name = ...`

`save` commits changes of an in-memory object to the database.

## Direct-to-DB operations

* `User.create({...})`
* `u.update_attribute(key, val)`
* `u.update_attributes({...})`

# Validations

* Secure data integrity at the model level
* Prevents bad data from entering the database
* Un-burdens controller or UI from messy validation code

!SLIDE

    @@@ruby
    class User < ActiveRecord::Base
        validates :name, :email
                  :presence => true,
                  :length   => {:maximum => 256}

        ...
    end

# Active Record API

* find
* create
* count
* valid?
* changed?
* errors
* attributes, attributes=
* update_attribute(s)
* destroy

Reference: <http://railsapi.com/>

# Queries

## Compose queries with

* where
* order
* limit
* include
* join

## Then execute them with

* first
* last
* all

Reference: <http://guides.rubyonrails.org/active_record_querying.html>

# SQL Injection Guards

## NEVER use STRING INTERPOLATION in SQL conditions!!!

    str = 'joe'

    User.where("name = #{str}")   # This is UNSAFE!!! DON'T DO IT!

    User.where(["name = ?", str])  # This is safe

    # Alternatively, with hash keys substitution
    User.where(["name = :str AND email = :email", :str => str, :email => 'joe@example.com'])

# Add RSpec
Open `Gemfile` and add:

    group :development do
      gem 'rspec-rails', '~> 2.6.0'
    end
    group :test do
      gem 'rspec', '~> 2.6.0'
    end

Then, run:

    bundle install

Bundler is a tool to manage a project's gem dependencies. This command will check and, if necessary, install all the gems and specific versions according to the `Gemfile` manifest.

The `~>` syntax for specifying versions mean that bundler will tolerate minor release revisions (e.g. `"~> 1.0.0"` will also accept version 1.0.1, but not 1.1.0)

# Run RSpec's installer

    rails g rspec:install


# CRUD Exercise
<!-- this should go into a 'learn_rails' project for TFT -->
## Class Methods

* new
* find
* create
* count

## Instance Methods

* attributes, attributes= (operates in-memory only)
* update_attribute(s) (saves right away)
* destroy

## Exercise

Create a new file in your Rails directory tree: `spec/models/user_spec.rb`. Then paste the following content in:

    require 'spec_helper'
    describe User do

      it 'can create a user and retrieve it back by ID' do
        u = User.create(:name => "Joe", :email => "joe@example.com")
        User.find(u.id).should == u
      end

      it 'can instantiate a user with new, then save it, and find it back by ID'

      it 'can update an existing user name with "attributes=" and save, then verify it is been written correctly'

      it 'can update an existing user name with "update_attribute" and save, then verify it is been written correctly'

      it 'can count the number of users'

      it 'can destroy a user, then verify it is gone'

    end

# Destroyed Objects

##... are frozen

    @@@ruby
    u = User.create(:name=>'Joe', :email=>'joe@example.com')
    u.destroy

    u.name  # => "Joe"
    u.id    # => ID

    # but:
    User.find(u.id)  # => Kaboom!

    u.frozen?  #  => true


Frozen objects can't be modified

# Query Exercise

* ActiveRelations Query language

## Compose queries with

* where
  * where(:name => 'Joe')
  * where(["email LIKE ?", '%example.com'])
* order
  * order('name ASC')
* limit
* include
* join

## Then execute them with

* first
* last
* all
* count


Create a new file in your Rails directory tree: `spec/models/user_queries_spec.rb`. Then paste the following content in:

    require 'spec_helper'

    describe User do

      it 'can retrieve a user by email address'

      it 'retrieves all users sorted by name'

      it 'retrieves all users containing the partial name "Smith"'

      it 'retrieves all users containing the partial name "Smith" sorted by email'

    end

# Validation Exercise

* valid?
* changed?
* errors

Create a new file in your Rails directory tree: `spec/models/user_validations_spec.rb`. Then paste the following content in:

    require 'spec_helper'

    describe User do

      it 'is invalid unless name is present'

      it 'the error message says "must be present"'

      it 'when changing an attribute, changed? flag is true'

      it 'it is only valid with both name and email given'

    end


# Microposts

    rails generate scaffold Micropost content:string user_id:integer

    rake db:migrate


# Validations for Micropost

    @@@ruby
    class Micopost < ActiveRecord::Base
        validates :content,
                  :presence => true,
                  :length   => {:maximum => 140}
        ...
    end

<http://guides.rubyonrails.org/active_record_validations_callbacks.html#validation-helpers>

# Associations

    @@@ruby
    class User < ActiveRecord::Base
      has_many :microposts
    end

    class Micropost < ActiveRecord::Base
      belongs_to :user
      
      validates :content, :length => { :maximum => 140 }
    end

!SLIDE

![Micropost User Association](images/micropost_user_association.png)

# A closer look

    rails console

    > u = User.find(1)
    > u.microposts.create(:content => "Hello World")
    
# Deploy

    # create .gitignore file
    git init
    git add .
    git commit -m 'initial commit'

    heroku create
    git push heroku master
    heroku rake db:migrate

# Onto a Real (Real) App

Rails Tutorial Chapter 3 (page 78ff)

     rails new sample_app -T

-T causes the rails generator to skip the default test unit directories. We'll be using RSpec instead

# Be a good citizen

## ...and put something useful into the README file.

Live Long and Prosper!

# Commit

It's a good time to commit now.

If you're using RubyMine, open your `.gitignore` file and add a line with `.idea/` to it. This excludes RubyMine configuration files from being tracked.

    git add .
    git commit -m "First commit, RSpec installed"


# Add Static Pages

See RailsTutorial, page 86

    rails generate controller Pages home contact about

Note the new controller file `pages_controller.rb`, the view templates for `pages` and the changes to `config/routes.rb`
 
# Adding content

For `app/views/pages/home.html.erb`

<http://ruby.railstutorial.org/chapters/static-pages#code:home_view_full_html>


For `app/views/pages/contact.html.erb`

<http://ruby.railstutorial.org/chapters/static-pages#code:contact_view_full_html>

For `app/views/pages/about.html.erb`

<http://ruby.railstutorial.org/chapters/static-pages#code:about_view_full_html>

# Setting Instance Variables in the Controller

    class PagesController < ApplicationController
      def home
        @title = "Home"
      end

      def contact
        @title = "Contact"
      end

      def about
        @title = "About"
      end
    end

<http://ruby.railstutorial.org/chapters/static-pages#code:pages_controller_with_title>

# Use them

## Replace the hard-coded strings "About" and "Contact" in the views with instance variables

(You may also want to add a `home` view template.)

# DRY things up with layouts

* Remove the repetitive parts of the about us and contact us templates
* Put them into a layout file which will call `yield` to insert the template

# Exercise

* Add a `home` and a `help` pages
* What files do you need to add?
* What additional declarations do you need to make?
* Try out things and see if the Rails error messages are helpful in guiding you.

# Exercise

* What is a new developer adds a new action, but forgets to set the `@title` variable?
* Or sometimes, we're happy with a generic title, and don't want to set it.
* Instead of using `@title` in the layout, define a `title` helper method in `application_helper.rb` that uses @title if defined, or sets a fallback string if not.

# Stylin'

Download: <http://github.com/joshuaclayton/blueprint-css/zipball/master>

Extract the `blueprint` subdirectory to `public/stylesheets`

And add stylesheet links to layout file:

    <%= stylesheet_link_tag 'blueprint/screen', :media => 'screen' %>
    <%= stylesheet_link_tag 'blueprint/print',  :media => 'print' %>

<http://ruby.railstutorial.org/chapters/rails-flavored-ruby#code:layout_with_stylesheets>

# Navigation

Add some styling and navigation links to the layout

<http://ruby.railstutorial.org/chapters/filling-in-the-layout#code:layout_new_structure>

Download the logo to `public/images`

<http://railstutorial.org/images/sample_app/logo.png>

# More CSS

Add a file `public/stylesheets/custom.css` from:

<http://ruby.railstutorial.org/chapters/filling-in-the-layout#code:header_content>

And append to it:

<http://ruby.railstutorial.org/chapters/filling-in-the-layout#code:navigation_css>

As well as:

<http://ruby.railstutorial.org/chapters/filling-in-the-layout#code:signup_css>

And for rounded corners:

<http://ruby.railstutorial.org/chapters/filling-in-the-layout#code:round_css>

And for the footer:

<http://ruby.railstutorial.org/chapters/filling-in-the-layout#code:footer_css>

# Use partials to de-clutter the layout

Create a stylesheets partial: `app/views/layouts/_stylesheets.html.erb`

<http://ruby.railstutorial.org/chapters/filling-in-the-layout#code:stylesheets_partial>

Create a header partial: `app/views/layouts/_header.html.erb`

<http://ruby.railstutorial.org/chapters/filling-in-the-layout#code:header_partial>

Create a footer partial: `app/views/layouts/_footer.html.erb`

<http://ruby.railstutorial.org/chapters/filling-in-the-layout#code:footer_partial>

Refactored layout:

<http://ruby.railstutorial.org/chapters/filling-in-the-layout#code:layout_with_partials>

**Note:** File names for partials start with an underscore `_` but when referred to in the `render` command, they
don't have the underscore!

# Named routes

The generator created entries in `config/routes` like so:

    get "pages/contact"

If you switch this to...

    match '/about', :to => 'pages#about'

...you'll get so-called "named routes" with methods:

    about_path  # relative url (without hostname)
    about_url   # absolute url (with hostname and protocol)

You can verify this by running

    rake routes

# Exercise

## Replace the HTML hand-coded links in the `_header` `_footer` and footer partials with the named route methods.

# Exercise

## Replace the local variable logo in the header partial with a helper method of the same name.
