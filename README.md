# How To Set Up JWT
- in the application_controller.rb change to:-
```
protect_from_forgery unless: -> { request.format.json? }
```
- add the following gem to the top of the jem file:-
```
gem 'dotenv-rails', groups: [:development, :test]
```
- create a .env file in the application root directory
- add gem:-
```
gem 'devise-jwt', '~> 0.5.5'
```
- set up devise
- in app/config/initializers/devise.rb add:-
```
Devise.setup do |config|
  # ...
  config.jwt do |jwt|
    jwt.secret = ENV['DEVISE_JWT_SECRET_KEY']
    jwt.expiration_time = 604800
  end
end
```
- and change the following (add :params_auth):-
```
config.skip_session_storage = [:http_auth, :params_auth]
```
- generate a new key and add to the .env file as follows:-
```
bundle exec rake secret
```
- add the entry in the .env file (where xxxxx is the new generated key):-
```
DEVISE_JWT_SECRET_KEY=xxxxxx
```
- create a white listed jwt file and migrate:-
```
class CreateWhitelistedJwts < ActiveRecord::Migration[5.1]
  def change
    create_table :whitelisted_jwts do |t|
      t.string :jti, null: false
      t.string :aud
      t.datetime :exp, null: false
      t.references :user, foreign_key: true
    end
    add_index :whitelisted_jwts, :jti, unique: true
  end
end
```
- change the user model to:-
```
class User < ApplicationRecord
  # Include default devise modules. Others available are:
  # :confirmable, :lockable, :timeoutable and :omniauthable
  
  include Devise::JWT::RevocationStrategies::Whitelist
  
  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :trackable, :validatable,
         :jwt_authenticatable, jwt_revocation_strategy: self
  
  has_many :whitelisted_jwts
end
```
- add the following to routes.rb
```
devise_for :users, defaults: { format: :json }
```
- add node package jwt-decode
```
yarn add jwt-decode
```