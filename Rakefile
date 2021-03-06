#!/usr/bin/env rake

require 'fileutils'
require 'foodcritic'
require 'rspec/core/rake_task'
require 'rubocop/rake_task'

task :default => [
  :style,
  :spec
]

# Style tests. Rubocop and Foodcritic
namespace :style do
  desc 'Run Ruby style checks'
  RuboCop::RakeTask.new(:ruby)

  desc 'Run Chef style checks'
  FoodCritic::Rake::LintTask.new(:chef) do |t|
    t.options = { fail_tags: ['correctness'] }
  end
end

desc 'Run all style checks'
task style: ['style:chef', 'style:ruby']

# Rspec and ChefSpec
desc 'Run ChefSpec examples'
RSpec::Core::RakeTask.new(:spec) do |t|
  t.verbose = false
end

namespace :generate do
  desc 'Create an integration kitchen test template'
  task :test_suite, :recipe_name do |_, arg|
    gemfile_relative_path = '../../helpers/serverspec/Gemfile'
    sspec_template = "#{Dir.pwd}/test/integration/helpers/serverspec/spec_template.rb"

    suite_name = "datadog_#{arg['recipe_name']}"
    spec_name = "#{arg['recipe_name']}_spec.rb"

    suite_dir = File.join(Dir.pwd, 'test/integration', suite_name)
    sspec_dir = File.join(suite_dir, 'serverspec')

    if Dir.exist? suite_dir
      abort("Test directory named '#{suite_name}' exists!")
    else
      FileUtils.makedirs(sspec_dir)
    end

    FileUtils.copy(sspec_template, File.join(sspec_dir, spec_name))

    # Symlinks need to be relative paths to work cross-system
    Dir.chdir(sspec_dir) do
      FileUtils.symlink(gemfile_relative_path, 'Gemfile')
    end

    puts <<-EOT.sub(/\n$/, '')
      New test suite created!
      Next steps:
      - Add a new entry to .kitchen.yml
      - Modify #{spec_name} in #{sspec_dir}
      - Run `kitchen list #{suite_name}` to confirm suite added.
      - Run `kitchen test #{suite_name}` to run tests.
    EOT
  end
end
