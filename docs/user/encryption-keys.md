---
title: Encryption keys
layout: en
permalink: encryption-keys/
---

Travis generates a pair of private and public RSA keys which can be used
to encrypt information which you will want to put into the `.travis.yml` file and
still keep it private. Currently we allow encryption of
[environment variables](/docs/user/build-configuration/#Secure-environment-variables), notification settings, and deploy api keys.

## Usage

The easiest way to encrypt something with the public key is to use Travis CLI.
This tool is written in Ruby and published as a gem. First, you need to install
the gem:

    gem install travis

Then, you can use `encrypt` command to encrypt data (This example assumes you are running the command in your project directory. If not, add `-r owner/project`):

    travis encrypt SOMEVAR=secretvalue

This will output a string looking something like:

    secure: ".... encrypted data ...."

Now you can place it in the `.travis.yml` file. 

Please note that the name of the environment variable and its value are both encoded in the string produced by "travis encrypt." You must add the entry to your .travis.yml with key "secure" (underneath the "env" key). This makes the environment variable SOMEVAR with value "secretvalue" available to your program.

You may add multiple entries to your .travis.yml with key "secure." They will all be available to your program.  

You can read more about
[secure environment variables](/docs/user/build-configuration/#Secure-environment-variables)
or [notifications](/docs/user/notifications).

### Notifications Example

We want to add campfire notifications to our .travis.yml file, but we don't want to publicly expose our API token.

The entry should be in this format:

    notifications:
      campfire: [subdomain]:[api token]@[room id]

For us, that is somedomain:abcxyz@14.

We encrypt this string

    travis encrypt somedomain:abcxyz@14

Which produces something like this

    Please add the following to your .travis.yml file:

      secure: "ABC5OwLpwB7L6Ca...."

We add to our .travis.yml file

    notifications:
      campfire:
        rooms:
          secure: "ABC5OwLpwB7L6Ca...."

And we're done.

### Detailed Discussion

The secure var system takes values of the form ```{ 'secure' => 'encrypted string' }``` in the (parsed YAML) configuration and replaces it with the decrypted string.

So

    notifications:
      campfire:
        rooms:
          secure: "encrypted string"

becomes

    notifications:
      campfire:
        rooms: "decrypted string"

while

    notifications:
      campfire:
        rooms:
          - secure: "encrypted string"

becomes

    notifications:
      campfire:
        rooms:
          - "decrypted string"

In the case of secure env vars

    env:
      - secure: "encrypted string"

becomes

    env:
      - "decrypted string"

## Fetching the public key for your repository

You can fetch the public key with Travis API, using `/repos/:owner/:name/key` or
`/repos/:id/key` endpoints, for example:

    https://api.travis-ci.org/repos/travis-ci/travis-ci/key

You can also use the `travis` tool for retrieving said key:

    travis pubkey

Or, if you're not in your project directory:

    travis pubkey -r owner/project
