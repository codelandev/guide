# Config folder and their must have

## Include default_url_options on your development.rb
`config.action_mailer.default_url_options = { host: 'localhost', port: 3000 }`

## Always include `staging.rb` on your environments
```
Rails.application.configure do
  config.action_mailer.default_url_options = { host: ENV['HOST_DOMAIN'] }

  # Settings specified here will take precedence over those in config/application.rb.

  # Code is not reloaded between requests.
  config.cache_classes = true

  # Eager load code on boot. This eager loads most of Rails and
  # your application in memory, allowing both threaded web servers
  # and those relying on copy on write to perform better.
  # Rake tasks automatically ignore this option for performance.
  config.eager_load = true

  # Full error reports are disabled and caching is turned on.
  config.consider_all_requests_local       = false
  config.action_controller.perform_caching = true

  # Disable serving static files from the `/public` folder by default since
  # Apache or NGINX already handles this.
  config.serve_static_files = ENV['RAILS_SERVE_STATIC_FILES'].present?

  # Compress JavaScripts and CSS.
  config.assets.js_compressor = :uglifier
  # config.assets.css_compressor = :sass

  # Do not fallback to assets pipeline if a precompiled asset is missed.
  config.assets.compile = false

  # Asset digests allow you to set far-future HTTP expiration dates on all assets,
  # yet still be able to expire them through the digest params.
  config.assets.digest = true

  # Use the lowest log level to ensure availability of diagnostic information
  # when problems arise.
  config.log_level = :debug

  # Enable locale fallbacks for I18n (makes lookups for any locale fall back to
  # the I18n.default_locale when a translation cannot be found).
  config.i18n.fallbacks = true

  # Send deprecation notices to registered listeners.
  config.active_support.deprecation = :notify

  # Use default logging formatter so that PID and timestamp are not suppressed.
  config.log_formatter = ::Logger::Formatter.new

  # Do not dump schema after migrations.
  config.active_record.dump_schema_after_migration = false
end
```

## Remember to configure SendGrid on your `environment.rb`

```
# Load the Rails application.
require File.expand_path('../application', __FILE__)

ActionMailer::Base.smtp_settings = {
  address:        'smtp.sendgrid.net',
  port:           '587',
  authentication: :plain,
  user_name:      ENV['SENDGRID_USERNAME'],
  password:       ENV['SENDGRID_PASSWORD'],
  domain:         'heroku.com',
  enable_starttls_auto: true
}

# Initialize the Rails application.
Rails.application.initialize!
```

## Avoid using handmade routes on your `routes.rb`
Always try to use `resources :examples` maximum as you can!

## Always keep your `secrets.yml` udpated for staging

You will probably have something like this:

```
staging:
  <<: *default
  aws_access_key: <%= ENV["AWS_ACCESS_KEY"] %>
  aws_bucket: <%= ENV["AWS_BUCKET"] %>
  aws_secret_key: <%= ENV["AWS_SECRET_KEY"] %>
  facebook_id: <%= ENV["FACEBOOK_APP_ID"] %>
  facebook_secret: <%= ENV["FACEBOOK_SECRET_ID"] %>
  secret_key_base: <%= ENV["SECRET_KEY_BASE"] %>
```

## Changes mailer_sender on `devise.rb`

`config.mailer_sender = 'contact@myapp.com'`

## Configure carrierwave initializer

```
CarrierWave.configure do |config|
  config.storage = (Rails.env.production? || Rails.env.staging?) ? :fog : :file
  config.enable_processing = !Rails.env.test?

  if Rails.env.production? || Rails.env.staging?
    config.fog_credentials = {
      provider:              'AWS',
      aws_access_key_id:     Rails.application.secrets.aws_access_key,
      aws_secret_access_key: Rails.application.secrets.aws_secret_key,
      # region:                'eu-west-1',
    }
    config.fog_directory  = Rails.application.secrets.aws_bucket
    config.fog_attributes = {
      'Cache-Control' => "max-age=#{365.days.to_i}",
    }
  end
end
```

## Remember to create locales for your models/enums

```
pt-BR:
  activerecord:
    models:
      receipt:
        one: 'Nota Fiscal'
        other: 'Notas Fiscais'

    attributes:
      receipt:
        client: 'Cliente'
        status: 'Status'
        amount: 'Valor'
        cpf_cnpj: 'CPF/CNPJ'
        description: 'Descrição'

        status:
          approved: 'Aprovada'
          rejected: 'Rejeitada'
          processing: 'Processando'

    errors:
      client:
        must_have_at_least_one_address: 'Deve ter ao menos 1 endereço.'
```
