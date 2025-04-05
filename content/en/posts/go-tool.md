+++
title = 'Go Tool: everything that nobody has asked for'
date = 2025-04-03T20:02:01-03:00
draft = false
tags = ["go", "1.24", "tools", "news", "opinion"]
+++

After many years working with Ruby, I migrate to Go without much experience with the language. My first friction was with dependency management because I always find it bad, with fuzzy commands and, the worst, without distinction between development and production dependencies, since both of them are included in the binary. Let's take a look at a `go.mod`  from a PoC:

```go.mod
module github.com/mfbmina/poc_circuit_breaker

go 1.24

require github.com/sony/gobreaker/v2 v2.0.0 // indirect
```

Time goes by, these issues were solved, I was able to understand how this management worked, and I'm still working mainly with Go. When I read that the 1.24 version will bring the "go tool" feature, I was excited about it, and usually I'm not the kind of guy that follows the upcoming versions, but after testing it out, I was disappointed.

The `go tools`, as the name says, is a feature to add the tools that the projects rely on, like goreleaser, golangci-lint, etc. To install a new tool, you run the command `$ go get -tool github.com/golangci/golangci-lint/v2/cmd/golangci-lint@v2.0.2`.  Once installed, you use it from Go's CLI: `$ go tool github.com/golangci/golangci-lint/v2/cmd/golangci-lint run`.  To list all installed tools, run `$ go tool`.

```go.mod
module github.com/mfbmina/poc_circuit_breaker

go 1.24

require github.com/sony/gobreaker/v2 v2.0.0

// omiting a lot of indirect dependencies...

tool github.com/golangci/golangci-lint/v2/cmd/golangci-lint
```

At first sight, it seems excellent since you can have all tools in your dependency file, but the main issue is still unresolved. It even can get worse, depending on how you see it, because it now includes all production and development dependencies on one `go.mod`, with the tools, without the option to install just some of them. Actually, tools like the [golanci-lint don't recommend](https://golangci-lint.run/welcome/install/#install-from-sources) this kind of installation, exactly because there isn't isolation between the dependencies. You can find some alternatives to using the `go tools`, but to be honest, I haven't tried it out because I lost interest in the feature.

What I was expecting was being able to create dependency groups like we can do in apps written in Ruby, JavaScript, PHP, etc. For instance, let's take a look at a Gemfile, a Ruby dependency file:

```Gemfile
# copied from https://github.com/sidekiq/sidekiq/blob/main/Gemfile
source "https://rubygems.org"

gemspec

gem "rake"
RAILS_VERSION = "~> 8.0"
gem "actionmailer", RAILS_VERSION
gem "actionpack", RAILS_VERSION
gem "activejob", RAILS_VERSION
gem "activerecord", RAILS_VERSION
gem "railties", RAILS_VERSION
gem "redis-client"
# gem "bumbler"
# gem "debug"

gem "sqlite3", "~> 2.2", platforms: :ruby
gem "activerecord-jdbcsqlite3-adapter", platforms: :jruby
gem "after_commit_everywhere", require: false
gem "yard"
gem "csv"
gem "vernier" unless RUBY_VERSION < "3"
gem "webrick"

group :test do
  gem "maxitest"
  gem "simplecov"
  gem "debug"
end

group :development, :test do
  gem "standard", require: false
end

group :load_test do
  gem "toxiproxy"
  gem "ruby-prof"
end
```

That way, we can install dependencies for just one group. This feature was everything that nobody's asked exactly because it's a solution for a non-real problem, or that doesn't bother much. Maybe the issue was mine, with unreal expectations on this feature...

Anyway, I still expect having my Go binaries without development and testing dependencies. Perhaps it will come in the next versions.
