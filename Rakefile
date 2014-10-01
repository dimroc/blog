desc "Deploy _site/ to master branch"
task :deploy do
  puts "building jekyll"
  system("jekyll build")
  system("mkdir -p tmp/deployment/_site")
  puts "copying"
  system("cp -R _site tmp/deployment/")
  system("cd /tmp/deployment/")
  system("git init")
  system("git remote add origin git@github.com:dimroc/dimroc.github.io.git")
  system("git add .")
  system("git commit -m 'Deployment #{Time.now}'")
  system("git push origin master -f")

  #puts "\n## Deleting master branch"
  #status = system("git branch -D master")
  #puts status ? "Success" : "Failed"
  #puts "\n## Creating new master branch and switching to it"
  #status = system("git checkout -b master")
  #puts status ? "Success" : "Failed"
  #puts "\n## Forcing the _site subdirectory to be project root"
  #status = system("git filter-branch --subdirectory-filter _site/ -f")
  #puts status ? "Success" : "Failed"
  #puts "\n## Switching back to source branch"
  #status = system("git checkout source")
  #puts status ? "Success" : "Failed"
  #puts "\n## Pushing all branches to origin"
  #status = system("git push --all origin")
  #puts status ? "Success" : "Failed"
end
