# frozen_string_literal: true

source "https://rubygems.org"

gem "jekyll", "~> 4.4"
gem "jekyll-theme-chirpy"
gem "jekyll-paginate", "~> 1.1"
gem "jekyll-redirect-from", "~> 0.16"
gem "jekyll-seo-tag", "~> 2.8"
gem "jekyll-archives", "~> 2.3"
gem "jekyll-sitemap", "~> 1.4"

# Performance-booster for watching directories on Windows
gem "wdm", "~> 0.2.0", :install_if => Gem.win_platform?

# Jekyll <= 4.2.0 compatibility with Ruby 3.0
gem "webrick", "~> 1.9"

group :test do
  gem "html-proofer", "~> 5.2"
end

# Windows and JRuby does not include zoneinfo files, so bundle the tzinfo-data gem
# and associated library.
install_if -> { RUBY_PLATFORM =~ %r!mingw|mswin|java! } do
  gem "tzinfo", "~> 2.0"
  gem "tzinfo-data"
end

