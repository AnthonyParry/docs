# Reference

## The Schema

```
type ScapholdSchema { 
    id: ID! 
    name: String! 
    description: String 
    types: [ScapholdType] 
} 

type ScapholdType { 
    id: ID! 
    name: String! 
    description: String 
    kind: "OBJECT" 
    fields: [ScapholdField] 
    interfaces: [String] 
    permissions: [ScapholdPermission]
} 

type ScapholdField { 
    name: String! 
    description: String 
    type: String! 
    ofType: String 
    nonNull: Boolean 
    unique: Boolean 
    reverseName: String 
} 

interface Node { 
    id: ID! 
}
```

Every application you build on Scaphold will have a schema. A schema consists of a set of types and a set of connections between those types. This connected web of types creates the 'graph' that will come to define your API.

One of GraphQL's bigget benefits is that provides a mechanism to express types at the API level. Types have existed in general purpose programming languages and other query languages like SQL for years so it's about time we brought it to APIs.

There are two main classes of types that you need to be aware of on Scaphold. Types that implement the **Node** interface, and types that don't.

Types that implement the **Node** interface are first-class entities in your Scaphold API. We will generate queries and mutations for them, you may create connections between them, and you may use them with event based integrations.

Types that don't implement the **Node** interface are auxillery structures in your GraphQL API and objects are stored inline within other Node types. Scaphold will not create queries and mutations for non node types nor can you use them to fire event based integrations. You can, however, still relate them to other objects via the List type or as an inline object.

Scaphold's [Schema Designer](https://scaphold.io/#/apps/schema) allows you to define complex schemas in a fraction of the time while also offering you a convenient place to setup access control rules (permissions) for your data. As you define the structure of your schema we'll generate the backend that will host your custom GraphQL API. By default we genereate a get, create, update, replace, and delete operation for each type in your schema. We'll also create mutations to handle user authentication, schema migration, and integration management. Start building today!

## Connections & Pagination

```
type _XConnection { 
    edges: [_XEdge]
    pageInfo: PageInfo  
} 

type _XEdge { 
    cursor: String
    node: X
} 

type PageInfo { 
    hasNextPage: Boolean! 
    hasPreviousPage: Boolean!
    count: Int 
}
```

We use the concept of **Connections** to provide a standardized way of paginating through large sets of objects. You can use the **Connection** type when your defining your schema whenever you need to model relations on potentially large sets of data. All connection fields will take the form:

<pre class="left">
query {
    connectionFieldOfTypeX (
            first: Int,
            after: String!,
            last: Int,
            before: String,
            orderBy: String
        ) {

        edges {
            cursor
            node {
                ...fields in type X
            }
        }
        pageInfo {
            hasNextPage
            hasPreviousPage
            count
        }
    }
}
</pre>

Although connections expose first, after, last, and before we strongly recommend that you only ever use either (first and after) or (last and before) together at once as unexpected behavior can occur if you use both at the same time. The orderBy paramater allows you to order your data with respect to a field in the connected type and we will maintain the order throughout the pagination.

Many-to-many connections are defined by having two types with Connection fields pointing to one another via their 'reverseName' and have a special workflow for managing edges between their objects.

For example suppose we have the following schema:

<pre class="left">
type User { 
    id: ID! 
    username: String! 
    posts: [Post] 
    edits: [Post] 
} 

type Post { 
    id: ID! 
    title: String! 
    content: String 
    author: User 
    editors: [User] 
}
</pre>

Scaphold will generate 4 special mutations for dealing with the many-to-many relations between **User** and **Post** via the edits & editors fields. These mutations will be:

1. addPostToUserEdits
2. removePostFromUserEdits
3. addUserToPostEditors
4. removeUserFromPostEditors

These mutations can be used to add edges between objects in the many to many connection. It is important to note that connections are bidirectional and thus if you use addPostToUserEdits() to add a post to a users set of edited posts then it will also add that user to the posts set of editors.

## API Operations

GraphQL designates two types of operations: queries & mutations. These two types of operations will allow you to interact with your data given the schema that you've created in your [Schema Designer](https://scaphold.io/#/home/docs#Schema-Designer). You'll use queries and mutations to handle your basic CRUD commands to manage your data through your Scaphold endpoint.

