# Rails-Postgres-on-Docker
This is a rails demo app that links to postgres and running on docker

## Prerequisites

- [What and Why Docker](https://zegetech.com/blog/2018/10/29/what-and-why-docker.html)
- [Developing with Docker](https://zegetech.com/blog/2018/11/08/developing-with-docker.html)
- [What and Why Ruby on Rails](https://zegetech.com/blog/2018/10/17/why-ruby-on-rails.html)
- [Postgresql for debian users](https://www.postgresql.org/download/linux/debian/)

### Step 1 - Setting and starting a new rails app

- Open your terminal and run
~~~
	rails new [your_app_name]
~~~

Then,
~~~
	rails server
~~~

- Once successful, open up your browser and go to localhost:3000 or the port that you've allocated to the rails server. 

You'll be able to see a rails welcome page. 

For kali users, you may run into problems because of its 'incompatibility' with ruby. You'll have to reinstall it using *rvm* and not directly with *apt-get*. More [here](https://amionrails.wordpress.com/2014/02/10/install-rvm-ruby-on-rails-and-ruby-on-kali-linux/)

### Step 2 - Dockerizing your rails app

Navigate to the root of your rails app. Then in the terminal,

~~~
	touch Dockerfile
~~~

This will create an empty dockerfile. Visit the link above on What and why docker for more on Dockerfile.

Open the newly created Dockerfile with your favorite editor, I use sublime. Then, copy the following:

~~~
FROM ruby:2.5.1
MAINTAINER your_name
RUN apt-get update && apt-get install -y build-essential && apt-get install -y nodejs
WORKDIR /
COPY Gemfile /
COPY Gemfile.lock /
RUN gem install bundler && bundle install
COPY . ./
EXPOSE 3000
ENTRYPOINT ["bundle", "exec"]
CMD ["rails","server","-b","0.0.0.0"]
~~~

Save it and run:

~~~
	docker build -t [tagname] .
~~~

This will build your image from the Dockerfile. At this point, you can actually run the docker image and view it on your browser with:

~~~
	docker run -p your_image_name 
~~~

If you are having problems with port problems, in your terminal,

~~~
	docker ps
~~~

This will list your running images. Use the port of your corresponding image


#### docker-compose

Let's introduce docker-compose. More [here](https://zegetech.com/blog/2018/11/08/developing-with-docker.html).

Inside your rails app root directory, create a new file called docker-compose.yml. In it paste the following:

~~~
	version: "3"
		services: 
  			web:
    			build: .
    			command: rails server -p 3000 -b '0.0.0.0'
    			volumes:
                 - .:/myapp
    			ports:
                 - 3000:3000
    			links:
                 - db
  			db:
    			image: postgres
			    ports:
                 - "5432"


~~~

Don't build the container yet but you are free to do so.

### Setup Postgresql

Open up Gemfile and include gem 'pg'. You can delete gem 'sqlite3' because we won't be using sqlite for our database needs. 

Then, open config/database.yml and replace its content with the following:

~~~
default: &default
  adapter: postgresql
  pool: 5
  timeout: 5000
  username: postgres
  host: db
  port: 5432

development:
  <<: *default
  database: myapp_db

# Warning: The database defined as "test" will be erased and
# re-generated from your development database when you run "rake".
# Do not set this db to the same as development or production.
test:
  <<: *default
  database: myapp_db_test

production:
  <<: *default
  database: myapp_db_prod

~~~

You can now build your container with

~~~
	docker-compose build
~~~

If you, start your container at this point, you'll stumble into issues with the app connecting to a non existing database. To solve this:

~~~
	docker-compose run web rake db:create
~~~

web is the name of the service as defined in the docker-compose.yml file.

Once successful, you can now do:

~~~
	docker-compose up
~~~

to start your application.

Yay, you've just dockerized your application!

> Did you know a snail can sleep for 3 years?

