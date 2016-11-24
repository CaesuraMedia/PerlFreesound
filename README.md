# NAME

WebService::Freesound - Perl wrapper around Freesound OAuth2 API!

# VERSION

Version 0.01

# SYNOPSIS

    #!/usr/bin/perl
    use WebService::Freesound;
    
    my %args = (
       client_id     => '03bbed9a541baf763526',
       client_secret => 'abcde1234598765fedcba78629091899aef32101',
       session_file  => '/var/www/myapp/freesoundrc',
    );

    my $freesound = WebService::Freesound->new (%args);

    # Freesound 3-Step OAuth process:
    #
    # Step 1.
    # Get authorisation URL
    #
    my $authorization_url = $freesound->get_authorization_url();

    # Step 2.
    # Redirect the user to authorization url, or paste into browser.
    # Or pop up an <iframe> with this URL as src.
    #
    use CGI;
    my $q = new CGI;
    $q->redirect($authorization_url);

    # OR in Toolkit::Template : Will put the Freesound.org's authentication
    # window on your site, with Approve or Deny buttons.
    #
    <iframe src=[% authorization_url %]>No iframe support</iframe>

    # Step 3.
    # A 'code' will be made available from the authorization_url from Freesound.
    # Use it here to get access_token, refresh_token and expiry times. These are
    # stored internally to the object and on disk. In theory, you should not need
    # to see them, but are accessible via their respective accessors.
    #
    my $rc = $freesound->get_new_oauth_tokens ($code));
    if ($rc) {
       print $freesound->error;
    }

    # At any time you can call check_authority to see if you are still authorised,
    # and this will return 1 or undef/$freesound->error.  It will refresh the tokens
    # if refresh_if_expired is set.
    #
    my $rc = $freesound->check_authority (refresh_if_expired => 1/undef);
    if ($rc) {
       print $freesound->error;
    }

    # All done with OAuth2 now it ..should.. just work forever (or until you revoke
    # the authorization at L<http://www.freesound.org/home/app_permissions/>, when
    # logged in, or set refresh_if_expired to undef).
    #
    # Get Freesound data, see L<https://www.freesound.org/docs/api/resources_apiv2.html>
    # Returns a L<HTTP::Response> or undef and $freesound->error.
    #
    my $rc = $freesound->check_authority (refresh_if_expired => 1);
    if ($rc) {
       my $response = $freesound->query ("query..." or "filter..." etc) # no query question mark required.
    }

    # Download the sample from a Freesound id into the specified directory with a
    # progress update in counter_file - for web apps get javascript to fire an Ajax call
    # to read this file until 100% complete.  Returns the path to the sample or undef
    # and $freesound->error.
    #
    my $rc = $freesound->check_authority (refresh_if_expired => 1);
    if ($rc) {
       my $file = $freesound->download ($id, '/var/www/myapp/downloads/', $counter_file);
    }

# DESCRIPTION