The names and headers of the query and mutation requests are generated by appending the operation type to the object name like the following for type User:

<pre class="left">
CREATE → createUser (input: $input)
READ → getUser (id: $id)
UPDATE → updateUser (input: $input)
DELETE → deleteUser (input: $input)
</pre>

### Queries

A GraphQL query is a read-only fetch on your data.

#### READ

Queries give you a way to make a GET request for your data, however you like it, giving you the exact data that you want; nothing more, nothing less. Here's an example:

<pre class="left">
// Query
query GetUser ($user: ID!) {
    getUser (id: $user) {
        id,
        username,
        createdAt,
        modifiedAt,
        lastLogin
    }
}

// Variables
{
    "user ": "xxx-xxx-xxx-xxx "
}

// Response
{
    "data": {
        "getUser": {
            "id": "eaf5822f-e440-40c7-a125-3c38b1a0820c",
            "username": "Elon Musk",
            "createdAt": "Wed May 25 2016 00:12:18 GMT-0700 (PDT)",
            "modifiedAt": "Wed May 25 2016 00:12:18 GMT-0700 (PDT)",
            "lastLogin": "Thu May 26 2016 12:15:36 GMT-0700 (PDT)"
        } 
    }
}
</pre>

All responses from your API will be formatted as JSON.

#### VIEWER

The viewer is an abstraction that allows you to easily paginate through the objects in your dataset. We generate a viewer field 'allX' for each type 'X' in your schema. Queries that go through the viewer are subject to your set permissions so users can only access the data you allow.

The viewer also contains a 'user' field that will always return information on the currently logged in user.

<pre class="left">
// Query
query GetThroughViewer {
    viewer {
        allUsers(first: $first, after: $after, orderBy: $orderBy) {
            edges {
                cursor
                node {
                    username
                    createdAt
                }
            }
        }
    }
}

// Variables
{
    "first": 3,
    "after": "eaf5822f-e440-40c7-a125-3c38b1a0820c",
    "orderBy": "-createdAt"
}

// Response
{
    "data": {
        "viewer": {
            "allUsers": [
                {
                    "username": "Steve Jobs",
                    "createdAt": "Wed May 24 2016 18:12:18 GMT-0700 (PDT)"
                },
                {
                    "username": "Bill Gates",
                    "createdAt": "Wed May 24 2016 17:29:22 GMT-0700 (PDT)"
                },
                {
                    "username": "Larry Page",
                    "createdAt": "Wed May 24 2016 15:43:13 GMT-0700 (PDT)"
                }
            ]
        } 
    }
}
</pre>

The viewer uses connections to enable pagination out of the box!

### Mutations

A GraphQL mutation is a write followed by a fetch in one operation.

#### CREATE

Mutations give you a way to make a POST request for your data, passing in parameters that correspond to the values of the fields that've been set in your app schema. Here's an example:

<pre class="left">
// Query
mutation CreateUser ($user: _CreateUserInput!) {
    createUser (input: $user) {
        changedUser {
            id,
            username
        }
    }
}

// Variables
{
    "user ": {
        "username": "elon@tesla.com",
        "password": "SuperSecretPassword"
    }
}

// Response
{
    "changedUser ": {
        "id ": "eaf5822f-e440-40c7-a125-3c38b1a0820c",
        "username": "elon@tesla.com"
    }
}
</pre>

In this request, JSON-formatted variables are used to send an object as part of the payload for the mutation request. The dollar sign ($) specifies a GraphQL variable, and the variable definition can be found in the variables section of the request. Another thing to note here is that when introducing variables in the mutation string, *$user: _CreateUserInput!*   means that that variable is of type _CreateUserInput! and is required (!).

#### UPDATE

You can also make a PUT request for your data, passing in parameters that correspond to the values of the fields that've been set in your app schema. Here's an example:

<pre class="left">
// Query
mutation UpdateUser ($user: _UpdateUserInput!) {
    updateUser (input: $user) {
        changedUser {
            id,
            username,
            biography
        }
    }
}

// Variables
{
    "user": {
        "id": "eaf5822f-e440-40c7-a125-3c38b1a0820c",
        "biography": "Spends his days saving the world with renewable energy."
    }
}

