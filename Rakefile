require 'bundler/gem_tasks'
require 'bundler/setup'
require 'rspec/core/rake_task'
require 'engine_cart/rake_task'
require 'rubocop/rake_task'
require 'solr_wrapper'
require 'fcrepo_wrapper'

Dir.glob('tasks/*.rake').each { |r| import r }

desc 'Run style checker'
RuboCop::RakeTask.new(:rubocop) do |task|
  task.requires << 'rubocop-rspec'
  task.fail_on_error = true
end

desc 'Run test suite and style checker'
task spec: :rubocop do
  RSpec::Core::RakeTask.new(:spec)
end

desc 'Spin up Solr & Fedora and run the test suite'
task ci: ['engine_cart:generate'] do
  # TODO: set port to nil (random port)
  solr_params = { port: '8985', verbose: true, managed: true }
  fcrepo_params = { port: '8986', verbose: true, managed: true }
  SolrWrapper.wrap(solr_params) do |solr|
    ENV['SOLR_TEST_PORT'] = solr.port
    solr.with_collection(name: 'hydra-test', dir: File.join(File.expand_path('.', File.dirname(__FILE__)), 'solr', 'config')) do
      FcrepoWrapper.wrap(fcrepo_params) do |fcrepo|
        ENV['FCREPO_TEST_PORT'] = fcrepo.port
        Rake::Task['spec'].invoke
      end
    end
  end
end

task clean: 'engine_cart:clean'
task default: :ci
