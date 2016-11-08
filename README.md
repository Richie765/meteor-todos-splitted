# meteor-todos-splitted
Meteor Todos Example with separate Client and Server application for testing hot code push.

This uses a modified meteor from: https://github.com/Richie765/meteor

This addresses this issue: meteor/meteor#3815

The following changes have been made:
* Hot code push will come from `ROOT_URL`
* Will connect to server at `DDP_URL` (alias of `DDP_DEFAULT_CONNECTION_URL`)
* Added `--mobile-root` option to `meteor run` and `meteor build`
* Changed `meteor build --server` to be `meteor build --mobile-server`

# Setting up
Clone repo
```
git clone --recursive https://github.com/Richie765/meteor
cd meteor
./meteor
ln -s meteor ~/bin/meteor-git
```

# Quick usage instructions

When running locally with 'meteor run'
* For server, just run as usual
* For client (browser and mobile) use the modified `meteor-git`
* For browser, use env vars `DDP_URL` to specify the server, use `ROOT_URL` to specify the root url, also used for hot code push.
* For mobile, specify the same vars, and use `--mobile-server` and `--mobile-root` with the same values to be compiled inside the App.

Building and running a bundle, server
* Build and run as usual

Building a bundle, client part
* Use the modified `meteor-git`
* When building for mobile, specify the `--mobile-server` and `--mobile-root`, the initial values to be compiled inside the App.

Running the bundle, client part
* Set `DDP_URL` and `ROOT_URL` to the same values used when building

# Test run locally

Start the server
```bash
cd server
# npm start
meteor --port 4000
```

Start client for web
```bash
cd client
# NOTE client requires modified meteor-git
# npm start
DDP_URL=http://localhost:4000 ROOT_URL=http://localhost:3000 meteor-git run
```

Start client for mobile (use your IP address or .local address on iOS+macOS)
```bash
cd client
# NOTE client requires modified meteor-git
ROOT_URL=http://192.168.1.5:3000 meteor-git --mobile-server=192.168.1.5:4000 run ios
ROOT_URL=http://myhost.local:3000 meteor-git --mobile-server=myhost.local:4000 run ios
ROOT_URL=http://192.168.1.5:3000 meteor-git --mobile-server=192.168.1.5:4000 run android
```

# Test run from bundle
Build and start the server
```bash
# Locally

cd server
meteor-git build --architecture=os.linux.x86_64 /bundle/path

# On sever

ROOT_URL=http://todos-server.com
MONGO_URL=mongodb://mongodb/todos
# run it
```

Build and start the client
```bash
# Locally

cd todos-client
meteor-git build --architecture=os.linux.x86_64 \
  --mobile-server=https://todos-server.com \
  --mobile-root=https://todos-client.com \
  /bundle/path

# On sever

DDP_URL=https://todos-server.com
ROOT_URL=http://todos-client.com
# MONGO_URL still needed but shouldn't be used. Point to a dummy DB
MONGO_URL=mongodb://mongodb/todos-dummy
# run it
```

Test mobile apps from simulator
```bash
cd todos-client
DDP_URL=https://todos-server.com ROOT_URL=http://todos-client.com meteor-git run ios --mobile-server=http://todos-client.com
DDP_URL=https://todos-server.com ROOT_URL=http://todos-client.com meteor-git run android --mobile-server=http://todos-client.com
```

# Relavant Settings

The URL for the DDP connection for publications and methods
```
# new, alias to DDP_DEFAULT_CONNECTION_URL
DDP_URL=http://
DDP_DEFAULT_CONNECTION_URL=http://
MOBILE_DDP_URL

meteor ios --mobile-server=10.10.10.125:4000

# new, same as --server
meteor build --mobile-server=http://
meteor build --server=https://

```

The root of your website, also used for hot code push
```
ROOT_URL=http://
MOBILE_ROOT_URL=http://
MOBILE_DDP_URL

meteor ios --mobile-root=10.10.10.125:4000

meteor build --mobile-root=https://
```

# Handling Account emails

As "reset-password", "verify-email" and "enroll-account" emails are sent by the server,
by default they'll contain the ROOT_URL of the server.
If you like to have them point to the client instead, you could use something like:

```javascript
// Run this on the server

let client_root = 'http://todos-client.com/';

['resetPassword', 'enrollAccount', 'verifyEmail'].forEach(func => {
  let old_func = Accounts.urls[func];
  Accounts.urls[func] = function(token) {
    let url = old_func(token);

    return url.replace(Meteor.absoluteUrl(), client_url);
  };
});

```
