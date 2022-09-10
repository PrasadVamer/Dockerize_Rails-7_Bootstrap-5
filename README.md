# Dockerize_Rails-7_Bootstrap-5
How to easily build a Ruby on Rails 7 with Bootstrap 5 on Docker Compose development environment


Operating environment

I just installed Docker on Windows 11, so basically, if you have a Docker environment, you should be able to start it anywhere.

    $ docker -v
    Docker version 20.10.17, build 100c701 


First, let's create five files.

All you need to create from scratch is the working folder and the following "five files".

Dockerfile

Gemfile

Gemfile.lock

docker-compose.yml

entrypoint.sh


# Dockerfile

    # syntax=docker/dockerfile:1
    FROM ruby:3.1.2
    RUN apt-get update -qq \
     && apt-get install -y nodejs postgresql-client npm \
     && rm -rf /var/lib/apt/lists/* \
     && npm install --global yarn
    WORKDIR /myapp
    COPY Gemfile /myapp/Gemfile
    COPY Gemfile.lock /myapp/Gemfile.lock
    RUN bundle install

    # Add a script to be executed every time the container starts.
    COPY entrypoint.sh /usr/bin/
    RUN chmod +x /usr/bin/entrypoint.sh
    ENTRYPOINT ["entrypoint.sh"]
    EXPOSE 3000

    # Configure the main process to run when running the image
    CMD ["rails", "server", "-b", "0.0.0.0"]

Regarding the Ruby version, it is a good idea to check the "current stable version" on the official website and update it.


# entrypoint.sh
It is a script that appears in the Dockerfile, but it seems that this area can be left as it is.

    #!/bin/bash
    set -e

    # Remove a potentially pre-existing server.pid for Rails.
    rm -f /myapp/tmp/pids/server.pid

    # Then exec the container's main process (what's set as CMD in the Dockerfile).
    exec "$@"


# Gemfile

    source 'https://rubygems.org'
    gem 'rails', '~> 7.0', '>= 7.0.3.1'
    

# Gemfile.lock

Create an empty Gemfile.lock and leave it like that.
    
    
# docker-compose.yml

For the time being, is it safer to fix the version of PostgreSQL as well

    version: "3"
    services:
      db:
        image: postgres:14.5-alpine
        volumes:
          - ./tmp/db:/var/lib/postgresql/data
        environment:
          POSTGRES_PASSWORD: password
      web:
        build: .
        command: bash -c "rm -f tmp/pids/server.pid && bundle exec rails s -p 3000 -b '0.0.0.0'"
        volumes:
          - .:/myapp
        ports:
          - "3000:3000"
        depends_on:
          - db

If you use it after a long time since updated here, the version may go up without you noticing it, causing a problem with the consistency with the mounted data, and you may not be able to start it.

# Let rails new's go.
Now, when the files are ready, just run the following command and the necessary files should be automatically generated.

    docker-compose run --rm --no-deps web rails new . --force --database=postgresql
    
    
Of the large number of generated files, only the database configuration file needs to be edited before launching the container.

In config/database.yml add these lines as in the image.

    host: db
    username: postgres
    password: password


![databaseyml_Update](https://user-images.githubusercontent.com/98350622/189475271-b6ebd6c5-97de-49c3-98f7-61c9988cd061.png)


# Dealing with the errors
Upon building the container we may encounter errors like this. Otherwise you are good to add bootstrap to your project.

  ![Rails_Errors](https://user-images.githubusercontent.com/98350622/189470423-69056915-e1d9-4a25-8aac-c92abaf09394.png)
  
  
Add sprocket-rails to the Gemfile. 

If sprockets-rails already exist in your Gemfile replace it with the following line;

    gem 'sprockets-rails', :require => 'sprockets/railtie'
    
Build the container

    docker-compose build

Install importmap

    docker-compose run --rm web rails importmap:install
    
Install turbo and stimulus

    docker-compose run --rm web rails  turbo:install stimulus:install
    

# Adding Bootstrap

Add cssbundling to the Gemfile.

    gem 'cssbundling-rails'
    
Build the container

    docker-compose build
    
Install Latest Bootstrap

    docker-compose run --rm web rails css:install:bootstrap
    
# Check Boostrap is working

Generate a controller with a view as folllows

    docker-compose run --rm web rails g controller home index

Copy paste the following Nav Bar taken from bootstrap 5.2 to the index file in, app\views\home\index.html.erb

    <nav class="navbar navbar-expand-lg bg-light">
      <div class="container-fluid">
        <a class="navbar-brand" href="#">Navbar</a>
        <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarSupportedContent" aria-controls="navbarSupportedContent" aria-expanded="false" aria-label="Toggle navigation">
          <span class="navbar-toggler-icon"></span>
        </button>
        <div class="collapse navbar-collapse" id="navbarSupportedContent">
          <ul class="navbar-nav me-auto mb-2 mb-lg-0">
            <li class="nav-item">
              <a class="nav-link active" aria-current="page" href="#">Home</a>
            </li>
            <li class="nav-item">
              <a class="nav-link" href="#">Link</a>
            </li>
            <li class="nav-item dropdown">
              <a class="nav-link dropdown-toggle" href="#" role="button" data-bs-toggle="dropdown" aria-expanded="false">
                Dropdown
              </a>
              <ul class="dropdown-menu">
                <li><a class="dropdown-item" href="#">Action</a></li>
                <li><a class="dropdown-item" href="#">Another action</a></li>
                <li><hr class="dropdown-divider"></li>
                <li><a class="dropdown-item" href="#">Something else here</a></li>
              </ul>
            </li>
            <li class="nav-item">
              <a class="nav-link disabled">Disabled</a>
            </li>
          </ul>
          <form class="d-flex" role="search">
            <input class="form-control me-2" type="search" placeholder="Search" aria-label="Search">
            <button class="btn btn-outline-success" type="submit">Search</button>
          </form>
        </div>
      </div>
    </nav>
  
Add the root path as follows in file config\routes.rb.

    root 'home#index'
      
Now let's start the container (while building) by running the following command:

    docker-compose up --build

Up to this point, the startup itself should have been completed, but an error will occur if the database to be connected does not exist, so create it with the following command.

    docker-compose run --rm web rails db:create
    
Start the container again

    docker-compose up --build

# View the page in your browser
Now, let's access http://localhost:3000/ and check if Rails and Bootstrap is working properly.

![DropDownNotWorking](https://user-images.githubusercontent.com/98350622/189471842-2aa50972-a94b-4988-bfab-4251b1692fc9.png)

Check the dropdown link is working or not.
If it is working you are good to go.

# DropDown not working
Add the following script tag taken from bootstrap 5.2 to the body of app/views/layouts/application.html.erb

    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.2.1/dist/js/bootstrap.bundle.min.js" integrity="sha384-u1OknCvxWvY5kfmNBILK2hRnQC3Pr17a+RTT6rIHI7NnikvbZlHgTPOOmMi466C8" crossorigin="anonymous"></script>
    
Your app\views\layouts\application.html.erb should look like as follows;

![ScriptInApplication html erb](https://user-images.githubusercontent.com/98350622/189471417-533ee4d6-7019-4b7b-a233-2adeb91160e8.png)

Refresh the page.
YOU ARE GOOD TO GO.

![DropDownWorking](https://user-images.githubusercontent.com/98350622/189471843-bb60ec9b-1d2b-4695-8132-9d8604d7ee38.png)
