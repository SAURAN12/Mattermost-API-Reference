# Mattermost-API-Reference
swagger: '2.0'
info:
  description: |
    There is also a work-in-progress [Postman API reference](https://documenter.getpostman.com/view/4508214/RW8FERUn).

  version: 4.0.0
  title: Mattermost API Reference
  termsOfService: 'https://about.mattermost.com/default-terms/'
  contact:
    email: feedback@mattermost.com
  x-logo:
    url: "https://www.mattermost.org/wp-content/uploads/2016/03/logoHorizontal_WS.png"
    backgroundColor: "#FFFFFF"
basePath: /api/v4
host: your-mattermost-url.com
tags:
  - name: introduction
    description: |
      The Mattermost Web Services API is used by Mattermost clients and third party applications to interact with the server. [JavaScript and Golang drivers for](/#tag/drivers) connecting to the APIs are also available.

      ### Support

      Mattermost core committers work with the community to keep the API documentation up-to-date.

      If you have questions on API routes not listed in this reference, please [join the Mattermost community server](https://pre-release.mattermost.com/signup_user_complete/?id=f1924a8db44ff3bb41c96424cdc20676) to ask questions in the Developers channel, [or post questions to our Developer Discussion forum](http://forum.mattermost.org/c/dev).

      [Bug reports](https://github.com/mattermost/mattermost-api-reference/issues) in the documentation or the API are also welcome, as are pull requests to fix the issues.

      ### Contributing

      When you have answers to API questions not addressed in our documentation we ask you to consider making a pull request to improve our reference. [Small changes](https://github.com/mattermost/mattermost-api-reference/commit/d574c0c1e95dc2228dc96663afd562f1305e3ece) and [larger changes](https://github.com/mattermost/mattermost-api-reference/commit/1ae3314f0935eebba8c885d8969dcad72f801501) are all welcome.

      We also have [Help Wanted tickets](https://github.com/mattermost/mattermost-api-reference/issues) available for community members who would like to help others more easily use the APIs. We acknowledge everyone's contribution in the [release notes of our next version](https://docs.mattermost.com/administration/changelog.html#contributors).

      The source code for this API reference is hosted at https://github.com/mattermost/mattermost-api-reference.
  - name: schema
    description: |
      All API access is through HTTP(S) requests at `your-mattermost-url.com/api/v4`. All request and response bodies are `application/json`.

      When using endpoints that require a user id, the string `me` can be used in place of the user id to indicate the action is to be taken for the logged in user.
  - name: drivers
    description: |
      The easiest way to interact with the Mattermost Web Service API is through a language specific driver.

      #### Official Drivers
      * [Mattermost JavaScript Driver](https://github.com/mattermost/mattermost-redux/blob/master/src/client/client4.js)
      * [Mattermost Golang Driver](https://github.com/mattermost/mattermost-server/blob/master/model/client4.go)

      #### Community-built Drivers
      * [PHP Driver](https://github.com/gnello/php-mattermost-driver) - built by [@gnello](https://github.com/gnello) and [@prixone](https://github.com/prixone)
      * [Python Driver](https://github.com/Vaelor/python-mattermost-driver) - built by [@Vaelor](https://github.com/Vaelor)

      For other community-built drivers and API wrappers, see [our app directory](https://about.mattermost.com/community-applications/#privateApps).

  - name: authentication
    description: |
      There are multiple ways to authenticate against the Mattermost API.

      All examples assume there is a Mattermost instance running at http://localhost:8065.

      #### Session Token

      Make an HTTP POST to `your-mattermost-url.com/api/v4/users/login` with a JSON body indicating the user’s `login_id`, `password` and optionally the MFA `token`. The `login_id` can be an email, username or an AD/LDAP ID depending on the system's configuration.

      ```
      curl -i -d '{"login_id":"someone@nowhere.com","password":"thisisabadpassword"}' http://localhost:8065/api/v4/users/login
      ```

      NOTE: If you're running cURL on windows, you will have to change the single quotes to double quotes, and escape the inner double quotes with backslash, like below:

      ```
      curl -i -d "{\"login_id\":\"someone@nowhere.com\",\"password\":\"thisisabadpassword\"}" http://localhost:8065/api/v4/users/login
      ```

      If successful, the response will contain a `Token` header and a user object in the body.

      ```
      HTTP/1.1 200 OK
      Set-Cookie: MMSID=hyr5dmb1mbb49c44qmx4whniso; Path=/; Max-Age=2592000; HttpOnly
      Token: hyr5dmb1mbb49c44qmx4whniso
      X-Ratelimit-Limit: 10
      X-Ratelimit-Remaining: 9
      X-Ratelimit-Reset: 1
      X-Request-Id: smda55ckcfy89b6tia58shk5fh
      X-Version-Id: developer
      Date: Fri, 11 Sep 2015 13:21:14 GMT
      Content-Length: 657
      Content-Type: application/json; charset=utf-8

      {{user object as json}}
      ```

      Include the `Token` as part of the `Authorization` header on your future API requests with the `Bearer` method.

      ```
      curl -i -H 'Authorization: Bearer hyr5dmb1mbb49c44qmx4whniso' http://localhost:8065/api/v4/users/me
      ```

      You should now be able to access the API as the user you logged in as.

      #### Personal Access Tokens

      Using [personal access tokens](https://docs.mattermost.com/developer/personal-access-tokens.html) is very similar to using a session token. The only real difference is that session tokens will expire, while personal access tokens will live until they are manually revoked by the user or an admin.

      Just like session tokens, include the personal access token as part of the `Authorization` header in your requests using the `Bearer` method. Assuming our personal access token is `9xuqwrwgstrb3mzrxb83nb357a`, we could use it as shown below.

      ```
      curl -i -H 'Authorization: Bearer 9xuqwrwgstrb3mzrxb83nb357a' http://localhost:8065/api/v4/users/me
      ```

      #### OAuth 2.0

      Mattermost has the ability to act as an [OAuth 2.0](https://tools.ietf.org/html/rfc6749) service provider.

      The official documentation for [using your Mattermost server as an OAuth 2.0 service provider can be found here.](https://docs.mattermost.com/developer/oauth-2-0-applications.html)

      For an example on how to register an OAuth 2.0 app with your Mattermost instance, please see the [Mattermost-Zapier integration documentation](https://docs.mattermost.com/integrations/zapier.html#register-zapier-as-an-oauth-2-0-application).

  - name: errors
    description: |
      All errors will return an appropriate HTTP response code along with the following JSON body:
      ```
      {
          "id": "the.error.id",
          "message": "Something went wrong", // the reason for the error
          "request_id": "", // the ID of the request
          "status_code": 0, // the HTTP status code
          "is_oauth": false // whether the error is OAuth specific
      }
      ```
  - name: rate limiting
    description: |
      Whenever you make an HTTP request to the Mattermost API you might notice the following headers included in the response:
      ```
      X-Ratelimit-Limit: 10
      X-Ratelimit-Remaining: 9
      X-Ratelimit-Reset: 1441983590
      ```

      These headers are telling you your current rate limit status.

      | Header | Description |
      | ------ | ----------- |
      | X-Ratelimit-Limit | The maximum number of requests you can make per second. |
      | X-Ratelimit-Remaining | The number of requests remaining in the current window. |
      | X-Ratelimit-Reset | The remaining UTC epoch seconds before the rate limit resets. |

      If you exceed your rate limit for a window you will receive the following error in the body of the response:

      ```
      HTTP/1.1 429 Too Many Requests
      Date: Tue, 10 Sep 2015 11:20:28 GMT
      X-RateLimit-Limit: 10
      X-RateLimit-Remaining: 0
      X-RateLimit-Reset: 1

      limit exceeded
      ```
  - name: WebSocket
    description: |
      In addition to the HTTP RESTful web service, Mattermost also offers a WebSocket event delivery system and some API functionality.

      To connect to the WebSocket follow the standard opening handshake as [defined by the RFC specification](https://tools.ietf.org/html/rfc6455#section-1.3) to the `/api/v4/websocket` endpoint of Mattermost.

      #### Authentication

      The Mattermost WebSocket can be authenticated by cookie or through an authentication challenge. If you're authenticating from a browser and have logged in with the Mattermost API, your authentication cookie should already be set, this is how the Mattermost webapp authenticates with the WebSocket.

      To authenticate with an authentication challenge, first connect the WebSocket and then send the following JSON over the connection:

      ```
      {
        "seq": 1,
        "action": "authentication_challenge",
        "data": {
          "token": "mattermosttokengoeshere"
        }
      }
      ```

      If successful, you will receive a standard OK response from the webhook:

      ```
      {
        "status": "OK",
        "seq_reply": 1
      }
      ```

      Once successfully authenticated, the server will pass a `hello` WebSocket event containing server version over the connection.

      #### Events

      WebSocket events are primarily used to alert the client to changes in Mattermost, such as delivering new posts or alerting the client that another user is typing in a channel.

      Events on the WebSocket will have the form:

      ```
      {
        "event": "hello",
        "data": {
          "server_version": "3.6.0.1451.1c38da627ebb4e3635677db6939e9195"
        },
        "broadcast":{
          "omit_users": null,
          "user_id": "ay5sq51sebfh58ktrce5ijtcwy",
          "channel_id": "",
          "team_id": ""
        }
      }
      ```

      The `event` field indicates the event type, `data` contains any data relevant to the event and `broadcast` contains information about who the event was sent to. For example, the above example has `user_id` set to "ay5sq51sebfh58ktrce5ijtcwy" meaning that only the user with that ID received this event broadcast. The `omit_users` field can contain an array of user IDs that were specifically omitted from receiving the event.

      The list of Mattermost WebSocket events are:
      - added_to_team
      - authentication_challenge
      - channel_converted
      - channel_created
      - channel_deleted
      - channel_member_updated
      - channel_updated
      - channel_viewed
      - config_changed
      - delete_team
      - direct_added
      - emoji_added
      - ephemeral_message
      - group_added
      - hello
      - leave_team
      - license_changed
      - memberrole_updated
      - new_user
      - plugin_disabled
      - plugin_enabled
      - plugin_statuses_changed
      - post_deleted
      - post_edited
      - post_unread
      - posted
      - preference_changed
      - preferences_changed
      - preferences_deleted
      - reaction_added
      - reaction_removed
      - response
      - role_updated
      - status_change
      - typing
      - update_team
      - user_added
      - user_removed
      - user_role_updated
      - user_updated
      - dialog_opened

      #### WebSocket API

      Mattermost has some basic support for WebSocket APIs. A connected WebSocket can make requests by sending the following over the connection:

      ```
      {
        "action": "user_typing",
        "seq": 2,
        "data": {
          "channel_id": "nhze199c4j87ped4wannrjdt9c",
          "parent_id": ""
        }
      }
      ```

      This is an example of making a `user_typing` request, with the purpose of alerting the server that the connected client has begun typing in a channel or thread. The `action` field indicates what is being requested, and performs a similar duty as the route in a HTTP API.

      The `seq` or sequence number is set by the client and should be incremented with every use. It is used to distinguish responses to requests that come down the WebSocket. For example, a standard response to the above request would be:

      ```
      {
        "status": "OK",
        "seq_reply": 2
      }
      ```

      Notice `seq_reply` is 2, matching the `seq` of the original request. Using this a client can distinguish which request the response is meant for.

      If there was any information to respond with, it would be encapsulated in a `data` field.

      In the case of an error, the response would be:

      ```
      {
        "status": "FAIL",
        "seq_reply": 2,
        "error": {
          "id": "some.error.id.here",
          "message": "Some error message here"
        }
      }
      ```

      The list of WebSocket API actions is:
      - user_typing
      - get_statuses
      - get_statuses_by_ids

      To see how these actions work, please refer to either the [Golang WebSocket driver](https://github.com/mattermost/mattermost-server/blob/master/model/websocket_client.go) or our [JavaScript WebSocket driver](https://github.com/mattermost/mattermost-redux/blob/master/src/client/websocket_client.js).

  - name: APIv3 Deprecation
    description: |
      Since Mattermost 4.6 released on January 16, 2018, API v3 has no longer been supported and it will be removed in Mattermost Server v5.0 on June 16, 2018. Follow these simple steps to migrate your integrations and apps to API v4. Otherwise your integrations may break once you upgrade to Mattermost 5.0

      1. Set your server's log level to `DEBUG` in **System Console > General > Logging > File Log Level** to print detailed logs for API requests.
      2. In **System Console > Logs**, search for requests hitting `/api/v3/` endpoints. Any requests hitting these endpoints are from integrations that should be migrated to API v4.
        - For in-house or self-built integrations, update them to use v4 with the help of [this API reference](https://api.mattermost.com). Most v3 endpoints have direct counterparts in v4 and should be migrated easily.
        - For third-party integrations, visit their homepage (on GitHub, GitLab, etc.). Check if they already have a version that uses the Mattermost v4 API. If they do not, consider opening an issue asking them if support is planned.
      3. Once all integrations have been migrated to API v4, review the server logs with log level set to `DEBUG`. Confirm no requests hit `/api/v3/` endpoints.
      4. Set **Allow use of API v3 endpoints** to `false` in **System Console > General > Configuration**, or set `EnableAPIv3` to `false` in `config.json`. This setting disables API v3 on your server. Any time a v3 endpoint is used, an error is logged in **System Console > Logs**.
      5. Set your server's log level back to `ERROR`. Use the error logs to help track down any remaining uses of API v3.

      Below are the major changes made between v3 and v4:

      1. Endpoint URLs only require team IDs when necessary. For example, getting a channel by ID no longer requires a team ID in v4.
      2. Collection endpoints now generally return lists and include paging as part of the query string.
      3. User ID is now included in most user endpoints. This allows admins to modify other users through v4 endpoints.

      If you have any questions about the API v3 deprecation, or about migrating from v3 to v4, [join our daily build server at pre-release.mattermost.com](https://pre-release.mattermost.com/signup_user_complete/?id=f1924a8db44ff3bb41c96424cdc20676) and ask questions in the [APIv4 channel](https://pre-release.mattermost.com/core/channels/apiv4).

  - name: users
    description: |
      Endpoints for creating, getting and interacting with users.

      When using endpoints that require a user id, the string `me` can be used in place of the user id to indicate the action is to be taken for the logged in user.
  - name: bots
    description: Endpoints for creating, getting and updating bot users.
  - name: teams
    description: Endpoints for creating, getting and interacting with teams.
  - name: channels
    description: Endpoints for creating, getting and interacting with channels.
  - name: posts
    description: Endpoints for creating, getting and interacting with posts.
  - name: files
    description: Endpoints for uploading and interacting with files.
  - name: preferences
    description: Endpoints for saving and modifying user preferences.
  - name: status
    description: Endpoints for getting and updating user statuses.
  - name: emoji
    description: Endpoints for creating, getting and interacting with emojis.
  - name: reactions
    description: Endpoints for creating, getting and removing emoji reactions.
  - name: webhooks
    description: Endpoints for creating, getting and updating webhooks.
  - name: commands
    description: Endpoints for creating, getting and updating slash commands.
  - name: OpenGraph
    description: Endpoint for getting Open Graph metadata.
  - name: system
    description: General endpoints for interating with the server, such as configuration and logging.
  - name: brand
    description: Endpoints related to custom branding and white-labeling. See [our branding documentation](https://docs.mattermost.com/administration/branding.html) for more information.
  - name: OAuth
    description: Endpoints for configuring and interacting with Mattermost as an OAuth 2.0 service provider.
  - name: SAML
    description: Endpoints for configuring and interacting with SAML.
  - name: LDAP
    description: Endpoints for configuring and interacting with LDAP.
  - name: groups
    description: Endpoints related to LDAP groups.
  - name: compliance
    description: Endpoints for creating, getting and downloading compliance reports.
  - name: cluster
    description: Endpoints for configuring and interacting with high availability clusters.
  - name: elasticsearch
    description: Endpoints for configuring and interacting with Elasticsearch.
  - name: dataretention
    description: Endpoint for getting data retention policy settings.
  - name: jobs
    description: Endpoints related to various background jobs that can be run by the server or separately by job servers.
  - name: plugins
    description: Endpoints related to uploading and managing plugins.
  - name: roles
    description: Endpoints for creating, getting and updating roles.
  - name: schemes
    description: Endpoints for creating, getting and updating and deleting schemes.
  - name: integration_actions
    description: Endpoints for interactive actions for use by integrations.
  - name: terms of service
    description: Endpoints for getting and updating custom terms of service.
x-tagGroups:
  - name: Overview
    tags:
      - introduction
      - schema
      - APIv3 Deprecation
  - name: Standard Features
    tags:
      - drivers
      - authentication
      - errors
      - rate limiting
      - WebSocket
  - name: Endpoints
    tags:
      - users
      - bots
      - teams
      - channels
      - posts
      - files
      - preferences
      - status
      - emoji
      - reactions
      - webhooks
      - commands
      - OpenGraph
      - system
      - brand
      - OAuth
      - SAML
      - LDAP
      - groups
      - compliance
      - cluster
      - elasticsearch
      - dataretention
      - jobs
      - plugins
      - roles
      - schemes
      - integration_actions
      - terms of service
schemes:
  - http
  - https
consumes:
  - application/json
produces:
  - application/json
responses:
  'Forbidden':
    description: Do not have appropriate permissions
    schema:
      $ref: '#/definitions/AppError'
  'Unauthorized':
    description: No access token provided
    schema:
      $ref: '#/definitions/AppError'
  'BadRequest':
    description: Invalid or missing parameters in URL or request body
    schema:
      $ref: '#/definitions/AppError'
  'NotFound':
    description: Resource not found
    schema:
      $ref: '#/definitions/AppError'
  'TooLarge':
    description: Content too large
    schema:
      $ref: '#/definitions/AppError'
  'NotImplemented':
    description: Feature is disabled
    schema:
      $ref: '#/definitions/AppError'
  'InternalServerError':
    description: Something went wrong with the server
    schema:
      $ref: '#/definitions/AppError'
paths:
  /users:
    post:
      tags:
        - users
      summary: Create a user
      description: |
        Create a new user on the system. Password is required for email login. For other authentication types such as LDAP or SAML, auth_data and auth_service fields are required.
        ##### Permissions
        No permission required but user creation can be controlled by server configuration.
      parameters:
        - name: t
          in: query
          description: Token id from an email invitation
          required: false
          type: string
        - name: iid
          in: query
          description: Token id from an invitation link
          required: false
          type: string
        - in: body
          name: body
          description: User object to be created
          required: true
          schema:
            type: object
            required:
              - email
              - username
            properties:
              email:
                type: string
              username:
                type: string
              first_name:
                 type: string
              last_name:
                type: string
              nickname:
                type: string
              auth_data:
                description: Service-specific authentication data, such as email address.
                type: string
              auth_service:
                description: The authentication service, one of "email", "gitlab", "ldap", "saml", "office365", "google", and "".
                type: string
              password:
                description: The password used for email authentication.
                type: string
              locale:
                type: string
              props:
                type: object
              notify_props:
                $ref: '#/definitions/UserNotifyProps'
      responses:
        '201':
          description: User creation successful
          schema:
            $ref: '#/definitions/User'
        '400':
          $ref: '#/responses/BadRequest'
        '403':
          $ref: '#/responses/Forbidden'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")

            user := &model.User{
                Username: "username",
                Email: "email@domain.com",
                Password: "Password1",
            }

            createdUser, response := Client.CreateUser(user)

    get:
      tags:
        - users
      summary: Get users
      description: |
        Get a page of a list of users. Based on query string parameters, select users from a team, channel, or select users not in a specific channel.

        Since server version 4.0, some basic sorting is available using the `sort` query parameter. Sorting is currently only supported when selecting users on a team.
        ##### Permissions
        Requires an active session and (if specified) membership to the channel or team being selected from.
      parameters:
        - name: page
          in: query
          description: The page to select.
          default: "0"
          type: string
        - name: per_page
          in: query
          description: The number of users per page. There is a maximum limit of 200 users per page.
          default: "60"
          type: string
        - name: in_team
          in: query
          description: The ID of the team to get users for.
          type: string
        - name: not_in_team
          in: query
          description: The ID of the team to exclude users for. Must not be used with "in_team" query parameter.
          type: string
        - name: in_channel
          in: query
          description: The ID of the channel to get users for.
          type: string
        - name: not_in_channel
          in: query
          description: The ID of the channel to exclude users for. Must be used with "in_channel" query parameter.
          type: string
        - name: group_constrained
          in: query
          description: When used with `not_in_channel` or `not_in_team`, returns only the users that are allowed to join the channel or team based on its group constrains.
          type: boolean
        - name: without_team
          in: query
          description: Whether or not to list users that are not on any team. This option takes precendence over `in_team`, `in_channel`, and `not_in_channel`.
          type: boolean
        - name: sort
          in: query
          description: |
            Sort is only available in conjunction with certain options below. The paging parameter is also always available.

            ##### `in_team`
            Can be "", "last_activity_at" or "create_at".
            When left blank, sorting is done by username.
            __Minimum server version__: 4.0
            ##### `in_channel`
            Can be "", "status".
            When left blank, sorting is done by username. `status` will sort by User's current status (Online, Away, DND, Offline), then by Username.
            __Minimum server version__: 4.7
          type: string
      responses:
        '200':
          description: User page retrieval successful
          schema:
            type: array
            items:
              $ref: '#/definitions/User'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")


            // page, perPage, etag
            users := Client.GetUsers(0, 60, "")
            users = Client.GetUsersInChannel("channelid", 0, 60, "")
            users = Client.GetUsersNotInChannel("teamid", "channelid", 0, 60, "")
            users = Client.GetUsersInTeam("teamid", 0, 60, "")
            users = Client.GetUsersNotInTeam("teamid", 0, 60, "")
            users = Client.GetUsersWithoutTeam(0, 60, "")

  /users/ids:
    post:
      tags:
        - users
      summary: Get users by ids
      description: |
        Get a list of users based on a provided list of user ids.
        ##### Permissions
        Requires an active session but no other permissions.
      parameters:
        - in: body
          name: body
          description: List of user ids
          required: true
          schema:
            type: array
            items:
              type: string
        - name: since
          in: query
          description: |
            Only return users that have been modified since the given Unix timestamp (in milliseconds).

            __Minimum server version__: 5.14
          type: integer
      responses:
        '200':
          description: User list retrieval successful
          schema:
            type: array
            items:
              $ref: '#/definitions/User'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'

  /users/group_channels:
    post:
      tags:
        - users
      summary: Get users by group channels ids
      description: |
        Get an object containing a key per group channel id in the
        query and its value as a list of users members of that group
        channel.

        The user must be a member of the group ids in the query, or
        they will be omitted from the response.
        ##### Permissions
        Requires an active session but no other permissions.

        __Minimum server version__: 5.14
      parameters:
        - in: body
          name: body
          description: List of group channel ids
          required: true
          schema:
            type: array
            items:
              type: string
      responses:
        '200':
          description: User list retrieval successful
          schema:
            type: object
            properties:
              "<CHANNEL_ID>":
                type: array
                items:
                  $ref: '#/definitions/User'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'

  /users/usernames:
    post:
      tags:
        - users
      summary: Get users by usernames
      description: |
        Get a list of users based on a provided list of usernames.
        ##### Permissions
        Requires an active session but no other permissions.
      parameters:
        - in: body
          name: body
          description: List of usernames
          required: true
          schema:
            type: array
            items:
              type: string
      responses:
        '200':
          description: User list retrieval successful
          schema:
            type: array
            items:
              $ref: '#/definitions/User'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            users, resp := Client.GetUsersByUsernames([]string{"username1", "username2"})

  /users/search:
    post:
      tags:
        - users
      summary: Search users
      description: |
        Get a list of users based on search criteria provided in the request body. Searches are typically done against username, full name, nickname and email unless otherwise configured by the server.
        ##### Permissions
        Requires an active session and `read_channel` and/or `view_team` permissions for any channels or teams specified in the request body.
      parameters:
        - in: body
          name: body
          description: Search criteria
          required: true
          schema:
            type: object
            required:
              - term
            properties:
              term:
                description: The term to match against username, full name, nickname and email
                type: string
              team_id:
                description: If provided, only search users on this team
                type: string
              not_in_team_id:
                description: If provided, only search users not on this team
                type: string
              in_channel_id:
                description: If provided, only search users in this channel
                type: string
              not_in_channel_id:
                description: If provided, only search users not in this channel. Must specifiy `team_id` when using this option
                type: string
              group_constrained:
                description: When used with `not_in_channel_id` or `not_in_team_id`, returns only the users that are allowed to join the channel or team based on its group constrains.
                type: boolean
              allow_inactive:
                description: When `true`, include deactivated users in the results
                type: boolean
              without_team:
                type: boolean
                description: Set this to `true` if you would like to search for users that are not on a team. This option takes precendence over `team_id`, `in_channel_id`, and `not_in_channel_id`.
              limit:
                description: |
                    The maximum number of users to return in the results

                    __Available as of server version 5.6. Defaults to `100` if not provided or on an earlier server version.__
                type: integer
                default: 100
      responses:
        '200':
          description: User list retrieval successful
          schema:
            type: array
            items:
              $ref: '#/definitions/User'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            teamID := "4xp9fdt77pncbef59f4k1qe83o"
            teamID2 := "JhMjDX9rAlCdBf0l9oyq4eGhxw"
            channelID := "Ej3SKOHlWIKAblkUTK5Xvkj2cm"
            channelID2 := "dWdfrUSdjJ7kyBvyBCgCav67Kz"

            users, resp := Client.SearchUsers(&model.UserSearch{
              Term:           "searchTerm",
              TeamId:         teamID,
              NotInTeamId:    teamID2,
              InChannelId:    channelID,
              NotInChannelId: channelID2,
              AllowInactive:  true,
              WithoutTeam:    true,
              Limit:          100,
              Role:           "admin",
            })

  /users/autocomplete:
    get:
      tags:
        - users
      summary: Autocomplete users
      description: |
        Get a list of users for the purpose of autocompleting based on the provided search term. Specify a combination of `team_id` and `channel_id` to filter results further.
        ##### Permissions
        Requires an active session and `view_team` and `read_channel` on any teams or channels used to filter the results further.
      parameters:
        - name: team_id
          in: query
          description: Team ID
          type: string
        - name: channel_id
          in: query
          description: Channel ID
          type: string
        - name: name
          in: query
          description: Username, nickname first name or last name
          required: true
          type: string
        - name: limit
          in: query
          description: |
              The maximum number of users to return in each subresult

              __Available as of server version 5.6. Defaults to `100` if not provided or on an earlier server version.__
          type: integer
          default: 100
      responses:
        '200':
          description: User autocomplete successful
          schema:
            $ref: '#/definitions/UserAutocomplete'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            teamID := "4xp9fdt77pncbef59f4k1qe83o"
            channelID := "Ej3SKOHlWIKAblkUTK5Xvkj2cm"
            username := "testUsername"

            users, resp := Client.AutocompleteUsersInChannel(teamID, channelID, username, 100, "")

  /users/stats:
    get:
      tags:
        - users
      summary: Get total count of users in the system
      description: |
        Get a total count of users in the system.
        ##### Permissions
        Must be authenticated.
      responses:
        '200':
          description: User stats retrieval successful
          schema:
            $ref: '#/definitions/UsersStats'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '404':
          $ref: '#/responses/NotFound'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            stats, resp := Client.GetTotalUsersStats("")

  /users/{user_id}:
    get:
      tags:
        - users
      summary: Get a user
      description: |
        Get a user a object. Sensitive information will be sanitized out.
        ##### Permissions
        Requires an active session but no other permissions.
      parameters:
        - name: user_id
          in: path
          description: User GUID
          required: true
          type: string
      responses:
        '200':
          description: User retrieval successful
          schema:
            $ref: '#/definitions/User'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '404':
          $ref: '#/responses/NotFound'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            userID := "4xp9fdt77pncbef59f4k1qe83o"

            user, resp := Client.GetUser(userID, "")

    put:
      tags:
        - users
      summary: Update a user
      description: |
        Update a user by providing the user object. The fields that can be updated are defined in the request body, all other provided fields will be ignored. Any fields not included in the request body will be set to null or reverted to default values.
        ##### Permissions
        Must be logged in as the user being updated or have the `edit_other_users` permission.
      parameters:
        - name: user_id
          in: path
          description: User GUID
          required: true
          type: string
        - in: body
          name: body
          description: User object that is to be updated
          required: true
          schema:
            type: object
            required:
              - id
            properties:
              id:
                type: string
              email:
                type: string
              username:
                type: string
              first_name:
                 type: string
              last_name:
                type: string
              nickname:
                type: string
              locale:
                type: string
              position:
                type: string
              props:
                type: object
              notify_props:
                $ref: '#/definitions/UserNotifyProps'
      responses:
        '200':
          description: User update successful
          schema:
            $ref: '#/definitions/User'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            userID := "4xp9fdt77pncbef59f4k1qe83o"
            email := "test@domain.com"
            username := "testUsername"
            firstName := "testFirstname"
            lastName := "testLastname"
            nickname := "testNickname"
            locale := "en"
            position := "testPosition"
            props := model.StringMap{}
            props["testPropKey"] = "testPropValue"
            notifyProps := model.StringMap{}
            notifyProps["comment"] = "somethingrandom"

            user, resp := Client.UpdateUser(&model.User{
              Id:          userID,
              Email:       email,
              Username:    username,
              FirstName:   firstName,
              LastName:    lastName,
              Nickname:    nickname,
              Locale:      locale,
              Position:    position,
              Props:       props,
              NotifyProps: notifyProps,
            })

    delete:
      tags:
        - users
      summary: Deactivate a user account.
      description: |
        Deactivates the user and revokes all its sessions by archiving its user object.
        ##### Permissions
        Must be logged in as the user being deactivated or have the `edit_other_users` permission.
      parameters:
        - name: user_id
          in: path
          description: User GUID
          required: true
          type: string
      responses:
        '200':
          description: User deactivation successful
          schema:
            $ref: '#/definitions/StatusOK'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '404':
          $ref: '#/responses/NotFound'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            userID := "4xp9fdt77pncbef59f4k1qe83o"

            ok, resp := Client.DeleteUser(userID)

  /users/{user_id}/patch:
    put:
      tags:
        - users
      summary: Patch a user
      description: |
        Partially update a user by providing only the fields you want to update. Omitted fields will not be updated. The fields that can be updated are defined in the request body, all other provided fields will be ignored.
        ##### Permissions
        Must be logged in as the user being updated or have the `edit_other_users` permission.
      parameters:
        - name: user_id
          in: path
          description: User GUID
          required: true
          type: string
        - in: body
          name: body
          description: User object that is to be updated
          required: true
          schema:
            type: object
            properties:
              email:
                type: string
              username:
                type: string
              first_name:
                 type: string
              last_name:
                type: string
              nickname:
                type: string
              locale:
                type: string
              position:
                type: string
              props:
                type: object
              notify_props:
                $ref: '#/definitions/UserNotifyProps'
      responses:
        '200':
          description: User patch successful
          schema:
            $ref: '#/definitions/User'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            userID := "4xp9fdt77pncbef59f4k1qe83o"

            patch := &model.UserPatch{}
            patch.Email = model.NewString("test@domain.com")
            patch.Username = model.NewString("testUsername")
            patch.FirstName = model.NewString("testFirstname")
            patch.LastName = model.NewString("testLastname")
            patch.Nickname = model.NewString("testNickname")
            patch.Locale = model.NewString("en")
            patch.Position = model.NewString("testPosition")
            patch.Props = model.StringMap{}
            patch.Props["testPropKey"] = "testPropValue"
            patch.NotifyProps = model.StringMap{}
            patch.NotifyProps["comment"] = "somethingrandom"

            user, resp := Client.PatchUser(userID, patch)

  /users/{user_id}/roles:
    put:
      tags:
        - users
      summary: Update a user's roles
      description: |
        Update a user's system-level roles. Valid user roles are "system_user", "system_admin" or both of them. Overwrites any previously assigned system-level roles.
        ##### Permissions
        Must have the `manage_roles` permission.
      parameters:
        - name: user_id
          in: path
          description: User GUID
          required: true
          type: string
        - in: body
          name: roles
          description: Space-delimited system roles to assign to the user
          required: true
          schema:
            type: object
            required:
              - roles
            properties:
              roles:
                type: string
      responses:
        '200':
          description: User roles update successful
          schema:
            $ref: '#/definitions/StatusOK'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            userID := "4xp9fdt77pncbef59f4k1qe83o"
            roles := "team_user team_admin"

            ok, resp = Client.UpdateUserRoles(userID, roles)

  /users/{user_id}/active:
    put:
      tags:
        - users
      summary: Update user active status
      description: |
        Update user active or inactive status.

        __Since server version 4.6, users using a SSO provider to login can be activated or deactivated with this endpoint. However, if their activation status in Mattermost does not reflect their status in the SSO provider, the next synchronization or login by that user will reset the activation status to that of their account in the SSO provider. Server versions 4.5 and before do not allow activation or deactivation of SSO users from this endpoint.__
        ##### Permissions
        User can deactivate themselves.
        User with `manage_system` permission can activate or deactivate a user.
      parameters:
        - name: user_id
          in: path
          description: User GUID
          required: true
          type: string
        - in: body
          name: body
          description: Use `true` to set the user active, `false` for inactive
          required: true
          schema:
            type: object
            required:
              - active
            properties:
              active:
                type: boolean
      responses:
        '200':
          description: User active status update successful
          schema:
            $ref: '#/definitions/StatusOK'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            userID := "4xp9fdt77pncbef59f4k1qe83o"

            ok, resp := Client.UpdateUserActive(userID, true)
  /users/{user_id}/image:
    get:
      tags:
        - users
      summary: Get user's profile image
      description: |
        Get a user's profile image based on user_id string parameter.
        ##### Permissions
        Must be logged in.
      parameters:
        - name: user_id
          in: path
          description: User GUID
          required: true
          type: string
      responses:
        '200':
          description: User's profile image
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '404':
          $ref: '#/responses/NotFound'
        '501':
          $ref: '#/responses/NotImplemented'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            userID := "4xp9fdt77pncbef59f4k1qe83o"

            data, resp := Client.GetProfileImage(userID, "")
    post:
      tags:
        - users
      summary: Set user's profile image
      description: |
        Set a user's profile image based on user_id string parameter.
        ##### Permissions
        Must be logged in as the user being updated or have the `edit_other_users` permission.
      consumes: ["multipart/form-data"]
      parameters:
        - name: image
          in: formData
          description: The image to be uploaded
          required: true
          type: file
        - name: user_id
          in: path
          description: User GUID
          required: true
          type: string
      responses:
        '200':
          description: Profile image set successful
          schema:
            $ref: '#/definitions/StatusOK'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '404':
          $ref: '#/responses/NotFound'
        '501':
          $ref: '#/responses/NotImplemented'
      x-code-samples:
        - lang: 'Go'
          source: |
            import (
              "io/ioutil"
              "log"

              "github.com/mattermost/mattermost-server/model"
            )

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            data, err := ioutil.ReadFile("profile_pic.png")
            if err != nil {
              log.Fatal(err)
            }

            userID := "4xp9fdt77pncbef59f4k1qe83o"

            ok, resp := Client.SetProfileImage(userID, data)
    delete:
      tags:
        - users
      summary: Delete user's profile image
      description: |
        Delete user's profile image and reset to default image based on user_id string parameter.
        ##### Permissions
        Must be logged in as the user being updated or have the `edit_other_users` permission.
        __Minimum server version__: 5.5
      parameters:
        - name: user_id
          in: path
          description: User GUID
          required: true
          type: string
      responses:
        '200':
          description: Profile image reset successful
          schema:
            $ref: '#/definitions/StatusOK'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '404':
          $ref: '#/responses/NotFound'
        '501':
          $ref: '#/responses/NotImplemented'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            userID := "4xp9fdt77pncbef59f4k1qe83o"

            // Deleting user's profile image consists on resetting it to default one
            ok, resp := Client.SetDefaultProfileImage(userID)
  /users/{user_id}/image/default:
    get:
      tags:
        - users
      summary: Return user's default (generated) profile image
      description: |
        Returns the default (generated) user profile image based on user_id string parameter.
        ##### Permissions
        Must be logged in.
        __Minimum server version__: 5.5
      parameters:
        - name: user_id
          in: path
          description: User GUID
          required: true
          type: string
      responses:
        '200':
          description: Default profile image
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '404':
          $ref: '#/responses/NotFound'
        '501':
          $ref: '#/responses/NotImplemented'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            userID := "4xp9fdt77pncbef59f4k1qe83o"

            ok, resp := Client.SetDefaultProfileImage(userID)

  /users/username/{username}:
    get:
      tags:
        - users
      summary: Get a user by username
      description: |
        Get a user object by providing a username. Sensitive information will be sanitized out.
        ##### Permissions
        Requires an active session but no other permissions.
      parameters:
        - name: username
          in: path
          description: Username
          required: true
          type: string
      responses:
        '200':
          description: User retrieval successful
          schema:
            $ref: '#/definitions/User'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '404':
          $ref: '#/responses/NotFound'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            userID := "4xp9fdt77pncbef59f4k1qe83o"

            user, resp := Client.GetUserByUsername(userID, "")

  /users/password/reset:
    post:
      tags:
        - users
      summary: Reset password
      description: |
        Update the password for a user using a one-use, timed recovery code tied to the user's account. Only works for non-SSO users.
        ##### Permissions
        No permissions required.
      parameters:
        - in: body
          name: body
          required: true
          schema:
            type: object
            required:
              - code
              - new_password
            properties:
              code:
                description: The recovery code
                type: string
              new_password:
                description: The new password for the user
                type: string
      responses:
        '200':
          description: User password update successful
          schema:
            $ref: '#/definitions/StatusOK'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            code := "4xp9fdt77pncbef59f4k1qe83o"
            newPassword := "awesomePassword"

            success, resp = Client.ResetPassword(code, newPassword)
  /users/{user_id}/mfa:
    put:
      tags:
        - users
      summary: Update a user's MFA
      description: |
        Activates multi-factor authentication for the user if `activate` is true and a valid `code` is provided. If activate is false, then `code` is not required and multi-factor authentication is disabled for the user.
        ##### Permissions
        Must be logged in as the user being updated or have the `edit_other_users` permission.
      parameters:
        - name: user_id
          in: path
          description: User GUID
          required: true
          type: string
        - in: body
          name: body
          required: true
          schema:
            type: object
            required:
              - activate
            properties:
              activate:
                description: Use `true` to activate, `false` to deactivate
                type: boolean
              code:
                description: The code produced by your MFA client. Required if `activate` is true
                type: string
      responses:
        '200':
          description: User MFA update successful
          schema:
            $ref: '#/definitions/StatusOK'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '501':
          $ref: '#/responses/NotImplemented'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            userID := "BbaYBYDV5IDOZFiJGBSzkw1k5u"
            code := "4xp9fdt77pncbef59f4k1qe83o"

            ok, resp := Client.UpdateUserMfa(userID, code, true)

  /users/{user_id}/mfa/generate:
    post:
      tags:
        - users
      summary: Generate MFA secret
      description: |
        Generates an multi-factor authentication secret for a user and returns it as a string and as base64 encoded QR code image.
        ##### Permissions
        Must be logged in as the user or have the `edit_other_users` permission.
      parameters:
        - name: user_id
          in: path
          description: User GUID
          required: true
          type: string
      responses:
        '200':
          description: MFA secret generation successful
          schema:
            type: object
            properties:
              secret:
                description: The MFA secret as a string
                type: string
              qr_code:
                description: A base64 encoded QR code image
                type: string
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '404':
          $ref: '#/responses/NotFound'
        '501':
          $ref: '#/responses/NotImplemented'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            userID := "BbaYBYDV5IDOZFiJGBSzkw1k5u"

            mfaSecret, resp = Client.GenerateMfaSecret(userID)
  /users/mfa:
    post:
      tags:
        - users
      summary: Check MFA
      description: |
        Check if a user has multi-factor authentication active on their account by providing a login id. Used to check whether an MFA code needs to be provided when logging in.
        ##### Permissions
        No permission required.
      parameters:
        - in: body
          name: body
          required: true
          schema:
            type: object
            required:
              - login_id
            properties:
              login_id:
                description: The email or username used to login
                type: string
      responses:
        '200':
          description: MFA check successful
          schema:
            type: object
            properties:
              mfa_required:
                description: Value will `true` if MFA is active, `false` otherwise
                type: boolean
        '400':
          $ref: '#/responses/BadRequest'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")

            loginID := "test@domain.com"

            required, resp := Client.CheckUserMfa(loginID)

  /users/{user_id}/password:
    put:
      tags:
        - users
      summary: Update a user's password
      description: |
        Update a user's password. New password must meet password policy set by server configuration. Current password is required if you're updating your own password.
        ##### Permissions
        Must be logged in as the user the password is being changed for or have `manage_system` permission.
      parameters:
        - name: user_id
          in: path
          description: User GUID
          required: true
          type: string
        - in: body
          name: body
          required: true
          schema:
            type: object
            required:
              - new_password
            properties:
              current_password:
                description: The current password for the user
                type: string
              new_password:
                description: The new password for the user
                type: string
      responses:
        '200':
          description: User password update successful
          schema:
            $ref: '#/definitions/StatusOK'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            userID := "BbaYBYDV5IDOZFiJGBSzkw1k5u"
            currentPassword := "badPassword"
            newPassword := "awesomePassword"

            ok, resp := Client.UpdateUserPassword(userID, currentPassword, newPassword)

  /users/password/reset/send:
    post:
      tags:
        - users
      summary: Send password reset email
      description: |
        Send an email containing a link for resetting the user's password. The link will contain a one-use, timed recovery code tied to the user's account. Only works for non-SSO users.
        ##### Permissions
        No permissions required.
      parameters:
        - in: body
          name: body
          required: true
          schema:
            type: object
            required:
              - email
            properties:
              email:
                description: The email of the user
                type: string
      responses:
        '200':
          description: Email sent if account exists
          schema:
            $ref: '#/definitions/StatusOK'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")

            email := "test@domain.com"

            pass, resp := Client.SendVerificationEmail(email)

  /users/email/{email}:
    get:
      tags:
        - users
      summary: Get a user by email
      description: |
        Get a user object by providing a user email. Sensitive information will be sanitized out.
        ##### Permissions
        Requires an active session and for the current session to be able to view another user's email based on the server's privacy settings.
      parameters:
        - name: email
          in: path
          description: User Email
          required: true
          type: string
      responses:
        '200':
          description: User retrieval successful
          schema:
            $ref: '#/definitions/User'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '404':
          $ref: '#/responses/NotFound'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            email := "test@domain.com"

            user, resp := Client.GetUserByEmail(email, "")

  /users/{user_id}/sessions:
    get:
      tags:
        - users
      summary: Get user's sessions
      description: |
        Get a list of sessions by providing the user GUID. Sensitive information will be sanitized out.
        ##### Permissions
        Must be logged in as the user being updated or have the `edit_other_users` permission.
      parameters:
        - name: user_id
          in: path
          description: User GUID
          required: true
          type: string
      responses:
        '200':
          description: User session retrieval successful
          schema:
            type: array
            items:
              $ref: '#/definitions/Session'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            userID := "zWEyrTZ7GZ22aBSfoX60iWryTY"

            sessions, resp := Client.GetSessions(userID, "")

  /users/{user_id}/sessions/revoke:
    post:
      tags:
        - users
      summary: Revoke a user session
      description: |
        Revokes a user session from the provided user id and session id strings.
        ##### Permissions
        Must be logged in as the user being updated or have the `edit_other_users` permission.
      parameters:
        - name: user_id
          in: path
          description: User GUID
          required: true
          type: string
        - in: body
          name: body
          required: true
          schema:
            type: object
            required:
              - session_id
            properties:
              session_id:
                description: The session GUID to revoke.
                type: string
      responses:
        '200':
          description: User session revoked successful
          schema:
            $ref: '#/definitions/StatusOK'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            userID := "zWEyrTZ7GZ22aBSfoX60iWryTY"
            sessionID := "adWv1qPZmHdtxk7Lmqh6RtxWxS"

            ok, resp = Client.RevokeSession(userID, sessionID)

  /users/{user_id}/sessions/revoke/all:
      post:
        tags:
          - users
        summary: Revoke all active sessions for a user
        description: |
          Revokes all user sessions from the provided user id and session id strings.
          ##### Permissions
          Must be logged in as the user being updated or have the `edit_other_users` permission.
          __Minimum server version__: 4.4
        parameters:
          - name: user_id
            in: path
            description: User GUID
            required: true
            type: string
        responses:
          '200':
            description: User sessions revoked successfully
            schema:
              $ref: '#/definitions/StatusOK'
          '400':
            $ref: '#/responses/BadRequest'
          '401':
            $ref: '#/responses/Unauthorized'
          '403':
            $ref: '#/responses/Forbidden'
        x-code-samples:
          - lang: 'Go'
            source: |
              import "github.com/mattermost/mattermost-server/model"

              Client := model.NewAPIv4Client("https://your-mattermost-url.com")
              Client.Login("email@domain.com", "Password1")

              userID := "zWEyrTZ7GZ22aBSfoX60iWryTY"

              ok, resp := Client.RevokeAllSessions(userID)

  /users/sessions/device:
    put:
      tags:
        - users
      summary: Attach mobile device
      description: |
        Attach a mobile device id to the currently logged in session. This will enable push notifications for a user, if configured by the server.
        ##### Permissions
        Must be authenticated.
      parameters:
        - in: body
          name: body
          required: true
          schema:
            type: object
            required:
              - device_id
            properties:
              device_id:
                description: Mobile device id. For Android prefix the id with `android:` and Apple with `apple:`
                type: string
      responses:
        '200':
          description: Device id attach successful
          schema:
            $ref: '#/definitions/StatusOK'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            deviceID := "zWEyrTZ7GZ22aBSfoX60iWryTY"

            pass, resp := Client.AttachDeviceId(deviceID)

  /users/{user_id}/audits:
    get:
      tags:
        - users
      summary: Get user's audits
      description: |
        Get a list of audit by providing the user GUID.
        ##### Permissions
        Must be logged in as the user or have the `edit_other_users` permission.
      parameters:
        - name: user_id
          in: path
          description: User GUID
          required: true
          type: string
      responses:
        '200':
          description: User audits retrieval successful
          schema:
            type: array
            items:
              $ref: '#/definitions/Audit'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            userID := "zWEyrTZ7GZ22aBSfoX60iWryTY"

            audits, resp := Client.GetUserAudits(userID, 0, 100, "")

  /users/email/verify:
    post:
      tags:
        - users
      summary: Verify user email
      description: |
        Verify the email used by a user to sign-up their account with.
        ##### Permissions
        No permissions required.
      parameters:
        - in: body
          name: body
          required: true
          schema:
            type: object
            required:
              - token
            properties:
              token:
                description: The token given to validate the email
                type: string
      responses:
        '200':
          description: User email verification successful
          schema:
            $ref: '#/definitions/StatusOK'
        '400':
          $ref: '#/responses/BadRequest'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")

            token := "zWEyrTZ7GZ22aBSfoX60iWryTY"

            ok, resp := Client.VerifyUserEmail(token)

  /users/email/verify/send:
    post:
      tags:
        - users
      summary: Send verification email
      description: |
        Send an email with a verification link to a user that has an email matching the one in the request body. This endpoint will return success even if the email does not match any users on the system.
        ##### Permissions
        No permissions required.
      parameters:
        - in: body
          name: body
          required: true
          schema:
            type: object
            required:
              - email
            properties:
              email:
                description: Email of a user
                type: string
      responses:
        '200':
          description: Email send successful if email exists
          schema:
            $ref: '#/definitions/StatusOK'
        '400':
          $ref: '#/responses/BadRequest'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")

            email := "test@domain.com"

            pass, resp := Client.SendVerificationEmail(email)

  /users/login/switch:
    post:
      tags:
        - users
      summary: Switch login method
      description: |
        Switch a user's login method from using email to OAuth2/SAML/LDAP or back to email. When switching to OAuth2/SAML, account switching is not complete until the user follows the returned link and completes any steps on the OAuth2/SAML service provider.

        To switch from email to OAuth2/SAML, specify `current_service`, `new_service`, `email` and `password`.

        To switch from OAuth2/SAML to email, specify `current_service`, `new_service`, `email` and `new_password`.

        To switch from email to LDAP/AD, specify `current_service`, `new_service`, `email`, `password`, `ldap_ip` and `new_password` (this is the user's LDAP password).

        To switch from LDAP/AD to email, specify `current_service`, `new_service`, `ldap_ip`, `password` (this is the user's LDAP password), `email`  and `new_password`.

        Additionally, specify `mfa_code` when trying to switch an account on LDAP/AD or email that has MFA activated.

        ##### Permissions
        No current authentication required except when switching from OAuth2/SAML to email.
      parameters:
        - in: body
          name: body
          required: true
          schema:
            type: object
            required:
              - current_service
              - new_service
            properties:
              current_service:
                description: The service the user currently uses to login
                type: string
              new_service:
                description: The service the user will use to login
                type: string
              email:
                description: The email of the user
                type: string
              password:
                description: The password used with the current service
                type: string
              mfa_code:
                description: The MFA code of the current service
                type: string
              ldap_id:
                description: The LDAP/AD id of the user
                type: string
      responses:
        '200':
          description: Login method switch or request successful
          schema:
            type: object
            properties:
              follow_link:
                description: The link for the user to follow to login or to complete the account switching when the current service is OAuth2/SAML
                type: string
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '404':
          $ref: '#/responses/NotFound'
        '501':
          $ref: '#/responses/NotImplemented'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")

            currentService := "email"
            newService := "gitlab"
            email := "test@domain.com"
            password := "awesomePassword"
            mfaCode := "adWv1qPZmHdtxk7Lmqh6RtxWxS"
            ldapLoginID := "RdDjEDlkWgt7ndjyVLwWGvnX8c"


            link, resp := Client.SwitchAccountType(&model.SwitchRequest{
              CurrentService: currentService,
              NewService:     newService,
              Email:          email,
              Password:       password,
              MfaCode:        mfaCode,
              LdapLoginId:    ldapLoginID,
            })

  /users/{user_id}/tokens:
    post:
      tags:
        - users
      summary: Create a user access token
      description: |
        Generate a user access token that can be used to authenticate with the Mattermost REST API.

        __Minimum server version__: 4.1

        ##### Permissions
        Must have `create_user_access_token` permission. For non-self requests, must also have the `edit_other_users` permission.
      parameters:
        - name: user_id
          in: path
          description: User GUID
          required: true
          type: string
        - in: body
          name: token
          required: true
          schema:
            type: object
            required:
              - description
            properties:
              description:
                description: A description of the token usage
                type: string
      responses:
        '201':
          description: User access token creation successful
          schema:
            $ref: '#/definitions/UserAccessToken'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            userID := "adWv1qPZmHdtxk7Lmqh6RtxWxS"

            userAccessToken, resp := Client.CreateUserAccessToken(userID, "test token")

    get:
      tags:
        - users
      summary: Get user access tokens
      description: |
        Get a list of user access tokens for a user. Does not include the actual authentication tokens. Use query parameters for paging.

        __Minimum server version__: 4.1

        ##### Permissions
        Must have `read_user_access_token` permission. For non-self requests, must also have the `edit_other_users` permission.
      parameters:
        - name: user_id
          in: path
          description: User GUID
          required: true
          type: string
        - name: page
          in: query
          description: The page to select.
          default: "0"
          type: string
        - name: per_page
          in: query
          description: The number of tokens per page.
          default: "60"
          type: string
      responses:
        '200':
          description: User access tokens retrieval successful
          schema:
            type: array
            items:
              $ref: '#/definitions/UserAccessTokenSanitized'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            userID := "adWv1qPZmHdtxk7Lmqh6RtxWxS"

            tokens, resp := Client.GetUserAccessTokensForUser(userID, 0, 100)

  /users/tokens:
    get:
      tags:
        - users
      summary: Get user access tokens
      description: |
        Get a page of user access tokens for users on the system. Does not include the actual authentication tokens. Use query parameters for paging.

        __Minimum server version__: 4.7

        ##### Permissions
        Must have `manage_system` permission.
      parameters:
        - name: page
          in: query
          description: The page to select.
          default: "0"
          type: string
        - name: per_page
          in: query
          description: The number of tokens per page.
          default: "60"
          type: string
      responses:
        '200':
          description: User access tokens retrieval successful
          schema:
            type: array
            items:
              $ref: '#/definitions/UserAccessTokenSanitized'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            tokens, resp := Client.GetUserAccessTokens(0, 100)

  /users/tokens/revoke:
    post:
      tags:
        - users
      summary: Revoke a user access token
      description: |
        Revoke a user access token and delete any sessions using the token.

        __Minimum server version__: 4.1

        ##### Permissions
        Must have `revoke_user_access_token` permission. For non-self requests, must also have the `edit_other_users` permission.
      parameters:
        - in: body
          name: token_id
          required: true
          schema:
            type: object
            required:
              - token_id
            properties:
              token_id:
                description: The user access token GUID to revoke
                type: string
      responses:
        '200':
          description: User access token revoke successful
          schema:
            $ref: '#/definitions/StatusOK'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            tokenID := "adWv1qPZmHdtxk7Lmqh6RtxWxS"

            ok, resp := Client.RevokeUserAccessToken(tokenID)

  /users/tokens/{token_id}:
    get:
      tags:
        - users
      summary: Get a user access token
      description: |
        Get a user access token. Does not include the actual authentication token.

        __Minimum server version__: 4.1

        ##### Permissions
        Must have `read_user_access_token` permission. For non-self requests, must also have the `edit_other_users` permission.
      parameters:
        - name: token_id
          in: path
          description: User access token GUID
          required: true
          type: string
      responses:
        '200':
          description: User access token retrieval successful
          schema:
            $ref: '#/definitions/UserAccessTokenSanitized'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '404':
          $ref: '#/responses/NotFound'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            tokenID := "adWv1qPZmHdtxk7Lmqh6RtxWxS"

            token, resp := Client.GetUserAccessToken(tokenID)

  /users/tokens/disable:
    post:
      tags:
        - users
      summary: Disable personal access token
      description: |
        Disable a personal access token and delete any sessions using the token. The token can be re-enabled using `/users/tokens/enable`.

        __Minimum server version__: 4.4

        ##### Permissions
        Must have `revoke_user_access_token` permission. For non-self requests, must also have the `edit_other_users` permission.
      parameters:
        - in: body
          name: token_id
          required: true
          schema:
            type: object
            required:
              - token_id
            properties:
              token_id:
                description: The personal access token GUID to disable
                type: string
      responses:
        '200':
          description: Personal access token disable successful
          schema:
            $ref: '#/definitions/StatusOK'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            tokenID := "adWv1qPZmHdtxk7Lmqh6RtxWxS"

            ok, resp := Client.DisableUserAccessToken(tokenID)

  /users/tokens/enable:
    post:
      tags:
        - users
      summary: Enable personal access token
      description: |
        Re-enable a personal access token that has been disabled.

        __Minimum server version__: 4.4

        ##### Permissions
        Must have `create_user_access_token` permission. For non-self requests, must also have the `edit_other_users` permission.
      parameters:
        - in: body
          name: token_id
          required: true
          schema:
            type: object
            required:
              - token_id
            properties:
              token_id:
                description: The personal access token GUID to enable
                type: string
      responses:
        '200':
          description: Personal access token enable successful
          schema:
            $ref: '#/definitions/StatusOK'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            tokenID := "adWv1qPZmHdtxk7Lmqh6RtxWxS"

            ok, resp := Client.EnableUserAccessToken(tokenID)

  /users/tokens/search:
    post:
      tags:
        - users
      summary: Search tokens
      description: |
        Get a list of tokens based on search criteria provided in the request body. Searches are done against the token id, user id and username.

        __Minimum server version__: 4.7

        ##### Permissions
        Must have `manage_system` permission.
      parameters:
        - in: body
          name: body
          description: Search criteria
          required: true
          schema:
            type: object
            required:
              - term
            properties:
              term:
                description: The search term to match against the token id, user id or username.
                type: string
      responses:
        '200':
          description: Personal access token search successful
          schema:
            type: array
            items:
              $ref: '#/definitions/UserAccessTokenSanitized'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            tokenID := "adWv1qPZmHdtxk7Lmqh6RtxWxS"

            userAccessTokens, resp = Client.SearchUserAccessTokens(&model.UserAccessTokenSearch{Term: tokenID})

  /users/{user_id}/auth:
    put:
      tags:
        - users
      summary: Update a user's authentication method
      description: |
        Updates a user's authentication method. This can be used to change them to/from LDAP authentication for example.

        __Minimum server version__: 4.6
        ##### Permissions
        Must have the `edit_other_users` permission.
      parameters:
        - name: user_id
          in: path
          description: User GUID
          required: true
          type: string
        - in: body
          name: body
          required: true
          schema:
            $ref: '#/definitions/UserAuthData'
      responses:
        '200':
          description: User auth update successful
          schema:
            $ref: '#/definitions/UserAuthData'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '501':
          $ref: '#/responses/NotImplemented'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            userID := "adWv1qPZmHdtxk7Lmqh6RtxWxS"
            user, resp := Client.GetUser(userID, "")
            userAuth := &model.UserAuth{}
            userAuth.AuthData = user.AuthData
            userAuth.AuthService = user.AuthService
            userAuth.Password = user.Password

            user, resp := Client.UpdateUserAuth(userID, userAuth)

  /users/{user_id}/terms_of_service:
    post:
      tags:
        - users
        - terms of service
      summary: Records user action when they accept or decline custom terms of service
      description: |
        Records user action when they accept or decline custom terms of service. Records the action in audit table.
        Updates user's last accepted terms of service ID if they accepted it.

        __Minimum server version__: 5.4
        ##### Permissions
        Must be logged in as the user being acted on.
      parameters:
        - name: user_id
          in: path
          description: User GUID
          required: true
          type: string
        - in: body
          name: body
          description: terms of service details
          required: true
          schema:
            type: object
            required:
            - serviceTermsId
            - accepted
            properties:
              serviceTermsId:
                description: terms of service ID on which the user is acting on
                type: string
              accepted:
                description: true or false, indicates whether the user accepted or rejected the terms of service.
                type: string
      responses:
        '200':
          description: Terms of service action recorded successfully
          schema:
            $ref: '#/definitions/StatusOK'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            userID := "adWv1qPZmHdtxk7Lmqh6RtxWxS"
            serviceTermsID := "RdDjEDlkWgt7ndjyVLwWGvnX8c"

            success, resp = Client.RegisterTermsOfServiceAction(userID, serviceTermsID, true)
    get:
      tags:
      - users
      - terms of service
      summary: Fetches user's latest terms of service action if the latest action was for acceptance.
      description: |
        Will be deprecated in v6.0
        Fetches user's latest terms of service action if the latest action was for acceptance.

        __Minimum server version__: 5.6
        ##### Permissions
        Must be logged in as the user being acted on.
      parameters:
      - name: user_id
        in: path
        description: User GUID
        required: true
        type: string
      responses:
        '200':
          description: User's accepted terms of service action
          schema:
            $ref: '#/definitions/UserTermsOfService'
        '404':
          description: User hasn't performed an action or the latest action was a rejection.
          schema:
            $ref: '#/definitions/AppError'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            userID := "adWv1qPZmHdtxk7Lmqh6RtxWxS"

            userTermsOfService, resp := Client.GetUserTermsOfService(userID, "")

  /users/sessions/revoke/all:
    post:
      tags:
        - users
      summary: Revoke all sessions from all users.
      description: |
        For any session currently on the server (including admin) it will be revoked.
        Clients will be notified to log out users.

        __Minimum server version__: 5.14

        ##### Permissions
        
        Must have `manage_system` permission.

      responses:
        '200':
          description: Sessions successfully revoked.
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            response, err := Client.RevokeSessionsFromAllUsers()
  /users/{user_id}/status:
    get:
      tags:
        - status
      summary: Get user status
      description: |
        Get user status by id from the server.
        ##### Permissions
        Must be authenticated.
      parameters:
        - name: user_id
          in: path
          description: User ID
          required: true
          type: string
      responses:
        '200':
          description: User status retrieval successful
          schema:
            $ref: '#/definitions/Status'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'

    put:
      tags:
        - status
      summary: Update user status
      description: |
        Manually set a user's status. When setting a user's status, the status will remain that value until set "online" again, which will return the status to being automatically updated based on user activity.
        ##### Permissions
        Must have `edit_other_users` permission for the team.
      parameters:
        - name: user_id
          in: path
          description: User ID
          required: true
          type: string
        - name: body
          in: body
          description: Status object that is to be updated
          required: true
          schema:
            type: object
            required:
              - status
              - user_id
            properties:
              user_id:
                type: string
                description: User ID
              status:
                type: string
                description: User status, can be `online`, `away`, `offline` and `dnd`
      responses:
        '200':
          description: User status update successful
          schema:
            $ref: '#/definitions/Status'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'

  /users/status/ids:
    post:
      tags:
        - status
      summary: Get user statuses by id
      description: |
        Get a list of user statuses by id from the server.
        ##### Permissions
        Must be authenticated.
      parameters:
        - name: post
          in: body
          description: List of user ids to fetch
          required: true
          schema:
            type: array
            items:
              type: string
      responses:
        '200':
          description: User statuses retrieval successful
          schema:
            type: array
            items:
              $ref: '#/definitions/Status'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
  /teams:
    post:
      tags:
        - teams
      summary: Create a team
      description: |
        Create a new team on the system.
        ##### Permissions
        Must be authenticated and have the `create_team` permission.
      parameters:
        - in: body
          name: body
          description: Team that is to be created
          required: true
          schema:
            type: object
            required:
              - name
              - display_name
              - type
            properties:
              name:
                type: string
                description: Unique handler for a team, will be present in the team URL
              display_name:
                type: string
                description: Non-unique UI name for the team
              type:
                type: string
                description: "`'O'` for open, `'I'` for invite only"
      responses:
        '201':
          description: Team creation successful
          schema:
            $ref: '#/definitions/Team'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            newTeam, err := Client.CreateTeam(&model.Team{
              Name:        "teamName",
              DisplayName: "TeamDisplayName",
              Type:        "O",
            })

    get:
      tags:
        - teams
      summary: Get teams
      description: |
        For regular users only returns open teams. Users with the "manage_system" permission will return teams regardless of type. The result is based on query string parameters - page and per_page.
        ##### Permissions
        Must be authenticated. "manage_system" permission is required to show all teams.
      parameters:
        - name: page
          in: query
          description: The page to select.
          default: "0"
          type: string
        - name: per_page
          in: query
          description: The number of teams per page.
          default: "60"
          type: string
        - name: include_total_count
          in: query
          type: boolean
          default: false
      responses:
        '200':
          description: Team list retrieval successful
          schema:
            type: array
            items:
              $ref: '#/definitions/Team'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            teams, resp := Client.GetAllTeams("", 0, 100)

  '/teams/{team_id}':
    get:
      tags:
        - teams
      summary: Get a team
      description: |
        Get a team on the system.
        ##### Permissions
        Must be authenticated and have the `view_team` permission.
      parameters:
        - name: team_id
          in: path
          description: Team GUID
          required: true
          type: string
      responses:
        '200':
          description: Team retrieval successful
          schema:
            $ref: '#/definitions/Team'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '404':
          $ref: '#/responses/NotFound'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            teamID := "4xp9fdt77pncbef59f4k1qe83o"

            t, err := Client.GetTeam(teamID, "")

    put:
      tags:
        - teams
      summary: Update a team
      description: |
        Update a team by providing the team object. The fields that can be updated are defined in the request body, all other provided fields will be ignored.
        ##### Permissions
        Must have the `manage_team` permission.
      parameters:
        - name: team_id
          in: path
          description: Team GUID
          required: true
          type: string
        - in: body
          name: body
          description: Team to update
          required: true
          schema:
            type: object
            required:
              - id
              - display_name
              - description
              - company_name
              - allowed_domains
              - invite_id
              - allow_open_invite
            properties:
              id:
                type: string
              display_name:
                type: string
              description:
                type: string
              company_name:
                type: string
              allowed_domains:
                type: string
              invite_id:
                type: string
              allow_open_invite:
                type: string
      responses:
        '200':
          description: Team update successful
          schema:
            $ref: '#/definitions/Team'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '404':
          $ref: '#/responses/NotFound'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            teamID := "4xp9fdt77pncbef59f4k1qe83o"
            inviteID := "qjda3stwafbgpqjaxej3k76sga"

            uteam, resp := Client.UpdateTeam(&model.Team{
              Id:              teamID,
              DisplayName:     "displayName",
              Description:     "description",
              CompanyName:     "companyName",
              AllowedDomains:  "allowedDomains",
              InviteId:        inviteID,
              AllowOpenInvite: false,
            })

    delete:
      tags:
        - teams
      summary: Delete a team
      description: |
        Soft deletes a team, by marking the team as deleted in the database. Soft deleted teams will not be accessible in the user interface.

        Optionally use the permanent query parameter to hard delete the team for compliance reasons. As of server version 5.0, to use this feature `ServiceSettings.EnableAPITeamDeletion` must be set to `true` in the server's configuration.
        ##### Permissions
        Must have the `manage_team` permission.
      parameters:
        - name: team_id
          in: path
          description: Team GUID
          required: true
          type: string
        - name: permanent
          in: query
          description: Permanently delete the team, to be used for compliance reasons only. As of server version 5.0, `ServiceSettings.EnableAPITeamDeletion` must be set to `true` in the server's configuration.
          required: false
          default: false
          type: boolean
      responses:
        '200':
          description: Team deletion successful
          schema:
            $ref: '#/definitions/StatusOK'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '404':
          $ref: '#/responses/NotFound'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            teamID := "4xp9fdt77pncbef59f4k1qe83o"

            // Non-permanent deletion
            ok, resp := Client.SoftDeleteTeam(&model.Team{Id: teamID})

            // Permanent deletion
            ok, resp := Client.PermanentDeleteTeam(&model.Team{Id: teamID})

  '/teams/{team_id}/patch':
    put:
      tags:
        - teams
      summary: Patch a team
      description: |
        Partially update a team by providing only the fields you want to update. Omitted fields will not be updated. The fields that can be updated are defined in the request body, all other provided fields will be ignored.
        ##### Permissions
        Must have the `manage_team` permission.
      parameters:
        - name: team_id
          in: path
          description: Team GUID
          required: true
          type: string
        - in: body
          name: body
          description: Team object that is to be updated
          required: true
          schema:
            type: object
            properties:
              display_name:
                type: string
              description:
                type: string
              company_name:
                 type: string
              invite_id:
                type: string
              allow_open_invite:
                type: boolean
      responses:
        '200':
          description: team patch successful
          schema:
            $ref: '#/definitions/Team'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            patch := &model.TeamPatch{}
            patch.DisplayName = model.NewString("Other name")
            patch.Description = model.NewString("Other description")
            patch.CompanyName = model.NewString("Other company name")
            patch.AllowOpenInvite = model.NewBool(true)

            teamID := "4xp9fdt77pncbef59f4k1qe83o"

            team, resp := Client.PatchTeam(teamID, patch)

  '/teams/name/{name}':
    get:
      tags:
        - teams
      summary: Get a team by name
      description: |
        Get a team based on provided name string
        ##### Permissions
        Must be authenticated, team type is open and have the `view_team` permission.
      parameters:
        - name: name
          in: path
          description: Team Name
          required: true
          type: string
      responses:
        '200':
          description: Team retrieval successful
          schema:
            $ref: '#/definitions/Team'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '404':
          $ref: '#/responses/NotFound'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            team, resp := Client.GetTeamByName("teamName", "")

  '/teams/search':
    post:
      tags:
        - teams
      summary: Search teams
      description: |
        Search teams based on search term provided in the request body.
        ##### Permissions
        Logged in user only shows open teams
        Logged in user with "manage_system" permission shows all teams
      parameters:
        - in: body
          name: body
          description: Search criteria
          required: true
          schema:
            type: object
            required:
              - term
            properties:
              term:
                description: The search term to match against the name or display name of teams
                type: string
      responses:
        '200':
          description: Teams search successful
          schema:
            type: array
            items:
              $ref: '#/definitions/Team'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '404':
          $ref: '#/responses/NotFound'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            teams, resp := Client.SearchTeams(&model.TeamSearch{Term: "searchTerm"})

  '/teams/name/{name}/exists':
    get:
      tags:
        - teams
      summary: Check if team exists
      description: |
        Check if the team exists based on a team name.
        ##### Permissions
        Must be authenticated.
      parameters:
        - name: name
          in: path
          description: Team Name
          required: true
          type: string
      responses:
        '200':
          description: Team retrieval successful
          schema:
            $ref: '#/definitions/TeamExists'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '404':
          $ref: '#/responses/NotFound'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            exists, resp := Client.TeamExists("teamName", "")

  '/users/{user_id}/teams':
    get:
      tags:
        - teams
      summary: Get a user's teams
      description: |
        Get a list of teams that a user is on.
        ##### Permissions
        Must be authenticated as the user or have the `manage_system` permission.
      parameters:
        - name: user_id
          in: path
          description: User GUID
          required: true
          type: string
      responses:
        '200':
          description: Team list retrieval successful
          schema:
            type: array
            items:
              $ref: '#/definitions/Team'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            userID := "4xp9fdt77pncbef59f4k1qe83o"

            teams, resp := Client.GetTeamsForUser(userID, "")

  '/teams/{team_id}/members':
    get:
      tags:
        - teams
      summary: Get team members
      description: |
        Get a page team members list based on query string parameters - team id, page and per page.
        ##### Permissions
        Must be authenticated and have the `view_team` permission.
      parameters:
        - name: team_id
          in: path
          description: Team GUID
          required: true
          type: string
        - name: page
          in: query
          description: The page to select.
          default: "0"
          type: string
        - name: per_page
          in: query
          description: The number of users per page.
          default: "60"
          type: string
      responses:
        '200':
          description: Team members retrieval successful
          schema:
            type: array
            items:
              $ref: '#/definitions/TeamMember'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '404':
          $ref: '#/responses/NotFound'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            teamID := "4xp9fdt77pncbef59f4k1qe83o"

            members, resp := Client.GetTeamMembers(teamID, 0, 100, "")
    post:
      tags:
        - teams
      summary: Add user to team
      description: |
        Add user to the team by user_id.
        ##### Permissions
        Must be authenticated and team be open to add self. For adding another user, authenticated user must have the `add_user_to_team` permission.
      parameters:
        - name: team_id
          in: path
          description: Team GUID
          required: true
          type: string
        - in: body
          name: body
          required: true
          schema:
            type: object
            properties:
              team_id:
                type: string
              user_id:
                type: string
      responses:
        '201':
          description: Team member creation successful
          schema:
            $ref: '#/definitions/TeamMember'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '404':
          $ref: '#/responses/NotFound'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            teamID := "4xp9fdt77pncbef59f4k1qe83o"
            userID := "qjda3stwafbgpqjaxej3k76sga"

            teamMember, resp := Client.AddTeamMember(teamID, userID)

  '/teams/members/invite':
    post:
      tags:
        - teams
      summary: Add user to team from invite
      description: |
        Using either an invite id or hash/data pair from an email invite link, add a user to a team.
        ##### Permissions
        Must be authenticated.
      parameters:
        - name: token
          in: query
          description: Token id from the invitation
          required: true
          type: string
      responses:
        '201':
          description: Team member creation successful
          schema:
            $ref: '#/definitions/TeamMember'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '404':
          $ref: '#/responses/NotFound'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            tokenID := "qjda3stwafbgpqjaxej3k76sga"

            tm, resp = Client.AddTeamMemberFromInvite(tokenID, "")

  '/teams/{team_id}/members/batch':
    post:
      tags:
        - teams
      summary: Add multiple users to team
      description: |
        Add a number of users to the team by user_id.
        ##### Permissions
        Must be authenticated. Authenticated user must have the `add_user_to_team` permission.
      parameters:
        - name: team_id
          in: path
          description: Team GUID
          required: true
          type: string
        - in: body
          name: body
          required: true
          schema:
            type: array
            items:
              $ref: '#/definitions/TeamMember'
      responses:
        '201':
          description: Team members created successfully.
          schema:
            type: array
            items:
              $ref: '#/definitions/TeamMember'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '404':
          $ref: '#/responses/NotFound'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            teamID := "IJyUQLwh1CO9ahbzaQwWwc0ZnV"

            userID := "zWEyrTZ7GZ22aBSfoX60iWryTY"
            userID2 := "NqCSr5HMDZjrWS74IEmedvlOYf"

            tm, resp := Client.AddTeamMembers(teamID, []string{userID, userID2})

  '/users/{user_id}/teams/members':
    get:
      tags:
        - teams
      summary: Get team members for a user
      description: |
        Get a list of team members for a user. Useful for getting the ids of teams the user is on and the roles they have in those teams.
        ##### Permissions
        Must be logged in as the user or have the `edit_other_users` permission.
      parameters:
        - name: user_id
          in: path
          description: User GUID
          required: true
          type: string
      responses:
        '200':
          description: Team members retrieval successful
          schema:
            type: array
            items:
              $ref: '#/definitions/TeamMember'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '404':
          $ref: '#/responses/NotFound'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            userID := "zWEyrTZ7GZ22aBSfoX60iWryTY"

            teamMembers, resp = Client.GetTeamMembersForUser(userID, "")

  '/teams/{team_id}/members/{user_id}':
    get:
      tags:
        - teams
      summary: Get a team member
      description: |
        Get a team member on the system.
        ##### Permissions
        Must be authenticated and have the `view_team` permission.
      parameters:
        - name: team_id
          in: path
          description: Team GUID
          required: true
          type: string
        - name: user_id
          in: path
          description: User GUID
          required: true
          type: string
      responses:
        '200':
          description: Team member retrieval successful
          schema:
            $ref: '#/definitions/TeamMember'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '404':
          $ref: '#/responses/NotFound'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            teamID := "zWEyrTZ7GZ22aBSfoX60iWryTY"
            userID := "NqCSr5HMDZjrWS74IEmedvlOYf"

            teamMember, resp = Client.GetTeamMember(teamID, userID, "")

    delete:
      tags:
        - teams
      summary: Remove user from team
      description: |
        Delete the team member object for a user, effectively removing them from a team.
        ##### Permissions
        Must be logged in as the user or have the `remove_user_from_team` permission.
      parameters:
        - name: team_id
          in: path
          description: Team GUID
          required: true
          type: string
        - name: user_id
          in: path
          description: User GUID
          required: true
          type: string
      responses:
        '200':
          description: Team member deletion successful
          schema:
            $ref: '#/definitions/StatusOK'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '404':
          $ref: '#/responses/NotFound'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            teamID := "zWEyrTZ7GZ22aBSfoX60iWryTY"
            userID := "NqCSr5HMDZjrWS74IEmedvlOYf"

            ok, resp = Client.RemoveTeamMember(teamID, userID)

  '/teams/{team_id}/members/ids':
    post:
      tags:
        - teams
      summary: Get team members by ids
      description: |
        Get a list of team members based on a provided array of user ids.
        ##### Permissions
        Must have `view_team` permission for the team.
      parameters:
        - name: team_id
          in: path
          description: Team GUID
          required: true
          type: string
        - in: body
          name: body
          description: List of user ids
          required: true
          schema:
            type: array
            items:
              type: string
      responses:
        '200':
          description: Team members retrieval successful
          schema:
            type: array
            items:
              $ref: '#/definitions/TeamMember'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            teamID := zWEyrTZ7GZ22aBSfoX60iWryTY

            userID := "NqCSr5HMDZjrWS74IEmedvlOYf"
            userID2 := "UAFalLvtKwNKABAnmwR7uGB5md"

            tm, resp := Client.GetTeamMembersByIds(teamID, []string{userID, userID2})

  '/teams/{team_id}/stats':
    get:
      tags:
        - teams
      summary: Get a team stats
      description: |
        Get a team stats on the system.
        ##### Permissions
        Must be authenticated and have the `view_team` permission.
      parameters:
        - name: team_id
          in: path
          description: Team GUID
          required: true
          type: string
      responses:
        '200':
          description: Team stats retrieval successful
          schema:
            $ref: '#/definitions/TeamStats'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '404':
          $ref: '#/responses/NotFound'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            teamID := "zWEyrTZ7GZ22aBSfoX60iWryTY"

            stats, resp := Client.GetTeamStats(teamID, "")

  '/teams/{team_id}/regenerate_invite_id':
    post:
      tags:
        - teams
      summary: Regenerate the Invite ID from a Team
      description: |
        Regenerates the invite ID used in invite links of a team
        ##### Permissions
        Must be authenticated and have the `manage_team` permission.
      parameters:
        - name: team_id
          in: path
          description: Team GUID
          required: true
          type: string
      responses:
        '200':
          description: Team Invite ID regenerated
          schema:
            $ref: '#/definitions/Team'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '404':
          $ref: '#/responses/NotFound'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            teamID := "zWEyrTZ7GZ22aBSfoX60iWryTY"

            team, resp := Client.RegenerateTeamInviteId(teamID)

  '/teams/{team_id}/image':
    get:
      tags:
        - teams
      summary: Get the team icon
      description: |
        Get the team icon of the team.

        __Minimum server version__: 4.9

        ##### Permissions
        User must be authenticated. In addition, team must be open or the user must have the `view_team` permission.
      parameters:
        - name: team_id
          in: path
          description: Team GUID
          required: true
          type: string
      responses:
        '200':
          description: Team icon retrieval successful
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '404':
          $ref: '#/responses/NotFound'
        '501':
          $ref: '#/responses/NotImplemented'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            teamID := "zWEyrTZ7GZ22aBSfoX60iWryTY"

            icon, resp = Client.GetTeamIcon(teamID, "")
    post:
      tags:
        - teams
      summary: Sets the team icon
      description: |
        Sets the team icon for the team.

        __Minimum server version__: 4.9

        ##### Permissions
        Must be authenticated and have the `manage_team` permission.
      consumes: ["multipart/form-data"]
      parameters:
        - name: team_id
          in: path
          description: Team GUID
          required: true
          type: string
        - name: image
          in: formData
          description: The image to be uploaded
          required: true
          type: file
      responses:
        '200':
          description: Team icon successfully set
          schema:
            $ref: '#/definitions/StatusOK'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '500':
          $ref: '#/responses/InternalServerError'
        '501':
          $ref: '#/responses/NotImplemented'
      x-code-samples:
        - lang: 'Go'
          source: |
            import (
              "io/ioutil"
              "log"

              "github.com/mattermost/mattermost-server/model"
            )

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            data, err := ioutil.ReadFile("icon.png")
            if err != nil {
              log.Fatal(err)
            }

            teamID := "zWEyrTZ7GZ22aBSfoX60iWryTY"

            ok, resp := Client.SetTeamIcon(teamID, data)
    delete:
      tags:
        - teams
      summary: Remove the team icon
      description: |
        Remove the team icon for the team.

        __Minimum server version__: 4.10

        ##### Permissions
        Must be authenticated and have the `manage_team` permission.
      parameters:
        - name: team_id
          in: path
          description: Team GUID
          required: true
          type: string
      responses:
        '200':
          description: Team icon successfully remove
          schema:
            $ref: '#/definitions/StatusOK'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '500':
          $ref: '#/responses/InternalServerError'
        '501':
          $ref: '#/responses/NotImplemented'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            teamID := "zWEyrTZ7GZ22aBSfoX60iWryTY"

            ok, resp = Client.RemoveTeamIcon(teamID)

  '/teams/{team_id}/members/{user_id}/roles':
    put:
      tags:
        - teams
      summary: Update a team member roles
      description: |
        Update a team member roles. Valid team roles are "team_user", "team_admin" or both of them. Overwrites any previously assigned team roles.
        ##### Permissions
        Must be authenticated and have the `manage_team_roles` permission.
      parameters:
        - name: team_id
          in: path
          description: Team GUID
          required: true
          type: string
        - name: user_id
          in: path
          description: User GUID
          required: true
          type: string
        - in: body
          name: body
          description: Space-delimited team roles to assign to the user
          required: true
          schema:
            type: object
            required:
              - roles
            properties:
              roles:
                type: string
      responses:
        '200':
          description: Team member roles update successful
          schema:
            $ref: '#/definitions/StatusOK'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '404':
          $ref: '#/responses/NotFound'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            teamID := "zWEyrTZ7GZ22aBSfoX60iWryTY"
            userID := "NqCSr5HMDZjrWS74IEmedvlOYf"

            ok, resp := Client.UpdateTeamMemberRoles(teamID, userID, "team_user team_admin")

  '/teams/{team_id}/members/{user_id}/schemeRoles':
    put:
      tags:
        - teams
      summary: Update the scheme-derived roles of a team member.
      description: |
        Update a team member's scheme_admin/scheme_user properties. Typically this should either be `scheme_admin=false, scheme_user=true` for ordinary team member, or `scheme_admin=true, scheme_user=true` for a team admin.

        __Minimum server version__: 5.0

        ##### Permissions
        Must be authenticated and have the `manage_team_roles` permission.
      parameters:
        - name: team_id
          in: path
          description: Team GUID
          required: true
          type: string
        - name: user_id
          in: path
          description: User GUID
          required: true
          type: string
        - in: body
          name: body
          description: Scheme properties.
          required: true
          schema:
            type: object
            required:
              - scheme_admin
              - scheme_user
            properties:
              scheme_admin:
                type: boolean
              scheme_user:
                type: boolean
      responses:
        '200':
          description: Team member's scheme-derived roles updated successfully.
          schema:
            $ref: '#/definitions/StatusOK'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '404':
          $ref: '#/responses/NotFound'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            teamID := "zWEyrTZ7GZ22aBSfoX60iWryTY"
            userID := "NqCSr5HMDZjrWS74IEmedvlOYf"

            ok, resp := Client.UpdateTeamMemberSchemeRoles(teamID, userID, &model.SchemeRoles{
              SchemeAdmin: true,
              SchemeUser:  true,
            })

  '/users/{user_id}/teams/unread':
    get:
      tags:
        - teams
      summary: Get team unreads for a user
      description: |
        Get the count for unread messages and mentions in the teams the user is a member of.
        ##### Permissions
        Must be logged in.
      parameters:
        - name: user_id
          in: path
          description: User GUID
          required: true
          type: string
        - name: exclude_team
          in: query
          description: Optional team id to be excluded from the results
          required: true
          type: string
      responses:
        '200':
          description: Team unreads retrieval successful
          schema:
            type: array
            items:
              $ref: '#/definitions/TeamUnread'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            userID := "NqCSr5HMDZjrWS74IEmedvlOYf"
            teamID := "zWEyrTZ7GZ22aBSfoX60iWryTY"

            teams, resp := Client.GetTeamsUnreadForUser(userID, teamID)

  '/users/{user_id}/teams/{team_id}/unread':
    get:
      tags:
        - teams
      summary: Get unreads for a team
      description: |
        Get the unread mention and message counts for a team for the specified user.
        ##### Permissions
        Must be the user or have `edit_other_users` permission and have `view_team` permission for the team.
      parameters:
        - name: user_id
          in: path
          description: User GUID
          required: true
          type: string
        - name: team_id
          in: path
          description: Team GUID
          required: true
          type: string
      responses:
        '200':
          description: Team unread count retrieval successful
          schema:
            $ref: '#/definitions/TeamUnread'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '404':
          $ref: '#/responses/NotFound'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            userID := "NqCSr5HMDZjrWS74IEmedvlOYf"
            teamID := "zWEyrTZ7GZ22aBSfoX60iWryTY"

            teamUnread, resp := Client.GetTeamUnread(userID, teamID)

  '/teams/{team_id}/invite/email':
    post:
      tags:
        - teams
      summary: Invite users to the team by email
      description: |
        Invite users to the existing team usign the user's email.
        ##### Permissions
        Must have `invite_to_team` permission for the team.
      parameters:
        - name: team_id
          in: path
          description: Team GUID
          required: true
          type: string
        - in: body
          name: body
          description: List of user's email
          required: true
          schema:
            type: array
            items:
              type: string
      responses:
        '200':
          description: Users invite successful
          schema:
            $ref: '#/definitions/StatusOK'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            teamID := "zWEyrTZ7GZ22aBSfoX60iWryTY"

            ok, resp := Client.InviteUsersToTeam(teamID, []string{"test@domain.com", "test2@domain.com"})

  '/teams/invites/email':
    delete:
      tags:
        - teams
      summary: Invalidate active email invitations
      description: |
        Invalidate active email invitations that have not been accepted by the user.
        ##### Permissions
        Must have `manage_system` permission.
      responses:
        '200':
          description: Email invites successfully revoked
          schema:
            $ref: '#/definitions/StatusOK'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            ok, resp := Client.InvalidateEmailInvites()

  '/teams/{team_id}/import':
    post:
      tags:
        - teams
      summary: Import a Team from other application
      description: |
        Import a team into a existing team. Import users, channels, posts, hooks.
        ##### Permissions
        Must have `permission_import_team` permission.
      consumes: ["multipart/form-data"]
      parameters:
        - name: file
          in: formData
          description: A file to be uploaded in zip format.
          required: true
          type: file
        - name: filesize
          in: formData
          description: The size of the zip file to be imported.
          required: true
          type: integer
        - name: importFrom
          in: formData
          description: String that defines from which application the team was exported to be imported into Mattermost.
          required: true
          allowEmptyValue: false
          type: string
        - name: team_id
          in: path
          description: Team GUID
          required: true
          type: string
      responses:
        '200':
          description: JSON object containing a base64 encoded text file of the import logs in its `results` property.
          schema:
            type: object
            properties:
              results:
                type: string
        '400':
          $ref: '#/responses/BadRequest'
        '403':
          $ref: '#/responses/Forbidden'
      x-code-samples:
        - lang: 'Go'
          source: |
            import (
              "encoding/binary"
              "io/ioutil"
              "log"

              "github.com/mattermost/mattermost-server/model"
            )

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            data, err = ioutil.ReadFile("to_import.zip")
            if err != nil && len(data) == 0 {
              log.Fatal("Error while reading file.")
            }

            teamID := "zWEyrTZ7GZ22aBSfoX60iWryTY"

            fileResp, resp := Client.ImportTeam(data, binary.Size(data), "slack", "to_import.zip", teamID)
  '/teams/invite/{invite_id}':
    get:
      tags:
        - teams
      summary: Get invite info for a team
      description: |
        Get the `name`, `display_name`, `description` and `id` for a team from the invite id.

        __Minimum server version__: 4.0

        ##### Permissions
        No authentication required.
      parameters:
        - name: invite_id
          in: path
          description: Invite id for a team
          required: true
          type: string
      responses:
        '200':
          description: Team invite info retrieval successful
          schema:
            type: object
            properties:
              id:
                type: string
              name:
                type: string
              display_name:
                type: string
              description:
                type: string
        '400':
          $ref: '#/responses/BadRequest'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")

            inviteID := "zWEyrTZ7GZ22aBSfoX60iWryTY"

            team, resp = Client.GetTeamInviteInfo(inviteID)

  '/teams/{team_id}/scheme':
    put:
      tags:
        - teams
      summary: Set a team's scheme
      description: |
        Set a team's scheme, more specifically sets the scheme_id value of a team record.

        ##### Permissions
        Must have `manage_system` permission.

        __Minimum server version__: 5.0
      parameters:
        - name: team_id
          in: path
          description: Team GUID
          required: true
          type: string
        - in: body
          name: body
          description: Scheme GUID
          required: true
          schema:
            type: object
            required:
              - scheme_id
            properties:
              scheme_id:
                type: string
                description: The ID of the scheme.
      responses:
        '200':
          description: Update team scheme successful
          schema:
            $ref: '#/definitions/StatusOK'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '501':
          $ref: '#/responses/NotImplemented'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            teamID := "4xp9fdt77pncbef59f4k1qe83o"
            schemeID := "qjda3stwafbgpqjaxej3k76sga"

            ok, resp := UpdateTeamScheme(teamID, schemeID)
        - lang: 'curl'
          source: |
            curl -X PUT \
              https://your-mattermost-url.com/api/v4/teams/4xp9fdt77pncbef59f4k1qe83o/scheme \
              -H 'Authorization: Bearer frn8fu5rtpyc5m4xy6q3oj4yur' \
              -H 'Content-Type: application/json' \
              -d '{"scheme_id": "qjda3stwafbgpqjaxej3k76sga"}'

  '/teams/{team_id}/members_minus_group_members':
    get:
      tags:
        - teams
      summary: Team members minus group members.
      description: |
        Get the set of users who are members of the team minus the set of users who are members of the given groups.
        Each user object contains an array of group objects representing the group memberships for that user.
        Each user object contains the boolean fields `scheme_guest`, `scheme_user`, and `scheme_admin` representing the roles that user has for the given team.

        ##### Permissions
        Must have `manage_system` permission.

        __Minimum server version__: 5.14
      parameters:
        - name: team_id
          in: path
          description: Team GUID
          required: true
          type: string
        - name: group_ids
          in: query
          description: A comma-separated list of group ids.
          required: true
          default: ""
          type: string
        - name: page
          in: query
          description: The page to select.
          default: "0"
          type: string
        - name: per_page
          in: query
          description: The number of users per page.
          default: "0"
          type: string
      responses:
        '200':
          description: Successfully returns users specified by the pagination, and the total_count.
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
      x-code-samples:
        - lang: 'curl'
          source: |
            curl 'http://your-mattermost-url.com/api/v4/teams/fcnst115y3y7xmzzp5uq34u8ce/members_minus_group_members?group_ids=eoezijg8zffgjmch8icy5bjd1e,ugaw6wjc3tfxpcr1eq5u5k8dhe&page=0&per_page=100' \
                -H 'Authorization: Bearer mq8rrfxpdfyafbnw3qfmhwkx6c' \
                -H 'Content-Type: application/json' \
                -H 'X-Requested-With: XMLHttpRequest'
  /channels:
    post:
      tags:
        - channels
      summary: Create a channel
      description: |
        Create a new channel.
        ##### Permissions
        If creating a public channel, `create_public_channel` permission is required. If creating a private channel, `create_private_channel` permission is required.
      parameters:
        - in: body
          name: body
          description: Channel object to be created
          required: true
          schema:
            type: object
            required:
              - name
              - display_name
              - type
              - team_id
            properties:
              team_id:
                type: string
                description: The team ID of the team to create the channel on
              name:
                type: string
                description: The unique handle for the channel, will be present in the channel URL
              display_name:
                type: string
                description: The non-unique UI name for the channel
              purpose:
                type: string
                description: A short description of the purpose of the channel
              header:
                type: string
                description: Markdown-formatted text to display in the header of the channel
              type:
                type: string
                description: "'O' for a public channel, 'P' for a private channel"
      responses:
        '201':
          description: Channel creation successful
          schema:
            $ref: '#/definitions/Channel'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"
            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            channel := &model.Channel{DisplayName: <YOUR CHANNEL DISPLAYNAME>, Name: <YOUR CHANNEL NAME>, Type: <CHANNEL TYPE OPEN/PRIVATE>, TeamId: <YOUR TEAM ID>}

            // CreateChannel
            rchannel, resp := Client.CreateChannel(channel)

  /channels/direct:
    post:
      tags:
        - channels
      summary: Create a direct message channel
      description: |
        Create a new direct message channel between two users.
        ##### Permissions
        Must be one of the two users and have `create_direct_channel` permission. Having the `manage_system` permission voids the previous requirements.
      parameters:
        - in: body
          name: body
          description: The two user ids to be in the direct message
          required: true
          schema:
            type: array
            items:
              type: string
            minItems: 2
            maxItems: 2
      responses:
        '201':
          description: Direct channel creation successful
          schema:
            $ref: '#/definitions/Channel'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"
            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            // CreateDirectChannel
            dm, resp := Client.CreateDirectChannel(<ID OF User1>, <ID OF User2>)

  /channels/group:
    post:
      tags:
        - channels
      summary: Create a group message channel
      description: |
        Create a new group message channel to group of users. If the logged in user's id is not included in the list, it will be appended to the end.
        ##### Permissions
        Must have `create_group_channel` permission.
      parameters:
        - in: body
          name: body
          description: User ids to be in the group message channel
          required: true
          schema:
            type: array
            items:
              type: string
      responses:
        '201':
          description: Group channel creation successful
          schema:
            $ref: '#/definitions/Channel'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"
            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            userIds := []string{<ID OF User1>, <ID OF User2>, <ID OF User3> ...}

            // CreateGroupChannel
            rgc, resp := Client.CreateGroupChannel(userIds)

  /group/search:
    post:
      tags:
        - channels
      summary: Search Group Channels
      description: |
        Get a list of group channels for a user which members' usernames match the search term.

        __Minimum server version__: 5.14
      parameters:
        - in: body
          name: body
          description: Search criteria
          required: true
          schema:
            type: object
            required:
              - term
            properties:
              term:
                description: The search term to match against the members' usernames of the group channels
                type: string
      responses:
        '200':
          description: Channels search successful
          schema:
            type: array
            items:
              $ref: '#/definitions/Channel'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"
            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            search := &model.ChannelSearch{Term: <MEMBER USERNAME>}

            // SearchGroupChannels
            channels, resp := Client.SearchGroupChannels(search)

  /teams/{team_id}/channels/ids:
    post:
      tags:
        - channels
      summary: Get a list of channels by ids
      description: |
        Get a list of public channels on a team by id.
        ##### Permissions
        `view_team` for the team the channels are on.
      parameters:
        - name: team_id
          in: path
          description: Team GUID
          required: true
          type: string
        - in: body
          name: body
          description: List of channel ids
          required: true
          schema:
            type: array
            items:
              type: string
      responses:
        '200':
          description: Channel list retrieval successful
          schema:
            type: array
            items:
              $ref: '#/definitions/Channel'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '404':
          $ref: '#/responses/NotFound'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"
            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            channelIds := []string{<ID OF CHANNEL1>, <ID OF CHANNEL2>, ...}

            // GetPublicChannelsByIdsForTeam
            channels, resp := Client.GetPublicChannelsByIdsForTeam(<TEAMID>, channelIds)

  /channels/{channel_id}/timezones:
    get:
      tags:
        - channels
      summary: Get timezones in a channel
      description: |
        Get a list of timezones for the users who are in this channel.

        __Minimum server version__: 5.6

        ##### Permissions
        Must have the `read_channel` permission.
      parameters:
        - name: channel_id
          in: path
          description: Channel GUID
          required: true
          type: string
      responses:
        '200':
          description: Timezone retrieval successful
          schema:
            type: array
            items:
              type: string
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"
            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            // GetChannelStats
            stats, resp := Client.GetChannelTimezones(<CHANNELID>)

  '/channels/{channel_id}':
    get:
      tags:
        - channels
      summary: Get a channel
      description: |
        Get channel from the provided channel id string.
        ##### Permissions
        `read_channel` permission for the channel.
      parameters:
        - name: channel_id
          in: path
          description: Channel GUID
          required: true
          type: string
      responses:
        '200':
          description: Channel retrieval successful
          schema:
            $ref: '#/definitions/Channel'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '404':
          $ref: '#/responses/NotFound'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"
            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            // GetChannel
            channel, resp := Client.GetChannel(<CHANNELID>, "")

    put:
      tags:
        - channels
      summary: Update a channel
      description: |
        Update a channel. The fields that can be updated are listed as parameters. Omitted fields will be treated as blanks.
        ##### Permissions
        If updating a public channel, `manage_public_channel_members` permission is required. If updating a private channel, `manage_private_channel_members` permission is required.
      parameters:
        - name: channel_id
          in: path
          description: Channel GUID
          required: true
          type: string
        - in: body
          name: body
          description: Channel object to be updated
          required: true
          schema:
            type: object
            required:
              - id
            properties:
              id:
                type: string
                description: The channel's id, not updatable
              name:
                type: string
                description: The unique handle for the channel, will be present in the channel URL
              display_name:
                type: string
                description: The non-unique UI name for the channel
              purpose:
                type: string
                description: A short description of the purpose of the channel
              header:
                type: string
                description: Markdown-formatted text to display in the header of the channel
              type:
                type: string
                description: "'O' for a public channel, 'P' for a private channel"
      responses:
        '200':
          description: Channel update successful
          schema:
            $ref: '#/definitions/Channel'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '404':
          $ref: '#/responses/NotFound'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"
            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            channel := &model.Channel{DisplayName: <YOUR CHANNEL NEW DISPLAYNAME>, ChannelId: <CHANNELID>, TeamId: <YOUR TEAM ID>}

            // UpdateChannel
            updatedChannel, resp := Client.UpdateChannel(channel)

    delete:
      tags:
        - channels
      summary: Delete a channel
      description: |
        Soft deletes a channel, by marking the channel as deleted in the database. Soft deleted channels will not be accessible in the user interface. Direct and group message channels cannot be deleted.
        ##### Permissions
        `delete_public_channel` permission if the channel is public,
        `delete_private_channel` permission if the channel is private,
        or have `manage_system` permission.
      parameters:
        - name: channel_id
          in: path
          description: Channel GUID
          required: true
          type: string
      responses:
        '200':
          description: Channel deletion successful
          schema:
            $ref: '#/definitions/StatusOK'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"
            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            // DeleteChannel
            pass, resp := Client.DeleteChannel(<CHANNELID>)

  '/channels/{channel_id}/patch':
    put:
      tags:
        - channels
      summary: Patch a channel
      description: |
        Partially update a channel by providing only the fields you want to update. Omitted fields will not be updated. The fields that can be updated are defined in the request body, all other provided fields will be ignored.
        ##### Permissions
        If updating a public channel, `manage_public_channel_members` permission is required. If updating a private channel, `manage_private_channel_members` permission is required.
      parameters:
        - name: channel_id
          in: path
          description: Channel GUID
          required: true
          type: string
        - in: body
          name: body
          description: Channel object to be updated
          required: true
          schema:
            type: object
            properties:
              name:
                type: string
                description: The unique handle for the channel, will be present in the channel URL
              display_name:
                type: string
                description: The non-unique UI name for the channel
              purpose:
                type: string
                description: A short description of the purpose of the channel
              header:
                type: string
                description: Markdown-formatted text to display in the header of the channel
      responses:
        '200':
          description: Channel patch successful
          schema:
            $ref: '#/definitions/Channel'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '404':
          $ref: '#/responses/NotFound'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"
            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            patch := &model.ChannelPatch{
              Name:        new(string),
              DisplayName: new(string),
              Header:      new(string),
              Purpose:     new(string),
            }
            *patch.Name = "<SOME_NEW_NAME>"
            *patch.DisplayName = "<SOME_NEW_DISPLAYNAME>"
            *patch.Header = "<SOME_NEW_HEADER>"
            *patch.Purpose = "<SOME_NEW_PURPOSE>"

            // PatchChannel
            channel, resp := Client.PatchChannel(<CHANNELID>, patch)

  '/channels/{channel_id}/convert':
      post:
        tags:
          - channels
        summary: Convert a channel from public to private
        description: |
          Convert into private channel from the provided channel id string.

          __Minimum server version__: 4.10

          ##### Permissions
          `manage_team` permission for the team of the channel.
        parameters:
          - name: channel_id
            in: path
            description: Channel GUID
            required: true
            type: string
        responses:
          '200':
            description: Channel conversion successful
            schema:
              $ref: '#/definitions/Channel'
          '400':
            $ref: '#/responses/BadRequest'
          '401':
            $ref: '#/responses/Unauthorized'
          '403':
            $ref: '#/responses/Forbidden'
          '404':
            $ref: '#/responses/NotFound'
        x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"
            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            // ConvertChannelToPrivate
            convertedChannel, resp := Client.ConvertChannelToPrivate(<CHANNELID>)

  '/channels/{channel_id}/restore':
      post:
        tags:
          - channels
        summary: Restore a channel
        description: |
          Restore channel from the provided channel id string.

          __Minimum server version__: 3.10

          ##### Permissions
          `manage_team` permission for the team of the channel.
        parameters:
          - name: channel_id
            in: path
            description: Channel GUID
            required: true
            type: string
        responses:
          '200':
            description: Channel restore successful
            schema:
              $ref: '#/definitions/Channel'
          '401':
            $ref: '#/responses/Unauthorized'
          '403':
            $ref: '#/responses/Forbidden'
          '404':
            $ref: '#/responses/NotFound'

  '/channels/{channel_id}/stats':
    get:
      tags:
        - channels
      summary: Get channel statistics
      description: |
        Get statistics for a channel.
        ##### Permissions
        Must have the `read_channel` permission.
      parameters:
        - name: channel_id
          in: path
          description: Channel GUID
          required: true
          type: string
      responses:
        '200':
          description: Channel statistics retrieval successful
          schema:
            $ref: '#/definitions/ChannelStats'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"
            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            // GetChannelStats
            stats, resp := Client.GetChannelStats(<CHANNELID>)

  '/channels/{channel_id}/pinned':
    get:
      tags:
        - channels
      summary: Get a channel's pinned posts
      description: Get a list of pinned posts for channel.
      parameters:
        - name: channel_id
          in: path
          description: Channel GUID
          required: true
          type: string
      responses:
        '200':
          description: The list of channel pinned posts
          schema:
            $ref: '#/definitions/PostList'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"
            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            // GetPinnedPosts
            posts, resp := Client.GetPinnedPosts(<CHANNELID>, "")

  '/teams/{team_id}/channels':
    get:
      tags:
        - channels
      summary: Get public channels
      description: |
        Get a page of public channels on a team based on query string parameters - page and per_page.
        ##### Permissions
        Must be authenticated and have the `list_team_channels` permission.
      parameters:
        - name: team_id
          in: path
          description: Team GUID
          required: true
          type: string
        - name: page
          in: query
          description: The page to select.
          default: "0"
          type: string
        - name: per_page
          in: query
          description: The number of public channels per page.
          default: "60"
          type: string
      responses:
        '200':
          description: Channels retrieval successful
          schema:
            type: array
            items:
              $ref: '#/definitions/Channel'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '404':
          $ref: '#/responses/NotFound'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"
            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")
            // GetPublicChannelsForTeam
            channels, resp := Client.GetPublicChannelsForTeam(<TEAMID>, 0, 100, "")

  '/teams/{team_id}/channels/deleted':
    get:
      tags:
        - channels
      summary: Get deleted channels
      description: |
        Get a page of deleted channels on a team based on query string parameters - team_id, page and per_page.

        __Minimum server version__: 3.10

        ##### Permissions
        Must be authenticated and have the `manage_team` permission.
      parameters:
        - name: team_id
          in: path
          description: Team GUID
          required: true
          type: string
        - name: page
          in: query
          description: The page to select.
          default: "0"
          type: string
        - name: per_page
          in: query
          description: The number of public channels per page.
          default: "60"
          type: string
      responses:
        '200':
          description: Channels retrieval successful
          schema:
            type: array
            items:
              $ref: '#/definitions/Channel'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '404':
          $ref: '#/responses/NotFound'

  '/teams/{team_id}/channels/autocomplete':
    get:
      tags:
        - channels
      summary: Autocomplete channels
      description: |
        Autocomplete public channels on a team based on the search term provided in the request URL.

        __Minimum server version__: 4.7

        ##### Permissions
        Must have the `list_team_channels` permission.
      parameters:
        - name: team_id
          in: path
          description: Team GUID
          required: true
          type: string
        - name: name
          in: query
          description: Name or display name
          required: true
          type: string
      responses:
        '200':
          description: Channels autocomplete successful
          schema:
            type: array
            items:
              $ref: '#/definitions/Channel'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '404':
          $ref: '#/responses/NotFound'

  '/teams/{team_id}/channels/search_autocomplete':
    get:
      tags:
        - channels
      summary: Autocomplete channels for search
      description: |
        Autocomplete your channels on a team based on the search term provided in the request URL.

        __Minimum server version__: 5.4

        ##### Permissions
        Must have the `list_team_channels` permission.
      parameters:
        - name: team_id
          in: path
          description: Team GUID
          required: true
          type: string
        - name: name
          in: query
          description: Name or display name
          required: true
          type: string
      responses:
        '200':
          description: Channels autocomplete successful
          schema:
            type: array
            items:
              $ref: '#/definitions/Channel'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '404':
          $ref: '#/responses/NotFound'

  '/teams/{team_id}/channels/search':
    post:
      tags:
        - channels
      summary: Search channels
      description: |
        Search public channels on a team based on the search term provided in the request body.
        ##### Permissions
        Must have the `list_team_channels` permission.

        In server version 5.16 and later, a user without the `list_team_channels` permission will be able to use this endpoint, with the search results limited to the channels that the user is a member of.
      parameters:
        - name: team_id
          in: path
          description: Team GUID
          required: true
          type: string
        - in: body
          name: body
          description: Search criteria
          required: true
          schema:
            type: object
            required:
              - term
            properties:
              term:
                description: The search term to match against the name or display name of channels
                type: string
      responses:
        '201':
          description: Channels search successful
          schema:
            type: array
            items:
              $ref: '#/definitions/Channel'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '404':
          $ref: '#/responses/NotFound'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"
            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            search := &model.ChannelSearch{Term: <CHANNEL DISPLAYNAME>}

            // SearchChannels
            channels, resp := Client.SearchChannels(<TEAMID>, search)

  '/teams/{team_id}/channels/name/{channel_name}':
    get:
      tags:
        - channels
      summary: Get a channel by name
      description: |
        Gets channel from the provided team id and channel name strings.
        ##### Permissions
        `read_channel` permission for the channel.
      parameters:
        - name: team_id
          in: path
          description: Team GUID
          required: true
          type: string
        - name: channel_name
          in: path
          description: Channel Name
          required: true
          type: string
        - name: include_deleted
          in: query
          description: Defines if deleted channels should be returned or not
          type: string
          enum: [true, false]
          default: "false"
      responses:
        '200':
          description: Channel retrieval successful
          schema:
            $ref: '#/definitions/Channel'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '404':
          $ref: '#/responses/NotFound'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"
            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            // GetChannelByName
            channel, resp := Client.GetChannelByName(<CHANNEL NAME>, <TEAMID>, "")

  '/teams/name/{team_name}/channels/name/{channel_name}':
    get:
      tags:
        - channels
      summary: Get a channel by name and team name
      description: |
        Gets a channel from the provided team name and channel name strings.
        ##### Permissions
        `read_channel` permission for the channel.
      parameters:
        - name: team_name
          in: path
          description: Team Name
          required: true
          type: string
        - name: channel_name
          in: path
          description: Channel Name
          required: true
          type: string
        - name: include_deleted
          in: query
          description: Defines if deleted channels should be returned or not
          type: string
          enum: [true, false]
          default: "false"
      responses:
        '200':
          description: Channel retrieval successful
          schema:
            $ref: '#/definitions/Channel'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '404':
          $ref: '#/responses/NotFound'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"
            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            // GetChannelByNameForTeamName
            channel, resp = Client.GetChannelByNameForTeamName(<CHANNEL NAME>, <TEAM NAME>, "")

  '/channels/{channel_id}/members':
    get:
      tags:
        - channels
      summary: Get channel members
      description: |
        Get a page of members for a channel.
        ##### Permissions
        `read_channel` permission for the channel.
      parameters:
        - name: channel_id
          in: path
          description: Channel GUID
          required: true
          type: string
        - name: page
          in: query
          description: The page to select.
          default: "0"
          type: string
        - name: per_page
          in: query
          description: The number of members per page.
          default: "60"
          type: string
      responses:
        '200':
          description: Channel members retrieval successful
          schema:
            type: array
            items:
              $ref: '#/definitions/ChannelMember'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"
            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            // GetChannelMembers
            members, resp := Client.GetChannelMembers(th.BasicChannel.Id, 0, 60, "")

    post:
      tags:
        - channels
      summary: Add user to channel
      description: Add a user to a channel by creating a channel member object.
      parameters:
        - name: channel_id
          in: path
          description: The channel ID
          required: true
          type: string
        - in: body
          name: body
          required: true
          schema:
            type: object
            required:
              - user_id
            properties:
              user_id:
                type: string
                description: The ID of user to add into the channel
              post_root_id:
                type: string
                description: The ID of root post where link to add channel member originates
      responses:
        '201':
          description: Channel member creation successful
          schema:
            $ref: '#/definitions/ChannelMember'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"
            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            // AddChannelMember
            cm, resp := Client.AddChannelMember(<CHANNEL ID>, <ID OF USER TO ADD>)

            // AddChannelMemberWithRootId
            cm, resp := Client.AddChannelMemberWithRootId(<CHANNEL ID>, <ID OF USER TO ADD>, <POST ROOT ID>)

  '/channels/{channel_id}/members/ids':
    post:
      tags:
        - channels
      summary: Get channel members by ids
      description: |
        Get a list of channel members based on the provided user ids.
        ##### Permissions
        Must have the `read_channel` permission.
      parameters:
        - name: channel_id
          in: path
          description: Channel GUID
          required: true
          type: string
        - in: body
          name: user_ids
          description: List of user ids
          required: true
          schema:
            type: array
            items:
              type: string
      responses:
        '200':
          description: Channel member list retrieval successful
          schema:
            type: array
            items:
              $ref: '#/definitions/ChannelMember'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '404':
          $ref: '#/responses/NotFound'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"
            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            usersIds := []string{<Id of User1>, <Id of User2>, ...}

            // GetChannelMembersByIds
            cm, resp := Client.GetChannelMembersByIds(<CHANNELID>, usersIds)

  '/channels/{channel_id}/members/{user_id}':
    get:
      tags:
        - channels
      summary: Get channel member
      description: |
        Get a channel member.
        ##### Permissions
        `read_channel` permission for the channel.
      parameters:
        - name: channel_id
          in: path
          description: Channel GUID
          required: true
          type: string
        - name: user_id
          in: path
          description: User GUID
          required: true
          type: string
      responses:
        '200':
          description: Channel member retrieval successful
          schema:
            $ref: '#/definitions/ChannelMember'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"
            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            // GetChannelMember
            member, resp := Client.GetChannelMember(<CHANNELID>, <USERID>, "")

    delete:
      tags:
        - channels
      summary: Remove user from channel
      description: |
        Delete a channel member, effectively removing them from a channel.

        In server version 5.3 and later, channel members can only be deleted from public or private channels.
        ##### Permissions
        `manage_public_channel_members` permission if the channel is public.
        `manage_private_channel_members` permission if the channel is private.
      parameters:
        - name: channel_id
          in: path
          description: Channel GUID
          required: true
          type: string
        - name: user_id
          in: path
          description: User GUID
          required: true
          type: string
      responses:
        '200':
          description: Channel member deletion successful
          schema:
            $ref: '#/definitions/StatusOK'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"
            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            // RemoveUserFromChannel
            pass, resp := Client.RemoveUserFromChannel(<CHANNELID>, <USERID>)

  '/channels/{channel_id}/members/{user_id}/roles':
    put:
      tags:
        - channels
      summary: Update channel roles
      description: |
        Update a user's roles for a channel.
        ##### Permissions
        Must have `manage_channel_roles` permission for the channel.
      parameters:
        - name: channel_id
          in: path
          description: Channel GUID
          required: true
          type: string
        - name: user_id
          in: path
          description: User GUID
          required: true
          type: string
        - in: body
          name: roles
          description: Space-delimited channel roles to assign to the user
          required: true
          schema:
            type: object
            required:
              - roles
            properties:
              roles:
                type: string
      responses:
        '200':
          description: Channel roles update successful
          schema:
            $ref: '#/definitions/StatusOK'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"
            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            // UpdateChannelRoles
            pass, resp := Client.UpdateChannelRoles(<CHANNELID>, <USERIDTOPROMOTE>, "channel_admin channel_user")

  '/channels/{channel_id}/members/{user_id}/schemeRoles':
    put:
      tags:
        - channels
      summary: Update the scheme-derived roles of a channel member.
      description: |
        Update a channel member's scheme_admin/scheme_user properties. Typically this should either be `scheme_admin=false, scheme_user=true` for ordinary channel member, or `scheme_admin=true, scheme_user=true` for a channel admin.
        __Minimum server version__: 5.0
        ##### Permissions
        Must be authenticated and have the `manage_channel_roles` permission.
      parameters:
        - name: channel_id
          in: path
          description: Channel GUID
          required: true
          type: string
        - name: user_id
          in: path
          description: User GUID
          required: true
          type: string
        - in: body
          name: body
          description: Scheme properties.
          required: true
          schema:
            type: object
            required:
              - scheme_admin
              - scheme_user
            properties:
              scheme_admin:
                type: boolean
              scheme_user:
                type: boolean
      responses:
        '200':
          description: Channel member's scheme-derived roles updated successfully.
          schema:
            $ref: '#/definitions/StatusOK'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '404':
          $ref: '#/responses/NotFound'

  '/channels/{channel_id}/members/{user_id}/notify_props':
    put:
      tags:
        - channels
      summary: Update channel notifications
      description: |
        Update a user's notification properties for a channel. Only the provided fields are updated.
        ##### Permissions
        Must be logged in as the user or have `edit_other_users` permission.
      parameters:
        - name: channel_id
          in: path
          description: Channel GUID
          required: true
          type: string
        - name: user_id
          in: path
          description: User GUID
          required: true
          type: string
        - in: body
          name: notify_props
          required: true
          schema:
            $ref: '#/definitions/ChannelNotifyProps'
      responses:
        '200':
          description: Channel notification properties update successful
          schema:
            $ref: '#/definitions/StatusOK'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '404':
          $ref: '#/responses/NotFound'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"
            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            props := map[string]string{}
            props[model.DESKTOP_NOTIFY_PROP] = model.CHANNEL_NOTIFY_MENTION
            props[model.MARK_UNREAD_NOTIFY_PROP] = model.CHANNEL_MARK_UNREAD_MENTION

            // UpdateChannelNotifyProps
            pass, resp := Client.UpdateChannelNotifyProps(<CHANNELID>, <USERID>, props)

  '/channels/members/{user_id}/view':
    post:
      tags:
        - channels
      summary: View channel
      description: |
        Perform all the actions involved in viewing a channel. This includes marking channels as read, clearing push notifications, and updating the active channel.
        ##### Permissions
        Must be logged in as user or have `edit_other_users` permission.

        __Response only includes `last_viewed_at_times` in Mattermost server 4.3 and newer.__
      parameters:
        - in: path
          name: user_id
          description: User ID to perform the view action for
          required: true
          type: string
        - in: body
          name: body
          description: Paremeters affecting how and which channels to view
          required: true
          schema:
            type: object
            required:
              - channel_id
            properties:
              channel_id:
                type: string
                description: The channel ID that is being viewed. Use a blank string to indicate that all channels have lost focus.
              prev_channel_id:
                type: string
                description: The channel ID of the previous channel, used when switching channels. Providing this ID will cause push notifications to clear on the channel being switched to.
      responses:
        '200':
          description: Channel view successful
          schema:
            type: object
            properties:
              status:
                type: string
                description: Value should be "OK" if successful
              last_viewed_at_times:
                type: object
                description: A JSON object mapping channel IDs to the channel view times
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"
            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            view := &model.ChannelView{
              ChannelId: <CHANNELID>,
            }
            // ViewChannel
            pass, resp := Client.ViewChannel(<USERID>, view)

  '/users/{user_id}/teams/{team_id}/channels/members':
    get:
      tags:
        - channels
      summary: Get channel members for user
      description: |
        Get all channel members on a team for a user.
        ##### Permissions
        Logged in as the user and `view_team` permission for the team. Having `manage_system` permission voids the previous requirements.
      parameters:
        - name: user_id
          in: path
          description: User GUID
          required: true
          type: string
        - name: team_id
          in: path
          description: Team GUID
          required: true
          type: string
      responses:
        '200':
          description: Channel members retrieval successful
          schema:
            type: array
            items:
              $ref: '#/definitions/ChannelMember'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"
            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            // GetChannelMembersForUser
            members, resp := Client.GetChannelMembersForUser(<USERID>, <TEAMID>, "")

  '/users/{user_id}/teams/{team_id}/channels':
    get:
      tags:
        - channels
      summary: Get channels for user
      description: |
        Get all the channels on a team for a user.
        ##### Permissions
        Logged in as the user, or have `edit_other_users` permission, and `view_team` permission for the team.
      parameters:
        - name: user_id
          in: path
          description: User GUID
          required: true
          type: string
        - name: team_id
          in: path
          description: Team GUID
          required: true
          type: string
      responses:
        '200':
          description: Channels retrieval successful
          schema:
            type: array
            items:
              $ref: '#/definitions/Channel'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '404':
          $ref: '#/responses/NotFound'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"
            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            // GetChannelsForTeamForUser
            channels, resp := Client.GetChannelsForTeamForUser(<TEAMID>, <USERID>, "")

  '/users/{user_id}/channels/{channel_id}/unread':
    get:
      tags:
        - channels
      summary: Get unread messages
      description: |
        Get the total unread messages and mentions for a channel for a user.
        ##### Permissions
        Must be logged in as user and have the `read_channel` permission, or have `edit_other_usrs` permission.
      parameters:
        - name: user_id
          in: path
          description: User GUID
          required: true
          type: string
        - name: channel_id
          in: path
          description: Channel GUID
          required: true
          type: string
      responses:
        '200':
          description: Channel unreads retrieval successful
          schema:
            $ref: '#/definitions/ChannelUnread'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '404':
          $ref: '#/responses/NotFound'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"
            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            // GetChannelUnread
            channelUnread, resp := Client.GetChannelUnread(<CHANNELID>, <USERID>)

  '/channels/{channel_id}/scheme':
    put:
      tags:
        - channels
      summary: Set a channel's scheme
      description: |
        Set a channel's scheme, more specifically sets the scheme_id value of a channel record.

        ##### Permissions
        Must have `manage_system` permission.

        __Minimum server version__: 4.10
      parameters:
        - name: channel_id
          in: path
          description: Channel GUID
          required: true
          type: string
        - in: body
          name: body
          description: Scheme GUID
          required: true
          schema:
            type: object
            required:
              - scheme_id
            properties:
              scheme_id:
                type: string
                description: The ID of the scheme.
      responses:
        '200':
          description: Update channel scheme successful
          schema:
            $ref: '#/definitions/StatusOK'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '501':
          $ref: '#/responses/NotImplemented'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"
            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            channelID := "4xp9fdt77pncbef59f4k1qe83o"
            schemeID := "qjda3stwafbgpqjaxej3k76sga"
            ok, resp := UpdateChannelScheme(channelID, schemeID)
        - lang: 'curl'
          source: |
            curl -X PUT \
              https://your-mattermost-url.com/api/v4/channels/4xp9fdt77pncbef59f4k1qe83o/scheme \
              -H 'Authorization: Bearer frn8fu5rtpyc5m4xy6q3oj4yur' \
              -H 'Content-Type: application/json' \
              -d '{"scheme_id": "qjda3stwafbgpqjaxej3k76sga"}'

  '/channels/{channel_id}/members_minus_group_members':
    get:
      tags:
        - channels
      summary: Channel members minus group members.
      description: |
        Get the set of users who are members of the channel minus the set of users who are members of the given groups.
        Each user object contains an array of group objects representing the group memberships for that user.
        Each user object contains the boolean fields `scheme_guest`, `scheme_user`, and `scheme_admin` representing the roles that user has for the given channel.

        ##### Permissions
        Must have `manage_system` permission.

        __Minimum server version__: 5.14
      parameters:
        - name: channel_id
          in: path
          description: Channel GUID
          required: true
          type: string
        - name: group_ids
          in: query
          description: A comma-separated list of group ids.
          required: true
          default: ""
          type: string
        - name: page
          in: query
          description: The page to select.
          default: "0"
          type: string
        - name: per_page
          in: query
          description: The number of users per page.
          default: "0"
          type: string
      responses:
        '200':
          description: Successfully returns users specified by the pagination, and the total_count.
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
      x-code-samples:
        - lang: 'curl'
          source: |
            curl -X GET \
              'http://your-mattermost-url.com/api/v4/channels/3wyp678obid8pggjmhmhwpah1r/members_minus_group_members?group_ids=eoezijg8zffgjmch8icy5bjd1e,ugaw6wjc3tfxpcr1eq5u5k8dhe&page=0&per_page=100' \
              -H 'Authorization: Bearer kno8tcdotpbx3dj1gzcbx9jrqy' \
              -H 'Content-Type: application/json' \
              -H 'X-Requested-With: XMLHttpRequest'
  /posts:
    post:
      tags:
        - posts
      summary: Create a post
      description: |
        Create a new post in a channel. To create the post as a comment on another post, provide `root_id`.
        ##### Permissions
        Must have `create_post` permission for the channel the post is being created in.
      parameters:
        - in: body
          name: post
          description: Post object to create
          required: true
          schema:
            type: object
            required:
              - channel_id
              - message
            properties:
              channel_id:
                type: string
                description: The channel ID to post in
              message:
                type: string
                description: The message contents, can be formatted with Markdown
              root_id:
                type: string
                description: The post ID to comment on
              file_ids:
                type: array
                description: A list of file IDs to associate with the post. Note that posts are limited to 5 files maximum. Please use additional posts for more files.
                items:
                  type: string
              props:
                description: A general JSON property bag to attach to the post
                type: object
      responses:
        '201':
          description: Post creation successful
          schema:
            $ref: '#/definitions/Post'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'

  /posts/ephemeral:
    post:
      tags:
        - posts
      summary: Create a ephemeral post
      description: |
        Create a new ephemeral post in a channel.
        ##### Permissions
        Must have `create_post_ephemeral` permission (currently only given to system admin)
      parameters:
        - in: body
          name: ephemeral_post
          description: Ephemeral Post object to send
          required: true
          schema:
            type: object
            required:
              - user_id
              - post
            properties:
              user_id:
                type: string
                description: The target user id for the ephemeral post
              post:
                type: object
                required:
                  - channel_id
                  - message
                description: Post object to create
                properties:
                  channel_id:
                    type: string
                    description: The channel ID to post in
                  message:
                    type: string
                    description: The message contents, can be formatted with Markdown
      responses:
        '201':
          description: Post creation successful
          schema:
            $ref: '#/definitions/Post'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
      x-code-samples:
        - lang: 'Go'
          source: |
            client := model.NewAPIv4Client("https://your-mattermost-url.com")
            client.Login("email@domain.com", "Password1")

            ephemeralPost := &model.PostEphemeral{
              UserID: "<ID OF THE USER THAT WOULD RECEIVE THE POST>",
              Post: &model.Post{
                ChannelId: "<ID OF CHANNEL>",
                Message:   "<YOUR MESSAGE>",
              },
            }

            createdPost, response := client.CreatePostEphemeral(ephemeralPost)

  '/posts/{post_id}':
    get:
      tags:
        - posts
      summary: Get a post
      description: |
        Get a single post.
        ##### Permissions
        Must have `read_channel` permission for the channel the post is in or if the channel is public, have the `read_public_channels` permission for the team.
      parameters:
        - name: post_id
          in: path
          description: ID of the post to get
          required: true
          type: string
      responses:
        '200':
          description: Post retrieval successful
          schema:
            $ref: "#/definitions/Post"
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'

    delete:
      tags:
        - posts
      summary: Delete a post
      description: |
        Soft deletes a post, by marking the post as deleted in the database. Soft deleted posts will not be returned in post queries.
        ##### Permissions
        Must be logged in as the user or have `delete_others_posts` permission.
      parameters:
        - name: post_id
          in: path
          description: ID of the post to delete
          required: true
          type: string
      responses:
        '200':
          description: Post deletion successful
          schema:
            $ref: "#/definitions/StatusOK"
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'

    put:
      tags:
        - posts
      summary: Update a post
      description: |
        Update a post. Only the fields listed below are updatable, omitted fields will be treated as blank.
        ##### Permissions
        Must have `edit_post` permission for the channel the post is in.
      parameters:
        - name: post_id
          in: path
          description: ID of the post to update
          required: true
          type: string
        - in: body
          name: body
          description: Post object that is to be updated
          required: true
          schema:
            type: object
            required:
              - id
            properties:
              id:
                description: ID of the post to update
                type: string
              is_pinned:
                description: Set to `true` to pin the post to the channel it is in
                type: boolean
              message:
                description: The message text of the post
                type: string
              has_reactions:
                description: Set to `true` if the post has reactions to it
                type: boolean
              props:
                description: A general JSON property bag to attach to the post
                type: string
      responses:
        '200':
          description: Post update successful
          schema:
            $ref: "#/definitions/Post"
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'

  '/user/{user_id}/posts/{post_id}/set_unread':
    post:
      tags:
        - posts
      summary: Mark as unread from a post.
      description: |
        Mark a channel as being unread from a given post.
        ##### Permissions
        Must have `read_channel` permission for the channel the post is in or if the channel is public, have the `read_public_channels` permission for the team.
        Must have `edit_other_users` permission if the user is not the one marking the post for himself.

        __Minimum server version__: 5.18
      parameters:
        - name: user_id
          in: path
          description: User GUID
          required: true
          type: string
        - name: post_id
          in: path
          description: Post GUID
          required: true
          type: string
      responses:
        '200':
          description: Post marked as unread successfully
          schema:
            $ref: "#/definitions/ChannelUnreadAt"
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '404':
          $ref: '#/responses/NotFound'

  '/posts/{post_id}/patch':
    put:
      tags:
        - posts
      summary: Patch a post
      description: |
        Partially update a post by providing only the fields you want to update. Omitted fields will not be updated. The fields that can be updated are defined in the request body, all other provided fields will be ignored.
        ##### Permissions
        Must have the `edit_post` permission.
      parameters:
        - name: post_id
          in: path
          description: Post GUID
          required: true
          type: string
        - in: body
          name: body
          description: Post object that is to be updated
          required: true
          schema:
            type: object
            properties:
              is_pinned:
                description: Set to `true` to pin the post to the channel it is in
                type: boolean
              message:
                description: The message text of the post
                type: string
              file_ids:
                 description: The list of files attached to this post
                 type: array
              has_reactions:
                description: Set to `true` if the post has reactions to it
                type: boolean
              props:
                description: A general JSON property bag to attach to the post
                type: string
      responses:
        '200':
          description: Post patch successful
          schema:
            $ref: '#/definitions/Post'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'

  '/posts/{post_id}/thread':
    get:
      tags:
        - posts
      summary: Get a thread
      description: |
        Get a post and the rest of the posts in the same thread.
        ##### Permissions
        Must have `read_channel` permission for the channel the post is in or if the channel is public, have the `read_public_channels` permission for the team.
      parameters:
        - name: post_id
          in: path
          description: ID of a post in the thread
          required: true
          type: string
      responses:
        '200':
          description: Post list retrieval successful
          schema:
            $ref: "#/definitions/PostList"
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'

  '/users/{user_id}/posts/flagged':
    get:
      tags:
        - posts
      summary: Get a list of flagged posts
      description: |
        Get a page of flagged posts of a user provided user id string. Selects from a channel, team or all flagged posts by a user.
        ##### Permissions
        Must be user or have `manage_system` permission.
      parameters:
        - name: user_id
          in: path
          description: ID of the user
          required: true
          type: string
        - name: team_id
          in: query
          description: Team ID
          type: string
        - name: channel_id
          in: query
          description: Channel ID
          type: string
        - name: page
          in: query
          description: The page to select
          default: "0"
          type: string
        - name: per_page
          in: query
          description: The number of posts per page
          default: "60"
          type: string
      responses:
        '200':
          description: Post list retrieval successful
          schema:
            type: array
            items:
              $ref: "#/definitions/PostList"
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'

  '/posts/{post_id}/files/info':
    get:
      tags:
        - posts
      summary: Get file info for post
      description: |
        Gets a list of file information objects for the files attached to a post.
        ##### Permissions
        Must have `read_channel` permission for the channel the post is in.
      parameters:
        - name: post_id
          in: path
          description: ID of the post
          required: true
          type: string
      responses:
        '200':
          description: File info retrieval successful
          schema:
            type: array
            items:
              $ref: "#/definitions/FileInfo"
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'

  '/channels/{channel_id}/posts':
    get:
      tags:
        - posts
      summary: Get posts for a channel
      description: |
        Get a page of posts in a channel. Use the query parameters to modify the behaviour of this endpoint. The parameters `since`, `before` and `after` must not be used together.
        ##### Permissions
        Must have `read_channel` permission for the channel.
      parameters:
        - name: channel_id
          in: path
          description: The channel ID to get the posts for
          required: true
          type: string
        - name: page
          in: query
          description: The page to select
          default: "0"
          type: string
        - name: per_page
          in: query
          description: The number of posts per page
          default: "60"
          type: string
        - name: since
          in: query
          description: Provide a non-zero value in Unix time milliseconds to select posts created after that time
          type: integer
        - name: before
          in: query
          description: A post id to select the posts that came before this one
          type: string
        - name: after
          in: query
          description: A post id to select the posts that came after this one
          type: string
      responses:
        '200':
          description: Post list retrieval successful
          schema:
            $ref: "#/definitions/PostList"
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'

  '/users/{user_id}/channels/{channel_id}/posts/unread':
    get:
      tags:
        - posts
      summary: Get posts around last unread
      description: |
        Get the oldest unread post in the channel for the given user as well as the posts around it.
        ##### Permissions
        Must be logged in as the user or have `edit_other_users` permission, and must have `read_channel` permission for the channel.
        __Minimum server version__: 5.14
      parameters:
        - name: user_id
          in: path
          description: ID of the user
          required: true
          type: string
        - name: channel_id
          in: path
          description: The channel ID to get the posts for
          required: true
          type: string
        - name: limit_before
          in: query
          description: Number of posts before the last unread posts. Maximum is 200 posts if limit is set greater than that.
          default: "60"
          type: string
        - name: limit_after
          in: query
          description: Number of posts after and including the last unread post. Maximum is 200 posts if limit is set greater than that.
          default: "60"
          type: string
      responses:
        '200':
          description: Post list retrieval successful
          schema:
            $ref: "#/definitions/PostList"
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'

  '/teams/{team_id}/posts/search':
    post:
      tags:
        - posts
      summary: Search for team posts
      description: |
        Search posts in the team and from the provided terms string.
        ##### Permissions
        Must be authenticated and have the `view_team` permission.
      parameters:
        - name: team_id
          in: path
          description: Team GUID
          required: true
          type: string
        - name: body
          in: body
          description: The search terms and logic to use in the search.
          required: true
          schema:
            type: object
            required:
              - terms
              - is_or_search
            properties:
              terms:
                type: string
                description: The search terms as inputed by the user. To search for posts from a user include `from:someusername`, using a user's username. To search in a specific channel include `in:somechannel`, using the channel name (not the display name).
              is_or_search:
                type: boolean
                description: Set to true if an Or search should be performed vs an And search.
              time_zone_offset:
                type: integer
                default: 0
                description: Offset from UTC of user timezone for date searches.
              include_deleted_channels:
                type: boolean
                description: Set to true if deleted channels should be included in the search. (archived channels)
              page:
                type: integer
                default: 0
                description: The page to select. (Only works with Elasticsearch)
              per_page:
                type: integer
                default: 60
                description: The number of posts per page. (Only works with Elasticsearch)
      responses:
        '200':
          description: Post list retrieval successful
          schema:
            $ref: "#/definitions/PostListWithSearchMatches"
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'

  '/posts/{post_id}/pin':
    post:
      tags:
        - posts
      summary: Pin a post to the channel
      description: |
        Pin a post to a channel it is in based from the provided post id string.
        ##### Permissions
        Must be authenticated and have the `read_channel` permission to the channel the post is in.
      parameters:
        - name: post_id
          in: path
          description: Post GUID
          required: true
          type: string
      responses:
        '200':
          description: Pinned post successful
          schema:
            $ref: "#/definitions/StatusOK"
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'

  '/posts/{post_id}/unpin':
    post:
      tags:
        - posts
      summary: Unpin a post to the channel
      description: |
        Unpin a post to a channel it is in based from the provided post id string.
        ##### Permissions
        Must be authenticated and have the `read_channel` permission to the channel the post is in.
      parameters:
        - name: post_id
          in: path
          description: Post GUID
          required: true
          type: string
      responses:
        '200':
          description: Unpinned post successful
          schema:
            $ref: "#/definitions/StatusOK"
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'

  '/posts/{post_id}/actions/{action_id}':
    post:
      tags:
        - posts
      summary: Perform a post action
      description: |
        Perform a post action, which allows users to interact with integrations through posts.
        ##### Permissions
        Must be authenticated and have the `read_channel` permission to the channel the post is in.
      parameters:
        - name: post_id
          in: path
          description: Post GUID
          required: true
          type: string
        - name: action_id
          in: path
          description: Action GUID
          required: true
          type: string
      responses:
        '200':
          description: Post action successful
          schema:
            $ref: "#/definitions/StatusOK"
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
  /users/{user_id}/preferences:
    get:
      tags:
        - preferences
      summary: Get the user's preferences
      description: |
        Get a list of the user's preferences.
        ##### Permissions
        Must be logged in as the user being updated or have the `edit_other_users` permission.
      parameters:
        - name: user_id
          in: path
          description: User GUID
          required: true
          type: string
      responses:
        '200':
          description: User preferences retrieval successful
          schema:
            type: array
            items:
              $ref: "#/definitions/Preference"
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
    put:
      tags:
        - preferences
      summary: Save the user's preferences
      description: |
        Save a list of the user's preferences.
        ##### Permissions
        Must be logged in as the user being updated or have the `edit_other_users` permission.
      parameters:
        - name: user_id
          in: path
          description: User GUID
          required: true
          type: string
        - in: body
          name: body
          description: List of preference object
          required: true
          schema:
            type: array
            items:
              $ref: '#/definitions/Preference'
      responses:
        '200':
          description: User preferences saved successful
          schema:
            $ref: "#/definitions/StatusOK"
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '404':
          $ref: '#/responses/NotFound'

  '/users/{user_id}/preferences/delete':
    post:
      tags:
        - preferences
      summary: Delete user's preferences
      description: |
        Delete a list of the user's preferences.
        ##### Permissions
        Must be logged in as the user being updated or have the `edit_other_users` permission.
      parameters:
        - name: user_id
          in: path
          description: User GUID
          required: true
          type: string
        - in: body
          name: body
          description: List of preference object
          required: true
          schema:
            type: array
            items:
              $ref: '#/definitions/Preference'
      responses:
        '200':
          description: User preferences saved successful
          schema:
            $ref: "#/definitions/StatusOK"
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'

  '/users/{user_id}/preferences/{category}':
    get:
      tags:
        - preferences
      summary: List a user's preferences by category
      description: |
        Lists the current user's stored preferences in the given category.
        ##### Permissions
        Must be logged in as the user being updated or have the `edit_other_users` permission.
      parameters:
        - name: user_id
          in: path
          description: User GUID
          required: true
          type: string
        - name: category
          in: path
          description: The category of a group of preferences
          required: true
          type: string
      responses:
        '200':
          description: A list of all of the current user's preferences in the given category
          schema:
            type: array
            items:
              $ref: "#/definitions/Preference"
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'

  '/users/{user_id}/preferences/{category}/name/{preference_name}':
    get:
      tags:
        - preferences
      summary: Get a specific user preference
      description: |
        Gets a single preference for the current user with the given category and name.
        ##### Permissions
        Must be logged in as the user being updated or have the `edit_other_users` permission.
      parameters:
        - name: user_id
          in: path
          description: User GUID
          required: true
          type: string
        - name: category
          in: path
          description: The category of a group of preferences
          required: true
          type: string
        - name: preference_name
          in: path
          description: The name of the preference
          required: true
          type: string        
      responses:
        '200':
          description: |
            A single preference for the current user in the current categorylist of all of the current user's preferences in the given category.
          schema:
            $ref: "#/definitions/Preference"
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
  /files:
    post:
      tags:
        - files
      summary: Upload a file
      description: |
        Uploads a file that can later be attached to a post.

        This request can either be a multipart/form-data request with a channel_id, files and optional
        client_ids defined in the FormData, or it can be a request with the channel_id and filename
        defined as query parameters with the contents of a single file in the body of the request.

        Only multipart/form-data requests are supported by server versions up to and including 4.7.
        Server versions 4.8 and higher support both types of requests.

        ##### Permissions
        Must have `upload_file` permission.
      consumes:
        - multipart/form-data
        - '*/*'
      produces:
        - application/json
      parameters:
        - name: files
          in: formData
          description: A file to be uploaded
          required: false
          type: file
        - name: channel_id
          in: formData
          description: The ID of the channel that this file will be uploaded to
          required: false
          type: string
        - name: client_ids
          in: formData
          description: A unique identifier for the file that will be returned in the response
          required: false
          allowEmptyValue: true
          type: string
        - name: channel_id
          in: query
          description: The ID of the channel that this file will be uploaded to
          required: false
          type: string
        - name: filename
          in: query
          description: The name of the file to be uploaded
          required: false
          type: string
      responses:
        '201':
          description: Corresponding lists of the provided client_ids and the metadata that has been stored in the database for each one
          schema:
            type: object
            properties:
              file_infos:
                description: A list of file metadata that has been stored in the database
                type: array
                items:
                  $ref: '#/definitions/FileInfo'
              client_ids:
                description: A list of the client_ids that were provided in the request
                type: array
                items:
                  type: string
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '413':
          $ref: '#/responses/TooLarge'
        '501':
          $ref: '#/responses/NotImplemented'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            file, err := os.Open("file.png")
            if err != nil {
              fmt.Fprintf(os.Stderr, "%v\n", err)
            }
            defer file.Close();

            buf := bytes.NewBuffer(nil)
            io.Copy(buf, file)
            data := buf.Bytes()

            channelID := "4xp9fdt77pncbef59f4k1qe83o"
            filename := "file.png"

            fileUploadResponse, response := Client.UploadFile(data, channelID, filename)

        - lang: 'Curl'
          source: |
            curl -F 'files=@PATH/TO/LOCAL/FILE' \
            -F 'channel_id=CHANNEL_ID' \
            --header 'authorization: Bearer c49adc55z3f53ck7xtp8ebq1ir'
            https://your-mattermost-url.com/api/v4/files


  '/files/{file_id}':
    get:
      tags:
        - files
      summary: Get a file
      description: |
        Gets a file that has been uploaded previously.
        ##### Permissions
        Must have `read_channel` permission or be uploader of the file.
      parameters:
        - name: file_id
          in: path
          description: The ID of the file to get
          required: true
          type: string
      responses:
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '404':
          $ref: '#/responses/NotFound'
        '501':
          $ref: '#/responses/NotImplemented'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            fileID := "4xp9fdt77pncbef59f4k1qe83o"

            data, resp := Client.GetFile(fileID)
  '/files/{file_id}/thumbnail':
    get:
      tags:
        - files
      summary: Get a file's thumbnail
      description: |
        Gets a file's thumbnail.
        ##### Permissions
        Must have `read_channel` permission or be uploader of the file.
      parameters:
        - name: file_id
          in: path
          description: The ID of the file to get
          required: true
          type: string
      responses:
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '404':
          $ref: '#/responses/NotFound'
        '501':
          $ref: '#/responses/NotImplemented'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            fileID := "4xp9fdt77pncbef59f4k1qe83o"

            data, resp := Client.GetFileThumbnail(fileID)

  '/files/{file_id}/preview':
    get:
      tags:
        - files
      summary: Get a file's preview
      description: |
        Gets a file's preview.
        ##### Permissions
        Must have `read_channel` permission or be uploader of the file.
      parameters:
        - name: file_id
          in: path
          description: The ID of the file to get
          required: true
          type: string
      responses:
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '404':
          $ref: '#/responses/NotFound'
        '501':
          $ref: '#/responses/NotImplemented'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            fileID := "4xp9fdt77pncbef59f4k1qe83o"

            data, resp := Client.GetFilePreview(fileID)

  '/files/{file_id}/link':
    get:
      tags:
        - files
      summary: Get a public file link
      description: |
        Gets a public link for a file that can be accessed without logging into Mattermost.
        ##### Permissions
        Must have `read_channel` permission or be uploader of the file.
      parameters:
        - name: file_id
          in: path
          description: The ID of the file to get a link for
          required: true
          type: string
      responses:
        '200':
          description: A publicly accessible link to the given file
          schema:
            type: object
            properties:
              link:
                type: string
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '404':
          $ref: '#/responses/NotFound'
        '501':
          $ref: '#/responses/NotImplemented'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            fileID := "4xp9fdt77pncbef59f4k1qe83o"

            data, resp := Client.GetFileLink(fileID)

  '/files/{file_id}/info':
    get:
      tags:
        - files
      summary: Get metadata for a file
      description: |
        Gets a file's info.
        ##### Permissions
        Must have `read_channel` permission or be uploader of the file.
      parameters:
        - name: file_id
          in: path
          description: The ID of the file info to get
          required: true
          type: string
      responses:
        '200':
          description: The stored metadata for the given file
          schema:
            $ref: "#/definitions/FileInfo"
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '404':
          $ref: '#/responses/NotFound'
        '501':
          $ref: '#/responses/NotImplemented'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            fileID := "4xp9fdt77pncbef59f4k1qe83o"

            info, resp := Client.GetFileInfo(fileID)
  '/jobs':
    get:
      tags:
        - jobs
      summary: Get the jobs.
      description: |
        Get a page of jobs. Use the query parameters to modify the behaviour of this endpoint.
        __Minimum server version: 4.1__
        ##### Permissions
        Must have `manage_jobs` permission.
      parameters:
        - name: page
          in: query
          description: The page to select.
          default: "0"
          type: string
        - name: per_page
          in: query
          description: The number of jobs per page.
          default: "60"
          type: string
      responses:
        '200':
          description: Job list retrieval successful
          schema:
            type: array
            items:
              $ref: '#/definitions/Job'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'

    post:
      tags:
      - jobs
      summary: Create a new job.
      description: |
        Create a new job.
        __Minimum server version: 4.1__
        ##### Permissions
        Must have `manage_jobs` permission.
      parameters:
        - in: body
          name: body
          description: Job object to be created
          required: true
          schema:
            type: object
            required:
              - type
            properties:
              type:
                type: string
                description: The type of job to create
              data:
                type: object
                description: An object containing any additional data required for this job type
      responses:
        '201':
          description: Job creation successful
          schema:
            $ref: '#/definitions/Job'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'

  '/jobs/{job_id}':
    get:
      tags:
        - jobs
      summary: Get a job.
      description: |
        Gets a single job.
        __Minimum server version: 4.1__
        ##### Permissions
        Must have `manage_jobs` permission.
      parameters:
        - name: job_id
          in: path
          description: Job GUID
          required: true
          type: string
      responses:
        '200':
          description: Job retrieval successful
          schema:
            $ref: '#/definitions/Job'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '404':
          $ref: '#/responses/NotFound'

  '/jobs/{job_id}/cancel':
    post:
      tags:
        - jobs
      summary: Cancel a job.
      description: |
        Cancel a job.
        __Minimum server version: 4.1__
        ##### Permissions
        Must have `manage_jobs` permission.
      parameters:
        - name: job_id
          in: path
          description: Job GUID
          required: true
          type: string
      responses:
        '200':
          description: Job canceled successfully
          schema:
            $ref: '#/definitions/StatusOK'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '404':
          $ref: '#/responses/NotFound'

  '/jobs/type/{type}':
    get:
      tags:
        - jobs
      summary: Get the jobs of the given type.
      description: |
        Get a page of jobs of the given type. Use the query parameters to modify the behaviour of this endpoint.
        __Minimum server version: 4.1__
        ##### Permissions
        Must have `manage_jobs` permission.
      parameters:
        - name: type
          in: path
          description: Job type
          required: true
          type: string
        - name: page
          in: query
          description: The page to select.
          default: "0"
          type: string
        - name: per_page
          in: query
          description: The number of jobs per page.
          default: "60"
          type: string
      responses:
        '200':
          description: Job list retrieval successful
          schema:
            type: array
            items:
              $ref: '#/definitions/Job'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
  '/system/ping':
    get:
      tags:
        - system
      summary: Check system health
      description: |
        Check if the server is up and healthy based on the configuration setting `GoRoutineHealthThreshold`. If `GoRoutineHealthThreshold` and the number of goroutines on the server exceeds that threshold the server is considered unhealthy. If `GoRoutineHealthThreshold` is not set or the number of goroutines is below the threshold the server is considered healthy.
        __Minimum server version__: 3.10
        ##### Permissions
        Must be logged in.
      responses:
        '200':
          description: Status of the system
          schema:
            $ref: "#/definitions/StatusOK"
        '500':
          $ref: '#/responses/InternalServerError'
          schema:
            type: object
            items:
              type: string
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            // GetPing
            status, resp := Client.GetPing()

  '/database/recycle':
    post:
      tags:
        - system
      summary: Recycle database connections
      description: |
        Recycle database connections by closing and reconnecting all connections to master and read replica databases.
        ##### Permissions
        Must have `manage_system` permission.
      responses:
        '200':
          description: Database recycle successful
          schema:
            $ref: "#/definitions/StatusOK"
        '403':
          $ref: '#/responses/Forbidden'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            ok, resp := Client.DatabaseRecycle()

  '/email/test':
    post:
      tags:
        - system
      summary: Send a test email
      description: |
        Send a test email to make sure you have your email settings configured correctly. Optionally provide a configuration in the request body to test. If no valid configuration is present in the request body the current server configuration will be tested.
        ##### Permissions
        Must have `manage_system` permission.
      parameters:
        - in: body
          name: body
          description: Mattermost configuration
          required: true
          schema:
            $ref: "#/definitions/Config"
      responses:
        '200':
          description: Email successful sent
          schema:
            $ref: "#/definitions/StatusOK"
        '403':
          $ref: '#/responses/Forbidden'
        '500':
          $ref: '#/responses/InternalServerError'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            config := model.Config{
              EmailSettings: model.EmailSettings{
                SMTPServer:    <SMTPServer>,
                SMTPPort:      <SMTPPort>,
                SMTPUsername:  <SMTPUsername>,
                SMTPPassword:  <SMTPPassword>,
              },
            }

            // TestEmail
            ok, resp := Client.TestEmail(&config)

  '/site_url/test':
    post:
      tags:
        - system
      summary: Checks the validity of a Site URL
      description: |
        Sends a Ping request to the mattermost server using the specified Site URL.

        ##### Permissions
        Must have `manage_system` permission.

        __Minimum server version__: 5.16
      parameters:
        - in: body
          name: body
          required: true
          schema:
            type: object
            required:
              - site_url
            properties:
              site_url:
                type: string
                description: The Site URL to test
      responses:
        '200':
          description: Site URL is valid
          schema:
            $ref: "#/definitions/StatusOK"
        '400':
          $ref: '#/responses/BadRequest'
        '403':
          $ref: '#/responses/Forbidden'
        '500':
          $ref: '#/responses/InternalServerError'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            siteURL := "https://your-new-mattermost-url.com"

            // TestSiteURL
            ok, resp := Client.TestSiteURL(siteUrl)

  '/file/s3_test':
    post:
      tags:
        - system
      summary: Test AWS S3 connection
      description: |
        Send a test to validate if can connect to AWS S3. Optionally provide a configuration in the request body to test. If no valid configuration is present in the request body the current server configuration will be tested.
        ##### Permissions
        Must have `manage_system` permission.
        __Minimum server version__: 4.8
      parameters:
        - in: body
          name: body
          description: Mattermost configuration
          required: true
          schema:
            $ref: "#/definitions/Config"
      responses:
        '200':
          description: S3 Test successful
          schema:
            $ref: "#/definitions/StatusOK"
        '403':
          $ref: '#/responses/Forbidden'
        '500':
          $ref: '#/responses/InternalServerError'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            config := model.Config{
              FileSettings: model.FileSettings{
                DriverName:              model.NewString(model.IMAGE_DRIVER_S3),
                AmazonS3AccessKeyId:     <AmazonS3AccessKeyId>,
                AmazonS3SecretAccessKey: <AmazonS3SecretAccessKey>,
                AmazonS3Bucket:          <AmazonS3Bucket>,
                AmazonS3Endpoint:        <AmazonS3Endpoint>
              },
            }

            // TestS3Connection
            ok, resp := Client.TestS3Connection(&config)

  '/config':
    get:
      tags:
        - system
      summary: Get configuration
      description: |
        Retrieve the current server configuration
        ##### Permissions
        Must have `manage_system` permission.
      responses:
        '200':
          description: Configuration retrieval successful
          schema:
            $ref: "#/definitions/Config"
        '400':
          $ref: '#/responses/BadRequest'
        '403':
          $ref: '#/responses/Forbidden'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            // GetConfig
            config, resp := Client.GetConfig()

    put:
      tags:
        - system
      summary: Update configuration
      description: |
        Submit a new configuration for the server to use. As of server version 4.8, the `PluginSettings.EnableUploads` setting cannot be modified by this endpoint.
        ##### Permissions
        Must have `manage_system` permission.
      parameters:
        - in: body
          name: body
          description: Mattermost configuration
          required: true
          schema:
            $ref: "#/definitions/Config"
      responses:
        '200':
          description: Configuration update successful
          schema:
            $ref: "#/definitions/Config"
        '400':
          $ref: '#/responses/BadRequest'
        '403':
          $ref: '#/responses/Forbidden'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            // GetConfig
            config, resp := Client.GetConfig()

            config.TeamSettings.SiteName = "MyFancyName"

            // UpdateConfig
            updatedConfig, resp := Client.UpdateConfig(config)

  '/config/reload':
    post:
      tags:
        - system
      summary: Reload configuration
      description: |
        Reload the configuration file to pick up on any changes made to it.
        ##### Permissions
        Must have `manage_system` permission.
      responses:
        '200':
          description: Configuration reload successful
          schema:
            $ref: "#/definitions/StatusOK"
        '400':
          $ref: '#/responses/BadRequest'
        '403':
          $ref: '#/responses/Forbidden'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            // ReloadConfig
            ok, resp := Client.ReloadConfig()

  '/config/client':
    get:
      tags:
        - system
      summary: Get client configuration
      description: |
        Get a subset of the server configuration needed by the client.
        ##### Permissions
        No permission required.
      parameters:
        - name: format
          in: query
          required: true
          description: Must be `old`, other formats not implemented yet
          type: string
      responses:
        '200':
          description: Configuration retrieval successful
        '400':
          $ref: '#/responses/BadRequest'
        '501':
          $ref: '#/responses/NotImplemented'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            // GetOldClientConfig
            ok, resp := Client.GetOldClientConfig()

  '/config/environment':
    get:
      tags:
        - system
      summary: Get configuration made through environment variables
      description: |
        Retrieve a json object mirroring the server configuration where fields are set to true
        if the corresponding config setting is set through an environment variable. Settings
        that haven't been set through environment variables will be missing from the object.

        __Minimum server version__: 4.10

        ##### Permissions
        Must have `manage_system` permission.
      responses:
        '200':
          description: Configuration retrieval successful
          schema:
            $ref: "#/definitions/EnvironmentConfig"
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'

  '/license':
    post:
      tags:
        - system
      summary: Upload license file
      description: |
        Upload a license to enable enterprise features.

        __Minimum server version__: 4.0

        ##### Permissions
        Must have `manage_system` permission.
      consumes: ["multipart/form-data"]
      parameters:
        - name: license
          in: formData
          description: The license to be uploaded
          required: true
          type: file
      responses:
        '201':
          description: License file upload successful
          schema:
            $ref: '#/definitions/StatusOK'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '413':
          $ref: '#/responses/TooLarge'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            file, err := os.Open("<Your license file>")
            if err != nil {
              return err
            }
            defer file.Close()

            data := &bytes.Buffer{}
            if _, err := io.Copy(data, file); err != nil {
              return err
            }

            ok, resp := Client.UploadLicenseFile(data.Bytes())

    delete:
      tags:
        - system
      summary: Remove license file
      description: |
        Remove the license file from the server. This will disable all enterprise features.

        __Minimum server version__: 4.0

        ##### Permissions
        Must have `manage_system` permission.
      responses:
        '200':
          description: License removal successful
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'

  '/license/client':
    get:
      tags:
        - system
      summary: Get client license
      description: |
        Get a subset of the server license needed by the client.
        ##### Permissions
        No permission required but having the `manage_system` permission returns more information.
      parameters:
        - name: format
          in: query
          required: true
          description: Must be `old`, other formats not implemented yet
          type: string
      responses:
        '200':
          description: License retrieval successful
        '400':
          $ref: '#/responses/BadRequest'
        '501':
          $ref: '#/responses/NotImplemented'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            // GetOldClientLicense
            license, resp := Client.GetOldClientLicense()

  '/audits':
    get:
      tags:
        - system
      summary: Get audits
      description: |
        Get a page of audits for all users on the system, selected with `page` and `per_page` query parameters.
        ##### Permissions
        Must have `manage_system` permission.
      parameters:
        - name: page
          in: query
          description: The page to select.
          default: "0"
          type: string
        - name: per_page
          in: query
          description: The number of audits per page.
          default: "60"
          type: string
      responses:
        '200':
          description: Audits retrieval successful
          schema:
            type: array
            items:
              $ref: "#/definitions/Audit"
        '403':
          $ref: '#/responses/Forbidden'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            // GetAudits
            audits, resp := Client.GetAudits(0, 100, "")

  '/caches/invalidate':
    post:
      tags:
        - system
      summary: Invalidate all the caches
      description: |
        Purge all the in-memory caches for the Mattermost server. This can have a temporary negative effect on performance while the caches are re-populated.
        ##### Permissions
        Must have `manage_system` permission.
      responses:
        '200':
          description: Caches invalidate successful
          schema:
            $ref: "#/definitions/StatusOK"
        '403':
          $ref: '#/responses/Forbidden'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            // InvalidateCaches
            ok, resp := Client.InvalidateCaches()

  '/logs':
    get:
      tags:
        - system
      summary: Get logs
      description: |
        Get a page of server logs, selected with `page` and `logs_per_page` query parameters.
        ##### Permissions
        Must have `manage_system` permission.
      parameters:
        - name: page
          in: query
          description: The page to select.
          default: "0"
          type: string
        - name: logs_per_page
          in: query
          description: The number of logs per page. There is a maximum limit of 10000 logs per page.
          default: "10000"
          type: string
      responses:
        '200':
          description: Logs retrieval successful
          schema:
            type: array
            items:
              type: string
        '403':
          $ref: '#/responses/Forbidden'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            // GetLogs
            logs, resp := Client.GetLogs(0, 10)

    post:
      tags:
        - system
      summary: Add log message
      description: |
        Add log messages to the server logs.
        ##### Permissions
        Users with `manage_system` permission can log ERROR or DEBUG messages.
        Logged in users can log ERROR or DEBUG messages when `ServiceSettings.EnableDeveloper` is `true` or just DEBUG messages when `false`.
        Non-logged in users can log ERROR or DEBUG messages when `ServiceSettings.EnableDeveloper` is `true` and cannot log when `false`.
      parameters:
        - in: body
          name: body
          required: true
          schema:
            type: object
            required:
              - level
              - message
            properties:
              level:
                type: string
                description: The error level, ERROR or DEBUG
              message:
                type: string
                description: Message to send to the server logs
      responses:
        '200':
          description: Logs sent successful
          schema:
            type: object
            items:
              type: string
        '403':
          $ref: '#/responses/Forbidden'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            message := make(map[string]string)
            message["level"] = "ERROR"
            message["message"] = "this is a test"

            // PostLog
            _, resp := Client.PostLog(message)

  '/analytics/old':
    get:
      tags:
        - system
      summary: Get analytics
      description: |
        Get some analytics data about the system. This endpoint uses the old format, the `/analytics` route is reserved for the new format when it gets implemented.

        The returned JSON changes based on the `name` query parameter but is always key/value pairs.

        __Minimum server version__: 4.0

        ##### Permissions
        Must have `manage_system` permission.
      parameters:
        - name: name
          in: query
          required: false
          description: Possible values are "standard", "post_counts_day", "user_counts_with_posts_day" or "extra_counts"
          default: "standard"
          type: string
        - name: team_id
          in: query
          required: false
          description: The team ID to filter the data by
          type: string
      responses:
        '200':
          description: Analytics retrieval successful
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
  /emoji:
    post:
      tags:
        - emoji
      summary: Create a custom emoji
      description: |
        Create a custom emoji for the team.
        ##### Permissions
        Must be authenticated.
      consumes: ["multipart/form-data"]
      parameters:
        - name: image
          in: formData
          description: A file to be uploaded
          required: true
          type: file
        - name: emoji
          in: formData
          description: A JSON object containing a `name` field with the name of the emoji and a `creator_id` field with the id of the authenticated user.
          required: true
          type: string
      responses:
        '201':
          description: Emoji creation successful
          schema:
            $ref: '#/definitions/Emoji'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '413':
          $ref: '#/responses/TooLarge'
        '501':
          $ref: '#/responses/NotImplemented'

    get:
      tags:
        - emoji
      summary: Get a list of custom emoji
      description: |
        Get a page of metadata for custom emoji on the system. Since server version 4.7, sort using the `sort` query parameter.
        ##### Permissions
        Must be authenticated.
      parameters:
        - name: page
          in: query
          description: The page to select.
          default: "0"
          type: string
        - name: per_page
          in: query
          description: The number of users per page.
          default: "60"
          type: string
        - name: sort
          in: query
          description: Either blank for no sorting or "name" to sort by emoji names. Minimum server version for sorting is 4.7.
          default: ""
          type: string
      responses:
        '200':
          description: Emoji list retrieval successful
          schema:
            $ref: '#/definitions/Emoji'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '501':
          $ref: '#/responses/NotImplemented'

  '/emoji/{emoji_id}':
    get:
      tags:
        - emoji
      summary: Get a custom emoji
      description: |
        Get some metadata for a custom emoji.
        ##### Permissions
        Must be authenticated.
      parameters:
        - name: emoji_id
          in: path
          description: Emoji GUID
          required: true
          type: string
      responses:
        '200':
          description: Emoji retrieval successful
          schema:
            $ref: '#/definitions/Emoji'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '404':
          $ref: '#/responses/NotFound'
        '501':
          $ref: '#/responses/NotImplemented'

    delete:
      tags:
        - emoji
      summary: Delete a custom emoji
      description: |
        Delete a custom emoji.
        ##### Permissions
        Must have the `manage_team` or `manage_system` permissions or be the user who created the emoji.
      parameters:
        - name: emoji_id
          in: path
          description: Emoji GUID
          required: true
          type: string
      responses:
        '200':
          description: Emoji delete successful
          schema:
            $ref: '#/definitions/Emoji'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '501':
          $ref: '#/responses/NotImplemented'

  '/emoji/name/{emoji_name}':
    get:
      tags:
        - emoji
      summary: Get a custom emoji by name
      description: |
        Get some metadata for a custom emoji using its name.
        ##### Permissions
        Must be authenticated.

        __Minimum server version__: 4.7
      parameters:
        - name: emoji_name
          in: path
          description: Emoji name
          required: true
          type: string
      responses:
        '200':
          description: Emoji retrieval successful
          schema:
            $ref: '#/definitions/Emoji'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '404':
          $ref: '#/responses/NotFound'
        '501':
          $ref: '#/responses/NotImplemented'

  '/emoji/{emoji_id}/image':
    get:
      tags:
        - emoji
      summary: Get custom emoji image
      description: |
        Get the image for a custom emoji.
        ##### Permissions
        Must be authenticated.
      parameters:
        - name: emoji_id
          in: path
          description: Emoji GUID
          required: true
          type: string
      responses:
        '200':
          description: Emoji image retrieval successful
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '404':
          $ref: '#/responses/NotFound'
        '500':
          $ref: '#/responses/InternalServerError'
        '501':
          $ref: '#/responses/NotImplemented'

  /emoji/search:
    post:
      tags:
        - emoji
      summary: Search custom emoji
      description: |
        Search for custom emoji by name based on search criteria provided in the request body. A maximum of 200 results are returned.
        ##### Permissions
        Must be authenticated.

        __Minimum server version__: 4.7
      parameters:
        - in: body
          name: body
          description: Search criteria
          required: true
          schema:
            type: object
            required:
              - term
            properties:
              term:
                description: The term to match against the emoji name.
                type: string
              prefix_only:
                description: Set to only search for names starting with the search term.
                type: string
      responses:
        '200':
          description: Emoji list retrieval successful
          schema:
            type: array
            items:
              $ref: '#/definitions/Emoji'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '501':
          $ref: '#/responses/NotImplemented'

  /emoji/autocomplete:
    get:
      tags:
        - emoji
      summary: Autocomplete custom emoji
      description: |
        Get a list of custom emoji with names starting with or matching the provided name. Returns a maximum of 100 results.
        ##### Permissions
        Must be authenticated.

        __Minimum server version__: 4.7
      parameters:
        - name: name
          in: query
          description: The emoji name to search.
          type: string
          required: true
      responses:
        '200':
          description: Emoji list retrieval successful
          schema:
            $ref: '#/definitions/Emoji'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '501':
          $ref: '#/responses/NotImplemented'
  /hooks/incoming:
    post:
      tags:
        - webhooks
      summary: Create an incoming webhook
      description: |
        Create an incoming webhook for a channel.
        ##### Permissions
        `manage_webhooks` for the channel the webhook is in.
      parameters:
        - in: body
          name: body
          description: Incoming webhook to be created
          required: true
          schema:
            type: object
            required:
              - channel_id
            properties:
              channel_id:
                type: string
                description: The ID of a public channel or private group that receives the webhook payloads.
              display_name:
                  type: string
                  description: The display name for this incoming webhook
              description:
                type: string
                description: The description for this incoming webhook
              username:
                type: string
                description: The username this incoming webhook will post as.
              icon_url:
                type: string
                description: The profile picture this incoming webhook will use when posting.
      responses:
        '201':
          description: Incoming webhook creation successful
          schema:
            $ref: "#/definitions/IncomingWebhook"
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'

    get:
      tags:
        - webhooks
      summary: List incoming webhooks
      description: |
        Get a page of a list of incoming webhooks. Optionally filter for a specific team using query parameters.
        ##### Permissions
        `manage_webhooks` for the system or `manage_webhooks` for the specific team.
      parameters:
        - name: page
          in: query
          description: The page to select.
          default: "0"
          type: string
        - name: per_page
          in: query
          description: The number of hooks per page.
          default: "60"
          type: string
        - name: team_id
          in: query
          description: The ID of the team to get hooks for.
          type: string
      responses:
        '200':
          description: Incoming webhooks retrieval successful
          schema:
            type: array
            items:
              $ref: "#/definitions/IncomingWebhook"
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'

  /hooks/incoming/{hook_id}:
    get:
      tags:
        - webhooks
      summary: Get an incoming webhook
      description: |
        Get an incoming webhook given the hook id.
        ##### Permissions
        `manage_webhooks` for system or `manage_webhooks` for the specific team or `manage_webhooks` for the channel.
      parameters:
        - name: hook_id
          in: path
          description: Incoming Webhook GUID
          required: true
          type: string
      responses:
        '200':
          description: Webhook retrieval successful
          schema:
            $ref: '#/definitions/IncomingWebhook'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '404':
          $ref: '#/responses/NotFound'

    put:
      tags:
        - webhooks
      summary: Update an incoming webhook
      description: |
        Update an incoming webhook given the hook id.
        ##### Permissions
        `manage_webhooks` for system or `manage_webhooks` for the specific team or `manage_webhooks` for the channel.
      parameters:
        - name: hook_id
          in: path
          description: Incoming Webhook GUID
          required: true
          type: string
        - in: body
          name: body
          description: Incoming webhook to be updated
          required: true
          schema:
            type: object
            required:
              - hook_id
              - channel_id
              - display_name
              - description
            properties:
              hook_id:
                type: string
                description: Incoming webhook GUID
              channel_id:
                type: string
                description: The ID of a public channel or private group that receives the webhook payloads.
              display_name:
                  type: string
                  description: The display name for this incoming webhook
              description:
                type: string
                description: The description for this incoming webhook
              username:
                type: string
                description: The username this incoming webhook will post as.
              icon_url:
                type: string
                description: The profile picture this incoming webhook will use when posting.
      responses:
        '200':
          description: Webhook update successful
          schema:
            $ref: '#/definitions/IncomingWebhook'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '404':
          $ref: '#/responses/NotFound'

  /hooks/outgoing:
    post:
      tags:
        - webhooks
      summary: Create an outgoing webhook
      description: |
        Create an outgoing webhook for a team.
        ##### Permissions
        `manage_webhooks` for the team the webhook is in.
      parameters:
        - in: body
          name: body
          description: Outgoing webhook to be created
          required: true
          schema:
            type: object
            required:
              - team_id
              - display_name
              - trigger_words
              - callback_urls
            properties:
              team_id:
                description: The ID of the team that the webhook watchs
                type: string
              channel_id:
                description: The ID of a public channel that the webhook watchs
                type: string
              description:
                description: The description for this outgoing webhook
                type: string
              display_name:
                description: The display name for this outgoing webhook
                type: string
              trigger_words:
                description: List of words for the webhook to trigger on
                type: array
                items:
                  type: string
              trigger_when:
                description: When to trigger the webhook, `0` when a trigger word is present at all and `1` if the message starts with a trigger word
                type: integer
              callback_urls:
                description: The URLs to POST the payloads to when the webhook is triggered
                type: array
                items:
                  type: string
              content_type:
                description: The format to POST the data in, either `application/json` or `application/x-www-form-urlencoded`
                default: "application/x-www-form-urlencoded"
                type: string
      responses:
        '201':
          description: Outgoing webhook creation successful
          schema:
            $ref: "#/definitions/OutgoingWebhook"
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '404':
          $ref: '#/responses/NotFound'

    get:
      tags:
        - webhooks
      summary: List outgoing webhooks
      description: |
        Get a page of a list of outgoing webhooks. Optionally filter for a specific team or channel using query parameters.
        ##### Permissions
        `manage_webhooks` for the system or `manage_webhooks` for the specific team/channel.
      parameters:
        - name: page
          in: query
          description: The page to select.
          default: "0"
          type: string
        - name: per_page
          in: query
          description: The number of hooks per page.
          default: "60"
          type: string
        - name: team_id
          in: query
          description: The ID of the team to get hooks for.
          type: string
        - name: channel_id
          in: query
          description: The ID of the channel to get hooks for.
          type: string
      responses:
        '200':
          description: Outgoing webhooks retrieval successful
          schema:
            type: array
            items:
              $ref: "#/definitions/OutgoingWebhook"
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '404':
          $ref: '#/responses/NotFound'

  /hooks/outgoing/{hook_id}:
    get:
      tags:
        - webhooks
      summary: Get an outgoing webhook
      description: |
        Get an outgoing webhook given the hook id.
        ##### Permissions
        `manage_webhooks` for system or `manage_webhooks` for the specific team or `manage_webhooks` for the channel.
      parameters:
        - name: hook_id
          in: path
          description: Outgoing webhook GUID
          required: true
          type: string
      responses:
        '200':
          description: Outgoing webhook retrieval successful
          schema:
            $ref: '#/definitions/OutgoingWebhook'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '404':
          $ref: '#/responses/NotFound'
    delete:
      tags:
        - webhooks
      summary: Delete an outgoing webhook
      description: |
        Delete an outgoing webhook given the hook id.
        ##### Permissions
        `manage_webhooks` for system or `manage_webhooks` for the specific team or `manage_webhooks` for the channel.
      parameters:
        - name: hook_id
          in: path
          description: Outgoing webhook GUID
          required: true
          type: string
      responses:
        '200':
          description: Webhook deletion successful
          schema:
            $ref: '#/definitions/StatusOK'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '404':
          $ref: '#/responses/NotFound'
    put:
      tags:
        - webhooks
      summary: Update an outgoing webhook
      description: |
        Update an outgoing webhook given the hook id.
        ##### Permissions
        `manage_webhooks` for system or `manage_webhooks` for the specific team or `manage_webhooks` for the channel.
      parameters:
        - name: hook_id
          in: path
          description: outgoing Webhook GUID
          required: true
          type: string
        - in: body
          name: body
          description: Outgoing webhook to be updated
          required: true
          schema:
            type: object
            required:
              - hook_id
              - channel_id
              - display_name
              - description
            properties:
              hook_id:
                type: string
                description: Outgoing webhook GUID
              channel_id:
                type: string
                description: The ID of a public channel or private group that receives the webhook payloads.
              display_name:
                  type: string
                  description: The display name for this incoming webhook
              description:
                type: string
                description: The description for this incoming webhook
      responses:
        '200':
          description: Webhook update successful
          schema:
            $ref: '#/definitions/OutgoingWebhook'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '404':
          $ref: '#/responses/NotFound'

  /hooks/outgoing/{hook_id}/regen_token:
    post:
      tags:
        - webhooks
      summary: Regenerate the token for the outgoing webhook.
      description: |
        Regenerate the token for the outgoing webhook.
        ##### Permissions
        `manage_webhooks` for system or `manage_webhooks` for the specific team or `manage_webhooks` for the channel.
      parameters:
        - name: hook_id
          in: path
          description: Outgoing webhook GUID
          required: true
          type: string
      responses:
        '200':
          description: Webhook token regenerate successful
          schema:
            $ref: '#/definitions/StatusOK'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '404':
          $ref: '#/responses/NotFound'
  /saml/metadata:
    get:
      tags:
        - SAML
      summary: Get metadata
      description: |
        Get SAML metadata from the server. SAML must be configured properly.
        ##### Permissions
        No permission required.
      responses:
        '200':
          description: SAML metadata retrieval successful
          schema:
            type: string
        '501':
          $ref: '#/responses/NotImplemented'

  /saml/certificate/idp:
    post:
      tags:
        - SAML
      summary: Upload IDP certificate
      description: |
        Upload the IDP certificate to be used with your SAML configuration. The server will pick a hard-coded filename for the IdpCertificateFile setting in your `config.json`.
        ##### Permissions
        Must have `manage_system` permission.
      consumes: ["multipart/form-data"]
      parameters:
        - name: certificate
          in: formData
          description: The IDP certificate file
          required: true
          type: file
      responses:
        '200':
          description: SAML certificate upload successful
          schema:
            $ref: '#/definitions/StatusOK'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '501':
          $ref: '#/responses/NotImplemented'

    delete:
      tags:
        - SAML
      summary: Remove IDP certificate
      description: |
        Delete the current IDP certificate being used with your SAML configuration. This will also disable SAML on your system as this certificate is required for SAML.
        ##### Permissions
        Must have `manage_system` permission.
      responses:
        '200':
          description: SAML certificate delete successful
          schema:
            $ref: '#/definitions/StatusOK'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '501':
          $ref: '#/responses/NotImplemented'

  /saml/certificate/public:
    post:
      tags:
        - SAML
      summary: Upload public certificate
      description: |
        Upload the public certificate to be used for encryption with your SAML configuration. The server will pick a hard-coded filename for the PublicCertificateFile setting in your `config.json`.
        ##### Permissions
        Must have `manage_system` permission.
      consumes: ["multipart/form-data"]
      parameters:
        - name: certificate
          in: formData
          description: The public certificate file
          required: true
          type: file
      responses:
        '200':
          description: SAML certificate upload successful
          schema:
            $ref: '#/definitions/StatusOK'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '501':
          $ref: '#/responses/NotImplemented'

    delete:
      tags:
        - SAML
      summary: Remove public certificate
      description: |
        Delete the current public certificate being used with your SAML configuration. This will also disable encryption for SAML on your system as this certificate is required for that.
        ##### Permissions
        Must have `manage_system` permission.
      responses:
        '200':
          description: SAML certificate delete successful
          schema:
            $ref: '#/definitions/StatusOK'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '501':
          $ref: '#/responses/NotImplemented'

  /saml/certificate/private:
    post:
      tags:
        - SAML
      summary: Upload private key
      description: |
        Upload the private key to be used for encryption with your SAML configuration. The server will pick a hard-coded filename for the PrivateKeyFile setting in your `config.json`.
        ##### Permissions
        Must have `manage_system` permission.
      consumes: ["multipart/form-data"]
      parameters:
        - name: certificate
          in: formData
          description: The private key file
          required: true
          type: file
      responses:
        '200':
          description: SAML certificate upload successful
          schema:
            $ref: '#/definitions/StatusOK'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '501':
          $ref: '#/responses/NotImplemented'

    delete:
      tags:
        - SAML
      summary: Remove private key
      description: |
        Delete the current private key being used with your SAML configuration. This will also disable encryption for SAML on your system as this key is required for that.
        ##### Permissions
        Must have `manage_system` permission.
      responses:
        '200':
          description: SAML certificate delete successful
          schema:
            $ref: '#/definitions/StatusOK'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '501':
          $ref: '#/responses/NotImplemented'

  /saml/certificate/status:
    get:
      tags:
        - SAML
      summary: Get certificate status
      description: |
        Get the status of the uploaded certificates and keys in use by your SAML configuration.
        ##### Permissions
        Must have `manage_system` permission.
      responses:
        '200':
          description: SAML certificate status retrieval successful
          schema:
            $ref: '#/definitions/SamlCertificateStatus'
        '403':
          $ref: '#/responses/Forbidden'
        '501':
          $ref: '#/responses/NotImplemented'
  /compliance/reports:
    post:
      tags:
        - compliance
      summary: Create report
      description: |
        Create and save a compliance report.
        ##### Permissions
        Must have `manage_system` permission.
      responses:
        '201':
          description: Compliance report creation successful
          schema:
            $ref: '#/definitions/Compliance'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '501':
          $ref: '#/responses/NotImplemented'

    get:
      tags:
        - compliance
      summary: Get reports
      description: |
        Get a list of compliance reports previously created by page, selected with `page` and `per_page` query parameters.
        ##### Permissions
        Must have `manage_system` permission.
      parameters:
        - name: page
          in: query
          description: The page to select.
          default: "0"
          type: string
        - name: per_page
          in: query
          description: The number of reports per page.
          default: "60"
          type: string
      responses:
        '200':
          description: Compliance reports retrieval successful
          schema:
            type: array
            items:
              $ref: '#/definitions/Compliance'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '501':
          $ref: '#/responses/NotImplemented'

  /compliance/reports/{report_id}:
    get:
      tags:
        - compliance
      summary: Get a report
      description: |
        Get a compliance reports previously created.
        ##### Permissions
        Must have `manage_system` permission.
      parameters:
        - name: report_id
          in: path
          description: Compliance report GUID
          required: true
          type: string
      responses:
        '200':
          description: Compliance report retrieval successful
          schema:
            $ref: '#/definitions/Compliance'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '501':
          $ref: '#/responses/NotImplemented'

  /compliance/reports/{report_id}/download:
    get:
      tags:
        - compliance
      summary: Download a report
      description: |
        Download the full contents of a report as a file.
        ##### Permissions
        Must have `manage_system` permission.
      parameters:
        - name: report_id
          in: path
          description: Compliance report GUID
          required: true
          type: string
      responses:
        '200':
          description: The compliance report file
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '501':
          $ref: '#/responses/NotImplemented'
  /ldap/sync:
    post:
      tags:
        - LDAP
      summary: Sync with LDAP
      description: |
        Synchronize any user attribute changes in the configured AD/LDAP server with Mattermost.
        ##### Permissions
        Must have `manage_system` permission.
      responses:
        '200':
          description: LDAP sync successful
          schema:
            $ref: '#/definitions/StatusOK'
        '501':
          $ref: '#/responses/NotImplemented'

  /ldap/test:
    post:
      tags:
        - LDAP
      summary: Test LDAP configuration
      description: |
        Test the current AD/LDAP configuration to see if the AD/LDAP server can be contacted successfully.
        ##### Permissions
        Must have `manage_system` permission.
      responses:
        '200':
          description: LDAP test successful
          schema:
            $ref: '#/definitions/StatusOK'
        '500':
          $ref: '#/responses/InternalServerError'
        '501':
          $ref: '#/responses/NotImplemented'

  '/channels/{channel_id}/groups':
    get:
      tags:
        - groups
      summary: Get channel groups
      description: |
        Retrieve the list of groups associated with a given channel.

        ##### Permissions
        Must have `manage_system` permission.

        __Minimum server version__: 5.11
      parameters:
        - name: channel_id
          in: path
          description: Channel GUID
          required: true
          type: string
        - name: page
          in: query
          description: The page to select.
          default: "0"
          type: string
        - name: per_page
          in: query
          description: The number of groups per page.
          default: "60"
          type: string
      responses:
        '200':
          description: Group list retrieval successful
          schema:
            type: array
            items:
              $ref: '#/definitions/Group'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'

  '/teams/{team_id}/groups':
    get:
      tags:
        - groups
      summary: Get team groups
      description: |
        Retrieve the list of groups associated with a given team.

        ##### Permissions
        Must have `manage_system` permission.

        __Minimum server version__: 5.11
      parameters:
        - name: team_id
          in: path
          description: Team GUID
          required: true
          type: string
        - name: page
          in: query
          description: The page to select.
          default: "0"
          type: string
        - name: per_page
          in: query
          description: The number of groups per page.
          default: "60"
          type: string
      responses:
        '200':
          description: Group list retrieval successful
          schema:
            type: array
            items:
              $ref: '#/definitions/Group'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
  /cluster/status:
    get:
      tags:
        - cluster
      summary: Get cluster status
      description: |
        Get a set of information for each node in the cluster, useful for checking the status and health of each node.
        ##### Permissions
        Must have `manage_system` permission.
      responses:
        '200':
          description: Cluster status retrieval successful
          schema:
            type: array
            items:
              $ref: '#/definitions/ClusterInfo'
        '403':
          $ref: '#/responses/Forbidden'

  /brand/image:
    get:
      tags:
        - brand
      summary: Get brand image
      description: |
        Get the previously uploaded brand image. Returns 404 if no brand image has been uploaded.
        ##### Permissions
        No permission required.
      responses:
        '200':
          description: Brand image retrieval successful
          schema:
            type: string
        '404':
          description: No brand image uploaded
          $ref: '#/responses/NotFound'
        '501':
          $ref: '#/responses/NotImplemented'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            // GetBrandImage
            img, err := Client.GetBrandImage()

    post:
      tags:
        - brand
      summary: Upload brand image
      description: |
        Uploads a brand image.
        ##### Permissions
        Must have `manage_system` permission.
      consumes: ["multipart/form-data"]
      parameters:
        - name: image
          in: formData
          description: The image to be uploaded
          required: true
          type: file
      responses:
        '201':
          description: Brand image upload successful
          schema:
            $ref: '#/definitions/StatusOK'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '413':
          $ref: '#/responses/TooLarge'
        '501':
          $ref: '#/responses/NotImplemented'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            file, err := os.Open("<Your image>")
            if err != nil {
              return err
            }
            defer file.Close()

            data := &bytes.Buffer{}
            if _, err := io.Copy(data, file); err != nil {
              return err
            }

            ok, resp := Client.UploadBrandImage(data.Bytes())

    delete:
      tags:
        - brand
      summary: Delete current brand image
      description: |
        Deletes the previously uploaded brand image. Returns 404 if no brand image has been uploaded.
        ##### Permissions
        Must have `manage_system` permission.
        __Minimum server version: 5.6__
      responses:
        '200':
          description: Brand image succesfully deleted
          schema:
            $ref: '#/definitions/StatusOK'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '404':
          description: No brand image uploaded
          $ref: '#/responses/NotFound'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            // Delete brand image
            resp := Client.DeleteBrandImage()
  /commands:
    post:
      tags:
        - commands
      summary: Create a command
      description: |
        Create a command for a team.
        ##### Permissions
        `manage_slash_commands` for the team the command is in.
      parameters:
        - in: body
          name: body
          description: command to be created
          required: true
          schema:
            type: object
            required:
              - team_id
              - method
              - trigger
              - url
            properties:
              team_id:
                type: string
                description: Team ID to where the command should be created
              method:
                type: string
                description: "`'P'` for post request, `'G'` for get request"
              trigger:
                type: string
                description: Activation word to trigger the command
              url:
                type: string
                description: The URL that the command will make the request
      responses:
        '201':
          description: Command creation successful
          schema:
            $ref: "#/definitions/Command"
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '501':
          $ref: '#/responses/NotImplemented'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            newCmd := &model.Command {
              TeamId:       <TEAMID>,
              URL:          "http://nowhere.com",
              Method:       model.COMMAND_METHOD_POST,
              Trigger:      "trigger",
              AutoComplete: false,
              Description:  "Description",
              DisplayName:  "Display name",
              IconURL:      "IconURL",
              Username:     "Username"
            }

            // CreateCommand
            createdCmd, resp := Client.CreateCommand(newCmd)

    get:
      tags:
        - commands
      summary: List commands for a team
      description: |
        List commands for a team.
        ##### Permissions
        `manage_slash_commands` if need list custom commands.
      parameters:
        - name: team_id
          in: query
          description: The team id.
          type: string
        - name: custom_only
          in: query
          description: |
            To get only the custom commands. If set to false will get the custom
            if the user have access plus the system commands, otherwise just the system commands.
          default: "false"
          type: string
      responses:
        '200':
          description: List Commands retrieve successful
          schema:
            type: array
            items:
              $ref: "#/definitions/Command"
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '501':
          $ref: '#/responses/NotImplemented'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            // ListCommands
            // The second parameter is to set if you want only custom commands (true) or defaults commands (false)
            listCommands, resp := Client.ListCommands(<TEAMID>, true)

  '/teams/{team_id}/commands/autocomplete':
    get:
      tags:
        - commands
      summary: List autocomplete commands
      description: |
        List autocomplete commands in the team.
        ##### Permissions
        `view_team` for the team.
      parameters:
        - name: team_id
          in: path
          description: Team GUID
          required: true
          type: string
      responses:
        '200':
          description: Autocomplete commands retrieval successful
          schema:
            type: array
            items:
              $ref: "#/definitions/Command"
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            // ListAutocompleteCommands
            listCommands, resp := Client.ListAutocompleteCommands(<TEAMID>)

  '/commands/{command_id}':
    put:
      tags:
        - commands
      summary: Update a command
      description: |
        Update a single command based on command id string and Command struct.
        ##### Permissions
        Must have `manage_slash_commands` permission for the team the command is in.
      parameters:
        - in: path
          name: command_id
          description: ID of the command to update
          required: true
          type: string
        - in: body
          name: body
          required: true
          schema:
            $ref: '#/definitions/Command'
      responses:
        '200':
          description: Command updated successful
          schema:
            $ref: "#/definitions/Command"
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            cmdToUpdate := &model.Command{
              CreatorId: <USERID>,
              TeamId:    <TEAMID>,
              URL:       "<http://nowhere.com/change>",
              Trigger:   <NEWTRIGGERNAME>,
              Id:        <COMMANDID>,
            }

            // UpdateCommand
            listCommands, resp := Client.UpdateCommand(cmdToUpdate)

    delete:
      tags:
        - commands
      summary: Delete a command
      description: |
        Delete a command based on command id string.
        ##### Permissions
        Must have `manage_slash_commands` permission for the team the command is in.
      parameters:
        - in: path
          name: command_id
          description: ID of the command to delete
          required: true
          type: string
      responses:
        '200':
          description: Command deletion successful
          schema:
            $ref: '#/definitions/StatusOK'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '404':
          $ref: '#/responses/NotFound'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            // DeleteCommand
            ok, resp := Client.DeleteCommand(<COMMANDID>)

  '/commands/{command_id}/regen_token':
    put:
      tags:
        - commands
      summary: Generate a new token
      description: |
        Generate a new token for the command based on command id string.
        ##### Permissions
        Must have `manage_slash_commands` permission for the team the command is in.
      parameters:
        - in: path
          name: command_id
          description: ID of the command to generate the new token
          required: true
          type: string
      responses:
        '200':
          description: Token generation successful
          schema:
            type: object
            properties:
              token:
                description: The new token
                type: string
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            // RegenCommandToken
            newToken, resp := Client.RegenCommandToken(<COMMANDID>)

  /commands/execute:
    post:
      tags:
        - commands
      summary: Execute a command
      description: |
        Execute a command on a team.
        ##### Permissions
        Must have `use_slash_commands` permission for the team the command is in.
      parameters:
        - in: body
          name: body
          description: command to be executed
          required: true
          schema:
            type: object
            required:
              - channel_id
              - command
            properties:
              channel_id:
                type: string
                description: Channel Id where the command will execute
              command:
                type: string
                description: The slash command to execute
      responses:
        '200':
          description: Command execution successful
          schema:
            $ref: "#/definitions/CommandResponse"
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '501':
          $ref: '#/responses/NotImplemented'
  /oauth/apps:
    post:
      tags:
        - OAuth
      summary: Register OAuth app
      description: |
        Register an OAuth 2.0 client application with Mattermost as the service provider.
        ##### Permissions
        Must have `manage_oauth` permission.
      parameters:
        - in: body
          name: body
          description: OAuth application to register
          required: true
          schema:
            type: object
            required:
              - name
              - description
              - callback_urls
              - homepage
            properties:
              name:
                type: string
                description: The name of the client application
              description:
                type: string
                description: A short description of the application
              icon_url:
                type: string
                description: A URL to an icon to display with the application
              callback_urls:
                type: array
                items:
                  type: string
                description: A list of callback URLs for the appliation
              homepage:
                type: string
                description: A link to the website of the application
              is_trusted:
                type: boolean
                description: Set this to `true` to skip asking users for permission
      responses:
        '201':
          description: App registration successful
          schema:
            $ref: '#/definitions/OAuthApp'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '501':
          $ref: '#/responses/NotImplemented'

    get:
      tags:
        - OAuth
      summary: Get OAuth apps
      description: |
        Get a page of OAuth 2.0 client applications registered with Mattermost.
        ##### Permissions
        With `manage_oauth` permission, the apps registered by the logged in user are returned. With `manage_system_wide_oauth` permission, all apps regardless of creator are returned.
      parameters:
        - name: page
          in: query
          description: The page to select.
          default: "0"
          type: string
        - name: per_page
          in: query
          description: The number of apps per page.
          default: "60"
          type: string
      responses:
        '200':
          description: OAuthApp list retrieval successful
          schema:
            type: array
            items:
              $ref: '#/definitions/OAuthApp'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '501':
          $ref: '#/responses/NotImplemented'

  '/oauth/apps/{app_id}':
    get:
      tags:
        - OAuth
      summary: Get an OAuth app
      description: |
        Get an OAuth 2.0 client application registered with Mattermost.
        ##### Permissions
        If app creator, must have `mange_oauth` permission otherwise `manage_system_wide_oauth` permission is required.
      parameters:
        - name: app_id
          in: path
          description: Application client id
          required: true
          type: string
      responses:
        '200':
          description: App retrieval successful
          schema:
            $ref: '#/definitions/OAuthApp'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '404':
          $ref: '#/responses/NotFound'
        '501':
          $ref: '#/responses/NotImplemented'

    put:
      tags:
        - OAuth
      summary: Update an OAuth app
      description: |
        Update an OAuth 2.0 client application based on OAuth struct.
        ##### Permissions
        If app creator, must have `mange_oauth` permission otherwise `manage_system_wide_oauth` permission is required.
      parameters:
        - name: app_id
          in: path
          description: Application client id
          required: true
          type: string
        - in: body
          name: body
          description: OAuth application to update
          required: true
          schema:
            type: object
            required:
              - id
              - name
              - description
              - callback_urls
              - homepage
            properties:
              id:
                type: string
                description: The id of the client application
              name:
                type: string
                description: The name of the client application
              description:
                type: string
                description: A short description of the application
              icon_url:
                type: string
                description: A URL to an icon to display with the application
              callback_urls:
                type: array
                items:
                  type: string
                description: A list of callback URLs for the appliation
              homepage:
                type: string
                description: A link to the website of the application
              is_trusted:
                type: boolean
                description: Set this to `true` to skip asking users for permission. It will be set to false if value is not provided.
      responses:
        '200':
          description: App update successful
          schema:
            $ref: '#/definitions/OAuthApp'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '404':
          $ref: '#/responses/NotFound'
        '501':
          $ref: '#/responses/NotImplemented'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            appToUpdate := &model.OAuthApp{
              Id:           <APP ID>,
              Name:         <APP NAME>,
              Description:  <APP DESCRIPTION>,
              IconURL:      <URL TO APP ICON>,
              CallbackUrls: [<CALLBACK URL1>, <CALLBACK URL2>],
              Homepage:     <URL TO APP HOMEPAGE>,
              IsTrusted:    <BOOLEAN>
            }

            // UpdateOAuthApp
            updatedApp, resp := Client.UpdateOAuthApp(appToUpdate)

    delete:
      tags:
        - OAuth
      summary: Delete an OAuth app
      description: |
        Delete and unregister an OAuth 2.0 client application 
        ##### Permissions
        If app creator, must have `mange_oauth` permission otherwise `manage_system_wide_oauth` permission is required.
      parameters:
        - name: app_id
          in: path
          description: Application client id
          required: true
          type: string
      responses:
        '200':
          description: App deletion successful
          schema:
            $ref: '#/definitions/StatusOK'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '404':
          $ref: '#/responses/NotFound'
        '501':
          $ref: '#/responses/NotImplemented'

  '/oauth/apps/{app_id}/regen_secret':
    post:
      tags:
        - OAuth
      summary: Regenerate OAuth app secret
      description: |
        Regenerate the client secret for an OAuth 2.0 client application registered with Mattermost.
        ##### Permissions
        If app creator, must have `mange_oauth` permission otherwise `manage_system_wide_oauth` permission is required.
      parameters:
        - name: app_id
          in: path
          description: Application client id
          required: true
          type: string
      responses:
        '200':
          description: Secret regeneration successful
          schema:
            $ref: '#/definitions/OAuthApp'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '404':
          $ref: '#/responses/NotFound'
        '501':
          $ref: '#/responses/NotImplemented'

  '/oauth/apps/{app_id}/info':
    get:
      tags:
        - OAuth
      summary: Get info on an OAuth app
      description: |
        Get public information about an OAuth 2.0 client application registered with Mattermost. The application's client secret will be blanked out.
        ##### Permissions
        Must be authenticated.
      parameters:
        - name: app_id
          in: path
          description: Application client id
          required: true
          type: string
      responses:
        '200':
          description: App retrieval successful
          schema:
            $ref: '#/definitions/OAuthApp'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '404':
          $ref: '#/responses/NotFound'
        '501':
          $ref: '#/responses/NotImplemented'

  '/users/{user_id}/oauth/apps/authorized':
    get:
      tags:
        - OAuth
      summary: Get authorized OAuth apps
      description: |
        Get a page of OAuth 2.0 client applications authorized to access a user's account.
        ##### Permissions
        Must be authenticated as the user or have `edit_other_users` permission.
      parameters:
        - name: user_id
          in: path
          description: User GUID
          required: true
          type: string
        - name: page
          in: query
          description: The page to select.
          default: "0"
          type: string
        - name: per_page
          in: query
          description: The number of apps per page.
          default: "60"
          type: string
      responses:
        '200':
          description: OAuthApp list retrieval successful
          schema:
            type: array
            items:
              $ref: '#/definitions/OAuthApp'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '501':
          $ref: '#/responses/NotImplemented'

  /elasticsearch/test:
    post:
      tags:
        - elasticsearch
      summary: Test Elasticsearch configuration
      description: |
        Test the current Elasticsearch configuration to see if the Elasticsearch server can be contacted successfully.
        Optionally provide a configuration in the request body to test. If no valid configuration is present in the
        request body the current server configuration will be tested.

        __Minimum server version__: 4.1
        ##### Permissions
        Must have `manage_system` permission.
      responses:
        '200':
          description: Elasticsearch test successful
          schema:
            $ref: '#/definitions/StatusOK'
        '400':
            $ref: '#/responses/BadRequest'
        '500':
          $ref: '#/responses/InternalServerError'
        '501':
          $ref: '#/responses/NotImplemented'

  /elasticsearch/purge_indexes:
    post:
      tags:
        - elasticsearch
      summary: Purge all Elasticsearch indexes
      description: |
        Deletes all Elasticsearch indexes and their contents. After calling this endpoint, it is
        necessary to schedule a new Elasticsearch indexing job to repopulate the indexes.
        __Minimum server version__: 4.1
        ##### Permissions
        Must have `manage_system` permission.
      responses:
        '200':
          description: Indexes purged successfully.
          schema:
            $ref: '#/definitions/StatusOK'
        '500':
          $ref: '#/responses/InternalServerError'
        '501':
          $ref: '#/responses/NotImplemented'
  /data_retention/policy:
    get:
      tags:
        - dataretention
      summary: Get the data retention policy details.
      description: |
        Gets the current data retention policy details from the server, including what data should be purged and the cutoff times for each data type that should be purged.
        __Minimum server version__: 4.3
        ##### Permissions
        Requires an active session but no other permissions.
      responses:
        '200':
          description: Data retention policy details retrieved successfully.
          schema:
            $ref: '#/definitions/DataRetentionPolicy'
        '500':
          $ref: '#/responses/InternalServerError'
        '501':
          $ref: '#/responses/NotImplemented'
  /plugins:
    post:
      tags:
        - plugins
      summary: Upload plugin
      description: |
        Upload a plugin that is contained within a compressed .tar.gz file. Plugins and plugin uploads must be enabled in the server's config settings.

        ##### Permissions
        Must have `manage_system` permission.

        __Minimum server version__: 4.4
      consumes: ["multipart/form-data"]
      parameters:
        - name: plugin
          in: formData
          description: The plugin image to be uploaded
          required: true
          type: file
        - name: force
          in: formData
          description: "Set to 'true' to overwrite a previously installed plugin with the same ID, if any"
          required: false
          type: string
      responses:
        '201':
          description: Plugin upload successful
          schema:
            $ref: '#/definitions/StatusOK'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '413':
          $ref: '#/responses/TooLarge'
        '501':
          $ref: '#/responses/NotImplemented'
      x-code-samples:
        - lang: 'Go'
          source: |
            import (
              "bytes"
              "io/ioutil"
              "log"

              "github.com/mattermost/mattermost-server/model"
            )

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            tarData, err := ioutil.ReadFile("plugin.tar.gz")
            if err != nil {
              log.Fatal("error while reading file")
            }

            // Not forced
            manifest, resp := Client.UploadPlugin(bytes.NewReader(tarData))

            // Forced
            manifest, resp := Client.UploadPluginForced(bytes.NewReader(tarData))
    get:
      tags:
        - plugins
      summary: Get plugins
      description: |
        Get a list of inactive and a list of active plugin manifests. Plugins must be enabled in the server's config settings.

        ##### Permissions
        Must have `manage_system` permission.

        __Minimum server version__: 4.4
      responses:
        '200':
          description: Plugins retrieval successful
          schema:
            type: object
            properties:
              active:
                type: array
                items:
                  $ref: '#/definitions/PluginManifest'
              inactive:
                type: array
                items:
                  $ref: '#/definitions/PluginManifest'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '501':
          $ref: '#/responses/NotImplemented'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            pluginsResp, resp := Client.GetPlugins()

  /plugins/install_from_url:
    post:
      tags:
        - plugins
      summary: Install plugin from url
      description: |
        Supply a URL to a plugin compressed in a .tar.gz file. Plugins must be enabled in the server's config settings.

        ##### Permissions
        Must have `manage_system` permission.

        __Minimum server version__: 5.14
      parameters:
        - name: plugin_download_url
          in: query
          description: URL used to download the plugin
          required: true
          type: string
        - name: force
          in: query
          description: "Set to 'true' to overwrite a previously installed plugin with the same ID, if any"
          required: false
          type: string
      responses:
        '201':
          description: Plugin install successful
          schema:
            $ref: '#/definitions/StatusOK'
        '400':
          $ref: '#/responses/BadRequest'
        '403':
          $ref: '#/responses/Forbidden'
        '501':
          $ref: '#/responses/NotImplemented'
      x-code-samples:
        - lang: 'Go'
          source: |
            import (
              "github.com/mattermost/mattermost-server/model"
            )

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            url := "https://mysite.com/my-plugin.tar.gz"

            // Not forced
            manifest, resp := Client.InstallPluginFromUrl(url, false)

            // Forced
            manifest, resp := Client.InstallPluginFromUrl(url, true)

  /plugins/{plugin_id}:
    delete:
      tags:
        - plugins
      summary: Remove plugin
      description: |
        Remove the plugin with the provided ID from the server. All plugin files are deleted. Plugins must be enabled in the server's config settings.

        ##### Permissions
        Must have `manage_system` permission.

        __Minimum server version__: 4.4
      parameters:
        - name: plugin_id
          in: path
          required: true
          type: string
      responses:
        '200':
            description: Plugin removed successfully
            schema:
              $ref: '#/definitions/StatusOK'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '501':
          $ref: '#/responses/NotImplemented'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            pluginID := "com.mattermost.demo-plugin"

            ok, resp = Client.RemovePlugin(pluginID)

  /plugins/{plugin_id}/enable:
    post:
      tags:
        - plugins
      summary: Enable plugin
      description: |
        Enable a previously uploaded plugin. Plugins must be enabled in the server's config settings.

        ##### Permissions
        Must have `manage_system` permission.

        __Minimum server version__: 4.4
      parameters:
        - name: plugin_id
          in: path
          required: true
          type: string
      responses:
        '200':
            description: Plugin enabled successfully
            schema:
              $ref: '#/definitions/StatusOK'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '501':
          $ref: '#/responses/NotImplemented'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            pluginID := "com.mattermost.demo-plugin"

            ok, resp = Client.EnablePlugin(pluginID)

  /plugins/{plugin_id}/disable:
    post:
      tags:
        - plugins
      summary: Disable plugin
      description: |
        Disable a previously enabled plugin. Plugins must be enabled in the server's config settings.

        ##### Permissions
        Must have `manage_system` permission.

        __Minimum server version__: 4.4
      parameters:
        - name: plugin_id
          in: path
          required: true
          type: string
      responses:
        '200':
            description: Plugin disabled successfully
            schema:
              $ref: '#/definitions/StatusOK'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '501':
          $ref: '#/responses/NotImplemented'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            pluginID := "com.mattermost.demo-plugin"

            ok, resp = Client.DisablePlugin(pluginID)

  /plugins/webapp:
    get:
      tags:
        - plugins
      summary: Get webapp plugins
      description: |
        Get a list of web app plugins installed and activated on the server.

        ##### Permissions
        No permissions required.

        __Minimum server version__: 4.4
      responses:
        '200':
            description: Plugin deactivated successfully
            schema:
              type: array
              items:
                $ref: '#/definitions/PluginManifestWebapp'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '501':
          $ref: '#/responses/NotImplemented'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")

            manifests, resp := Client.GetWebappPlugins()
  '/roles/{role_id}':
    get:
      tags:
        - roles
      summary: Get a role
      description: |
        Get a role from the provided role id.

        ##### Permissions
        Requires an active session but no other permissions.

        __Minimum server version__: 4.9
      parameters:
        - name: role_id
          in: path
          description: Role GUID
          required: true
          type: string
      responses:
        '200':
          description: Role retrieval successful
          schema:
            $ref: '#/definitions/Role'
        '401':
          $ref: '#/responses/Unauthorized'
        '404':
          $ref: '#/responses/NotFound'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"
            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            role, resp := Client.GetRole(<ROLEID>, "")

  '/roles/name/{role_name}':
    get:
      tags:
        - roles
      summary: Get a role
      description: |
        Get a role from the provided role name.

        ##### Permissions
        Requires an active session but no other permissions.

        __Minimum server version__: 4.9
      parameters:
        - name: role_name
          in: path
          description: Role Name
          required: true
          type: string
      responses:
        '200':
          description: Role retrieval successful
          schema:
            $ref: '#/definitions/Role'
        '401':
          $ref: '#/responses/Unauthorized'
        '404':
          $ref: '#/responses/NotFound'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"
            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            role, resp := Client.GetRoleByName(<ROLENAME>, "")

  '/roles/{role_id}/patch':
    put:
      tags:
        - roles
      summary: Patch a role
      description: |
        Partially update a role by providing only the fields you want to update. Omitted fields will not be updated. The fields that can be updated are defined in the request body, all other provided fields will be ignored.

        ##### Permissions
        `manage_system` permission is required.

        __Minimum server version__: 4.9
      parameters:
        - name: role_id
          in: path
          description: Role GUID
          required: true
          type: string
        - in: body
          name: body
          description: Role object to be updated
          required: true
          schema:
            type: object
            properties:
              permissions:
                type: array
                items:
                  type: string
                description: The permissions the role should grant.
      responses:
        '200':
          description: Role patch successful
          schema:
            $ref: '#/definitions/Role'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '404':
          $ref: '#/responses/NotFound'

  /roles/names:
      post:
        tags:
          - roles
        summary: Get a list of roles by name
        description: |
          Get a list of roles from their names.

          ##### Permissions
          Requires an active session but no other permissions.

          __Minimum server version__: 4.9
        parameters:
          - in: body
            name: body
            description: List of role names
            required: true
            schema:
              type: array
              items:
                type: string
        responses:
          '200':
            description: Role list retrieval successful
            schema:
              type: array
              items:
                $ref: '#/definitions/Role'
          '400':
            $ref: '#/responses/BadRequest'
          '401':
            $ref: '#/responses/Unauthorized'
          '404':
            $ref: '#/responses/NotFound'
        x-code-samples:
          - lang: 'Go'
            source: |
              import "github.com/mattermost/mattermost-server/model"
              Client := model.NewAPIv4Client("https://your-mattermost-url.com")
              Client.Login("email@domain.com", "Password1")

              roleNames := []string{<NAME OF ROLE1>, <NAME OF ROLE2>, ...}

              roles, resp := Client.GetRolesByNames(roleNames)
  '/schemes':
    get:
      tags:
        - schemes
      summary: Get the schemes.
      description: |
        Get a page of schemes. Use the query parameters to modify the behaviour of this endpoint.

        ##### Permissions
        Must have `manage_system` permission.

        __Minimum server version__: 5.0
      parameters:
        - name: scope
          in: query
          description: Limit the results returned to the provided scope, either `team` or `channel`.
          default: ""
          type: string
        - name: page
          in: query
          description: The page to select.
          default: "0"
          type: string
        - name: per_page
          in: query
          description: The number of schemes per page.
          default: "60"
          type: string
      responses:
        '200':
          description: Scheme list retrieval successful
          schema:
            type: array
            items:
              $ref: '#/definitions/Scheme'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'

    post:
      tags:
        - schemes
      summary: Create a scheme
      description: |
        Create a new scheme.

        ##### Permissions
        Must have `manage_system` permission.

        __Minimum server version__: 5.0
      parameters:
        - in: body
          name: scheme
          description: Scheme object to create
          required: true
          schema:
            type: object
            required:
              - name
              - scope
            properties:
              name:
                type: string
                description: The name of the scheme
              description:
                type: string
                description: The description of the scheme
              scope:
                type: string
                description: The scope of the scheme ("team" or "channel")
      responses:
        '201':
          description: Scheme creation successful
          schema:
            $ref: '#/definitions/Scheme'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '501':
          $ref: '#/responses/NotImplemented'

  '/schemes/{scheme_id}':
    get:
      tags:
        - schemes
      summary: Get a scheme
      description: |
        Get a scheme from the provided scheme id.

        ##### Permissions
        Must have `manage_system` permission.

        __Minimum server version__: 5.0
      parameters:
        - name: scheme_id
          in: path
          description: Scheme GUID
          required: true
          type: string
      responses:
        '200':
          description: Scheme retrieval successful
          schema:
            $ref: '#/definitions/Scheme'
        '401':
          $ref: '#/responses/Unauthorized'
        '404':
          $ref: '#/responses/NotFound'
        '501':
          $ref: '#/responses/NotImplemented'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"
            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            scheme, resp := Client.GetScheme(<SCHEMEID>, "")

    delete:
      tags:
        - schemes
      summary: Delete a scheme
      description: |
        Soft deletes a scheme, by marking the scheme as deleted in the database.

        ##### Permissions
        Must have `manage_system` permission.

        __Minimum server version__: 5.0
      parameters:
        - name: scheme_id
          in: path
          description: ID of the scheme to delete
          required: true
          type: string
      responses:
        '200':
          description: Scheme deletion successful
          schema:
            $ref: "#/definitions/StatusOK"
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '501':
          $ref: '#/responses/NotImplemented'

  '/schemes/{scheme_id}/patch':
    put:
      tags:
        - schemes
      summary: Patch a scheme
      description: |
        Partially update a scheme by providing only the fields you want to update. Omitted fields will not be updated. The fields that can be updated are defined in the request body, all other provided fields will be ignored.

        ##### Permissions
        `manage_system` permission is required.

        __Minimum server version__: 5.0
      parameters:
        - name: scheme_id
          in: path
          description: Scheme GUID
          required: true
          type: string
        - in: body
          name: body
          description: Scheme object to be updated
          required: true
          schema:
            type: object
            properties:
              name:
                type: string
                description: The human readable name of the scheme
              description:
                type: string
                description: The description of the scheme
      responses:
        '200':
          description: Scheme patch successful
          schema:
            $ref: '#/definitions/Scheme'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '404':
          $ref: '#/responses/NotFound'
        '501':
          $ref: '#/responses/NotImplemented'

  '/schemes/{scheme_id}/teams':
     get:
       tags:
         - schemes
       summary: Get a page of teams which use this scheme.
       description: |
         Get a page of teams which use this scheme. The provided Scheme ID should be for a Team-scoped Scheme.
         Use the query parameters to modify the behaviour of this endpoint.

         ##### Permissions
         `manage_system` permission is required.

         __Minimum server version__: 5.0
       parameters:
         - name: scheme_id
           in: path
           description: Scheme GUID
           required: true
           type: string
         - name: page
           in: query
           description: The page to select.
           default: "0"
           type: string
         - name: per_page
           in: query
           description: The number of teams per page.
           default: "60"
           type: string
       responses:
         '200':
           description: Team list retrieval successful
           schema:
             type: array
             items:
               $ref: '#/definitions/Team'
         '400':
           $ref: '#/responses/BadRequest'
         '401':
           $ref: '#/responses/Unauthorized'
         '403':
           $ref: '#/responses/Forbidden'
         '404':
           $ref: '#/responses/NotFound'

  '/schemes/{scheme_id}/channels':
      get:
        tags:
          - schemes
        summary: Get a page of channels which use this scheme.
        description: |
          Get a page of channels which use this scheme. The provided Scheme ID should be for a Channel-scoped Scheme.
          Use the query parameters to modify the behaviour of this endpoint.

          ##### Permissions
          `manage_system` permission is required.

          __Minimum server version__: 5.0
        parameters:
          - name: scheme_id
            in: path
            description: Scheme GUID
            required: true
            type: string
          - name: page
            in: query
            description: The page to select.
            default: "0"
            type: string
          - name: per_page
            in: query
            description: The number of channels per page.
            default: "60"
            type: string
        responses:
          '200':
            description: Channel list retrieval successful
            schema:
              type: array
              items:
                $ref: '#/definitions/Channel'
          '400':
            $ref: '#/responses/BadRequest'
          '401':
            $ref: '#/responses/Unauthorized'
          '403':
            $ref: '#/responses/Forbidden'
          '404':
            $ref: '#/responses/NotFound'
  /opengraph:
    post:
      tags:
        - OpenGraph
      summary: Get open graph metadata for url
      description: |
        Get Open Graph Metadata for a specif URL. Use the Open Graph protocol to get some generic metadata about a URL. Used for creating link previews.

        __Minimum server version__: 3.10

        ##### Permissions
        No permission required but must be logged in.
      parameters:
        - name: body
          in: body
          required: true
          schema:
            type: object
            required:
              - url
            properties:
              url:
                type: string
                description: The URL to get Open Graph Metadata.
      responses:
        '200':
          description: Open Graph retrieval successful
          schema:
            $ref: '#/definitions/OpenGraph'
        '501':
          $ref: '#/responses/NotImplemented'
  /reactions:
    post:
      tags:
        - reactions
      summary: Create a reaction
      description: |
        Create a reaction.
        ##### Permissions
        Must have `read_channel` permission for the channel the post is in.
      parameters:
        - in: body
          name: reaction
          description: The user's reaction with its post_id, user_id, and emoji_name fields set
          required: true
          schema:
            $ref: '#/definitions/Reaction'
      responses:
        '201':
          description: Reaction creation successful
          schema:
            $ref: '#/definitions/Reaction'
        '400':
          $ref: '#/responses/BadRequest'
        '403':
          $ref: '#/responses/Forbidden'

  '/posts/{post_id}/reactions':
    get:
      tags:
        - reactions
      summary: Get a list of reactions to a post
      description: |
        Get a list of reactions made by all users to a given post.
        ##### Permissions
        Must have `read_channel` permission for the channel the post is in.
      parameters:
        - name: post_id
          in: path
          description: ID of a post
          required: true
          type: string
      responses:
        '200':
          description: List reactions retrieve successful
          schema:
            type: array
            items:
              $ref: "#/definitions/Reaction"
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'

  '/users/{user_id}/posts/{post_id}/reactions/{emoji_name}':
    delete:
      tags:
        - reactions
      summary: Remove a reaction from a post
      description: |
        Deletes a reaction made by a user from the given post.
        ##### Permissions
        Must be user or have `manage_system` permission.
      parameters:
        - name: user_id
          in: path
          description: ID of the user
          required: true
          type: string
        - name: post_id
          in: path
          description: ID of the post
          required: true
          type: string
        - name: emoji_name
          in: path
          description: emoji name
          required: true
          type: string
      responses:
        '200':
          description: Reaction deletion successful
          schema:
            $ref: "#/definitions/StatusOK"
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'

  '/posts/ids/reactions':
    post:
      tags:
        - reactions
      summary: Bulk get the reaction for posts
      description: |
        Get a list of reactions made by all users to a given post.
        ##### Permissions
        Must have `read_channel` permission for the channel the post is in.

        __Minimum server version__: 5.8
      parameters:
        - in: body
          name: postIds
          description: Array of post IDs
          required: true
          schema:
            type: array
            items:
              type: string
      responses:
        '200':
          description: Reactions retrieval successful
          schema:
            $ref: "#/definitions/PostIdToReactionsMap"
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
  '/actions/dialogs/open':
    post:
      tags:
        - integration_actions
      summary: Open a dialog
      description: |
        Open an interactive dialog using a trigger ID provided by a slash command, or some other action payload. See https://docs.mattermost.com/developer/interactive-dialogs.html for more information on interactive dialogs.
        __Minimum server version: 5.6__
      parameters:
        - name: body
          in: body
          description: Metadata for the dialog to be opened
          required: true
          schema:
            type: object
            required:
              - trigger_id
              - url
              - dialog
            properties:
              trigger_id:
                type: string
                description: Trigger ID provided by other action
              url:
                type: string
                description: The URL to send the submitted dialog payload to
              dialog:
                type: object
                required:
                  - title
                  - elements
                description: Post object to create
                properties:
                  callback_id:
                    type: string
                    description: Set an ID that will be included when the dialog is submitted
                  title:
                    type: string
                    description: Title of the dialog
                  elements:
                    type: array
                    description: Input elements, see https://docs.mattermost.com/developer/interactive-dialogs.html#elements
                    items:
                      type: object
                  submit_label:
                    type: string
                    description: Label on the submit button
                  notify_on_cancel:
                    type: boolean
                    description: Set true to receive payloads when user cancels a dialog
                  state:
                    type: string
                    description: Set some state to be echoed back with the dialog submission
      responses:
        '200':
          description: Dialog open successful
          schema:
            $ref: '#/definitions/StatusOK'
        '400':
          $ref: '#/responses/BadRequest'

  '/actions/dialogs/submit':
    post:
      tags:
        - integration_actions
      summary: Submit a dialog
      description: |
        Endpoint used by the Mattermost clients to submit a dialog. See https://docs.mattermost.com/developer/interactive-dialogs.html for more information on interactive dialogs.
        __Minimum server version: 5.6__
      parameters:
        - name: body
          in: body
          description: Dialog submission data
          required: true
          schema:
            type: object
            required:
              - url
              - submission
              - channel_id
              - team_id
            properties:
              url:
                type: string
                description: The URL to send the submitted dialog payload to
              channel_id:
                type: string
                description: Channel ID the user submitted the dialog from
              team_id:
                type: string
                description: Team ID the user submitted the dialog from
              submission:
                type: object
                description: String map where keys are element names and values are the element input values
              callback_id:
                type: string
                description: Callback ID sent when the dialog was opened
              state:
                type: string
                description: State sent when the dialog was opened
              cancelled:
                type: boolean
                description: Set to true if the dialog was cancelled
                
      responses:
        '200':
          description: Dialog submission successful
          schema:
            $ref: '#/definitions/StatusOK'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'

  /bots:
    post:
      tags:
        - bots
      summary: Create a bot
      description: |
        Create a new bot account on the system. Username is required.
        ##### Permissions
        Must have `create_bot` permission.
        __Minimum server version__: 5.10
      parameters:
        - in: body
          name: body
          description: Bot to be created
          required: true
          schema:
            type: object
            required:
              - username
            properties:
              username:
                type: string
              display_name:
                type: string
              description:
                type: string
      responses:
        '201':
          description: Bot creation successful
          schema:
            $ref: '#/definitions/Bot'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
    get:
      tags:
        - bots
      summary: Get bots
      description: |
        Get a page of a list of bots.
        ##### Permissions
        Must have `read_bots` permission for bots you are managing, and `read_others_bots` permission for bots others are managing.
        __Minimum server version__: 5.10
      parameters:
        - name: page
          in: query
          description: The page to select.
          default: "0"
          type: string
        - name: per_page
          in: query
          description: The number of users per page. There is a maximum limit of 200 users per page.
          default: "60"
          type: string
        - name: include_deleted
          in: query
          description: If deleted bots should be returned.
          type: boolean
        - name: only_orphaned
          in: query
          description: When true, only orphaned bots will be returned. A bot is consitered orphaned if it's owner has been deactivated.
          type: boolean
      responses:
        '200':
          description: Bot page retrieval successful
          schema:
            type: array
            items:
              $ref: '#/definitions/Bot'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'

  /bots/{bot_user_id}:
    put:
      tags:
        - bots
      summary: Patch a bot
      description: |
        Partially update a bot by providing only the fields you want to update. Omitted fields will not be updated. The fields that can be updated are defined in the request body, all other provided fields will be ignored.
        ##### Permissions
        Must have `manage_bots` permission. 
        __Minimum server version__: 5.10
      parameters:
        - name: bot_user_id
          in: path
          description: Bot user ID
          required: true
          type: string
        - in: body
          name: body
          description: Bot to be created
          required: true
          schema:
            type: object
            required:
              - username
            properties:
              username:
                type: string
              display_name:
                type: string
              description:
                type: string
      responses:
        '200':
          description: Bot patch successful
          schema:
            $ref: '#/definitions/Bot'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
    get:
      tags:
        - bots
      summary: Get a bot
      description: |
        Get a bot specified by its bot id.
        ##### Permissions
        Must have `read_bots` permission for bots you are managing, and `read_others_bots` permission for bots others are managing.
        __Minimum server version__: 5.10
      parameters:
        - name: bot_user_id
          in: path
          description: Bot user ID
          required: true
          type: string
        - name: include_deleted
          type: boolean
          in: query
          description: If deleted bots should be returned.
      responses:
        '200':
          description: Bot successfully retrieved.
          schema:
            $ref: '#/definitions/Bot'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'

  /bots/{bot_user_id}/disable:
    post:
      tags:
        - bots
      summary: Disable a bot
      description: |
        Disable a bot.
        ##### Permissions
        Must have `manage_bots` permission. 
        __Minimum server version__: 5.10
      parameters:
        - name: bot_user_id
          in: path
          description: Bot user ID
          required: true
          type: string
      responses:
        '200':
          description: Bot successfully disabled.
          schema:
            $ref: '#/definitions/Bot'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'

  /bots/{bot_user_id}/enable:
    post:
      tags:
        - bots
      summary: Enable a bot
      description: |
        Enable a bot.
        ##### Permissions
        Must have `manage_bots` permission. 
        __Minimum server version__: 5.10
      parameters:
        - name: bot_user_id
          in: path
          description: Bot user ID
          required: true
          type: string
      responses:
        '200':
          description: Bot successfully enabled.
          schema:
            $ref: '#/definitions/Bot'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'

  /bots/{bot_user_id}/assign/{user_id}:
    post:
      tags:
        - bots
      summary: Assign a bot to a user
      description: |
        Assign a bot to a specified user.
        ##### Permissions
        Must have `manage_bots` permission. 
        __Minimum server version__: 5.10
      parameters:
        - name: bot_user_id
          in: path
          description: Bot user ID
          required: true
          type: string
        - name: user_id
          in: path
          description: The user ID to assign the bot to.
          required: true
          type: string
      responses:
        '200':
          description: Bot successfully assigned.
          schema:
            $ref: '#/definitions/Bot'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'

  /bots/{bot_user_id}/icon:
    get:
      tags:
        - bots
      summary: Get bot's LHS icon
      description: |
        Get a bot's LHS icon image based on bot_user_id string parameter.
        ##### Permissions
        Must be logged in.
        __Minimum server version__: 5.14
      parameters:
        - name: bot_user_id
          in: path
          description: Bot user ID
          required: true
          type: string
      responses:
        '200':
          description: Bot's LHS icon image
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '404':
          $ref: '#/responses/NotFound'
        '500':
          $ref: '#/responses/InternalServerError'
        '501':
          $ref: '#/responses/NotImplemented'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            botUserID := "4xp9fdt77pncbef59f4k1qe83o"

            data, resp := Client.GetBotIconImage(botUserID)
    post:
      tags:
        - bots
      summary: Set bot's LHS icon image
      description: |
        Set a bot's LHS icon image based on bot_user_id string parameter. Icon image must be SVG format, all other formats are rejected.
        ##### Permissions
        Must have `manage_bots` permission.
        __Minimum server version__: 5.14
      consumes: ["multipart/form-data"]
      parameters:
        - name: image
          in: formData
          description: SVG icon image to be uploaded
          required: true
          type: file
        - name: bot_user_id
          in: path
          description: Bot user ID
          required: true
          type: string
      responses:
        '200':
          description: SVG icon image set successful
          schema:
            $ref: '#/definitions/StatusOK'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '413':
          $ref: '#/responses/TooLarge'
        '500':
          $ref: '#/responses/InternalServerError'
        '501':
          $ref: '#/responses/NotImplemented'
      x-code-samples:
        - lang: 'Go'
          source: |
            import (
              "io/ioutil"
              "log"

              "github.com/mattermost/mattermost-server/model"
            )

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            data, err := ioutil.ReadFile("icon_image.svg")
            if err != nil {
              log.Fatal(err)
            }

            botUserID := "4xp9fdt77pncbef59f4k1qe83o"

            ok, resp := Client.SetBotIconImage(botUserID, data)
    delete:
      tags:
        - bots
      summary: Delete bot's LHS icon image
      description: |
        Delete bot's LHS icon image based on bot_user_id string parameter.
        ##### Permissions
        Must have `manage_bots` permission.
        __Minimum server version__: 5.14
      parameters:
        - name: bot_user_id
          in: path
          description: Bot user ID
          required: true
          type: string
      responses:
        '200':
          description: Icon image deletion successful
          schema:
            $ref: '#/definitions/StatusOK'
        '400':
          $ref: '#/responses/BadRequest'
        '401':
          $ref: '#/responses/Unauthorized'
        '403':
          $ref: '#/responses/Forbidden'
        '404':
          $ref: '#/responses/NotFound'
        '500':
          $ref: '#/responses/InternalServerError'
        '501':
          $ref: '#/responses/NotImplemented'
      x-code-samples:
        - lang: 'Go'
          source: |
            import "github.com/mattermost/mattermost-server/model"

            Client := model.NewAPIv4Client("https://your-mattermost-url.com")
            Client.Login("email@domain.com", "Password1")

            botUserID := "4xp9fdt77pncbef59f4k1qe83o"

            ok, resp := Client.DeleteBotIconImage(botUserID)
definitions:
  User:
    type: object
    properties:
      id:
        type: string
      create_at:
        description: The time in milliseconds a user was created
        type: integer
        format: int64
      update_at:
        description: The time in milliseconds a user was last updated
        type: integer
        format: int64
      delete_at:
        description: The time in milliseconds a user was deleted
        type: integer
        format: int64
      username:
        type: string
      first_name:
        type: string
      last_name:
        type: string
      nickname:
        type: string
      email:
        type: string
      email_verified:
        type: boolean
      auth_service:
        type: string
      roles:
        type: string
      locale:
        type: string
      notify_props:
        description: Field only visible to self and admins
        $ref: '#/definitions/UserNotifyProps'
      props:
        type: object
      last_password_update:
        type: integer
      last_picture_update:
        type: integer
      failed_attempts:
        type: integer
      mfa_active:
        type: boolean
      timezone:
        $ref: '#/definitions/Timezone'
      terms_of_service_id:
        description: ID of accepted terms of service, if any. This field is not present if empty.
        type: string
      terms_of_service_create_at:
        description: The time in milliseconds the user accepted the terms of service
        type: integer
        format: int64

  UsersStats:
    type: object
    properties:
      total_users_count:
        type: integer

  Team:
    type: object
    properties:
      id:
        type: string
      create_at:
        description: The time in milliseconds a team was created
        type: integer
        format: int64
      update_at:
        description: The time in milliseconds a team was last updated
        type: integer
        format: int64
      delete_at:
        description: The time in milliseconds a team was deleted
        type: integer
        format: int64
      display_name:
        type: string
      name:
        type: string
      description:
        type: string
      email:
        type: string
      type:
        type: string
      allowed_domains:
        type: string
      invite_id:
        type: string
      allow_open_invite:
        type: boolean

  TeamStats:
    type: object
    properties:
      team_id:
        type: string
      total_member_count:
        type: integer
      active_member_count:
        type: integer

  TeamExists:
    type: object
    properties:
      exists:
        type: boolean

  Channel:
    type: object
    properties:
      id:
        type: string
      create_at:
        description: The time in milliseconds a channel was created
        type: integer
        format: int64
      update_at:
        description: The time in milliseconds a channel was last updated
        type: integer
        format: int64
      delete_at:
        description: The time in milliseconds a channel was deleted
        type: integer
        format: int64
      team_id:
        type: string
      type:
        type: string
      display_name:
        type: string
      name:
        type: string
      header:
        type: string
      purpose:
        type: string
      last_post_at:
        description: The time in milliseconds of the last post of a channel
        type: integer
      total_msg_count:
        type: integer
      extra_update_at:
        description: Deprecated in Mattermost 5.0 release
        type: integer
        format: int64
      creator_id:
        type: string

  ChannelStats:
    type: object
    properties:
      channel_id:
        type: string
      member_count:
        type: integer

  ChannelMember:
    type: object
    properties:
      channel_id:
        type: string
      user_id:
        type: string
      roles:
        type: string
      last_viewed_at:
        description: The time in milliseconds the channel was last viewed by the user
        type: integer
        format: int64
      msg_count:
        type: integer
      mention_count:
        type: integer
      notify_props:
        description: Field only visible to self and admins
        $ref: '#/definitions/ChannelNotifyProps'
      last_update_at:
        description: The time in milliseconds the channel member was last updated
        type: integer
        format: int64

  ChannelData:
    type: object
    properties:
      channel:
        $ref: '#/definitions/Channel'
      member:
        $ref: '#/definitions/ChannelMember'

  Post:
    type: object
    properties:
      id:
        type: string
      create_at:
        description: The time in milliseconds a post was created
        type: integer
        format: int64
      update_at:
        description: The time in milliseconds a post was last updated
        type: integer
        format: int64
      delete_at:
        description: The time in milliseconds a post was deleted
        type: integer
        format: int64
      edit_at:
        type: integer
        format: int64
      user_id:
        type: string
      channel_id:
        type: string
      root_id:
        type: string
      parent_id:
        type: string
      original_id:
        type: string
      message:
        type: string
      type:
        type: string
      props:
        type: object
      hashtag:
        type: string
      filenames:
        description: This field will only appear on some posts created before Mattermost 3.5 and has since been deprecated.
        type: array
        items:
          type: string
      file_ids:
        type: array
        items:
          type: string
      pending_post_id:
        type: string
      metadata:
        description: |
          Additional information used to display the post. This field is only used to send information from the server
          to the client, and the server will ignore it if it receives it from a client.

          This field will only be returned by servers running Mattermost 5.6 or higher with the experimental
          EnablePostMetadata setting enabled.
        $ref: '#/definitions/PostMetadata'

  PostList:
    type: object
    properties:
      order:
        type: array
        items:
            type: string
        example: ["post_id1", "post_id12"]
      posts:
        type: object
        additionalProperties:
          $ref: '#/definitions/Post'
      next_post_id:
        type: string
        description: The ID of next post. Not omitted when empty or not relevant.
      prev_post_id:
        type: string
        description: The ID of previous post. Not omitted when empty or not relevant.

  PostListWithSearchMatches:
    type: object
    properties:
      order:
        type: array
        items:
            type: string
        example: ["post_id1", "post_id12"]
      posts:
        type: object
        additionalProperties:
          $ref: '#/definitions/Post'
      matches:
        description: A mapping of post IDs to a list of matched terms within the post. This field will only be populated on servers running version 5.1 or greater with Elasticsearch enabled.
        type: object
        additionalProperties:
          type: array
          items:
            type: string
        example: {"post_id1": ["search match 1", "search match 2"]}

  PostMetadata:
    type: object
    description: Additional information used to display a post.
    properties:
      embeds:
        type: array
        description: |
          Information about content embedded in the post including OpenGraph previews, image link previews, and
          message attachments. This field will be null if the post does not contain embedded content.
        items:
          type: object
          properties:
            type:
              type: string
              description: The type of content that is embedded in this point.
              enum: ["image", "message_attachment", "opengraph", "link"]
            url:
              type: string
              description: The URL of the embedded content, if one exists.
            data:
              type: object
              description: |
                Any additional information about the embedded content. Only used at this time to store OpenGraph metadata.
                This field will be null for non-OpenGraph embeds.
      emojis:
        type: array
        description: |
          The custom emojis that appear in this point or have been used in reactions to this post. This field will be
          null if the post does not contain custom emojis.
        items:
          $ref: '#/definitions/Emoji'
      files:
        type: array
        description: |
          The FileInfo objects for any files attached to the post. This field will be null if the post does not have
          any file attachments.
        items:
          $ref: '#/definitions/FileInfo'
      images:
        type: object
        description: |
          An object mapping the URL of an external image to an object containing the dimensions of that image. This
          field will be null if the post or its embedded content does not reference any external images.
        items:
          type: object
          properties:
            height:
              type: integer
            width:
              type: integer
      reactions:
        type: array
        description: |
          Any reactions made to this point. This field will be null if no reactions have been made to this post.
        items:
          $ref: '#/definitions/Reaction'

  TeamMap:
    type: object
    description: A mapping of teamIds to teams.
    properties:
      team_id:
        $ref: '#/definitions/Team'

  TeamMember:
    type: object
    properties:
      team_id:
        description: The ID of the team this member belongs to.
        type: string
      user_id:
        description: The ID of the user this member relates to.
        type: string
      roles:
        description: The complete list of roles assigned to this team member, as a space-separated list of role names, including any roles granted implicitly through permissions schemes.
        type: string
      delete_at:
        description: The time in milliseconds that this team member was deleted.
        type: integer
      scheme_user:
        description: Whether this team member holds the default user role defined by the team's permissions scheme.
        type: boolean
      scheme_admin:
        description: Whether this team member holds the default admin role defined by the team's permissions scheme.
        type: boolean
      explicit_roles:
        description: The list of roles explicitly assigned to this team member, as a space separated list of role names. This list does *not* include any roles granted implicitly through permissions schemes.
        type: string

  TeamUnread:
    type: object
    properties:
      team_id:
        type: string
      msg_count:
        type: integer
      mention_count:
        type: integer

  ChannelUnread:
    type: object
    properties:
      team_id:
        type: string
      channel_id:
        type: string
      msg_count:
        type: integer
      mention_count:
        type: integer

  ChannelUnreadAt:
    type: object
    properties:
      team_id:
        description: The ID of the team the channel belongs to.
        type: string
      channel_id:
        description: The ID of the channel the user has access to..
        type: string
      msg_count:
        description: No. of messages the user has already read.
        type: integer
      mention_count:
        description: No. of mentions the user has within the unread posts of the channel.
        type: integer
      last_viewed_at:
        description: time in milliseconds when the user last viewed the channel.
        type: integer

  Session:
    type: object
    properties:
      create_at:
        description: The time in milliseconds a session was created
        type: integer
        format: int64
      device_id:
        type: string
      expires_at:
        description: The time in milliseconds a session will expire
        type: integer
        format: int64
      id:
        type: string
      is_oauth:
        type: boolean
      last_activity_at:
        description: The time in milliseconds of the last activity of a session
        type: integer
        format: int64
      props:
        type: object
      roles:
        type: string
      team_members:
        type: array
        items:
          $ref: '#/definitions/TeamMember'
      token:
        type: string
      user_id:
        type: string

  FileInfo:
    type: object
    properties:
      id:
        description: The unique identifier for this file
        type: string
      user_id:
        description: The ID of the user that uploaded this file
        type: string
      post_id:
        description: If this file is attached to a post, the ID of that post
        type: string
      create_at:
        description: The time in milliseconds a file was created
        type: integer
        format: int64
      update_at:
        description: The time in milliseconds a file was last updated
        type: integer
        format: int64
      delete_at:
        description: The time in milliseconds a file was deleted
        type: integer
        format: int64
      name:
        description: The name of the file
        type: string
      extension:
        description: The extension at the end of the file name
        type: string
      size:
        description: The size of the file in bytes
        type: integer
      mime_type:
        description: The MIME type of the file
        type: string
      width:
        description: If this file is an image, the width of the file
        type: integer
      height:
        description: If this file is an image, the height of the file
        type: integer
      has_preview_image:
        description: If this file is an image, whether or not it has a preview-sized version
        type: boolean

  Preference:
    type: object
    properties:
      user_id:
        description: The ID of the user that owns this preference
        type: string
      category:
        type: string
      name:
        type: string
      value:
        type: string

  UserAuthData:
    type: object
    properties:
      auth_data:
        description: Service-specific authentication data
        type: string
      auth_service:
        description: The authentication service such as "email", "gitlab", or "ldap"
        type: string
      password:
        description: The password used for email authentication
        type: string

  UserAutocomplete:
    type: object
    properties:
      users:
        description: A list of users that are the main result of the query
        type: array
        items:
          $ref: '#/definitions/User'
      out_of_channel:
        description: A special case list of users returned when autocompleting in a specific channel. Omitted when empty or not relevant
        type: array
        items:
          $ref: '#/definitions/User'

  UserAutocompleteInTeam:
    type: object
    properties:
      in_team:
        description: A list of user objects in the team
        type: array
        items:
          $ref: '#/definitions/User'

  UserAutocompleteInChannel:
    type: object
    properties:
      in_channel:
        description: A list of user objects in the channel
        type: array
        items:
          $ref: '#/definitions/User'
      out_of_channel:
        description: A list of user objects not in the channel
        type: array
        items:
          $ref: '#/definitions/User'

  IncomingWebhook:
    type: object
    properties:
      id:
        description: The unique identifier for this incoming webhook
        type: string
      create_at:
        description: The time in milliseconds a incoming webhook was created
        type: integer
        format: int64
      update_at:
        description: The time in milliseconds a incoming webhook was last updated
        type: integer
        format: int64
      delete_at:
        description: The time in milliseconds a incoming webhook was deleted
        type: integer
        format: int64
      channel_id:
        description: The ID of a public channel or private group that receives the webhook payloads
        type: string
      description:
        description: The description for this incoming webhook
        type: string
      display_name:
        description: The display name for this incoming webhook
        type: string

  OutgoingWebhook:
    type: object
    properties:
      id:
        description: The unique identifier for this outgoing webhook
        type: string
      create_at:
        description: The time in milliseconds a outgoing webhook was created
        type: integer
        format: int64
      update_at:
        description: The time in milliseconds a outgoing webhook was last updated
        type: integer
        format: int64
      delete_at:
        description: The time in milliseconds a outgoing webhook was deleted
        type: integer
        format: int64
      creator_id:
        description: The Id of the user who created the webhook
        type: string
      team_id:
        description: The ID of the team that the webhook watchs
        type: string
      channel_id:
        description: The ID of a public channel that the webhook watchs
        type: string
      description:
        description: The description for this outgoing webhook
        type: string
      display_name:
        description: The display name for this outgoing webhook
        type: string
      trigger_words:
        description: List of words for the webhook to trigger on
        type: array
        items:
          type: string
      trigger_when:
        description: When to trigger the webhook, `0` when a trigger word is present at all and `1` if the message starts with a trigger word
        type: integer
      callback_urls:
        description: The URLs to POST the payloads to when the webhook is triggered
        type: array
        items:
          type: string
      content_type:
        description: The format to POST the data in, either `application/json` or `application/x-www-form-urlencoded`
        default: "application/x-www-form-urlencoded"
        type: string

  Reaction:
    type: object
    properties:
      user_id:
        description: The ID of the user that made this reaction
        type: string
      post_id:
        description: The ID of the post to which this reaction was made
        type: string
      emoji_name:
        description: The name of the emoji that was used for this reaction
        type: string
      create_at:
        description: The time in milliseconds this reaction was made
        type: integer
        format: int64

  Emoji:
    type: object
    properties:
      id:
        description: The ID of the emoji
        type: string
      creator_id:
        description: The ID of the user that made the emoji
        type: string
      name:
        description: The name of the emoji
        type: string
      create_at:
        description: The time in milliseconds the emoji was made
        type: integer
        format: int64
      update_at:
        description: The time in milliseconds the emoji was last updated
        type: integer
        format: int64
      delete_at:
        description: The time in milliseconds the emoji was deleted
        type: integer
        format: int64

  Command:
    type: object
    properties:
      id:
        description: The ID of the slash command
        type: string
      token:
        description: The token which is used to verify the source of the payload
        type: string
      create_at:
        description: The time in milliseconds the command was created
        type: integer
      update_at:
        description: The time in milliseconds the command was last updated
        type: integer
        format: int64
      deleted_at:
        description: The time in milliseconds the command was deleted, 0 if never deleted
        type: integer
        format: int64
      creator_id:
        description: The user id for the commands creator
        type: string
      team_id:
        description: The team id for which this command is configured
        type: string
      trigger:
        description: The string that triggers this command
        type: string
      method:
        description: Is the trigger done with HTTP Get ('G') or HTTP Post ('P')
        type: string
      username:
        description: What is the username for the response post
        type: string
      icon_url:
        description: The url to find the icon for this users avatar
        type: string
      auto_complete:
        description: Use auto complete for this command
        type: boolean
      auto_complete_desc:
        description: The description for this command shown when selecting the command
        type: string
      auto_complete_hint:
        description: The hint for this command
        type: string
      display_name:
        description: Display name for the command
        type: string
      description:
        description: Description for this command
        type: string
      url:
        description: The URL that is triggered
        type: string

  CommandResponse:
    type: object
    properties:
      ResponseType:
        description: The response type either in_channel or ephemeral
        type: string
      Text:
        type: string
      Username:
        type: string
      IconURL:
        type: string
      GotoLocation:
        type: string
      Attachments:
        type: array
        items:
          $ref: '#/definitions/SlackAttachment'

  SlackAttachment:
    type: object
    properties:
      Id:
        type: string
      Fallback:
        type: string
      Color:
        type: string
      Pretext:
        type: string
      AuthorName:
        type: string
      AuthorLink:
        type: string
      AuthorIcon:
        type: string
      Title:
        type: string
      TitleLink:
        type: string
      Text:
        type: string
      Fields:
        type: array
        items:
          $ref: '#/definitions/SlackAttachmentField'
      ImageURL:
        type: string
      ThumbURL:
        type: string
      Footer:
        type: string
      FooterIcon:
        type: string
      Timestamp:
        description: The timestamp of the slack attachment, either type of string or integer
        type: string

  SlackAttachmentField:
    type: object
    properties:
      Title:
        type: string
      Value:
        description: The value of the attachment, set as string but capable with golang interface
        type: string
      Short:
        type: boolean

  StatusOK:
    type: object
    properties:
      status:
        description: Will contain "ok" if the request was successful and there was nothing else to return
        type: string

  OpenGraph:
    type: object
    description: OpenGraph metadata of a webpage
    properties:
      type:
        type: string
      url:
        type: string
      title:
        type: string
      description:
        type: string
      determiner:
        type: string
      site_name:
        type: string
      locale:
        type: string
      locales_alternate:
        type: array
        items:
          type: string
      images:
        type: array
        items:
          type: object
          description: Image object used in OpenGraph metadata of a webpage
          properties:
            url:
              type: string
            secure_url:
              type: string
            type:
              type: string
            width:
              type: integer
            height:
              type: integer
      videos:
        type: array
        items:
          type: object
          description: Video object used in OpenGraph metadata of a webpage
          properties:
            url:
              type: string
            secure_url:
              type: string
            type:
              type: string
            width:
              type: integer
            height:
              type: integer

      audios:
        type: array
        items:
          type: object
          description: Audio object used in OpenGraph metadata of a webpage
          properties:
            url:
              type: string
            secure_url:
              type: string
            type:
              type: string
      article:
        type: object
        description: Article object used in OpenGraph metadata of a webpage, if type is article
        properties:
          published_time:
            type: string
          modified_time:
            type: string
          expiration_time:
            type: string
          section:
            type: string
          tags:
            type: array
            items:
              type: string
          authors:
            type: array
            items:
              type: object
              properties:
                first_name:
                  type: string
                last_name:
                  type: string
                username:
                  type: string
                gender:
                  type: string
      book:
        type: object
        description: Book object used in OpenGraph metadata of a webpage, if type is book
        properties:
          isbn:
            type: string
          release_date:
            type: string
          tags:
            type: array
            items:
              type: string
          authors:
            type: array
            items:
              type: object
              properties:
                first_name:
                  type: string
                last_name:
                  type: string
                username:
                  type: string
                gender:
                  type: string
      profile:
        type: object
        properties:
          first_name:
            type: string
          last_name:
            type: string
          username:
            type: string
          gender:
            type: string

  Audit:
    type: object
    properties:
      id:
        type: string
      create_at:
        description: The time in milliseconds a audit was created
        type: integer
        format: int64
      user_id:
        type: string
      action:
        type: string
      extra_info:
        type: string
      ip_address:
        type: string
      session_id:
        type: string

  Config:
    type: object
    properties:
      ServiceSettings:
        type: object
        properties:
          SiteURL:
            type: string
          ListenAddress:
            type: string
          ConnectionSecurity:
            type: string
          TLSCertFile:
            type: string
          TLSKeyFile:
            type: string
          UseLetsEncrypt:
            type: boolean
          LetsEncryptCertificateCacheFile:
            type: string
          Forward80To443:
            type: boolean
          ReadTimeout:
            type: integer
          WriteTimeout:
            type: integer
          MaximumLoginAttempts:
            type: integer
          SegmentDeveloperKey:
            type: string
          GoogleDeveloperKey:
            type: string
          EnableOAuthServiceProvider:
            type: boolean
          EnableIncomingWebhooks:
            type: boolean
          EnableOutgoingWebhooks:
            type: boolean
          EnableCommands:
            type: boolean
          EnableOnlyAdminIntegrations:
            type: boolean
          EnablePostUsernameOverride:
            type: boolean
          EnablePostIconOverride:
            type: boolean
          EnableTesting:
            type: boolean
          EnableDeveloper:
            type: boolean
          EnableSecurityFixAlert:
            type: boolean
          EnableInsecureOutgoingConnections:
            type: boolean
          EnableMultifactorAuthentication:
            type: boolean
          EnforceMultifactorAuthentication:
            type: boolean
          AllowCorsFrom:
            type: string
          SessionLengthWebInDays:
            type: integer
          SessionLengthMobileInDays:
            type: integer
          SessionLengthSSOInDays:
            type: integer
          SessionCacheInMinutes:
            type: integer
          WebsocketSecurePort:
            type: integer
          WebsocketPort:
            type: integer
          WebserverMode:
            type: string
          EnableCustomEmoji:
            type: boolean
          RestrictCustomEmojiCreation:
            type: string
      TeamSettings:
        type: object
        properties:
          SiteName:
            type: string
          MaxUsersPerTeam:
            type: integer
          EnableTeamCreation:
            type: boolean
          EnableUserCreation:
            type: boolean
          EnableOpenServer:
            type: boolean
          RestrictCreationToDomains:
            type: string
          EnableCustomBrand:
            type: boolean
          CustomBrandText:
            type: string
          CustomDescriptionText:
            type: string
          RestrictDirectMessage:
            type: string
          RestrictTeamInvite:
            type: string
          RestrictPublicChannelManagement:
            type: string
          RestrictPrivateChannelManagement:
            type: string
          RestrictPublicChannelCreation:
            type: string
          RestrictPrivateChannelCreation:
            type: string
          RestrictPublicChannelDeletion:
            type: string
          RestrictPrivateChannelDeletion:
            type: string
          UserStatusAwayTimeout:
            type: integer
          MaxChannelsPerTeam:
            type: integer
          MaxNotificationsPerChannel:
            type: integer
      SqlSettings:
        type: object
        properties:
          DriverName:
            type: string
          DataSource:
            type: string
          DataSourceReplicas:
            type: array
            items:
              type: string
          MaxIdleConns:
            type: integer
          MaxOpenConns:
            type: integer
          Trace:
            type: boolean
          AtRestEncryptKey:
            type: string
      LogSettings:
        type: object
        properties:
          EnableConsole:
            type: boolean
          ConsoleLevel:
            type: string
          EnableFile:
            type: boolean
          FileLevel:
            type: string
          FileLocation:
            type: string
          EnableWebhookDebugging:
            type: boolean
          EnableDiagnostics:
            type: boolean
      PasswordSettings:
        type: object
        properties:
          MinimumLength:
            type: integer
          Lowercase:
            type: boolean
          Number:
            type: boolean
          Uppercase:
            type: boolean
          Symbol:
            type: boolean
      FileSettings:
        type: object
        properties:
          MaxFileSize:
            type: integer
          DriverName:
            type: string
          Directory:
            type: string
          EnablePublicLink:
            type: boolean
          PublicLinkSalt:
            type: string
          ThumbnailWidth:
            type: integer
          ThumbnailHeight:
            type: integer
          PreviewWidth:
            type: integer
          PreviewHeight:
            type: integer
          ProfileWidth:
            type: integer
          ProfileHeight:
            type: integer
          InitialFont:
            type: string
          AmazonS3AccessKeyId:
            type: string
          AmazonS3SecretAccessKey:
            type: string
          AmazonS3Bucket:
            type: string
          AmazonS3Region:
            type: string
          AmazonS3Endpoint:
            type: string
          AmazonS3SSL:
            type: boolean
      EmailSettings:
        type: object
        properties:
          EnableSignUpWithEmail:
            type: boolean
          EnableSignInWithEmail:
            type: boolean
          EnableSignInWithUsername:
            type: boolean
          SendEmailNotifications:
            type: boolean
          RequireEmailVerification:
            type: boolean
          FeedbackName:
            type: string
          FeedbackEmail:
            type: string
          FeedbackOrganization:
            type: string
          SMTPUsername:
            type: string
          SMTPPassword:
            type: string
          SMTPServer:
            type: string
          SMTPPort:
            type: string
          ConnectionSecurity:
            type: string
          InviteSalt:
            type: string
          PasswordResetSalt:
            type: string
          SendPushNotifications:
            type: boolean
          PushNotificationServer:
            type: string
          PushNotificationContents:
            type: string
          EnableEmailBatching:
            type: boolean
          EmailBatchingBufferSize:
            type: integer
          EmailBatchingInterval:
            type: integer
      RateLimitSettings:
        type: object
        properties:
          Enable:
            type: boolean
          PerSec:
            type: integer
          MaxBurst:
            type: integer
          MemoryStoreSize:
            type: integer
          VaryByRemoteAddr:
            type: boolean
          VaryByHeader:
            type: string
      PrivacySettings:
        type: object
        properties:
          ShowEmailAddress:
            type: boolean
          ShowFullName:
            type: boolean
      SupportSettings:
        type: object
        properties:
          TermsOfServiceLink:
            type: string
          PrivacyPolicyLink:
            type: string
          AboutLink:
            type: string
          HelpLink:
            type: string
          ReportAProblemLink:
            type: string
          SupportEmail:
            type: string
      GitLabSettings:
        type: object
        properties:
          Enable:
            type: boolean
          Secret:
            type: string
          Id:
            type: string
          Scope:
            type: string
          AuthEndpoint:
            type: string
          TokenEndpoint:
            type: string
          UserApiEndpoint:
            type: string
      GoogleSettings:
        type: object
        properties:
          Enable:
            type: boolean
          Secret:
            type: string
          Id:
            type: string
          Scope:
            type: string
          AuthEndpoint:
            type: string
          TokenEndpoint:
            type: string
          UserApiEndpoint:
            type: string
      Office365Settings:
        type: object
        properties:
          Enable:
            type: boolean
          Secret:
            type: string
          Id:
            type: string
          Scope:
            type: string
          AuthEndpoint:
            type: string
          TokenEndpoint:
            type: string
          UserApiEndpoint:
            type: string
      LdapSettings:
        type: object
        properties:
          Enable:
            type: boolean
          LdapServer:
            type: string
          LdapPort:
            type: integer
          ConnectionSecurity:
            type: string
          BaseDN:
            type: string
          BindUsername:
            type: string
          BindPassword:
            type: string
          UserFilter:
            type: string
          FirstNameAttribute:
            type: string
          LastNameAttribute:
            type: string
          EmailAttribute:
            type: string
          UsernameAttribute:
            type: string
          NicknameAttribute:
            type: string
          IdAttribute:
            type: string
          PositionAttribute:
            type: string
          SyncIntervalMinutes:
            type: integer
          SkipCertificateVerification:
            type: boolean
          QueryTimeout:
            type: integer
          MaxPageSize:
            type: integer
          LoginFieldName:
            type: string
      ComplianceSettings:
        type: object
        properties:
          Enable:
            type: boolean
          Directory:
            type: string
          EnableDaily:
            type: boolean
      LocalizationSettings:
        type: object
        properties:
          DefaultServerLocale:
            type: string
          DefaultClientLocale:
            type: string
          AvailableLocales:
            type: string
      SamlSettings:
        type: object
        properties:
          Enable:
            type: boolean
          Verify:
            type: boolean
          Encrypt:
            type: boolean
          IdpUrl:
            type: string
          IdpDescriptorUrl:
            type: string
          AssertionConsumerServiceURL:
            type: string
          IdpCertificateFile:
            type: string
          PublicCertificateFile:
            type: string
          PrivateKeyFile:
            type: string
          FirstNameAttribute:
            type: string
          LastNameAttribute:
            type: string
          EmailAttribute:
            type: string
          UsernameAttribute:
            type: string
          NicknameAttribute:
            type: string
          LocaleAttribute:
            type: string
          PositionAttribute:
            type: string
          LoginButtonText:
            type: string
      NativeAppSettings:
        type: object
        properties:
          AppDownloadLink:
            type: string
          AndroidAppDownloadLink:
            type: string
          IosAppDownloadLink:
            type: string
      ClusterSettings:
        type: object
        properties:
          Enable:
            type: boolean
          InterNodeListenAddress:
            type: string
          InterNodeUrls:
            type: array
            items:
              type: string
      MetricsSettings:
        type: object
        properties:
          Enable:
            type: boolean
          BlockProfileRate:
            type: integer
          ListenAddress:
            type: string
      AnalyticsSettings:
        type: object
        properties:
          MaxUsersForStatistics:
            type: integer

  EnvironmentConfig:
    type: object
    properties:
      ServiceSettings:
        type: object
        properties:
          SiteURL:
            type: boolean
          ListenAddress:
            type: boolean
          ConnectionSecurity:
            type: boolean
          TLSCertFile:
            type: boolean
          TLSKeyFile:
            type: boolean
          UseLetsEncrypt:
            type: boolean
          LetsEncryptCertificateCacheFile:
            type: boolean
          Forward80To443:
            type: boolean
          ReadTimeout:
            type: boolean
          WriteTimeout:
            type: boolean
          MaximumLoginAttempts:
            type: boolean
          SegmentDeveloperKey:
            type: boolean
          GoogleDeveloperKey:
            type: boolean
          EnableOAuthServiceProvider:
            type: boolean
          EnableIncomingWebhooks:
            type: boolean
          EnableOutgoingWebhooks:
            type: boolean
          EnableCommands:
            type: boolean
          EnableOnlyAdminIntegrations:
            type: boolean
          EnablePostUsernameOverride:
            type: boolean
          EnablePostIconOverride:
            type: boolean
          EnableTesting:
            type: boolean
          EnableDeveloper:
            type: boolean
          EnableSecurityFixAlert:
            type: boolean
          EnableInsecureOutgoingConnections:
            type: boolean
          EnableMultifactorAuthentication:
            type: boolean
          EnforceMultifactorAuthentication:
            type: boolean
          AllowCorsFrom:
            type: boolean
          SessionLengthWebInDays:
            type: boolean
          SessionLengthMobileInDays:
            type: boolean
          SessionLengthSSOInDays:
            type: boolean
          SessionCacheInMinutes:
            type: boolean
          WebsocketSecurePort:
            type: boolean
          WebsocketPort:
            type: boolean
          WebserverMode:
            type: boolean
          EnableCustomEmoji:
            type: boolean
          RestrictCustomEmojiCreation:
            type: boolean
      TeamSettings:
        type: object
        properties:
          SiteName:
            type: boolean
          MaxUsersPerTeam:
            type: boolean
          EnableTeamCreation:
            type: boolean
          EnableUserCreation:
            type: boolean
          EnableOpenServer:
            type: boolean
          RestrictCreationToDomains:
            type: boolean
          EnableCustomBrand:
            type: boolean
          CustomBrandText:
            type: boolean
          CustomDescriptionText:
            type: boolean
          RestrictDirectMessage:
            type: boolean
          RestrictTeamInvite:
            type: boolean
          RestrictPublicChannelManagement:
            type: boolean
          RestrictPrivateChannelManagement:
            type: boolean
          RestrictPublicChannelCreation:
            type: boolean
          RestrictPrivateChannelCreation:
            type: boolean
          RestrictPublicChannelDeletion:
            type: boolean
          RestrictPrivateChannelDeletion:
            type: boolean
          UserStatusAwayTimeout:
            type: boolean
          MaxChannelsPerTeam:
            type: boolean
          MaxNotificationsPerChannel:
            type: boolean
      SqlSettings:
        type: object
        properties:
          DriverName:
            type: boolean
          DataSource:
            type: boolean
          DataSourceReplicas:
            type: boolean
          MaxIdleConns:
            type: boolean
          MaxOpenConns:
            type: boolean
          Trace:
            type: boolean
          AtRestEncryptKey:
            type: boolean
      LogSettings:
        type: object
        properties:
          EnableConsole:
            type: boolean
          ConsoleLevel:
            type: boolean
          EnableFile:
            type: boolean
          FileLevel:
            type: boolean
          FileLocation:
            type: boolean
          EnableWebhookDebugging:
            type: boolean
          EnableDiagnostics:
            type: boolean
      PasswordSettings:
        type: object
        properties:
          MinimumLength:
            type: boolean
          Lowercase:
            type: boolean
          Number:
            type: boolean
          Uppercase:
            type: boolean
          Symbol:
            type: boolean
      FileSettings:
        type: object
        properties:
          MaxFileSize:
            type: boolean
          DriverName:
            type: boolean
          Directory:
            type: boolean
          EnablePublicLink:
            type: boolean
          PublicLinkSalt:
            type: boolean
          ThumbnailWidth:
            type: boolean
          ThumbnailHeight:
            type: boolean
          PreviewWidth:
            type: boolean
          PreviewHeight:
            type: boolean
          ProfileWidth:
            type: boolean
          ProfileHeight:
            type: boolean
          InitialFont:
            type: boolean
          AmazonS3AccessKeyId:
            type: boolean
          AmazonS3SecretAccessKey:
            type: boolean
          AmazonS3Bucket:
            type: boolean
          AmazonS3Region:
            type: boolean
          AmazonS3Endpoint:
            type: boolean
          AmazonS3SSL:
            type: boolean
      EmailSettings:
        type: object
        properties:
          EnableSignUpWithEmail:
            type: boolean
          EnableSignInWithEmail:
            type: boolean
          EnableSignInWithUsername:
            type: boolean
          SendEmailNotifications:
            type: boolean
          RequireEmailVerification:
            type: boolean
          FeedbackName:
            type: boolean
          FeedbackEmail:
            type: boolean
          FeedbackOrganization:
            type: boolean
          SMTPUsername:
            type: boolean
          SMTPPassword:
            type: boolean
          SMTPServer:
            type: boolean
          SMTPPort:
            type: boolean
          ConnectionSecurity:
            type: boolean
          InviteSalt:
            type: boolean
          PasswordResetSalt:
            type: boolean
          SendPushNotifications:
            type: boolean
          PushNotificationServer:
            type: boolean
          PushNotificationContents:
            type: boolean
          EnableEmailBatching:
            type: boolean
          EmailBatchingBufferSize:
            type: boolean
          EmailBatchingInterval:
            type: boolean
      RateLimitSettings:
        type: object
        properties:
          Enable:
            type: boolean
          PerSec:
            type: boolean
          MaxBurst:
            type: boolean
          MemoryStoreSize:
            type: boolean
          VaryByRemoteAddr:
            type: boolean
          VaryByHeader:
            type: boolean
      PrivacySettings:
        type: object
        properties:
          ShowEmailAddress:
            type: boolean
          ShowFullName:
            type: boolean
      SupportSettings:
        type: object
        properties:
          TermsOfServiceLink:
            type: boolean
          PrivacyPolicyLink:
            type: boolean
          AboutLink:
            type: boolean
          HelpLink:
            type: boolean
          ReportAProblemLink:
            type: boolean
          SupportEmail:
            type: boolean
      GitLabSettings:
        type: object
        properties:
          Enable:
            type: boolean
          Secret:
            type: boolean
          Id:
            type: boolean
          Scope:
            type: boolean
          AuthEndpoint:
            type: boolean
          TokenEndpoint:
            type: boolean
          UserApiEndpoint:
            type: boolean
      GoogleSettings:
        type: object
        properties:
          Enable:
            type: boolean
          Secret:
            type: boolean
          Id:
            type: boolean
          Scope:
            type: boolean
          AuthEndpoint:
            type: boolean
          TokenEndpoint:
            type: boolean
          UserApiEndpoint:
            type: boolean
      Office365Settings:
        type: object
        properties:
          Enable:
            type: boolean
          Secret:
            type: boolean
          Id:
            type: boolean
          Scope:
            type: boolean
          AuthEndpoint:
            type: boolean
          TokenEndpoint:
            type: boolean
          UserApiEndpoint:
            type: boolean
      LdapSettings:
        type: object
        properties:
          Enable:
            type: boolean
          LdapServer:
            type: boolean
          LdapPort:
            type: boolean
          ConnectionSecurity:
            type: boolean
          BaseDN:
            type: boolean
          BindUsername:
            type: boolean
          BindPassword:
            type: boolean
          UserFilter:
            type: boolean
          FirstNameAttribute:
            type: boolean
          LastNameAttribute:
            type: boolean
          EmailAttribute:
            type: boolean
          UsernameAttribute:
            type: boolean
          NicknameAttribute:
            type: boolean
          IdAttribute:
            type: boolean
          PositionAttribute:
            type: boolean
          SyncIntervalMinutes:
            type: boolean
          SkipCertificateVerification:
            type: boolean
          QueryTimeout:
            type: boolean
          MaxPageSize:
            type: boolean
          LoginFieldName:
            type: boolean
      ComplianceSettings:
        type: object
        properties:
          Enable:
            type: boolean
          Directory:
            type: boolean
          EnableDaily:
            type: boolean
      LocalizationSettings:
        type: object
        properties:
          DefaultServerLocale:
            type: boolean
          DefaultClientLocale:
            type: boolean
          AvailableLocales:
            type: boolean
      SamlSettings:
        type: object
        properties:
          Enable:
            type: boolean
          Verify:
            type: boolean
          Encrypt:
            type: boolean
          IdpUrl:
            type: boolean
          IdpDescriptorUrl:
            type: boolean
          AssertionConsumerServiceURL:
            type: boolean
          IdpCertificateFile:
            type: boolean
          PublicCertificateFile:
            type: boolean
          PrivateKeyFile:
            type: boolean
          FirstNameAttribute:
            type: boolean
          LastNameAttribute:
            type: boolean
          EmailAttribute:
            type: boolean
          UsernameAttribute:
            type: boolean
          NicknameAttribute:
            type: boolean
          LocaleAttribute:
            type: boolean
          PositionAttribute:
            type: boolean
          LoginButtonText:
            type: boolean
      NativeAppSettings:
        type: object
        properties:
          AppDownloadLink:
            type: boolean
          AndroidAppDownloadLink:
            type: boolean
          IosAppDownloadLink:
            type: boolean
      ClusterSettings:
        type: object
        properties:
          Enable:
            type: boolean
          InterNodeListenAddress:
            type: boolean
          InterNodeUrls:
            type: boolean
      MetricsSettings:
        type: object
        properties:
          Enable:
            type: boolean
          BlockProfileRate:
            type: boolean
          ListenAddress:
            type: boolean
      AnalyticsSettings:
        type: object
        properties:
          MaxUsersForStatistics:
            type: boolean

  SamlCertificateStatus:
    type: object
    properties:
      idp_certificate_file:
        description: Status is good when `true`
        type: boolean
      public_certificate_file:
        description: Status is good when `true`
        type: boolean
      private_key_file:
        description: Status is good when `true`
        type: boolean

  Compliance:
    type: object
    properties:
      id:
        type: string
      create_at:
        type: integer
        format: int64
      user_id:
        type: string
      status:
        type: string
      count:
        type: integer
      desc:
        type: string
      type:
        type: string
      start_at:
        type: integer
        format: int64
      end_at:
        type: integer
        format: int64
      keywords:
        type: string
      emails:
        type: string

  ClusterInfo:
    type: object
    properties:
      id:
        description: The unique ID for the node
        type: string
      version:
        description: The server version the node is on
        type: string
      config_hash:
        description: The hash of the configuartion file the node is using
        type: string
      internode_url:
        description: The URL used to communicate with those node from other nodes
        type: string
      hostname:
        description: The hostname for this node
        type: string
      last_ping:
        description: The time of the last ping to this node
        type: integer
      is_alive:
        description: Whether or not the node is alive and well
        type: boolean

  AppError:
    type: object
    properties:
      status_code:
        type: integer
      id:
        type: string
      message:
        type: string
      request_id:
        type: string

  Status:
    type: object
    properties:
      user_id:
        type: string
      status:
        type: string
      manual:
        type: boolean
      last_activity_at:
        type: integer
        format: int64

  OAuthApp:
    type: object
    properties:
      id:
        type: string
        description: The client id of the application
      client_secret:
        type: string
        description: The client secret of the application
      name:
        type: string
        description: The name of the client application
      description:
        type: string
        description: A short description of the application
      icon_url:
        type: string
        description: A URL to an icon to display with the application
      callback_urls:
        type: array
        items:
          type: string
        description: A list of callback URLs for the appliation
      homepage:
        type: string
        description: A link to the website of the application
      is_trusted:
        type: boolean
        description: Set this to `true` to skip asking users for permission
      create_at:
        type: integer
        description: The time of registration for the application
        format: int64
      update_at:
        type: integer
        description: The last time of update for the application
        format: int64

  Job:
    type: object
    properties:
      id:
        type: string
        description: The unique id of the job
      type:
        type: string
        description: The type of job
      create_at:
        type: integer
        description: The time at which the job was created
        format: int64
      start_at:
        type: integer
        description: The time at which the job was started
        format: int64
      last_activity_at:
        type: integer
        description: The last time at which the job had activity
        format: int64
      status:
        type: string
        description: The status of the job
      progress:
        type: integer
        description: The progress (as a percentage) of the job
      data:
        type: object
        description: A freeform data field containing additional information about the job

  UserAccessToken:
    type: object
    properties:
      id:
        type: string
        description: Unique identifier for the token
      token:
        type: string
        description: The token used for authentication
      user_id:
        type: string
        description: The user the token authenticates for
      description:
        type: string
        description: A description of the token usage

  UserAccessTokenSanitized:
    type: object
    properties:
      id:
        type: string
        description: Unique identifier for the token
      user_id:
        type: string
        description: The user the token authenticates for
      description:
        type: string
        description: A description of the token usage
      is_active:
        type: boolean
        description: Indicates whether the token is active

  DataRetentionPolicy:
    type: object
    properties:
      message_deletion_enabled:
        type: boolean
        description: Indicates whether data retention policy deletion of messages is enabled.
      file_deletion_enabled:
        type: boolean
        description: Indicates whether data retention policy deletion of file attachments is enabled.
      message_retention_cutoff:
        type: integer
        description: The current server timestamp before which messages should be deleted.
      file_retention_cutoff:
        type: integer
        description: The current server timestamp before which files should be deleted.

  UserNotifyProps:
    type: object
    properties:
      email:
        type: string
        description: Set to "true" to enable email notifications, "false" to disable. Defaults to "true".
      push:
        type: string
        description: Set to "all" to receive push notifications for all activity, "mention" for mentions and direct messages only, and "none" to disable. Defaults to "mention".
      desktop:
        type: string
        description: Set to "all" to receive desktop notifications for all activity, "mention" for mentions and direct messages only, and "none" to disable. Defaults to "all".
      desktop_sound:
        type: string
        description: Set to "true" to enable sound on desktop notifications, "false" to disable. Defaults to "true".
      mention_keys:
        type: string
        description: A comma-separated list of words to count as mentions. Defaults to username and @username.
      channel:
        type: string
        description: Set to "true" to enable channel-wide notifications (@channel, @all, etc.), "false" to disable. Defaults to "true".
      first_name:
        type: string
        description: Set to "true" to enable mentions for first name. Defaults to "true" if a first name is set, "false" otherwise.

  Timezone:
    type: object
    properties:
      useAutomaticTimezone:
        type: string
        description: Set to "true" to use the browser/system timezone, "false" to set manually. Defaults to "true".
      manualTimezone:
        type: string
        description: Value when setting manually the timezone, i.e. "Europe/Berlin".
      automaticTimezone:
        type: string
        description: This value is set automatically when the "useAutomaticTimezone" is set to "true".

  ChannelNotifyProps:
    type: object
    properties:
      email:
        type: string
        description: Set to "true" to enable email notifications, "false" to disable, or "default" to use the global user notification setting.
      push:
        type: string
        description: Set to "all" to receive push notifications for all activity, "mention" for mentions and direct messages only, "none" to disable, or "default" to use the global user notification setting.
      desktop:
        type: string
        description: Set to "all" to receive desktop notifications for all activity, "mention" for mentions and direct messages only, "none" to disable, or "default" to use the global user notification setting.
      mark_unread:
        type: string
        description: Set to "all" to mark the channel unread for any new message, "mention" to mark unread for new mentions only. Defaults to "all".

  PluginManifest:
    type: object
    properties:
      id:
        type: string
        description: Globally unique identifier that represents the plugin.
      name:
        type: string
        description: Name of the plugin.
      description:
        type: string
        description: Description of what the plugin is and does.
      version:
        type: string
        description: Version number of the plugin.
      min_server_version:
        type: string
        description: |
          The minimum Mattermost server version required for the plugin.

          Available as server version 5.6.
      backend:
        type: object
        description: Deprecated in Mattermost 5.2 release.
        properties:
          executable:
            type: string
            description: Path to the executable binary.
      server:
        type: object
        properties:
          executables:
            type: object
            description: Paths to executable binaries, specifying multiple entry points for different platforms when bundled together in a single plugin.
            properties:
              linux-amd64:
                type: string
              darwin-amd64:
                type: string
              windows-amd64:
                type: string
          executable:
            type: string
            description: Path to the executable binary.
      webapp:
        type: object
        properties:
          bundle_path:
            type: string
            description: Path to the webapp JavaScript bundle.
      settings_schema:
        type: object
        description: Settings schema used to define the System Console UI for the plugin.

  PluginManifestWebapp:
    type: object
    properties:
      id:
        type: string
        description: Globally unique identifier that represents the plugin.
      version:
        type: string
        description: Version number of the plugin.
      webapp:
        type: object
        properties:
          bundle_path:
            type: string
            description: Path to the webapp JavaScript bundle.

  Role:
    type: object
    properties:
      id:
        type: string
        description: The unique identifier of the role.
      name:
        type: string
        description: The unique name of the role, used when assigning roles to users/groups in contexts.
      display_name:
        type: string
        description: The human readable name for the role.
      description:
        type: string
        description: A human readable description of the role.
      permissions:
        type: array
        items:
          type: string
        description: A list of the unique names of the permissions this role grants.
      scheme_managed:
        type: boolean
        description: indicates if this role is managed by a scheme (true), or is a custom stand-alone role (false).

  Scheme:
    type: object
    properties:
      id:
        type: string
        description: The unique identifier of the scheme.
      name:
        type: string
        description: The human readable name for the scheme.
      description:
        type: string
        description: A human readable description of the scheme.
      create_at:
        type: integer
        format: int64
        description: The time at which the scheme was created.
      update_at:
        type: integer
        format: int64
        description: The time at which the scheme was last updated.
      delete_at:
        type: integer
        format: int64
        description: The time at which the scheme was deleted.
      scope:
        type: string
        description: The scope to which this scheme can be applied, either "team" or "channel".
      default_team_admin_role:
        type: string
        description: The id of the default team admin role for this scheme.
      default_team_user_role:
        type: string
        description: The id of the default team user role for this scheme.
      default_channel_admin_role:
        type: string
        description: The id of the default channel admin role for this scheme.
      default_channel_user_role:
        type: string
        description: The id of the default channel user role for this scheme.

  TermsOfService:
    type: object
    properties:
      id:
        type: string
        description: The unique identifier of the terms of service.
      create_at:
        type: integer
        format: int64
        description: The time at which the terms of service was created.
      user_id:
        type: string
        description: The unique identifier of the user who created these terms of service.
      text:
        type: string
        description: The text of terms of service. Supports Markdown.

  UserTermsOfService:
    type: object
    properties:
      user_id:
        type: string
        description: The unique identifier of the user who performed this terms of service action.
      terms_of_service_id:
        type: string
        description: The unique identifier of the terms of service the action was performed on.
      create_at:
        description: The time in milliseconds that this action was performed.
        type: integer
        format: int64

  PostIdToReactionsMap:
    type: object
    additionalProperties:
      type: array
      items:
        $ref: '#/definitions/Reaction'

  Group:
    type: object
    properties:
      id:
        type: string
      name:
        type: string
      display_name:
        type: string
      description:
        type: string
      source:
        type: string
      remote_id:
        type: string
      create_at:
        type: integer
        format: int64
      update_at:
        type: integer
        format: int64
      delete_at:
        type: integer
        format: int64
      has_syncables:
        type: boolean

  Bot:
    description: A bot account
    type: object
    properties:
      user_id:
        description: The user id of the associated user entry.
        type: string
      create_at:
        description: The time in milliseconds a bot was created
        type: integer
        format: int64
      update_at:
        description: The time in milliseconds a bot was last updated
        type: integer
        format: int64
      delete_at:
        description: The time in milliseconds a bot was deleted
        type: integer
        format: int64
      username:
        type: string
      display_name:
        type: string
      description:
        type: string
      owner_id:
        description: The user id of the user that currently owns this bot.
        type: string

externalDocs:
  description: Find out more about Mattermost
  url: 'https://about.mattermost.com'
