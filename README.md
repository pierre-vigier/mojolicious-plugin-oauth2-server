# NAME

Mojolicious::Plugin::OAuth2::Server - Easier implementation of an OAuth2
Authorization Server / Resource Server with Mojolicious

[![Build Status](https://travis-ci.org/Humanstate/mojolicious-plugin-oauth2-server.svg)](https://travis-ci.org/Humanstate/mojolicious-plugin-oauth2-server)
[![Coverage Status](https://coveralls.io/repos/github/Humanstate/mojolicious-plugin-oauth2-server/badge.svg?branch=master)](https://coveralls.io/github/Humanstate/mojolicious-plugin-oauth2-server?branch=master)

# VERSION

0.38

# SYNOPSIS

    use Mojolicious::Lite;

    plugin 'OAuth2::Server' => {
        ... # see SYNOPSIS in Net::OAuth2::AuthorizationServer::Manual
    };

    group {
      # /api - must be authorized
      under '/api' => sub {
        my ( $c ) = @_;

        return 1 if $c->oauth; # must be authorized via oauth

        $c->render( status => 401, text => 'Unauthorized' );
        return undef;
      };

      any '/annoy_friends' => sub { shift->render( text => "Annoyed Friends" ); };
      any '/post_image'    => sub { shift->render( text => "Posted Image" ); };
    };

    any '/track_location' => sub {
      my ( $c ) = @_;

      my $oauth_details = $c->oauth( 'track_location' )
          || return $c->render( status => 401, text => 'You cannot track location' );

      $c->render( text => "Target acquired: @{[$oauth_details->{user_id}]}" );
    };

    app->start;

Or full fat app:

    use Mojo::Base 'Mojolicious';

    ...

    sub startup {
      my $self = shift;

      ...

      $self->plugin( 'OAuth2::Server' => $oauth2_auth_code_grant_config );
    }

Then in your controller:

     sub my_route_name {
       my ( $c ) = @_;
    
       if ( my $oauth_details = $c->oauth( qw/required scopes/ ) ) {
         ... # do something, user_id, client_id, etc, available in $oauth_details
       } else {
         return $c->render( status => 401, text => 'Unauthorized' );
       }

       ...
     }

# DESCRIPTION

This plugin implements the various OAuth2 grant types flow as described at
[http://tools.ietf.org/html/rfc6749](http://tools.ietf.org/html/rfc6749). It is a complete implementation of
RFC6749, with the exception of the "Extension Grants" as the description of
that grant type is rather hand-wavy.

The bulk of the functionality is implemented in the [Net::OAuth2::AuthorizationServer](https://metacpan.org/pod/Net::OAuth2::AuthorizationServer)
distribution, you should see that for more comprehensive documentation and
examples of usage.

The examples here use the "Authorization Code Grant" flow as that is considered
the most secure and most complete form of OAuth2.

# METHODS

## register

Registers the plugin with your app - note that you must pass callbacks for
certain functions that the plugin expects to call if you are not using the
plugin in its simplest form.

    $self->register($app, \%config);

Registering the plugin will call the [Net::OAuth2::AuthorizationServer](https://metacpan.org/pod/Net::OAuth2::AuthorizationServer)
and create a `auth_code_grant` that can be accessed using the defined
`authorize_route` and `access_token_route`. The arguments passed to the
plugin are passed straight through to the `auth_code_grant` method in
the [Net::OAuth2::AuthorizationServer](https://metacpan.org/pod/Net::OAuth2::AuthorizationServer) module.

## oauth

Checks if there is a valid Authorization: Bearer header with a valid access
token and if the access token has the requisite scopes. The scopes are optional:

    unless ( my $oauth_details = $c->oauth( @scopes ) ) {
      return $c->render( status => 401, text => 'Unauthorized' );
    }

This calls the [Net::OAuth2::AuthorizationServer::AuthorizationCodeGrant](https://metacpan.org/pod/Net::OAuth2::AuthorizationServer::AuthorizationCodeGrant)
module (`verify_token_and_scope` method) to validate the access/refresh token.

# SEE ALSO

[Net::OAuth2::AuthorizationServer](https://metacpan.org/pod/Net::OAuth2::AuthorizationServer) - The dist that handles the bulk of the
functionality used by this plugin

# AUTHOR

Lee Johnson - `leejo@cpan.org`

# LICENSE

This library is free software; you can redistribute it and/or modify it under
the same terms as Perl itself. If you would like to contribute documentation
or file a bug report then please raise an issue / pull request:

    https://github.com/Humanstate/mojolicious-plugin-oauth2-server
