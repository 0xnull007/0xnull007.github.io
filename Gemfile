source "https://rubygems.org"

# Use GitHub Pages (this pins Jekyll & all compatible dependencies)
gem "github-pages", "~> 228", group: :jekyll_plugins

# Plugins
group :jekyll_plugins do
  gem "jekyll-feed", "~> 0.12"
  gem "jekyll-remote-theme"  # <-- Add this line (for remote themes like pages-themes/hacker)
end

# Needed for Ruby >= 3.0 (Jekyll 3.x no longer ships with it)
gem "webrick", "~> 1.7"

# Optional: fixes SSL certificate issues on some macOS systems
gem "openssl", ">= 3.0", require: false

# Windows & JRuby platform fixes
platforms :mingw, :x64_mingw, :mswin, :jruby do
  gem "tzinfo", "~> 1.2"
  gem "tzinfo-data"
end

# Optional (Windows): fast file watcher
gem "wdm", "~> 0.1.1", :platforms => [:mingw, :x64_mingw, :mswin]
