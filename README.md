# README

https://confluence.freshworks.com/display/FRES/Ruby+SDK+Integration

```require 'net/http'
require 'uri'
require 'json'
require 'base64'
require 'securerandom'
 
CLIENT_ID = 'xxxx'
CLIENT_SECRET = 'yyyy'
ENDPOINT = 'accounts.freshworks360.io'
TOKEN_URI = URI.parse("https://#{ENDPOINT}/oauth/token?grant_type=client_credentials")
 
def fetch_access_token
  request = Net::HTTP::Post.new(TOKEN_URI)
  request['Content-Type'] = 'application/json'
  request['Authorization'] = "Basic #{Base64.strict_encode64("#{CLIENT_ID}:#{CLIENT_SECRET}")}"
  request.body = '{}'
   
  response = Net::HTTP.start(TOKEN_URI.hostname, TOKEN_URI.port, use_ssl: true) do |http|
    http.request(request)
  end
 
  raise 'Failed to fetch access token' unless response.is_a?(Net::HTTPSuccess)
   
  JSON.parse(response.body)['access_token']
end
 
def configure_freshworks(endpoint, client_id, client_secret)
  Freshworks.config = {
    endpoint: endpoint,
    client_id: client_id,
    client_secret: client_secret,
    timeout: 10,
    trace_config: {
      b3_injector: -> { { "x-request-id" => SecureRandom.uuid } }
    }
  }
end
 
configure_freshworks(ENDPOINT, CLIENT_ID, CLIENT_SECRET)
user_client = Freshworks::User::V2::UserService::Client.new
account_client = Freshworks::Account::V2::AccountService::Client.new
 
user_request_payload = {
  product_account_id: { value: '2220000170' },
  product_user_id: { value: '1022' }
}
 
account_request_payload = {
  product_account_id: { value: '2220000170' }
}
 
user_request_object = Freshworks::User::V2::GetUserByProductIdRequest.new(user_request_payload)
account_request_object = Freshworks::Account::V2::GetAccountByProductIdRequest.new(account_request_payload)
access_token  = fetch_access_token
 
user_client.send('get_user_by_product_user_id', user_request_object, access_token)
 
account_client.send('get_account_by_product_account_id', account_request_object, access_token)```
