auto-deploy-with-capistrano
===========================
capistrano is a great tools for deployment.
 
for installing  capistrano first we need to install ruby .

1. Ruby Install:

   we can download ruby from ruby offical website. http://www.ruby-lang.org/en/downloads/
   
   For mac osx ruby is installed default.
   For windows user download ruby from here http://rubyinstaller.org/
   
   RVM is one of the best way to install ruby.
   https://rvm.io/rvm/install
   capistrano has dependency on  Ruby ≥ 1.8.7
   
  2. Install Capistrano
  
    In order to install Capistrano you will need Ruby and RubyGems installed on your computer. 
    Then, run the following command from a Terminal or command line:

   $ gem install capistrano
   
   after that put this command
   
   $ gem install capistrano-ext
   
   $ gem install railsless-deploy
   
  3.Server Dependencies
  
    It’s important to make sure that your server is POSIX-compilant and has SSH access. 
    Don’t forget to setup your SSH keys (Mac or Windows). If you can’t SSH into your server, 
    Capistrano isn’t going to work for you.
    
  4.Create and configure the test website
   
    Create a new site
    $ mkdir mahfuz
    $ cd mahfuz
    $ echo "This is a Test Capistrano Deploy" > index.html   
    

 5.Capify your site
   Navigate to your application’s root directory in Terminal and run the following command:
   
   $ capify .
   
   
   This command creates a special file called Capfile in your project, and adds a template deployment recipe at config/deploy.rb in your Rails project. 
   The Capfile helps Capistrano load your recipes and libraries properly, but you don’t need to edit it for now.

   Instead, open the deploy.rb file in your favorite text editor. This file is where all the magic happens!
   You can start by deleting everything in the template file, as this guide will help you fill it with the correct code for a successful deployment recipe.
   
   first add your server info
   
   # replace these with your server's information
    set :domain,  "example.com"
    set :user,    "user"
    
  # name this the same thing as the directory on your server
    set :application, "test"  
    
    
    # use your local repository as the source
     set :repository, "file://#{File.expand_path('.')}"

   # or use a hosted repository
    #set :repository, "ssh://user@example.com/~/git/test.git"
    server "#{domain}", :app, :web, :db, :primary => true
    set :deploy_via, :copy
    set :copy_exclude, [".git", ".DS_Store"]
    set :scm, :git
    set :branch, "master"
  # set this path to be correct on yoru server
    set :deploy_to, "/home/#{user}/public_html/#{application}"
    set :use_sudo, false
    set :keep_releases, 2
    set :git_shallow_clone, 1

   ssh_options[:paranoid] = false

   # this tells capistrano what to do when you deploy
    namespace :deploy do

  desc <<-DESC
  A macro-task that updates the code and fixes the symlink.
  DESC
  task :default do
    transaction do
      update_code
      symlink
    end
  end

  task :update_code, :except => { :no_release => true } do
    on_rollback { run "rm -rf #{release_path}; true" }
    strategy.deploy!
  end

  task :after_deploy do
    cleanup
  end

end


6.Validating your Recipe

  $ cap deploy:setup
  
  When you run this command, Capistrano will SSH to your server, enter the directory that you specified in the 
  deploy_to variable, and create a special directory structure that’s required for Capistrano to work properly. 
  If something is wrong with the permissions or SSH access,
  you will get error messages. Look closely at the output Capistrano gives you while the command is running.

  The last step before we can do an actual deployment with Capistrano is to make sure that everything is set up correctly on the server from the setup command. 
  There’s a simple way to verify, with the command:
  
  $cap deploy:check
  
  This command will check your local environment and your server and locate likely problems. 
  If you see any error messages, fix them and run the command again. Once you can run cap deploy:check without errors, you can proceed!
  
  7.Deploy Using your New Recipe
  
  $cap deploy
  
  This will perform a deployment to your default stage, which is staging. If you want to deploy to production, you’d run the following:
  
  $cap production deploy
  
  Once that finishes, go again to your server shell and list the directory. You’ll see something like:
  
  test/
  current/
  releases/
  shared/

  This is the second, you need to point your webserver document root to the current directory. 
  In my example, using apache, I’d set it to /home/user/public_html/test/current. Of course,
  if your web root is a sub folder of your repo then you’d do ../current/public or whatever.

  Congratulations! You’ve now deployed the site.