This module provides a Perl wrapper around the [http://Freesound.org](http://Freesound.org) RESTful API.

Freesound is a collaborative database of Creative Commons Licensed sounds. It allows
you to browse, download and share sounds.  This Perl wrapper at present allows you
'read-only' access to Freesound, ie browse and download.  Upcoming versions could provide
upload, describe and edit your own sounds (though I expect it might just be easier to use
their website for this).

The complete Freesound API is documented at [https://www.freesound.org/docs/api/index.html](https://www.freesound.org/docs/api/index.html)

In order to use this Perl module you will need get an account at Freesound
([http://www.freesound.org/home/register/](http://www.freesound.org/home/register/)) and then to register your application with them
at [http://www.freesound.org/apiv2/apply](http://www.freesound.org/apiv2/apply). Your application will then be given a client ID and
a client secret which you will need to use to get OAuth2 authorisation.

The OAuth2 Dance is described at Freesound, [https://www.freesound.org/docs/api/authentication.html](https://www.freesound.org/docs/api/authentication.html)
and officially at [RFC6749](http://tools.ietf.org/html/rfc6749).  It is a three step process as 
suggested above.

This module should look after the authorisation once done, ie when the expiry time arrives
it can automatically refresh the tokens.  The auth tokens are therefore kept as a file specified by
"_session\_file_", which should be read-only by you/www-data only.

When downloading a sound sample from Freesound a progress meter is available in "_counter\_file_"
which is useful in web contexts as a progress bar.  Format of the file is :

&lt;bytes-written>:&lt;byes-total>:&lt;percentage>
\# for example "10943051:12578220:87", ie 87% of 12578220 bytes written.

This is optional.

Also the download will download the sample file as its name and type suffix (some Freesound names have 
suffixes, some don't), so something like "/var/www/myapp/downloads/Pretty tune on piano.wav", 
".../violin tremolo G5.aif" etc.

The query method allows you to put any text string into its parameter so that you have the full
capabilities of Freesound search, filter, sort etc, as described here :
[https://www.freesound.org/docs/api/resources\_apiv2.html](https://www.freesound.org/docs/api/resources_apiv2.html)

If used as part of a web app, then the process could be : 

- Check for _session\_file_. If none then put up an iframe with the src set to output
of `$freesound-`get\_authorization\_url();>
- User clicks Authorise with a callback run (set in Freesound API credentials :
[http://www.freesound.org/apiv2/apply](http://www.freesound.org/apiv2/apply) (ie http://localhost/cgi-bin/mayapp/do\_auth.cgi)
which calls `$freesound-`get\_oauth\_tokens ($code))> - the code will be a parameter in
the CGI (ie `$q-`param ('code')>).  
- Text search box on main webpage can then be used as inputs to `$freesound-`query> -
the output formatted into HTML (from XML or json) as you
wish.  With Freesound you get a picture of the waveform and a low quality sound preview so you can engineer
your website to show the waveform and have start/stop/pause buttons.  Best not to replicate the entire
Freesound website, as this might contravene their terms and conditions.
- A Freesound sample will have an id. This can be used in `$freesound-`download ($id, $dir, $counter\_file)>.  
- Show download progress bar by continually polling the contents
of _counter\_file_ (with an Ajax call) and drawing a CSS bar.  Actually downloads to your server, not the
web-browser users Downloads directory.

## METHODS

- new ( _client\_id_, _client\_secret_ and _session\_file_ )

    Creates a new Freesound object for authorisation, queries and downloads.
    _client\_id_, _client\_secret_ and _session\_file_ are required.

- client\_id

    Accessor for the Client ID that was provided when you registered your
    application with Freesound.org.

- client\_secret

    Accessor for the client secret that was provided when you registered your
    application with Freesound.org.

- session\_file

    Accessor for the session file that stores the authorisation codes.

- ua

    Accessor for the User Agent.

- error

    Accessor for the error messages that may occur.

- access\_token

    Accessor for the OAuth2 access\_token.

- refresh\_token

    Accessor for the OAuth2 refresh\_token.

- expires\_in

    Accessor for the OAuth2 expiry time.

- get\_authorization\_url

    This returns the URL to start with when no auth is offered or accepted. Use it in an
    iframe if using this in a CGI environment (ie send to [Template::Toolkit](https://metacpan.org/pod/Template::Toolkit)).

- get\_new\_oauth\_tokens ( _code_ )

    Takes the resultant 'code' displayed when the user authorises this app on Freesound.org and 
    then sets the internal OAuth tokens, along with expiry time.  This method seriailises
    this in the session\_file for later use.  This is Step 3 in the process as described in
    [https://www.freesound.org/docs/api/authentication.html](https://www.freesound.org/docs/api/authentication.html).
    Returns 1 if succesful, undef and $freesound->error if not.

- check\_authority

    Checks the session file exists, has a current token.  If no session file, then
    returns URI to get the initial code from. If session file exists and and has not
    expired then it checks with Freesound.org for existing authority.  If the tokens
    need refreshing and refresh\_if\_expired is set, it attempts a refresh.  If that's
    successful, then updates the session file with new oauth tokens.  Return error if
    the refresh didn't work (or refreshable but not asked to) - maybe because the
    authority has been revoked.  See [http://www.freesound.org/home/app\_permissions/](http://www.freesound.org/home/app_permissions/)
    when logged into Freesound.org.  Return error if there is no authorisation at 
    Freesound.org.

- query ( _query-string_ )

    Does the querying of the Freesound database, see 
    [https://www.freesound.org/docs/api/resources\_apiv2.html](https://www.freesound.org/docs/api/resources_apiv2.html)
    Should just let any string go into the query like filter, tag, sort, geotag etc.  Just a string. 
    Returns whatever Freesound returns in an [HTTP::Response](https://metacpan.org/pod/HTTP::Response).

- download ( _sample-id_, _download-directory_, {_counter-file_} )

    The _sample-id_ is unique to a sample on Freesound, use `$freesound-`query>. The _download-directory_
    is where the downloaded file should go, the actual sound file will be named after its name on Freesound
    and will have the correct extension (wav, mp3, aif etc).  The _counter\_file_ is optional - it keeps
    a running count of the download progress .  In a web environment a Javascript Ajax call can read
    this in real-time to give a progress bar.  _counter\_file_ probably needs to be named with a session id
    of some sort. Returns the path of the file or undef (then see `$freesound-`error>).

- get\_filename\_from\_id ( _id_ )

    Does a query to get two fields - name and type (wav, mp3, aif etc) from the Freesound _id_ of the sample.
    Returns undef and $freesound->error if can't find a name/type for this id.

# INTERNAL SUBROUTINES/METHODS

Please don't use these as they may change on a whim.

- \_post\_request

    Updates the objects oauth tokens and session file from User Agent response. Returns
    1 or undef and $freesound->error.

# AUTHOR

Andy Cragg, `<andyc at caesuramedia.org>`

# BUGS

This is beta code and may contain bugs - please feel free to fix them and send patches.

# SUPPORT

You can find documentation for this module with the perldoc command.

    perldoc WebService::Freesound

You can also look for information at:

- RT: CPAN's request tracker (report bugs here)

    [http://rt.cpan.org/NoAuth/Bugs.html?Dist=WebService-Freesound](http://rt.cpan.org/NoAuth/Bugs.html?Dist=WebService-Freesound)

- AnnoCPAN: Annotated CPAN documentation

    [http://annocpan.org/dist/WebService-Freesound](http://annocpan.org/dist/WebService-Freesound)

- CPAN Ratings

    [http://cpanratings.perl.org/d/WebService-Freesound](http://cpanratings.perl.org/d/WebService-Freesound)

- Search CPAN

    [http://search.cpan.org/dist/WebService-Freesound/](http://search.cpan.org/dist/WebService-Freesound/)

# ACKNOWLEDGEMENTS

I had a look at [WebService::Soundlcoud](https://metacpan.org/pod/WebService::Soundlcoud) by Mohan Prasad Gutta, [http://search.cpan.org/~mpgutta/](http://search.cpan.org/~mpgutta/) for some ideas.

# LICENSE AND COPYRIGHT

Copyright 2016 Andy Cragg.

This program is free software; you can redistribute it and/or modify it
under the terms of the the Artistic License (2.0). You may obtain a
copy of the full license at:

[http://www.perlfoundation.org/artistic\_license\_2\_0](http://www.perlfoundation.org/artistic_license_2_0)

Any use, modification, and distribution of the Standard or Modified
Versions is governed by this Artistic License. By using, modifying or
distributing the Package, you accept this license. Do not use, modify,
or distribute the Package, if you do not accept this license.

If your Modified Version has been derived from a Modified Version made
by someone other than you, you are nevertheless required to ensure that
your Modified Version complies with the requirements of this license.

This license does not grant you the right to use any trademark, service
mark, tradename, or logo of the Copyright Holder.

This license includes the non-exclusive, worldwide, free-of-charge
patent license to make, have made, use, offer to sell, sell, import and
otherwise transfer the Package with respect to any patent claims
licensable by the Copyright Holder that are necessarily infringed by the
Package. If you institute patent litigation (including a cross-claim or
counterclaim) against any party alleging that the Package constitutes
direct or contributory patent infringement, then this Artistic License
to you shall terminate on the date that such litigation is filed.

Disclaimer of Warranty: THE PACKAGE IS PROVIDED BY THE COPYRIGHT HOLDER
AND CONTRIBUTORS "AS IS' AND WITHOUT ANY EXPRESS OR IMPLIED WARRANTIES.
THE IMPLIED WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR
PURPOSE, OR NON-INFRINGEMENT ARE DISCLAIMED TO THE EXTENT PERMITTED BY
YOUR LOCAL LAW. UNLESS REQUIRED BY LAW, NO COPYRIGHT HOLDER OR
CONTRIBUTOR WILL BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, OR
CONSEQUENTIAL DAMAGES ARISING IN ANY WAY OUT OF THE USE OF THE PACKAGE,
EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
