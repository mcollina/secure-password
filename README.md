# `secure-password`

> Making Password storage safer for all

## Usage

```js
var securePassword = require('secure-password')

// Initialise our password policy
var pwd = securePassword({
  memlimit: securePassword.MEMLIMIT_DEFAULT,
  opslimit: securePassword.OPSLIMIT_DEFAULT
})

var userPassword = Buffer.from('my secret password')

// Register user
pwd.hash(userPassword, function (err, hash) {
  if (err) return console.error('Try again later')

  // Save hash somewhere

  pwd.verify(userPassword, hash, function (err, result) {
    if (err) return console.error('Try again later')
    if (result === securePassword.VALID) return console.log('Yay you made it')
    if (result === securePassword.INVALID) return console.log('Imma call the cops')
    if (result === securePassword.NEEDS_REHASH) {
      console.log('Yay you made it, wait for us to improve your safety')

      pwd.hash(userPassword, function (err, improvedHash) {
        if (err) console.error('You are authenticated, but we could not improve your safety this time around')

        // Save improvedHash somewhere
      })
    }
  })
})
```

## API

### `var pwd = new SecurePassword(opts)`

Make a new instance of `SecurePassword` which will contain your settings. You
can view this as a password policy for your application. `opts` currently takes
two keys `opts.memlimit` and `opts.opslimit`. They're both constrained by the
constants `SecurePassword.MEMLIMIT_MIN` - `SecurePassword.MEMLIMIT_MAX` and
`SecurePassword.OPSLIMIT_MIN` - `SecurePassword.OPSLIMIT_MAX`. If not provided
they will be given the default values `SecurePassword.MEMLIMIT_DEFAULT` and
`SecurePassword.OPSLIMIT_DEFAULT` which should be fast enough for a general
purpose web server without your users noticing too much of a load time. However
your should set these as high as possible to make any kind of cracking as costly
as possible. A load time of 1s seems reasonable for login, so test various
settings in your production environment.

The settings can be easily increased at a later time as hardware most likely
improves (Moore's law) and adversaries therefore get more powerful. Passwords
secured by an older policy will still verify just fine, but you will get
get a special return code signalling that you need to rehash the plaintext
password according to the updated policy. In contrast to other modules, this
module will not increase these settings automatically according to some formula
as this can have ill effects on services that are not carefully monitored.

### `pwd.hash(password, function (err, hash) {})`

Takes Buffer `password` and hashes it. The hashing is done by a seperate worker
as to not block the event loop, so normal execution and I/O can continue.
The callback is invoked with a potential error, or the Buffer `hash`.

`password` must be a Buffer of length `SecurePassword.PASSWORD_BYTES_MIN` - `SecurePassword.PASSWORD_BYTES_MAX`.  
`hash` will be a Buffer of length `SecurePassword.HASH_BYTES`.

### `var hash = pwd.hashSync(password)`

Takes Buffer `password` and hashes it. The hashing is done on the same thread as
the event loop, therefore normal execution and I/O will be blocked.
The function may `throw` a potential error, but most likely return
the Buffer `hash`.

`password` must be a Buffer of length `SecurePassword.PASSWORD_BYTES_MIN` - `SecurePassword.PASSWORD_BYTES_MAX`.  
`hash` will be a Buffer of length `SecurePassword.HASH_BYTES`.

### `pwd.verify(password, hash, function (err, enum) {})`

Takes Buffer `password` and hashes it and then safely compares it to the
Buffer `hash`. The hashing is done by a seperate worker as to not block the
event loop, so normal execution and I/O can continue.
The callback is invoked with a potential error, or one of the enums
`SecurePassword.VALID`, `SecurePassword.INVALID` or `SecurePassword.NEEDS_REHASH`.
Check with strict equality for one the cases as in the example above.

If `enum === SecurePassword.NEEDS_REHASH` you should call `pwd.hash` with
`password` and save the new `hash` for next time. Be careful not to introduce a
bug where a user trying to login multiple times, successfully, in quick succession
makes your server do unnecessary work.

`password` must be a Buffer of length `SecurePassword.PASSWORD_BYTES_MIN` - `SecurePassword.PASSWORD_BYTES_MAX`.  
`hash` will be a Buffer of length `SecurePassword.HASH_BYTES`.

### `var enum = pwd.verifySync(password, hash)`

Takes Buffer `password` and hashes it and then safely compares it to the
Buffer `hash`. The hashing is done on the same thread as the event loop,
therefore normal execution and I/O will be blocked.
The function may `throw` a potential error, or return one of the enums
`SecurePassword.VALID`, `SecurePassword.INVALID` or `SecurePassword.NEEDS_REHASH`.
Check with strict equality for one the cases as in the example above.

### `SecurePassword.VALID`

The password was verified and is valid

### `SecurePassword.INVALID`

The password was invalid

### `SecurePassword.NEEDS_REHASH`

The password was verified and is valid, but needs to be rehashed with new
parameters

### `SecurePassword.HASH_BYTES`

Size of the `hash` Buffer returned by `hash` and `hashSync` and used by `verify`
and `verifySync`.

### `SecurePassword.PASSWORD_BYTES_MIN`

Minimum length of the `password` Buffer.

### `SecurePassword.PASSWORD_BYTES_MAX`

Maximum length of the `password` Buffer.

### `SecurePassword.MEMLIMIT_MIN`

Minimum value for the `opts.memlimit` option.

### `SecurePassword.MEMLIMIT_MAX`

Maximum value for the `opts.memlimit` option.

### `SecurePassword.OPSLIMIT_MIN`

Minimum value for the `opts.opslimit` option.

### `SecurePassword.OPSLIMIT_MAX`

Maximum value for the `opts.opslimit` option.

### `SecurePassword.MEMLIMIT_DEFAULT`

Default value for the `opts.memlimit` option.

### `SecurePassword.OPSLIMIT_DEFAULT`

Minimum value for the `opts.opslimit` option.

## TODO

- [ ] Implement rehash checking

## Install

```sh
npm install secure-password
```

## License

[ISC](LICENSE.md)
