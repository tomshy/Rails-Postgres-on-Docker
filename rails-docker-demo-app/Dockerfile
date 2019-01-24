FROM ruby:2.5.1
MAINTAINER Tom Nyongesa
RUN apt-get update && apt-get install -y build-essential && apt-get install -y nodejs
WORKDIR /
COPY Gemfile /
COPY Gemfile.lock /
RUN gem install bundler && bundle install
COPY . ./
EXPOSE 3000
ENTRYPOINT ["bundle", "exec"]
CMD ["rails","server","-b","0.0.0.0"]