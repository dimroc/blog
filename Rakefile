desc "Deploy _site/ to master branch"
task :deploy do
  puts "building jekyll"
  system("jekyll build")
  system("mkdir -p tmp/deployment/_site")
  puts "copying"
  system("cp -R _site tmp/deployment/")

  Dir.chdir "tmp/deployment/_site"
  system("touch .nojekyll")
  system("git init")
  system("git remote add origin git@github.com:dimroc/dimroc.github.io.git")
  system("git add .")
  system("git commit -m 'Deployment #{Time.now}'")
  system("git push origin master -f")
end
