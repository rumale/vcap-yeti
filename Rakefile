$:.unshift(File.join(File.dirname(__FILE__), "lib"))
require 'rspec/core/rake_task'
require "harness"
#require "harness/color_helper"
include BVT::Harness::ColorHelpers

task :default => [:help]

desc "List help commands"
task :help do
  puts "Usage: rake [command]"
  puts "  admin\t\trun admin test cases"
  puts "  tests\t\trun core tests in parallel, e.g. rake test[5] (default to 10, max = 16)\n"
  puts "       \t\tOptions: VCAP_BVT_LONGEVITY=N can loop this task.\n"
  puts "       \t\te.g. rake tests[8] VCAP_BVT_LONGEVITY=10"
  puts "       \t\tVCAP_BVT_CONFIG_FILE=[path_to_config_file] to specify config file.\n"
  puts "       \t\te.g. rake tests VCAP_BVT_CONFIG_FILE=/home/czhang/my_test.yml\n"
  puts "       \t\tAbove options are also usable in other tasks."
  puts "  full\t\trun full tests in parallel, e.g. rake full[5] (default to 10, max = 16)"
  puts "  random\trun all bvts randomly, e.g. rake random[1023] to re-run seed 1023"
  puts "  java\t\trun java tests (spring, java_web) in parallel\n" +
          "\t\te.g. rake java[5] (default to 10, max = 16)"
  puts "  jvm\t\trun jvm tests (spring, java_web, grails, lift) in parallel\n" +
          "\t\te.g. rake jvm[5] (default to 10, max = 16)"
  puts "  ruby\t\trun ruby tests (rails3, sinatra, rack) in parallel\n" +
          "\t\te.g. rake ruby[5] (default to 10, max = 16)"
  puts "  services\trun service tests (mongodb/redis/mysql/postgres/rabbitmq/neo4j/vblob) in parallel\n" +
          "\t\te.g. rake services[5] (default to 10, max = 16)"
  puts "  clean\t\tclean up test environment(only run this task after interruption).\n" +
           "\t\t  1, Remove all apps and services under test user\n" +
           "\t\t  2, Remove all apps and services under parallel users"

  puts "  core\t\trun core tests for verifying that an installation meets\n" +
       "\t\tminimal Cloud Foundry compatibility requirements"

  puts "  mcf\t\trun Micro Cloud Foundry tests\n"

  puts "  help\t\tlist help commands"
end

desc "run full tests (not include admin cases)"
task :full, :thread_number do |t, args|
  threads = 10
  threads = args[:thread_number].to_i if args[:thread_number]
  BVT::Harness::RakeHelper.generate_config_file
  BVT::Harness::RakeHelper.check_environment
  if threads == 1
    longevity('sh("rspec --format Fuubar --color spec/ --tag ~admin --tag ~slow")')
  else
    longevity("BVT::Harness::ParallelHelper.run_tests(#{threads}, {'tags' => '~admin,~slow'})")
  end
end

desc "run tests subset"
task :tests, :thread_number do |t, args|
  threads = 10
  threads = args[:thread_number].to_i if args[:thread_number]
  BVT::Harness::RakeHelper.generate_config_file
  BVT::Harness::RakeHelper.check_environment
  if threads == 1
    longevity('sh("rspec --format Fuubar --color spec/ --tag ~admin --tag p1 --tag ~slow")')
  else
    longevity("BVT::Harness::ParallelHelper.run_tests(#{threads}, {'tags' => 'p1,~admin,~slow'})")
  end
end

