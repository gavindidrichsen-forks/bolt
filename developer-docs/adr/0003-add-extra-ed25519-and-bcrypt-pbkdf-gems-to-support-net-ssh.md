# 3. Add extra ed25519 and bcrypt_pbkdf gems to support net-ssh

Date: 2024-12-10

## Status

Accepted

## Context

On provisioning a docker image using `bundle exec rake 'litmus:provision[docker, litmusimage/debian:12]'` the following error occurred:

```bash
unsupported key type `ssh-ed25519'
net-ssh requires the following gems for ed25519 support:
 * ed25519 (>= 1.2, < 2.0)
 * bcrypt_pbkdf (>= 1.0, < 2.0)
See https://github.com/net-ssh/net-ssh/issues/565 for more information
Gem::LoadError : "ed25519 is not part of the bundle. Add it to your Gemfile."
```

A `bundle why net-ssh` shows that `bolt` is the point at which the `net-ssh` is injected:

```bash
➜  puppetlabs-motd-OLD git:(main) ✗ bundle why net-ssh
puppet_litmus -> bolt -> net-scp -> net-ssh
serverspec -> specinfra -> net-scp -> net-ssh
puppet_litmus -> bolt -> net-ssh-krb -> net-ssh
puppet_litmus -> bolt -> net-ssh
serverspec -> specinfra -> net-ssh
```

It seems that although the latest `net-ssh` supports `ed255519`, it doesn't by default include the missing dependencies above.  The best place to add the dependencies is probably the `bolt` gem.

## Decision

Therefore, I decided to add the following to the bolt.gemspec:

```ruby
gem "ed25519", '>= 1.2', '< 2.0',              require: false
gem "bcrypt_pbkdf", '>= 1.0', '< 2.0',         require: false
```

## Consequences

Now, puppet_litmus works fine under the above circumstances.
