require 'html/proofer'

task :test do
  sh "bundle exec jekyll build"
  HTML::Proofer.new("./_site", {:disable-external => true, :checks-to-ignore => "ImageCheck"}).run
end
