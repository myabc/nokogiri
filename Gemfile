# frozen_string_literal: true

source "https://rubygems.org"

gemspec

group :development do
  # ruby 3.4.0-dev removed some gems from the default set
  #
  # TODO: we should be able to remove these as our gem dependencies sort it out and we pull them in
  # transitively.
  gem "mutex_m"

  # bootstrapping
  gem "bundler", "~> 2.3"
  gem "rake", "13.2.1"

  # building extensions
  gem "rake-compiler", "1.2.7"
  gem "rake-compiler-dock", "1.5.0"

  # parser generator
  gem "rexical", "= 1.0.7"

  # tests
  gem "minitest", "5.22.3"
  gem "minitest-parallel_fork", "2.0.0"
  gem "ruby_memcheck", "3.0.0"
  gem "rubyzip", "~> 2.3.2"
  gem "simplecov", "= 0.21.2"

  # rubocop
  if Gem::Requirement.new("~> 3.0").satisfied_by?(Gem::Version.new(RUBY_VERSION))
    gem "rubocop", "1.63.5"
    gem "rubocop-minitest", "0.34.5"
    gem "rubocop-packaging", "0.5.2"
    gem "rubocop-performance", "1.21.0"
    gem "rubocop-rake", "= 0.6.0"
    gem "rubocop-shopify", "2.15.1"
  end
end

# If Psych doesn't build, you can disable this group locally by running
# `bundle config set --local without rdoc`
# Then re-run `bundle install`.
group :rdoc do
  gem "rdoc", "6.6.3.1"
end
