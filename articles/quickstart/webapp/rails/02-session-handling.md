---
title: Session Handling
description: Learn how to store user data in your session and clean it up upon logout.
budicon: 448
---

<%= include('../../../_includes/_package', {
  org: 'auth0-samples',
  repo: 'auth0-rubyonrails-sample',
  path: '02-Session-Handling',
  requirements: [
    'Ruby 2.3.1',
    'Rails 5.0.0'
  ]
}) %>

## Store Session Data on Login

Upon successful authentication, **OmniAuth** sets the authentication hash of a request to `/auth/oauth2/callback`. To handle this request, add a new route in your routes file.

```ruby
get "/auth/oauth2/callback" => "auth0#callback"
```

Store the user information in the session in `auth0_controller\callback`.

```ruby
  def callback
    # This stores all the user information that came from Auth0
    # and the IdP
    session[:userinfo] = request.env['omniauth.auth']

    # Redirect to the URL you want after successful auth
    redirect_to '/dashboard'
  end
```

## Logout Action

To clear out all the objects stored within the session, call the `reset_session` method within the `logout_controller\logout` method. [Learn more about `reset_session` here](http://api.rubyonrails.org/classes/ActionController/Base.html#M000668).

```ruby
class LogoutController < ApplicationController
  include LogoutHelper
  def logout
    reset_session
    redirect_to logout_url.to_s
  end
end
```

You can use the Auth0 SDK to generate the logout URL.

```ruby
module LogoutHelper
  def logout_url
    domain = Rails.application.secrets.auth0_domain
    client_id = Rails.application.secrets.auth0_client_id
    request_params = {
      returnTo: root_url,
      client_id: client_id
    }

    URI::HTTPS.build(host: domain, path: '/logout', query: to_query(request_params))
  end

  private

  def to_query(hash)
    hash.map { |k, v| "#{k}=#{URI.escape(v)}" unless v.nil? }.reject(&:nil?).join('&')
  end
end
```

::: note
The final destination URL (the `returnTo` value) needs to be in the list of `Allowed Logout URLs`. See the [logout documentation](/logout#redirecting-users-after-logout) for more.
:::
