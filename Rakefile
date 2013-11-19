# -*- ruby -*-
require 'rubygems'

gem 'hoe'
require 'hoe'
Hoe.plugin :debugging
Hoe.plugin :git
Hoe.plugin :gemspec
Hoe.plugin :bundler
Hoe.add_include_dirs '.'

GENERATED_PARSER    = "lib/nokogiri/css/parser.rb"
GENERATED_TOKENIZER = "lib/nokogiri/css/tokenizer.rb"
CROSS_DIR           =  File.join(File.dirname(__FILE__), 'ports')

def java?
  !! (RUBY_PLATFORM =~ /java/)
end

ENV['LANG'] = "en_US.UTF-8" # UBUNTU 10.04, Y U NO DEFAULT TO UTF-8?

require 'tasks/nokogiri.org'

HOE = Hoe.spec 'nokogiri' do
  developer 'Aaron Patterson', 'aaronp@rubyforge.org'
  developer 'Mike Dalessio',   'mike.dalessio@gmail.com'
  developer 'Yoko Harada',     'yokolet@gmail.com'
  developer 'Tim Elliott',     'tle@holymonkey.com'
  developer 'Akinori MUSHA',   'knu@idaemons.org'

  self.readme_file  = ['README',    ENV['HLANG'], 'rdoc'].compact.join('.')
  self.history_file = ['CHANGELOG', ENV['HLANG'], 'rdoc'].compact.join('.')

  self.extra_rdoc_files = FileList['*.rdoc','ext/nokogiri/*.c']


  self.clean_globs += [
    'nokogiri.gemspec',
    'lib/nokogiri/nokogiri.{bundle,jar,rb,so}',
    'lib/nokogiri/{1.9,2.0}',
    'ports/*.installed',
    'ports/i686-w64-mingw32',
    'ports/x86_64-w64-mingw32',
    'ports/i586-mingw32msvc',
    'ports/libxml2',
    'ports/libxslt',
    # GENERATED_PARSER,
    # GENERATED_TOKENIZER
  ]

  self.extra_deps += [
    ["mini_portile",    "~> 0.5.2"],
  ]

  self.extra_dev_deps += [
    ["hoe-bundler",     ">= 1.1"],
    ["hoe-debugging",   ">= 1.0.3"],
    ["hoe-gemspec",     ">= 1.0"],
    ["hoe-git",         ">= 1.4"],
    ["minitest",        "~> 2.2.2"],
    ["rake",            ">= 0.9"],
    ["rake-compiler",   "~> 0.9.1"],
    ["racc",            ">= 1.4.6"],
    ["rexical",         ">= 1.0.5"]
  ]

  if java?
    self.spec_extras = { :platform => 'java' }
  else
    self.spec_extras = {
      :extensions => ["ext/nokogiri/extconf.rb"],
      :required_ruby_version => '>= 1.9.2'
    }
  end

  self.testlib = :minitest
end
HOE.spec.licenses = ['MIT']

# ----------------------------------------

def add_file_to_gem relative_path
  target_path = File.join gem_build_path, relative_path
  target_dir = File.dirname(target_path)
  mkdir_p target_dir unless File.directory?(target_dir)
  rm_f target_path
  safe_ln relative_path, target_path
  HOE.spec.files += [relative_path]
end

def gem_build_path
  File.join 'pkg', HOE.spec.full_name
end

if java?
  # TODO: clean this section up.
  require "rake/javaextensiontask"
  Rake::JavaExtensionTask.new("nokogiri", HOE.spec) do |ext|
    jruby_home = RbConfig::CONFIG['prefix']
    ext.ext_dir = 'ext/java'
    ext.lib_dir = 'lib/nokogiri'
    jars = ["#{jruby_home}/lib/jruby.jar"] + FileList['lib/*.jar']
    ext.classpath = jars.map { |x| File.expand_path x }.join ':'
  end

  task gem_build_path => [:compile] do
    add_file_to_gem 'lib/nokogiri/nokogiri.jar'
  end