desc "Run all bvts randomly, add [N] to specify a seed"
task :random, :seed do |t, args|
  BVT::Harness::RakeHelper.generate_config_file
  BVT::Harness::RakeHelper.check_environment
  if args[:seed] != nil
    longevity("sh 'bundle exec rspec spec/ --tag ~admin --tag p1 --tag ~slow' +
       ' --seed #{args[:seed]} --format d -c'")
  else
    longevity('sh "bundle exec rspec spec/ --tag ~admin --tag p1 --tag ~slow" +
       " --order rand --format d -c"')
  end
end

desc "Run admin test cases"
task :admin do
  BVT::Harness::RakeHelper.generate_config_file(true)
  BVT::Harness::RakeHelper.check_environment
  sh "bundle exec rspec --format Fuubar --color spec/users/ --tag admin"
end

desc "Run java tests (spring, java_web)"
task :java, :thread_number, :longevity, :fail_fast do |t, args|
  threads = 10
  threads = args[:thread_number].to_i if args[:thread_number]
  BVT::Harness::RakeHelper.generate_config_file
  BVT::Harness::RakeHelper.check_environment
  if threads == 1
    longevity('sh "bundle exec rspec --format Fuubar --color -P spec/**/*_spring_spec.rb," +
     "spec/**/*_java_web_spec.rb"')
  else
    longevity("BVT::Harness::ParallelHelper.run_tests(#{threads}, {'pattern' => /_(spring|java_web)_spec\.rb/})")
  end
end

desc "Run jvm tests (spring, java_web, grails, lift)"
task :jvm, :thread_number do |t, args|
  threads = 10
  threads = args[:thread_number].to_i if args[:thread_number]
  BVT::Harness::RakeHelper.generate_config_file
  BVT::Harness::RakeHelper.check_environment
  if threads == 1
    longevity('sh "bundle exec rspec --format Fuubar --color -P spec/**/*_spring_spec.rb,spec" +
     "/**/*_java_web_spec.rb,spec/**/*_grails_spec.rb,spec/**/*_lift_spec.rb"')
  else
    longevity("BVT::Harness::ParallelHelper.run_tests(#{threads},
      {'pattern' => /_(spring|java_web|grails|lift)_spec\.rb/})")
  end
end

desc "Run ruby tests (rails3, sinatra, rack)"
task :ruby, :thread_number do |t, args|
  threads = 10
  threads = args[:thread_number].to_i if args[:thread_number]
  BVT::Harness::RakeHelper.generate_config_file
  BVT::Harness::RakeHelper.check_environment
  if threads == 1
    longevity('sh "bundle exec rspec --format Fuubar --color -P spec/**/ruby18_*_spec.rb," +
     "spec/**/ruby19_*_spec.rb"')
  else
    longevity("BVT::Harness::ParallelHelper.run_tests(#{threads}, {'pattern' => /ruby1[89]_.+_spec\.rb/})")
  end
end

desc "Run service tests (mongodb, redis, mysql, postgres, rabbitmq, neo4j, vblob)"
task :services, :thread_number do |t, args|
  threads = 10
  threads = args[:thread_number].to_i if args[:thread_number]
  BVT::Harness::RakeHelper.generate_config_file
  BVT::Harness::RakeHelper.check_environment
  if threads == 1
    longevity('sh "bundle exec rspec --format Fuubar --color spec/ --tag mongodb --tag rabbitmq " +
     "--tag mysql --tag redis --tag postgresql --tag neo4j --tag vblob"')
  else
    longevity("BVT::Harness::ParallelHelper.run_tests(#{threads}, {'tags' =>
      '~admin,mongodb,rabbitmq,mysql,redis,postgresql,neo4j,vblob'})")
  end
end

desc "Clean up test environment"
task :clean do
  BVT::Harness::RakeHelper.cleanup!
end

desc "sync yeti assets binaries"
task :sync_assets do
  BVT::Harness::RakeHelper.sync_assets
end

desc 'run core tests for verifying that an installation meets minimal Cloud Foundry compatibility requirements'
RSpec::Core::RakeTask.new(:core) do |t|
  t.rspec_opts = '--tag cfcore'
end

desc 'run Micro Cloud Foundry tests'
RSpec::Core::RakeTask.new(:mcf) do |t|
  t.rspec_opts = '--tag mcf'
end

def get_longevity_time
  return ENV['VCAP_BVT_LONGEVITY'].to_i if ENV['VCAP_BVT_LONGEVITY']
  return 1
end

def longevity(cmd)
  loop_times = get_longevity_time
  if loop_times == 1
    eval(cmd)
    return
  elsif loop_times < 1
    puts red("longevity input error")
    return
  end
  time_start = Time.now
  puts yellow("loop times: #{loop_times}")
  $stdout.flush
  loop_times.times {|i|
    puts yellow("This is #{i} run.")
    eval(cmd)
  }
  puts yellow("longevity finished!")
  puts yellow("loop times:    #{loop_times}")
  t1 = Time.now
  running_time = (t1 - time_start).to_i
  puts yellow("total running time: #{running_time} seconds")
end
