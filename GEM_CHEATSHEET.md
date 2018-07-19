# Instructions for gem pushing

 - code goes in lib/
 - mother_ducker.gemspec # metainfo for gem

# create a new gem

  - run `bundle gem name_of_new_gem`

# modify a project to make it a gem

  - folder structure (for terminal app)
  ```bash
    - bin/
      - executable #name of program when run on terminal
    - lib/
      - executable/ # folder which holds the code for the gem
          - version.rb # holds constant EXECUTABLE::VERSION
      - executable.rb # entry point for the app, file which loads other files
    - executable.gemspec
    - Rakefile
  ```
  - executable file in bin/
    ```ruby
      #!/usr/bin/env ruby
      
      # ruby code to launch your gem goes in here
    ```
  - Rakefile
    ```ruby
      require "bundler/gem_tasks"
      task :default => :install
    ```
  - executable.gemspec (very basic version)
    ```ruby
      Gem::Specification.new do |gem|
          gem.name = 'executable'
          gem.summary = "summary"

          gem.version = "2.0.1"
          gem.files = Dir['lib/**/*']
          gem.executables = ["executable"]

          gem.author = ['author']
          gem.licenses = ['MIT']

          gem.add_development_dependency "bundler", "~> 1.16"
          gem.add_development_dependency "rake", "~> 10.0"
      end
    ```
   - .gitignore (default from bundle gem)
    ```text
      /.bundle/
      /.yardoc
      /_yardoc/
      /coverage/
      /doc/
      /pkg/
      /spec/reports/
      /tmp/

    ```
# cheatsheet

 - with rake

 - build and install for testing

  ```bash
    $ rake install
  ```

 - release to the wider world

  ```bash
    $ rake release
  ```

- check if gem is published

  ```bash
    $ gem list -r motherducker
  ```

- if version tag is not updated

  ```bash
    $ gem yank motherducker -v VERSION
  ```

## outdated
- manually

  building the gem
  ```bash
    $ gem build mother_ducker.gemspec
  ```

  pushing the gem
  ```bash
    $ gem push {file_created_from_build}.gem
  ```

# notes

 - version can be hardcoded into the gemspec, will need to be bumped up as we go


 [guide](https://guides.rubygems.org/make-your-own-gem/)
 [another, better guide](http://robdodson.me/how-to-write-a-command-line-ruby-gem/)