else
  mingw_available = true
  begin
    require 'rake/extensioncompiler'
    # Ensure mingw compiler is installed
    Rake::ExtensionCompiler.mingw_host
  rescue
    puts "WARNING: cross compilation not available: #{$!}"
    mingw_available = false
  end
  require "rake/extensiontask"

  HOE.spec.files.reject! { |f| f =~ %r{\.(java|jar)$} }

  windows_p = RbConfig::CONFIG['target_os'] == 'mingw32' || RbConfig::CONFIG['target_os'] =~ /mswin/

  unless windows_p || java?
    task gem_build_path do
      add_file_to_gem "dependencies.yml"

      dependencies = YAML.load_file("dependencies.yml")
      %w[libxml2 libxslt].each do |lib|
        version = dependencies[lib]
        archive = File.join("ports", "archives", "#{lib}-#{version}.tar.gz")
        add_file_to_gem archive
      end
    end
  end

  Rake::ExtensionTask.new("nokogiri", HOE.spec) do |ext|
    ext.lib_dir = File.join(*['lib', 'nokogiri', ENV['FAT_DIR']].compact)
    ext.config_options << ENV['EXTOPTS']
    if mingw_available
      ext.cross_compile  = true
      ext.cross_platform = ["x86-mingw32", "x64-mingw32"]
      ext.cross_config_options << "--enable-cross-build"
    end
  end
end

# To reduce the gem file size strip mingw32 dlls before packaging
ENV['RUBY_CC_VERSION'].to_s.split(':').each do |ruby_version|
  task "copy:nokogiri:x86-mingw32:#{ruby_version}" do |t|
    sh "i686-w64-mingw32-strip -S tmp/x86-mingw32/stage/lib/nokogiri/#{ruby_version[/^\d+\.\d+/]}/nokogiri.so"
  end

  task "copy:nokogiri:x64-mingw32:#{ruby_version}" do |t|
    sh "x86_64-w64-mingw32-strip -S tmp/x64-mingw32/stage/lib/nokogiri/#{ruby_version[/^\d+\.\d+/]}/nokogiri.so"
  end
end

# ----------------------------------------

desc "Generate css/parser.rb and css/tokenizer.rex"
task 'generate' => [GENERATED_PARSER, GENERATED_TOKENIZER]
task 'gem:spec' => 'generate' if Rake::Task.task_defined?("gem:spec")

# This is a big hack to make sure that the racc and rexical
# dependencies in the Gemfile are constrainted to ruby platforms
# (i.e. MRI and Rubinius). There's no way to do that through hoe,
# and any solution will require changing hoe and hoe-bundler.
old_gemfile_task = Rake::Task['bundler:gemfile'] rescue nil
task 'bundler:gemfile' do
  old_gemfile_task.invoke if old_gemfile_task

  lines = File.open('Gemfile', 'r') { |f| f.readlines }.map do |line|
    line =~ /racc|rexical/ ? "#{line.strip}, :platform => :ruby" : line
  end
  File.open('Gemfile', 'w') { |f| lines.each { |line| f.puts line } }
end

file GENERATED_PARSER => "lib/nokogiri/css/parser.y" do |t|
  racc = RbConfig::CONFIG['target_os'] =~ /mswin32/ ? '' : `which racc`.strip
  racc = "#{::RbConfig::CONFIG['bindir']}/racc" if racc.empty?
  racc = %x{command -v racc}.strip if racc.empty?
  sh "#{racc} -l -o #{t.name} #{t.prerequisites.first}"
end

file GENERATED_TOKENIZER => "lib/nokogiri/css/tokenizer.rex" do |t|
  sh "rex --independent -o #{t.name} #{t.prerequisites.first}"
end

[:compile, :check_manifest].each do |task_name|
  Rake::Task[task_name].prerequisites << GENERATED_PARSER
  Rake::Task[task_name].prerequisites << GENERATED_TOKENIZER
end

# ----------------------------------------

desc "set environment variables to build and/or test with debug options"
task :debug do
  ENV['NOKOGIRI_DEBUG'] = "true"
  ENV['CFLAGS'] ||= ""
  ENV['CFLAGS'] += " -DDEBUG"
end

require 'tasks/test'

