# Rakefile for GRATR        -*- ruby -*-
begin
  require 'rubygems'
  require 'rake/gempackagetask'
rescue Exception
  nil
end

require 'rake/clean'
require 'rake/testtask'
require 'rake/rdoctask'

$LOAD_PATH << 'lib'
SPEC = eval(File.read(File.join(File.dirname(__FILE__), 'gratr.gemspec')))

# Determine the current version of the software
SOURCES = FileList['lib/**/*.rb']
RDOC_DIR = './gratr'

# The default task is run if rake is given no explicit arguments.

desc "Default Task"
task :default => :test

# Define a test task.

Rake::TestTask.new { |t|
  t.libs << "tests"
  t.pattern = 'tests/Test*.rb'
  t.verbose = true
}

task :test

# Define a test that will run all the test targets.
desc "Run all test targets"
task :testall => [:test ]

# Install gratr using the standard install.rb script.

desc "Install the application"
task :install do
  ruby "install.rb"
end

# CVS Tasks ----------------------------------------------------------

desc "Tag all the CVS files with the latest release number (TAG=x)"
task :tag do
  rel = "REL_" + GRATR::VERSION.gsub(/\./, '_')
  rel << ENV['TAG'] if ENV['TAG']
  puts rel
  sh %{cvs commit -m 'pre-tag commit'}
  sh %{cvs tag #{rel}}
end

# Create a task to build the RDOC documentation tree.

rd = Rake::RDocTask.new("rdoc") { |rdoc|
  rdoc.rdoc_dir = RDOC_DIR
  rdoc.title    = SPEC.summary
  rdoc.options << '--line-numbers' << '--inline-source' <<
     '--main' << 'README'
  rdoc.rdoc_files.include(SOURCES, 'README' ) #, 'examples/examples.rb')
}

# ====================================================================
# Create a task that will package the gratr software into distributable
# tar, zip and gem files.

if ! defined?(Gem)
  puts "Package Target requires RubyGEMs"
else
  Rake::GemPackageTask.new(SPEC) do |pkg|
    #pkg.need_zip = true
    pkg.need_tar = true
  end
end

# TAGS ---------------------------------------------------------------

file 'tags' => SOURCES do
  print "Running ctags..."
  sh %{ctags #{SOURCES.join(' ')}}             # vi tags
  puts "done."
end

file 'TAGS' => SOURCES do
  sh %{ctags -e #{SOURCES.join(' ')}}          # emacs TAGS
end

# Misc tasks =========================================================

def count_lines(filename)
  lines = 0
  codelines = 0
  open(filename) { |f|
    f.each do |line|
      lines += 1
      next if line =~ /^\s*$/
      next if line =~ /^\s*#/
      codelines += 1
    end
  }
  [lines, codelines]
end

def show_line(msg, lines, loc)
  printf "%6s %6s   %s\n", lines.to_s, loc.to_s, msg
end

desc "Count lines in the main files"
task :lines do
  total_lines = 0
  total_code = 0
  show_line("File Name", "LINES", "LOC")
  SOURCES.each do |fn|
    lines, codelines = count_lines(fn)
    show_line(fn, lines, codelines)
    total_lines += lines
    total_code  += codelines
  end
  show_line("TOTAL", total_lines, total_code)
end

ARCHIVEDIR = '/mnt/flash'

task :archive => [:package] do
  cp FileList["pkg/*.tgz", "pkg/*.zip", "pkg/*.gem"], ARCHIVEDIR
end

desc "Copy rdoc html to rubyforge"
task :rdoc2rf => [:rdoc] do
  sh "scp -r #{RDOC_DIR} monora@rubyforge.org:/var/www/gforge-projects/gratr"
  sh "scp examples/*.jpg monora@rubyforge.org:/var/www/gforge-projects/gratr/examples"
end

STATS_DIRECTORIES = [
  %w(Libraries          lib/gratr),
  %w(Unit\ tests        tests),
].collect { |name, dir| [ name, "./#{dir}" ] }.select { |name, dir| File.directory?(dir) }

desc "Report code statistics (KLOCs, etc) from the application"
task :stats do
  require 'stats/code_statistics'
  CodeStatistics.new(*STATS_DIRECTORIES).to_s
end
