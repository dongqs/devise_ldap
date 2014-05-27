Devise with LDAP
================

Gemfile
-------

    gem "devise"
    gem "devise_ldap_authenticatable"

run
---

    bundle install
    rails g devise:install
    rails g devise user
    rails g devise:views
    rails g devise_ldap_authenticatable:install
    rails g migration add_username_to_users username:string:index
    bundle exec rake db:migrate

app/controllers/application_controller.rb
-----------------------------------------

    before_action :authenticate_user!
    before_action :configure_permitted_parameters, if: :devise_controller?

    protected

    def configure_permitted_parameters
      devise_parameter_sanitizer.for(:sign_in) << :username
    end

app/models/user.rb
------------------

    class User < ActiveRecord::Base
      # Include default devise modules. Others available are:
      # :confirmable, :lockable, :timeoutable and :omniauthable
      devise :ldap_authenticatable, :registerable,
         :recoverable, :rememberable, :trackable, :validatable

      validates :username, presence: true, uniqueness: true

      before_validation :get_ldap_email
      def get_ldap_email
        self.email = Devise::LDAP::Adapter.get_ldap_param(self.username,"mail").first
      end

      # use ldap uid as primary key
      before_validation :get_ldap_id
      def get_ldap_id
        self.id = Devise::LDAP::Adapter.get_ldap_param(self.username,"uidnumber").first
      end

      # hack for remember_token
      def authenticatable_token
        Digest::SHA1.hexdigest(email)[0,29]
      end
    end

app/views/devise/sessions/new.html.slim
---------------------------------------

    chagne email to username

config/initializers/devise.rb
-----------------------------

    Devise.setup do |config|
      # ==> LDAP Configuration 
      config.ldap_logger = true
      config.ldap_create_user = true
      config.ldap_update_password = true
      config.ldap_use_admin_to_bind = true

      config.authentication_keys = [ :username ]
      config.password_length = 0..128 # if your ldap has a weak password police

config/ldap.yml
---------------
