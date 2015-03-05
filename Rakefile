require 'html/proofer'

task default: %w[test]

task :test do
  sh "bundle exec jekyll build"
  HTML::Proofer.new("./_site", disable_external: true, checks_to_ignore: ["ImageCheck"], href_ignore: ["/blog.xml"]).run
end
