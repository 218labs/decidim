# frozen_string_literal: true

require "bundler/gem_tasks"
require "rspec/core/rake_task"
require "decidim/gem_manager"

RSpec::Core::RakeTask.new(:spec)

task default: :spec

desc "Runs all tests in all Decidim engines"
task test_all: [:test_main, :test_subgems]

desc "Runs all tests in decidim subgems"
task test_subgems: :test_app do
  Decidim::GemManager.run_all("rake", include_root: false)
end

desc "Runs all tests in the main decidim gem"
task :test_main do
  Decidim::GemManager.new(__dir__).run("rake")
end

desc "Update version in all gems to the one set in the `.decidim-version` file"
task :update_versions do
  Decidim::GemManager.replace_versions
end

Decidim::GemManager.all_dirs(include_root: false) do |dir|
  manager = Decidim::GemManager.new(dir)
  name = manager.short_name

  desc "Runs tests on #{name}"
  task "test_#{name}" do
    manager.run("rake")
  end
end

desc "Runs tests for a random participatory space"
task :test_participatory_space do
  Decidim::GemManager.test_participatory_space
end

desc "Runs tests for a random component"
task :test_component do
  Decidim::GemManager.test_component
end

desc "Installs all local gem versions globally"
task :install_all do
  Decidim::GemManager.install_all
end

desc "Uninstalls all local gem versions"
task :uninstall_all do
  Decidim::GemManager.uninstall_all
end

desc "Pushes a new build for each gem."
task release_all: [:update_versions, :check_locale_completeness, :webpack] do
  Decidim::GemManager.run_all("rake release")
end

desc "Makes sure all official locales are complete and clean."
task :check_locale_completeness do
  system({ "ENFORCED_LOCALES" => "en,ca,es", "SKIP_NORMALIZATION" => "true" }, "rspec spec/i18n_spec.rb")
end

load "decidim-dev/lib/tasks/generators.rake"

desc "Generates a dummy app for testing"
task test_app: "decidim:generate_external_test_app"

desc "Generates a development app."
task development_app: "decidim:generate_external_development_app"

desc "Build webpack bundle files"
task :webpack do
  sh "npm install && npm run build:prod"
end

desc "Bundle all Gemfiles"
task :bundle do
  [".", "decidim-generators", "decidim_app-design"].each do |dir|
    Bundler.with_original_env do
      Dir.chdir(dir) { sh "bundle install" }
    end
  end
end

desc "Parses git-log output and produces CHANGELOG friendly entries"
task :parse_git_log, [:log_path] do |t, args|
  log_path= args[:log_path]
  puts "Usage: bin/rake parse_git_log[path_to_log_file]" unless log_path.present? && File.exists?(log_path)
  puts "Parsing: #{log_path}"

  full_log= File.open(log_path).read.strip
  entries= full_log.split(/^commit \w+[^\n]*$/)
  entries.shift # remove first empty entry from split
  puts "Found #{entries.size} entries"

  categorized= {}
  uncategorized= []
  entries.each do |entry|
    puts "ENTRY:-------------------"
    content, notes= entry.split(/^Notes:$/)
    content= content.strip
    notes= notes&.strip
    if notes.present?
      puts "CONTENT: #{content}"
      puts "NOTES: #{notes}"
      type, modules= notes.split(':')
      categorized[type]||= []
      categorized[type] << "- #{modules}: #{content}"
    else
      next if content.start_with?("New Crowdin updates")
      uncategorized << content
    end
  end
  puts "CHANGELOG ENTRIES:"
  categorized.keys.each do |type|
    puts "#{type}:"
    categorized[type].each do |entry|
      puts entry
    end
  end
  puts "UNCATEGORIZED ENTRIES:"
  puts uncategorized.join("\n")
end
