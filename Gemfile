source "https://rubygems.org"

# Jekyll-Version. Starten mit:
#
#     bundle exec jekyll serve --livereload
#
gem "jekyll", "~> 4.4"

# Wird für "jekyll serve" unter Ruby 3 benötigt
gem "webrick"

# Das Minima-Theme liegt direkt im Projekt (_layouts, _includes, _sass),
# daher wird kein Theme-Gem benötigt.

# Plugins (müssen auch in _config.yml unter "plugins:" stehen)
group :jekyll_plugins do
  gem "jekyll-feed", "~> 0.17"
  gem "jekyll-seo-tag", "~> 2.8"
end

# Windows und JRuby bringen keine Zeitzonendaten mit
platforms :windows, :jruby do
  gem "tzinfo", ">= 1", "< 3"
  gem "tzinfo-data"
end