// Response
{
    "changedUser": {
    "id ": "eaf5822f-e440-40c7-a125-3c38b1a0820c ",
    "username": "elon@tesla.com",
    "biography": "Spends his days saving the world with renewable energy."
    }
}
</pre>

The update operation performs a non-destructive update to an object in your dataset. I.E. Update only updates the fields that you include as part of the input. If you want to replace an object entirely, use the replaceX mutation. The object's 'id' is required in every updateX mutation so that we can uniquely identify the object you would like to update.

#### DELETE

Use delete operations to remove data from your dataset. A delete operation take an input _DeleteXInput! that contains a single 'id' field. This operation will delete the object that is uniquely identified by that id field.

<pre class="left">
// Query
mutation DeleteUser ($user: _DeleteUserInput!) {
    deleteUser (input: $user) {
        id
    }
}

// Variables
{
    "user ": {
    "id ": "eaf5822f-e440-40c7-a125-3c38b1a0820c"
    }
}

// Response
{
    "id ": "eaf5822f-e440-40c7-a125-3c38b1a0820c "
}

</pre>

Delete an object in your dataset given an id.

## Authentication

Scaphold handles user authentication for you. Each of Scaphold application comes with a default User model which includes a 'username' and 'password' that are used to authenticate your users. We securely encrypt and store each user's password as well as ensure that they are not readable.

Logging in a user is simple. Use the loginUser mutation we provide you and we will return a JSON Web Token if the credentials match. To authenticate a user, you simply set the 'Authentication' http header of your request with the format 'Bearer {token-from-loginUser}'.

This token informs your API what user is logged in at any given time and enables our permissions system to layer access control rules on your data.

<pre class="left">
mutation Login($input: _LoginUserInput!) {
    loginUser(input: $input) {
        id
        token
        # Use this token in the header 
        # Authentication: Bearer {token} 
    }
}
</pre>

For social authentication, Scaphold has a solution for that as well! Integrating with OAuth providers like Facebook, Google, and Twtiter has never been easier with Scaphold's Auth0 integration. Find out how to [integrate social authentication here](#social-auth).

## Permissions

Scaphold implements a permissions system that allows your to define powerful access control rules by leveraging a combination of features from role based access control systems (RBAC) as well as the connections in your API's graph to define what user's can access what information.

Each type in your schema has an optional set of permissions applied to it. When a user tries to complete an operation, your API checks the types permissions and validates that the user is authorized to complete that operation. When you login to the Scaphold portal, you are logged in as an admin user and will have complete access to your application. It is also important to note that if a type has no permissions then anyone can perform any operations on it so make sure you add them before launching!

**EVERYONE** scoped permissions are the loosest available. Everyone can access the type. No login required.

**AUTHENTICATED** scoped permissions required a valid auth token to be present in the request headers (See authentication for more details). Scaphold provides a number of authentication mechanisms that all work seamlessly with your permissions.


**RELATED** scoped permissions use the connections in your data's graph to authorize behaviors. When you add a RELATED permission you tell use what 'User Fields' to inspect to check for the logged in user. For example, a **Post** model might have an 'author'. You could specify that only the author of the post can update it by adding the following permission to the **Post** type:

<pre class="left">
{
    scope: "RELATED",
    userFields: ["author"],
    update: true
}
</pre>

There is a special RELATED scope permission that gives a user sole access to their own user object. To enable this behavior create a RELATED scope permission with the desired operation and the user field 'me'. A common use case for the 'me' user field is to give a user sole access to update their own user object.

**ROLE** permissions allow you to layer more generic role based authentication methods on top of the connection and user based permissions you already have. We have added a few queries and mutations that make it easy to manage your new roles and permissions.

1. **createRole**

        Create a new 'public' or 'private' role and enrolls the creator as a role admin. Private roles require an admin to enroll a user. Anyone can join themself into a public role as a non-admin.

2. **deleteRole**

        Removes a role. Only role admins can delete roles.

3. **enrollUser**

        Enrolls a user as a member of a role. Only admins can create other admins and you must be an admin to enroll a user into a private role.

4. **disenrollUser**

        Removes a user from a role. Anyone can disenroll themself and admins can disenroll anyone.

