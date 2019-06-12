Firebase Realtime Database Android Quickstart
----------------------------------------------
This quickstart sample app demonstrates how to use the Firebase Realtime Database and the Firebase Android SDK to build and deploy a simple and secure blogging application for an Android client device.

Getting Started
---------------
To build this quickstart Android app, follow these steps:

1. [Add Firebase to your Android Project](https://firebase.google.com/docs/android/setup).
2. Log in to the [Firebase Console](https://console.firebase.google.com).
3. Go to **Authentication** tab and then enable the **Email/Password Sign-in Method**.
4. Run the sample on Android device or emulator.

Result
-----------
<img src="/app/src/screen.png" height="534" width="300">

Database Model
--------------
The database model is structured by four "root" nodes:

  1. `users` - a list of `User` objects, keyed by user ID. So
    `/users/<ID>/email` is the email address of the user with id=`<ID>`.
  2. `posts` - a list of `Post` objects, keyed by randomly generated push ID.
    Each `Post` contains the `uid` and `author` properties to determine the
    identity of the author without a JOIN-style query.
    * Posts contain a `stars` property which is a `Map` of user IDs to boolean
      values.  If `/posts/<POST-ID>/stars/<USER-ID>` is `true`, this means
      the user with ID `<USER-ID>` has starred the post with ID `<POST-ID>`.
      This data nesting makes it easy to tell if a specific user has already
      starred a specific post, but would not scale to large numbers of stars
      per post as it would make loading the Post data more expensive.
  3. `user-posts` - a list of posts by the user.  `/user-posts/<USER-ID>` is a list
     of all posts made by a specific user, keyed by the same push ID used in
     the `posts` tree. This makes it easy to query "all posts by a specific
     user" without filtering through all Post objects.
  4. `post-comments` - comments on a particular posts, where
    `/post-comments/<POST-ID>` is a list of all comments on post with id
    `<POST-ID>`.  Each comment has a randomly generated push key. By keeping
    this data in its own tree rather than nesting it under `posts`, we make it
    possible to load a post without loading all comments while still
    having a known path to access all comments for a particular post.

Database Rules
---------------
Below are some samples rules that limit access and validate data:

```javascript

{
  "rules": {
    // User profiles are only readable/writable by the user who owns it
    "users": {
      "$UID": {
        ".read": "auth.uid == $UID",
        ".write": "auth.uid == $UID"
      }
    },

    // Posts can be read by anyone but only written by logged-in users.
    "posts": {
      ".read": true,
      ".write": "auth.uid != null",

      "$POSTID": {
        // UID must match logged in user and is fixed once set
        "uid": {
          ".validate": "(data.exists() && data.val() == newData.val()) || newData.val() == auth.uid"
        },

        // User can only update own stars
        "stars": {
          "$UID": {
              ".validate": "auth.uid == $UID"
          }
        }
      }
    },

    // User posts can be read by anyone but only written by the user that owns it,
    // and with a matching UID
    "user-posts": {
      ".read": true,

      "$UID": {
        "$POSTID": {
          ".write": "auth.uid == $UID",
        	".validate": "data.exists() || newData.child('uid').val() == auth.uid"
        }
      }
    },


    // Comments can be read by anyone but only written by a logged in user
    "post-comments": {
      ".read": true,
      ".write": "auth.uid != null",

      "$POSTID": {
        "$COMMENTID": {
          // UID must match logged in user and is fixed once set
          "uid": {
              ".validate": "(data.exists() && data.val() == newData.val()) || newData.val() == auth.uid"
          }
        }
      }
    }
  }
}
```

Additional Resources
--------------------

- [Stack Overflow](https://stackoverflow.com/questions/tagged/firebase-database)
- [Firebase Support](https://firebase.google.com/support/)

License
-------
&copy; Chris Oung, 2019. [Apache 2.0 License](https://github.com/chrisoung/firebase-realtime-database-android/blob/master/LICENSE).