task :java_debug do
  ENV['JAVA_OPTS'] = '-Xdebug -Xrunjdwp:transport=dt_socket,address=8000,server=y,suspend=y' if java? && ENV['JAVA_DEBUG']
end

if java?
  task :test_18 => :test
  task :test_19 do
    ENV['JRUBY_OPTS'] = "--1.9"
    Rake::Task["test"].invoke
  end
end

Rake::Task[:test].prerequisites << :compile
Rake::Task[:test].prerequisites << :java_debug
Rake::Task[:test].prerequisites << :check_extra_deps unless java?

if Hoe.plugins.include?(:debugging)
  ['valgrind', 'valgrind:mem', 'valgrind:mem0'].each do |task_name|
    Rake::Task["test:#{task_name}"].prerequisites << :compile
  end
end

# ----------------------------------------

def verify_dll(dll, fformat, dll_imports=[])
  host = 'x86_64-w64-mingw32'
  dump = `env LANG=C #{host}-objdump -p #{dll.inspect}`
  raise "unexpected file format for generated dll #{dll}" unless dump =~ /file format #{fformat}\s/
  raise "export function Init_nokogiri not in dll #{dll}" unless dump =~ /Table.*\sInit_nokogiri\s/mi

  # Verify that the expected DLL dependencies match the actual dependencies
  # and that no further dependencies exist.
  dll_imports_is = dump.scan(/DLL Name: (.*)$/).map(&:first).map(&:downcase).uniq
  if dll_imports_is.sort != dll_imports.sort
    raise "unexpected dll imports #{dll_imports_is.inspect} in #{dll}"
  end
end

desc "build a windows gem without all the ceremony."
task "gem:windows" => "gem" do
  cross_rubies = [
    ["x86-mingw32", "1.9.3-p448"],
    ["x86-mingw32", "2.0.0-p247"],
    ["x64-mingw32", "2.0.0-p247"],
  ]
  ruby_cc_version = cross_rubies.collect { |platform, version| version.split("-").first }.uniq.join(":") # e.g., "1.8.7:1.9.2"
  rake_compiler_config_path = "#{ENV['HOME']}/.rake-compiler/config.yml"

  unless File.exists? rake_compiler_config_path
    raise "rake-compiler has not installed any cross rubies. try running 'env --unset=HOST rake-compiler cross-ruby VERSION=#{cross_rubies.first.last}'"
  end
  rake_compiler_config = YAML.load_file(rake_compiler_config_path)

  # check that rake-compiler config contains the right patchlevels. see #279 for background,
  # and http://blog.mmediasys.com/2011/01/22/rake-compiler-updated-list-of-supported-ruby-versions-for-cross-compilation/
  # for more up-to-date docs.
  cross_rubies.each do |platform, version|
    majmin, patchlevel = version.split("-")
    rbconfig = "rbconfig-#{platform}-#{majmin}"
    unless rake_compiler_config.key?(rbconfig) && rake_compiler_config[rbconfig] =~ /-#{patchlevel}/
      raise "rake-compiler '#{rbconfig}' not #{patchlevel}. try running 'env --unset=HOST rake-compiler cross-ruby VERSION=#{version}'"
    end
  end

  sh("env RUBY_CC_VERSION=#{ruby_cc_version} rake cross native gem") || raise("build failed!")

  # Do some sanity checks on the built DLLs
  verify_dll 'tmp/x86-mingw32/nokogiri/1.9.3/nokogiri.so', 'pei-i386',
      %w[kernel32.dll msvcrt.dll ws2_32.dll msvcrt-ruby191.dll]
  verify_dll 'tmp/x86-mingw32/nokogiri/2.0.0/nokogiri.so', 'pei-i386',
      %w[kernel32.dll msvcrt.dll user32.dll ws2_32.dll msvcrt-ruby200.dll]
  verify_dll 'tmp/x64-mingw32/nokogiri/2.0.0/nokogiri.so', 'pei-x86-64',
      %w[kernel32.dll msvcrt.dll user32.dll ws2_32.dll x64-msvcrt-ruby200.dll]
end

# vim: syntax=Ruby