By default, we create a special 'admin' role for each of your apps. Users that are enrolled into the 'admin' role are given full access to your GraphQL API without having to specify any custom permissions. The 'admin' role is private so, to get started, add one of your users to the 'admin' role using Scaphold GraphiQL explorer in the portal.

<pre class="left">
// Query
mutation EnrollUser($input: _EnrollUserInput!) {
  enrollUser(input: $input) {
    role
    isAdmin
  }
}

// Variables
{
    "input": {
        "userId": < admin-user-id >,
        "role": "admin",
        "isAdmin": true
    }
}
</pre>

Role scoped permissions can be used alongside existing RELATED scoped permissions to easily create complex AC rules. Let's take an example where we would like to have a set of notes that only executives of my company can see. Part of our schema might look something like this:

<pre class="left">
// Schema
type User {
    id: ID!
    username: String!
    authoredExecNotes: ExecutiveNoteConnection
}

type ExecutiveNote {
    id: ID!
    author: User
    content: String
}

// Permissions
[{
    scope: "ROLE",
    roles: ["Executives"],
    read: true,
    create: true
}
{
    scope: "RELATED",
    userFields: ["author"],
    update: true,
    delete: true
}]
</pre>


## Files

```javascript
// An example using superagent.js 
import request from 'superagent'; 
const query = 'mutation Create($input: _CreateUserInput!) { 
    createUser(input: $input) { 
        changedUser { 
            id 
            username 
            profilePicture { 
                url name mimeType 
            } 
        } 
    } 
}'; 

const variables = { 
    input: { 
        username: "michael", 
        password: "password", 
        profilePicture: { 
            name: "HikingAngelsLanding.jpg", 
            fieldName: "profPic" 
            // The same name that we use in request.attach() 
        } 
    }
}; 

request 
    .post('https://api.scaphold.io/graphql/my-api')
    .field('query', query) 
    .field('variables', JSON.stringify(variables)) 
    .attach('profPic', '/path/to/file.jpg')
    .end(function(err, res) { 
        console.log("That was easy") 
    })
```

You can use your Scaphold API to upload, download, and manage files! Here's how it works:

Go to the schema designer and add a field with type 'File'

<img src="images/news/filetype-min.png" alt="Add the file type in the schema designer">

Upload a file using the same GraphQL conventions you love! Simply attach your file to a multipart/form-data request and point to it using the 'fieldName' attribute of the FileInput type which looks like.

<pre class="left">
Type FileInput { 
    // Overrides the default file name 
    name: String;

    // Points us to the file in the request
    fieldName: String; 
}
</pre>

Querying file types is just as simple. Let's look at the File type.

<pre class="left">
// Querying a FileType will generate a pre-signed url 
// so only your users can access your files.
Type File { 
    name: String; 
    url: String; 
    mimeType: String; 
} 
</pre>

## Location Services

Build real-world apps with our built in location support!

