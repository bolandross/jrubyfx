require 'open-uri'
task :default => [:build, :run]

jar = ENV['jar'] || "jar"
target = ENV['target'] || "target"
dist = ENV['dist'] || "dist"
output_jar = ENV['output_jar'] || "rubyfx-app.jar"
main_script = ENV['main_script'] || nil
src = ENV['src'] || 'src/*'
jruby_version = ENV['jruby_version'] || JRUBY_VERSION || "1.7.1" #if they want speedy raking, use the default so they can use MRI or other rubies

base_dir = File.dirname(__FILE__)
cd base_dir
main_script = nil if main_script == "nil"

desc "Clean all build artifacts except #{dist}/jruby-complete.jar"
task :clean do
  rm_rf target
  FileList["jrubyfxml-*-java.gem"].each do |file|
    rm file
  end
end

desc "Clean all build artifacts INCLUDING #{dist}/jruby-complete.jar"
task :clean_jruby => :clean do
  rm_rf dist
end

desc "Run a script without installing the gem"
task :run do
  ruby "-I lib '#{main_script||'samples/fxml/Demo.rb'}'"
end

desc "Build the gem"
task :build => :clean do
  sh "gem build jrubyfxml.gemspec"
end

# Alias :)
desc "Build the gem"
task :gem => :build

desc "Build and install the gem"
task :install => :build do
  sh "gem install jrubyfxml-*-java.gem"
end

task :download_jruby_jar do
	unless File.exists?("#{dist}/jruby-complete.jar") && File.size("#{dist}/jruby-complete.jar") > 0
		mkdir_p dist
		cd dist
		puts "JRuby complete jar not found. Downloading... (May take awhile)"
		download(jruby_version)
		cd base_dir
	end
end

desc "Create a full jar with embedded JRuby and given script (via main_script and src ENV var)"
task :jar => [:clean, :download_jruby_jar] do
  mkdir_p target
  
  #copy jruby jar file in, along with script and our rb files
  cp "#{dist}/jruby-complete.jar", "#{target}/#{output_jar}"
  
  #copy source in
  FileList[src].each do |iv_srv|
    cp iv_srv, "#{target}/#{File.basename(iv_srv)}" if main_script == nil || main_script != iv_srv
  end
  cp main_script, "#{target}/jar-bootstrap.rb" unless main_script == nil
  
  unless File.exists? "#{target}/jar-bootstrap.rb"
    puts "@"*79
    puts "@#{"!!!WARNING!!!".center(79-2)}@"
    puts "@#{"jar-bootstrap.rb NOT FOUND!".center(79-2)}@"
    puts "@#{"Did you set main_src= or have jar-bootstrap in src= ?".center(79-2)}@"
    puts "@"*79
  end
  
  #copy our libs in
  FileList['lib/*'].each do |librb|
    cp_r librb, target
  end
  
  # edit the jar
  cd target
  sh "#{jar} ufe '#{output_jar}' org.jruby.JarBootstrapMain *"
  chmod 0775, output_jar
  cd base_dir
end

desc "Create a full jar and run it"
task :run_jar => :jar do
  sh "java -jar #{target}/#{output_jar}"
end

BASE_URL='http://repository.codehaus.org/org/jruby/jruby-complete'

def download(version_string)
  File.open("jruby-complete.jar","wb") do |f|
    f.write(open("#{BASE_URL}/#{version_string}/jruby-complete-#{version_string}.jar").read)
  end
end
