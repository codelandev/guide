# Controllers

## Before we go

1. Always keep your API controllers separated from the web app controllers
2. Add version name to your API folder (like v1, v2...)
3. You should have an `BaseController` to put general logic of your APIs
4. You should put general logic of wep app on `ApplicationController`

## Example of API `BaseController`

This is just an example of what you should have inside your `BaseController`,
you don't need to follow this strict, but at least this will could serve to
you don't miss some important methods.

```ruby
class Api::V1::BaseController < ApplicationController
  include Pundit # if you use pundit
  force_ssl if: :ssl_configured?

  protect_from_forgery with: :null_session

  before_action :destroy_session

  rescue_from Pundit::NotAuthorizedError, with: :unauthorized!
  rescue_from ActiveRecord::RecordNotFound, with: :not_found
  rescue_from ActionController::ParameterMissing, with: :parameter_missing

  protected

  def verify_application!
    if params[:client_id].blank? || params[:secret_key].blank?
      api_error(status: :bad_request, errors: 'Missing client_id or secret_key')
    elsif app.nil?
      api_error(status: :bad_request, errors: 'Please verify client_id and secret_key')
    end
  end

  def authenticate_user!
    token, options = ActionController::HttpAuthentication::Token.token_and_options(request)
    user_email = options.blank? ? nil : options[:email]
    user = user_email && User.find_by(email: user_email)

    existing_token = if user
                       user.authorizations.find_by(token: token).try(:token) || ''
                     else
                       ''
                     end

    if user && ActiveSupport::SecurityUtils.secure_compare(existing_token, token)
      @current_user = user
    else
      return unauthenticated!
    end
  end

  def current_user
    @current_user
  end

  def app
    @external_app ||= ExternalApplication.find_by(client_id: params[:client_id],
                                                  secret_key: params[:secret_key])
  end

  def destroy_session
    request.session_options[:skip] = true
  end

  def not_found
    api_error(status: :not_found, errors: 'Not found')
  end

  def parameter_missing
    api_error(status: :bad_request, errors: 'Missing params key. View documentation')
  end

  def unauthenticated!
    response.headers['WWW-Authenticate'] = "Token realm=Application"
    api_error(status: :unauthorized, errors: 'Bad credentials')
  end

  def unauthorized!
    api_error(status: :forbidden, errors: 'Not authorized')
  end

  def invalid_resource!(errors = [])
    api_error(status: :unprocessable_entity, errors: errors)
  end

  def api_error(status: 500, errors: [])
    unless Rails.env.production? || Rails.env.test?
      puts errors.full_messages if errors.respond_to?(:full_messages)
    end
    head(status: status) and return if errors.empty?

    content = {
      code: Rack::Utils.status_code(status),
      error: jsonapi_format(errors)
    }
    return render json: content.to_json, status: status
  end

  def jsonapi_format(errors)
    return errors if errors.is_a? String
    errors_hash = {}
    errors.messages.each do |attribute, error|
      array_hash = []
      error.each do |message|
        array_hash << message
      end
      errors_hash.merge!({ attribute => array_hash })
    end
    errors_hash
  end

  def ssl_configured?
    ActiveRecord::ConnectionAdapters::Column::TRUE_VALUES.include?(ENV['API_SSL'])
  end
end
```

## Example of `ApplicationController`

1. Include libraries that are commom to all controllers
2. Put all rescues (for authentication or whatever) here
3. Use `protected` instead of `private` ( [here is why](http://stackoverflow.com/questions/2028495/when-do-you-write-a-private-method-versus-protected) )

```ruby
class ApplicationController < ActionController::Base
  include Pundit
  protect_from_forgery with: :exception
  rescue_from Pundit::NotAuthorizedError, with: :user_not_authorized
  before_action :configure_permitted_parameters, if: :devise_controller?

  def configure_permitted_parameters
    devise_parameter_sanitizer.for(:sign_up) do |u|
      u.permit(*policy(User).permitted_attributes)
    end
    devise_parameter_sanitizer.for(:account_update) do |u|
      u.permit(*policy(User).permitted_attributes)
    end
  end

  protected

  def user_not_authorized
    flash[:error] = "Você não tem permissão para fazer isso."
    redirect_to(request.referrer || root_path)
  end
end
```

## Example of an API Controller

1. Keep your controller organizated and indented
2. Don't 'tab' method names after `private/protected` flag
3. Put all your callbacks like `before_action` on the top of the controller
4. Try to not create methods that change objects other than the object of your controller.
   Example: `def new_purchase` to change Purchase object inside `BusinessesController`

```ruby
class Api::V1::BusinessesController < Api::V1::BaseController
  before_action :authenticate_user!

  def index
    businesses = current_user.businesses

    render json: businesses, each_serializer: Api::V1::BusinessesSerializer
  end

  def show
    business = current_user.businesses.find(params[:id])

    render json: business, serializer: Api::V1::BusinessSerializer
  end

  def create
    business = current_user.businesses.build(permitted_params)

    if business.save
      render json: business, serializer: Api::V1::BusinessSerializer
    else
      invalid_resource!(business.errors)
    end
  end

  def update
    business = current_user.businesses.find(params[:id])

    if business.update(permitted_params)
      render json: business, serializer: Api::V1::BusinessSerializer
    else
      invalid_resource!(business.errors)
    end
  end

  protected

  def permitted_params
    attrs = policy(Business).permitted_attributes
    params.require(:business).permit(*attrs)
  end
end
```

## Example of an WebApp Controller

1. Keep your controller organizated and indented
2. Don't 'tab' method names after `private/protected` flag
3. Put all your callbacks like `before_action` on the top of the controller
4. `has_scope` came right after the callbacks and at the top of the methods
5. Try to not create methods that change objects other than the object of your controller.
   Example: `def new_purchase` to change Purchase object inside `DemandsController`
6. Always include a error case and show a message (flash) to user
7. Put resource (Object.find(params[:id])) and collection (Object.all) on seperated methods always as possible

```ruby
class DemandsController < ApplicationController
  before_action :authenticate_user!
  after_action :verify_authorized
  after_action :verify_policy_scoped, only: :index
  helper_method :resource, :collection

  has_scope :by_status, default: [
    Demand.statuses[:created],
    Demand.statuses[:pending],
  ], only: :index, type: :array

  def index;end

  def new
    authorize(@demand = Demand.new)
  end

  def create
    authorize(@demand = Demand.new(permitted_params))
    if @demand.save
      flash[:notice] = 'Successful created!'
      redirect_to demand_path(@demand)
    else
      render :new
    end
  end

  def show
    authorize(resource)
  end

  def edit
    authorize(resource)
  end

  def update
    authorize(resource)
    if resource.update_attributes(permitted_params)
      flash[:notice] = 'Successful updated!'
      redirect_to demand_path(resource)
    else
      render :edit
    end
  end

  def destroy
    authorize(resource)
    if resource.destroy
      flash[:notice] = 'Successful deleted!'
    else
      flash[:alert] = 'Error deleting object!'
    end
    redirect_to root_path
  end

  protected

  def permitted_params
    params.require(:demand).permit(*policy(Demand).permitted_attributes).merge(user: current_user)
  end
  
  def resource
    @demand ||= Demand.find(params[:id])
  end

  def collection
    @demands ||= Demand.all.order(updated_at: :desc).page(params[:page]).per(12)
  end
end
```