Use the [Schema Designer](https://scaphold.io/#/apps/schema) to add a field with type Location

<img src="images/news/location-field.png" alt="Add a Location field.">

Scaphold detect's your Location fields and adds geographic queries automatically. Go to the doc explorer in GraphiQL and open the Viewer query field type to see the new functionality!

### Create objects with locations

<pre class="left">
// Query
mutation CreateArtist($input:_CreateArtistInput!) {
  createArtist(input: $input) {
    changedArtist {
      id
      name
      location {
        lat
        lon
      }
    }
  }
}

// Variables
{
    "input": {
        "location": {
            "lat": 47.606209,
            "lon": -122.332071
        },
        "name": "Macklemore"
    }
}
</pre>

### Query objects nearest to a location

<pre class="left">
# Get the artists nearest Seattle
query ClosestArtistsToSeattle($loc: _GeoLocationInput!, $unit: _GeoUnit) {
  viewer {
    getNearestArtistsByLocation(location: $loc, maxResults: 30, maxDist: 100, unit: $unit) {
      dist
      node {
        id
        name
        location {
          lat
          lon
        }
      }
    }
  }
}

// Variables
{
    "loc": {
        "lat": 47.644826,
        "lon": -122.334480
    },
    "unit": "mi"
}
</pre>

### Query all objects within a geographic circle

<pre class="left">
# Get the artists within 1000 meters of Golden Gate Park
query ArtistsWithinGoldenGatePark($circle: _GeoCircleInput!) {
  viewer {
    getUsersInsideCircleByLocation(circle: $circle) {
      id
      name
      location {
          lat
          lon
      }
    }
  }
}

// Variables
{
    "circle": {
        "center": {
            "lat": 37.769040,
            "lon": -122.483519
        },
        "radius": 1000,
        "unit": "m"
    }
}
</pre>

### Query all objects within an arbitrary geograhic area

<pre class="left">
# Get all artists from NY to Chicago to DC
query ArtistsFromNYToChicagoToDC($area: _GeoAreaInput!) {
  viewer {
    getUsersInsideAreaByLocation(area: $area) {
      id
      name
      location {
          lat
          lon
      }
    }
  }
}

// Variables
{
    "area": {
        "points": [
            {
                "lat": 40.712784,
                "lon": -74.005941
            },
            {
                "lat": 41.878114,
                "lon": -87.629798
            },
            {
                "lat": 38.907192,
                "lon": -77.036871
            }
        ]
    }
}
</pre>

## Aliases

Simplify your API URL with an Alias!

Now when you create an app, you can choose to set an alias. An alias is a unique name for your application that can be used in the URL to access your API. For example, if I set an alias for my app of **my-awesome-app** then I could access my application at the url **https://api.scaphold.io/graphql/my-awesome-app**

Alias names must be 6 characters or greater in length, must contain only lowercase letters, numbers, and dashes, and must not end or begin in a dash.

You can also change an alias at any time by clicking **My API** at the top of the page.

<img src="images/news/app-alias-min.png" alt="Copy and paste the code into your app.">

# Starter Kits

For a simple way to get up and running you can download one of our [starter kits on GitHub](https://github.com/scaphold-io)

## React & Relay
[Download for React/Relay](https://github.com/scaphold-io/react-relay-starter-kit)

Run this code by running `npm start` in the base directory of the application. Once the application has built, go to <code>localhost:3001</code> in your browser to access your new website.

Congratulations! you've successfully launched your first website with Scaphold. How is this all working? When the website runs, the starter kit points to a Scaphold endpoint that we've created with some basic data. Your website leverages the GraphQL server that we've already set up for you, so you can start creating users and logging in. After you've successfully logged in, you'll be able to see a HackerNews clone with several articles listed. Getting this list of items is another request that is being made via a GraphQL query.

To demonstrate, upon logging in, here is the query that is being made to fetch the HackerNews data:

<pre class="left">
// js/routes/HomeRoute.js

query {
    viewer {
    ${Component.getFragment('allHackerNewsItems', {orderBy: variables.orderBy})}
    }
}

. . .

// js/components/HackerNewsClone/Home.js

fragment on Viewer {
    allHackerNewsItems (first: 10, orderBy: $orderBy) {
        edges {
            cursor
            node {
                id,
                createdAt,
                modifiedAt,
                title,
                score,
                url,
                author {
                    id,
                    username
                }
            }
        }
    }
}
</pre>

These two pieces of code go together to form one query that grabs the first 10 HackerNewsItems from the app's dataset with the corresponding return fields like title, score, url, and author. To see what other queries you can make, feel free to navigate to the GraphiQL page from the header of the website and open up the document explorer on the top right. From here, you'll find a reference for query and mutation templates that are allowed for this particular app, and a sandbox to execute these particular requests.

If you would like to have a more involved or custom dataset to play around with, the beauty of Scaphold is that you can create your own in a matter of minutes! Jump right into the [tutorial](https://scaphold.io/#/docs#Tutorial) to start building your first app.

This starter kit is yours to keep, and if you have any questions, don't hesitate to reach out to us through our chat support or email us at support@scaphold.io.

But wait, there's more! Here's the list of boilerplates that we provide that works right out of the box with Scaphold:

* [React & Relay](https://github.com/scaphold-io/react-relay-starter-kit)
* [React Native & Relay](https://github.com/scaphold-io/react-native-starter-kit)
* More to come... On the docket: AngularJS, iOS, Android
