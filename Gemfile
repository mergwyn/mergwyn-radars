source ENV['GEM_SOURCE'] || 'https://rubygems.org'

def location_for(place_or_version, fake_version = nil)
  git_url_regex = %r{\A(?<url>(https?|git)[:@][^#]*)(#(?<branch>.*))?}
  file_url_regex = %r{\Afile:\/\/(?<path>.*)}

  if place_or_version && (git_url = place_or_version.match(git_url_regex))
    [fake_version, { git: git_url[:url], branch: git_url[:branch], require: false }].compact
  elsif place_or_version && (file_url = place_or_version.match(file_url_regex))
    ['>= 0', { path: File.expand_path(file_url[:path]), require: false }]
  else
    [place_or_version, { require: false }]
  end
end

ruby_version_segments = Gem::Version.new(RUBY_VERSION.dup).segments
minor_version = ruby_version_segments[0..1].join('.')

group :development do
# These where already set in pdk templates and so may not be relevant to you
  gem "json", '~> 2.0',                                require: false
  gem "voxpupuli-puppet-lint-plugins", '~> 3.0',       require: false

# The main group of rules we are adding
  gem "facterdb", '~> 1.18',                           require: false
  gem "metadata-json-lint", '>= 2.0.2', '< 4.0.0',     require: false
  gem "puppetlabs_spec_helper", '>= 2.9.0', '< 4.0.0', require: false
  gem "rspec-puppet-facts", '>= 1.10.0', '< 3.0.0',    require: false
  gem "codecov", '~> 0.2',                             require: false
  gem "dependency_checker", '~> 0.2',                  require: false
  gem "parallel_tests", '~> 3.4',                      require: false
  gem "pry", '~> 0.10',                                require: false
  gem "simplecov-console", '~> 0.5',                   require: false
  gem "puppet-debugger", '~> 1.0',                     require: false

## We pin these in order to keep the style rules static
  gem "rubocop", '= 1.6.1',                            require: false
  gem "rubocop-performance", '= 1.9.1',                require: false
  gem "rubocop-rspec", '= 2.0.1',                      require: false
  gem "rb-readline", '= 0.5.5',                        require: false, platforms: [:mswin, :mingw, :x64_mingw]

# Added locally and used by us as part of the release process, may not be relevant to you
  gem "github_changelog_generator",                    require: false
end

group :system_tests do
  gem "puppet_litmus", '< 1.0.0', require: false, platforms: [:ruby]
  gem "serverspec", '~> 2.41',    require: false
end
puppet_version = ENV['PUPPET_GEM_VERSION']
facter_version = ENV['FACTER_GEM_VERSION']
hiera_version = ENV['HIERA_GEM_VERSION']

gems = {}

gems['puppet'] = location_for(puppet_version)

# If facter or hiera versions have been specified via the environment
# variables

gems['facter'] = location_for(facter_version) if facter_version
gems['hiera'] = location_for(hiera_version) if hiera_version

if Gem.win_platform? && puppet_version =~ %r{^(file:///|git://)}
  # If we're using a Puppet gem on Windows which handles its own win32-xxx gem
  # dependencies (>= 3.5.0), set the maximum versions (see PUP-6445).
  gems['win32-dir'] =      ['<= 0.4.9', require: false]
  gems['win32-eventlog'] = ['<= 0.6.5', require: false]
  gems['win32-process'] =  ['<= 0.7.5', require: false]
  gems['win32-security'] = ['<= 0.2.5', require: false]
  gems['win32-service'] =  ['0.8.8', require: false]
end

gems.each do |gem_name, gem_params|
  gem gem_name, *gem_params
end

# Evaluate Gemfile.local and ~/.gemfile if they exist
extra_gemfiles = [
  "#{__FILE__}.local",
  File.join(Dir.home, '.gemfile'),
]

extra_gemfiles.each do |gemfile|
  if File.file?(gemfile) && File.readable?(gemfile)
    eval(File.read(gemfile), binding)
  end
end
# vim: syntax=ruby
