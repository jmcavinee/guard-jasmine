# Guard::Jasmine [![Build Status](https://secure.travis-ci.org/guard/guard-jasmine.png)](https://travis-ci.org/guard/guard-jasmine)

Guard::Jasmine automatically tests your Jasmine specs when files are modified.

Tested on MRI Ruby 1.9.3, 2.0.0, 2.0.1 and the latest versions of JRuby and Rubinius.

If you have any questions please join us on our [Google group](http://groups.google.com/group/guard-dev) or on `#guard`
(irc.freenode.net).

## Highlights

* Continuous testing based on file modifications by [Guard][], manifold configuration by writing rules with RegExp and
Ruby.

* Fast headless testing on [PhantomJS][], a full featured WebKit browser with native support for
various web standards: DOM handling, CSS selector, JSON, Canvas, and SVG.

* Runs the standard Jasmine test runner, which can optionally integrate [Jasmine][] into the
[Rails asset pipeline][], enabling you to write your specs in [CoffeeScript][].

* Integrates [Istanbul](https://github.com/gotwarlost/istanbul) to instrument your code in the asset pipeline and
generate coverage reports.

* Custom console logger implementation for pretty printing JavaScript objects and DOM elements.

* Can be used to run [Jasmine-species](http://rudylattae.github.io/jasmine-species/) acceptance tests provided by
[Jasmine Stories](https://github.com/DominikGuzei/jasmine-stories).

* Thor and Rake command line helper for CI server integration.

* Runs on Mac OS X, Linux and Windows.

## ScreenCast

If you are a [RailsCast Pro](http://railscasts.com/pro) subscriber, I recommend to watch
[#261 Testing JavaScript with Jasmine (revised)](http://railscasts.com/episodes/261-testing-javascript-with-jasmine-revised)
for an introduction to Jasmine, Jasminerice and Guard::Jasmine.

## Installation

### Guard and Guard::Jasmine

The simplest way to install Guard is to use [Bundler](http://bundler.io).
Please make sure to have [Guard][] installed.

Add Guard::Jasmine to your `Gemfile`:

```ruby
group :development, :test do
  gem 'guard-jasmine'
end
```

Add the default Guard::Jasmine template to your `Guardfile` by running:

```bash
$ guard init jasmine
```

Please have a look at the [CHANGELOG](https://github.com/guard/guard-jasmine/blob/master/CHANGELOG.md) when
upgrading to a newer Guard::Jasmine version.

### PhantomJS

You need the PhantomJS browser installed on your system. You can download binaries for Mac OS X and Windows from
[the PhantomJS download section][].

Alternatively you can install [Homebrew][] on Mac OS X and install it with:

```bash
$ brew install phantomjs
```

If you are using Ubuntu 12.04 or above, phantomjs is in the official repositories and can be installed with apt:

```bash
$ sudo apt-get install phantomjs
```

For older versions of Ubuntu, you will need to add a repository first:

```bash
$ sudo add-apt-repository ppa:jerome-etienne/neoip
$ sudo apt-get update
$ sudo apt-get install phantomjs
```

You can also build it from source for several other operating systems, please consult the
[PhantomJS build instructions][].

## Rails with the asset pipeline setup

With Rails 3.1 and later you can write your Jasmine specs in addition to JavaScript with CoffeeScript, fully integrated
into the Rails asset pipeline. You have full access to your running Rails app, but it's a good
practice to fake the server response. Check out the excellent [Sinon.JS][] documentation to learn more about this topic.

Guard::Jasmine will start a Rails Rack server to run your specs.  Step by step instructions on configuring this are below.

### How it works

![Guard Jasmine](https://github.com/guard/guard-jasmine/raw/master/resources/guard-jasmine-with-asset-pipeline.jpg)

1. Guard is triggered by a file modification.
2. Guard::Jasmine executes the [PhantomJS script][].
3. The PhantomJS script requests the Jasmine test runner via HTTP.
4. Rails uses the asset pipeline to get the Jasmine runner, the code to be tested and the specs.
5. The asset pipeline prepares the assets, compiles the CoffeeScripts if necessary.
6. The asset pipeline has finished to prepare the needed assets.
7. Rails returns the Jasmine runner HTML.
8. PhantomJS requests linked assets and runs the Jasmine tests headless.
9. The PhantomJS script collects the Jasmine runner results and returns a JSON report.
10. Guard::Jasmine reports the results to the console and system notifications.

#### Configuring Guard Jasmine to integrate with the asset pipeline.

 * Install Guard Jasmine into the Rails project's Gemfile:

   `gem "guard-jasmine", "~> 2.0"`

 * Install Jasmine: `rails generate jasmine:install`
 * (Optionally) install Jasmine examples: `rails generate jasmine:examples`
 * (Optionally) Edit spec/javascripts/support/jasmine.yml to enable running CoffeeScript.
   * Change the `spec_files:` section to `'**/*[sS]pec.{js,coffee}'`
 * Install GuardJasmine's sample Guardfile `rails generate  guard_jasmine:install`
 * Run GuardJasmine: `guard`

A sample Rails project configured in this way is at https://github.com/nathanstitt/guard-jasmine-rails-test


## Usage

Please read the [Guard usage documentation](https://github.com/guard/guard#readme).

## Guardfile

Guard::Jasmine can be adapted to all kind of projects. Please read the
[Guard documentation](https://github.com/guard/guard#readme) for more information about the Guardfile DSL.

```ruby
guard 'jasmine' do
  watch(%r{spec/javascripts/spec\.(js\.coffee|js|coffee)$})         { "spec/javascripts" }
  watch(%r{spec/javascripts/.+_spec\.(js\.coffee|js|coffee)$})
  watch(%r{app/assets/javascripts/(.+?)\.(js\.coffee|js|coffee)$})  { |m| "spec/javascripts/#{m[1]}_spec.#{m[2]}" }
end
```

## Options

There are many options that can customize Guard::Jasmine to your needs. Options are simply supplied as hash when
defining the Guard in your `Guardfile`:

```ruby
guard 'jasmine', all_on_start: false, specdoc: :always do
  ...
end
```

### Server options

The server options configures the server environment that is needed to run Guard::Jasmine:

```ruby
server:  :jasmine_gem                         # Jasmine server to use, either :auto, :none,
                                              # :webrick, :mongrel, :thin, :unicorn, :jasmine_gem, :puma
                                              # default: :auto

server_env:  :test                            # Jasmine server Rails environment to set,
                                              # e.g. :development or :test
                                              # default: RAILS_ENV is exists, otherwise :development

server_timeout:  30                           # The number of seconds to wait for the Jasmine spec server
                                              # default: 15

port:  8888                                   # Jasmine server port to use. Note that some ports (e.g. 6665) won't work.
                                              # This is due to the fact that Webkit considers them unsafe.
                                              # default: a random, free server port

phantomjs_bin:  '~/bin/phantomjs'             # Path to phantomjs.
                                              # default: auto-detect 'phantomjs'

timeout:  20                                  # The time in seconds to wait for the spec runner to finish.
                                              # default: 10

rackup_config:  'spec/dummy/config.ru'        # Path to rackup config file (i.e. for webrick, mongrel, thin, unicorn, puma).
                                              # default: ./config.ru
                                              # This option is useful when using guard-jasmine in a mountable engine
                                              # and the config.ru is within the dummy app
```

If you're setting the `:server` option to `:none` or need to access your specs on a other host than `localhost`, you can
supply the Jasmine runner url manually:

```ruby
jasmine_url:  'http://192.168.1.5:1234/jasmine'  # URL where Jasmine is served.
                                                   # default: nil
```
You may want to have also a fixed port instead of the random generated one.

The reason why the Server environment is set to `development` by default is that in development mode
the asset pipeline doesn't concatenate the JavaScripts and you'll see the line number in the real file,
instead of a ridiculous high line number in a single, very large JavaScript.

#### Use a custom server

If you supply an unknown server name as the `:server` option, then Guard::Jasmine will execute
a `rake` task with the given server name as task in a child process. For example, if you configure
`server:  'start_my_server'`, then the command `rake start_my_server` will be executed and
you have to make sure the server starts on the port that you can get from the `JASMINE_PORT`
environment variable.

### Spec runner options

The spec runner options configures the behavior driven development (or BDD) cycle:

```ruby
spec_dir:  'app/spec'                         # Directory with the Jasmine specs.
                                              # default: 'spec/javascripts'

clean:  false                                 # Clean the spec list by only keep Jasmine specs within the project.
                                              # default: true

all_on_start:  false                          # Run all suites on start.
                                              # default: true

keep_failed:  false                           # Keep failed suites and add them to the next run again.
                                              # default: true

all_after_pass:  false                        # Run all suites after a suite has passed again
                                              # after failing.
                                              # default: true
```

The `:keep_failed` failed option remembers failed suites and not failed specs. The reason for this decision is to
avoid additional round trip time to request the Jasmine test runner for each single spec, which is mostly more expensive
than running a whole suite.

In general you want to leave the `:clean` flag on, which ensures that only Jasmine specs (files ending with `_spec.js`,
`_spec.coffee` and `_spec.js.coffee` inside your project are passed to the runner. If you have a custom project
structure or spec naming convention, you can set `:clean` to false to skip that file filter.

### Specdoc options

Guard::Jasmine can generate an RSpec like specdoc in the console after running the specs and you can set when it will
be shown in the console:

```ruby
specdoc: :always                              # Specdoc output options,
                                              # either :always, :never or :failure
                                              # default: :failure

focus: false                                  # Specdoc focus to hide successful specs when
                                              # at least one spec fails.
                                              # default: true

console: :always                              # Console.log output options,
                                              # either :always, :never or :failure
                                              # default: :failure

errors: :always                               # Error output options,
                                              # either :always, :never or :failure
                                              # default: :failure
```

With the option set to `:always`, the specdoc is shown with and without errors in your spec, whereas on with the option
set to `:never`, there is no output at all, instead just a summary of the spec run is shown. The default option
`:failure` shows the specdoc when at least one spec failed.

When `:focus` is enabled, only the failing specs are shown in the specdoc when at least one spec is failing.

The `:errors` option is partially working when using at least PhantomJS version 1.5. Please see
[Issue #166](http://code.google.com/p/phantomjs/issues/detail?id=166) for the actual status of retreiving the JavaScript
stack trace.

### Overwrite options when running all specs

You may want to have different options when the spec runner runs all specs. You can specify the `:run_all` option
as a Hash that contains any valid runner option and will overwrite the general options.

```ruby
run_all: { specdoc:  :never }                 # Run all options,
                                              # Takes any valid option
                                              # default: {}
```

#### Console logs

The `:console` options adds captured console logs from the spec runner and adds them to the specdoc. Guard:Jasmine
contains its own minimalistic console implementation. The following console methods are supported:

* `console.log`
* `console.info`
* `console.warn`
* `console.error`
* `console.debug`

The difference for each of this log methods is simply a prefix that is added to the log statement. The log also
supports the common format placeholders:

* `%s`
* `%i`, `%d`
* `%f`
* `%o`

You can further customize the log output by implement one of these methods:

* `toString()` - must return a string that describes the object
* `toJSON()` - must return an object that is used instead of the actual object.

In addition, the console can log jQuery collections and outputs the HTML representation of the element by using the
jQuery `html()` method.

### Coverage options

Guard::Jasmine supports coverage reports generated by [Istanbul](https://github.com/gotwarlost/istanbul). You need to
have `istanbul` in your path in order to make this feature work. You can install it with NPM

```ruby
$ npm install -g istanbul
```

You also need to explicit enable the coverage support in the options:

```ruby
coverage: true                                # Enable/disable JavaScript coverage support
                                              # default: false
```

### Instrumentation

Istanbul needs to instrument the implementation files so that the execution path can be detected. Guard::Jasmine comes
with a tilt template that generates instrumented implementation files when using in the asset pipeline. If you do not
use asset pipeline, than you need to instrument your files on your own, either manually (basic example: `istanbul instrument --output instrumented_scripts scripts`) or by using something like
[Guard::Process](https://github.com/guard/guard-process). You can get more information about the
instrumentation with `istanbul help instrument`. You'll also need to update your `:spec_dir` or `jasmine.yml/src_dir` settings to point Guard::Jasmine to these instrumented source files.

**Important**: You need to clear the asset cache when you change this setting, so that already compiled assets will be
recompiled. Just use the Sprockets supplied Rake task:

```ruby
$ rake assets:clean
```

#### Check coverage

By default Guard::Jasmine just outputs the coverage when enable without any effect on the spec run result. You can
make Guard::Jasmine fail the spec run when a given threshold is not met. You can set the following thresholds:

```ruby
statements_threshold: 95                      # Statements coverage threshold
                                              # default: 0

functions_threshold: 85                       # Functions coverage threshold
                                              # default: 0

branches_threshold: -10                       # Branches coverage threshold
                                              # default: 0

lines_threshold: -15                          # Lines coverage threshold
                                              # default: 0
```

A positive threshold is taken to be the minimum percentage required, a negative threshold represents the maximum number
of uncovered entities allowed.

#### Coverage report

Guard::Jasmine always shows the Istanbul text report after a spec run that contains the coverage results per file. You
can also enable two more reports:

```ruby
coverage_html: true                           # Enable Istanbul HTML coverage report
                                              # default: false

coverage_html_dir: './coverage'               # Directory to write Istanbul HTML coverage report to
                                              # default: './coverage'

coverage_summary: true                        # Enable Istanbul summary coverage report
                                              # default: false

ignore_instrumentation: 'vendor'              # Ignore files matching this regex when instrumenting
                                              # default: ''
```

The `:coverage_summary` options disables the detailed file based coverage report by a small summary coverage report.

Both of these results are more useful if they are run against the coverage data from a full spec run, so it's strongly
advised to enable the `:all_on_start` option.

With Jasmine in the asset pipeline all instrumented implementation files are available in the runtime and when you
execute a partial spec run it reports a lower coverage for the excluded files, since their associated specs aren't run.
Guard::Jasmine tries to work around this by merge only the coverage data for the changed files (Istanbul knows the file
name in opposite to Jasmine).

#### Jenkins CI integration

You can use the Cobertura format to bring coverage support to Jenkins CI, even that Guard::Jasmine has no built in
support for it. The trick is to post-process the coverage data with istanbul after the spec run:

```ruby
desc "Run all JavaScript specs with Istanbul"
task :jscov => :environment do
  # Run Jasmine tests with code coverage on and generate Jenkins-compatible Jasmine code coverage report file.
  # Must run assets:clean first to force re-compilation.
  # Make sure to fail this task if there are unit test failures.
  # For some reason does not work if run as separate exec's, so combine into one.
  exec('rake assets:clean; guard-jasmine --coverage --coverage-html --coverage-summary; \
        code=$?; if [ $code != "0" ]; then exit $code; fi; istanbul report cobertura')
end
```

### System notifications options

These options affects what system notifications are shown after a spec run:

```ruby
notification: false                           # Show success and error notifications.
                                              # default: true

hide_success: true                            # Disable successful spec run notification.
                                              # default: false

max_error_notify: 5                           # Maximum error notifications to show.
                                              # default: 3
```

## Mapping file changes to the spec filter

Jasmine doesn't know anything about your test files, it only knows the name of your specs that you specify in the
`describe` function. When a file change is detected, Guard::Jasmine extracts the first spec name of the file and uses
that spec description as spec filter.

So if you want to have a precise spec detection, you should:

* Use only one top-level description per spec file.
* Make each top-level description unique.

To get a feeling how your naming strategy works, play with the web based Jasmine runner and modify the `spec` query
parameter.

## Guard::Jasmine outside of Guard

### Thor command line utility

Guard::Jasmine includes a little command line utility to run your specs once and output the specdoc to the console.

```bash
$ guard-jasmine
```

You can get help on the available options with the `help` task:

```bash
$ guard-jasmine help spec

Usage:
  guard-jasmine spec

Options:
  -s, [--server=SERVER]                 # Server to start, either `auto`, `webrick`, `mongrel`, `thin`, `puma`
                                        # `unicorn`, `jasmine_gem` or `none`
                                        # Default: auto
  -p, [--port=N]                        # Server port to use
                                        # Default: Random free port
      [--verbose]                       # Show the server output in the console
  -e, [--server-env=SERVER_ENV]         # The server environment to use, for example `development`, `test` etc.
                                        # Default: test
      [--server-timeout=N]              # The number of seconds to wait for the Jasmine spec server
                                        # Default: 15
  -b, [--bin=BIN]                       # The location of the PhantomJS binary
  -d, [--spec-dir=SPEC_DIR]             # The directory with the Jasmine specs
                                        # Default: spec/javascripts
  -l, [--line-number=N]                 # The line which identifies the spec to be run
  -u, [--url=URL]                       # The url of the Jasmine test runner_options
                                        # Default: nil
  -t, [--timeout=N]                     # The maximum time in seconds to wait for the spec
                                        # runner to finish
                                        # Default: 10
      [--console=CONSOLE]               # Whether to show console.log statements in the spec runner,
                                        # either `always`, `never` or `failure`
                                        # Default: failure
      [--errors=ERRORS]                 # Whether to show errors in the spec runner,
                                        # either `always`, `never` or `failure`
                                        # Default: failure
      [--focus]                         # Specdoc focus to hide successful tests when at least one test fails
                                        # Default: true
      [--specdoc=SPECDOC]               # Whether to show successes in the spec runner, either `always`, `never` or `failure`
                                        # Default: always
      [--coverage]                      # Whether to enable the coverage support or not
      [--coverage-html]                 # Whether to generate html coverage report. Implies --coverage
      [--coverage-html-dir=REPORT_DIR]  # Where to save html coverage reports. Defaults to ./coverage. Implies --coverage-html
      [--coverage-summary]              # Whether to generate html coverage summary. Implies --coverage
      [--statements-threshold=N]        # Statements coverage threshold
                                        # Default: 0
      [--functions-threshold=N]         # Functions coverage threshold
                                        # Default: 0
      [--branches-threshold=N]          # Branches coverage threshold
                                        # Default: 0
      [--lines-threshold=N]             # Lines coverage threshold
                                        # Default: 0
Run the Jasmine spec runner
```

By default all specs are run, but you can supply multiple paths to your specs to run only a subset:

```bash
$ guard-jasmine spec/javascripts/a_spec.js.coffee spec/javascripts/another_spec.js.coffee
```

### Rake task integration

Guard::Jasmine provides a Rake task wrapper around the Thor command line utility. Simply create a JasmineTask within
your `Rakefile`:

```ruby
require 'guard/jasmine/task'
Guard::JasmineTask.new
```

You can configure the CLI options either by providing the options as parameter or use a block:

```ruby
require 'guard/jasmine/task'

Guard::JasmineTask.new do |task|
  task.options = '-t 15 -e test'
end

Guard::JasmineTask.new(:jasmine_no_server, '-s none')
```

Now you'll set your tasks in the Rake task list:

```bash
$ rake -T guard
rake guard:jasmine            # Run all Jasmine specs
rake guard:jasmine_no_server  # Run all Jasmine specs
```

and you can execute the Guard::Jasmine specs on the console by running
the task:

```bash
$ rake guard:jasmine
```

### Travis CI integration

With the given `guard-jasmine` script you're able to configure [Travis CI](https://travis-ci.org/) to run Guard::Jasmine.
Simply use the `script` setting in your `.travis.yml`:

```yaml
script: 'bundle exec guard-jasmine --server-timeout=60'
```

You can also run your Guard::Jasmine specs after your specs that are ran with `rake` by using `&&`:

```yaml
script: 'rake spec && bundle exec guard-jasmine'
```

When using a PhantomJS version prior to 1.5, you need to start `xvfb` before running the specs:

```yaml
before_script:
  - "export DISPLAY=:99.0"
  - "sh -e /etc/init.d/xvfb start"
```

**Tip**: It's highly recommended the have a server timeout of at least 60 seconds, since the performance of the Travis VMs seems to
vary quite a bit; sometimes the Jasmine server starts in 5 seconds, sometimes it takes as long as 50 seconds.

## How to test a Rails engine with Jasmine Gem

When building an engine, your code lives at the root but the dummy Rails app is in another folder (like `test/dummy` or `spec/dummy`).

So you have to import the Jasmine task in your `Rakefile`:

```bash
$ echo "import 'lib/tasks/jasmine.rake'" > Rakefile
$ bundle exec rake -T jasmine
rake jasmine     # Run specs via server
rake jasmine:ci  # Run continuous integration tests
```

Given your configuration, you could also need to set:

* `jasmine_url` in your `Guardfile` as explained above

* the server url in the command line: `bundle exec guard-jasmine -u http://localhost:8888/`

## Alternatives

There are many ways to get your Jasmine specs run within a headless environment. If Guard::Jasmine isn't for you,
I recommend to check out these other brilliant Jasmine runners:

* [guard-konacha][], Automatically run [konacha][] tests through Guard.
* [guard-teabag][], Guard-Teabag: Run Javascript tests with Guard and all the features of [Teabag][]
* [Karma][] spectacular Test Runner for JavaScript.

## How to file an issue

You can report issues and feature requests to [GitHub Issues](https://github.com/guard/guard-jasmine/issues). Try to figure out
where the issue belongs to: Is it an issue with Guard itself or with Guard::Jasmine? Please don't
ask question in the issue tracker, instead join us in our [Google group](http://groups.google.com/group/guard-dev) or on
`#guard` (irc.freenode.net).

When you file an issue, please try to follow to these simple rules if applicable:

* Make sure you have study the README carefully.
* Make sure you run Guard with `bundle exec` first.
* Add debug information to the issue by running Guard with `debug: true` set in the options.
* Add your `Guardfile` and `Gemfile` to the issue.
* Make sure that the issue is reproducible with your description.

## Development information

- Documentation hosted at [RubyDoc](http://www.rubydoc.info/github/guard/guard-jasmine/master/frames).
- Source hosted at [GitHub](https://github.com/guard/guard-jasmine).

Pull requests are very welcome! Please try to follow these simple rules if applicable:

* Please create a topic branch for every separate change you make.
* Make sure your patches are well tested.
* Update the [Yard](http://yardoc.org/) documentation.
* Update the README.
* Update the CHANGELOG for noteworthy changes.
* Please **do not change** the version number.

For questions please join us in our [Google group](http://groups.google.com/group/guard-dev) or on
`#guard` (irc.freenode.net).

### Open Commit Bit

Guard has an open commit bit policy: Anyone with an accepted pull request gets added as a repository collaborator.
Please try to follow these simple rules:

* Commit directly onto the master branch only for typos, improvements to the readme and documentation (please add
  `[ci skip]` to the commit message).
* Create a feature branch and open a pull-request early for any new features to get feedback.
* Make sure you adhere to the general pull request rules above.

### The guard-jasmine-debug executable

This Guard comes with a small executable `guard-jasmine-debug` that can be used to run the Jasmine test runner on PhantomJS
and see the JSON result that gets evaluated by Guard::Jasmine. This comes handy when there is an issue with your specs
and you want to see the output of the PhantomJS script.

```bash
$ guard-jasmine-debug
```

The only argument that the script takes is the URL to the Jasmine runner, which defaults to
`http://127.0.0.1:3000/jasmine`. So you can for example just run a subset of the specs by changing the URL:

```bash
$ guard-jasmine-debug http://127.0.0.1:3000/Jasmine?spec=YourSpec
```

## Author

Developed by Michael Kessler, sponsored by [FlinkFinger](http://www.flinkfinger.com).

If you like Guard::Jasmine, you can watch the repository at [GitHub](https://github.com/guard/guard-jasmine).

## Maintainer

Maintained by [Nathan Stitt](https://github.com/nathanstitt).

## Contributors

See the [CHANGELOG](https://github.com/guard/guard-jasmine/blob/master/CHANGELOG.md) and the GitHub list of
[contributors](https://github.com/guard/guard-jasmine/contributors).

## Acknowledgment

- [Ariya Hidayat][] for [PhantomJS][], a powerful headless WebKit browser.
- [Brad Phelan][] for [Jasminerice][], an elegant solution for [Jasmine][] in the Rails 3.1 asset pipeline.
- [Pivotal Labs][] for their beautiful [Jasmine][] BDD testing framework that makes JavaScript testing fun.
- [Jeremy Ashkenas][] for [CoffeeScript][], that little language that compiles into JavaScript and makes me enjoy the
front-end.
- The [Guard Team][] for giving us such a nice piece of software that is so easy to extend, one *has* to make a plugin
for it!
- All the authors of the numerous [Guards][] available for making the Guard ecosystem so much growing and comprehensive.

## License

(The MIT License)

Copyright (c) 2011-2014 Michael Kessler

Permission is hereby granted, free of charge, to any person obtaining
a copy of this software and associated documentation files (the
'Software'), to deal in the Software without restriction, including
without limitation the rights to use, copy, modify, merge, publish,
distribute, sublicense, and/or sell copies of the Software, and to
permit persons to whom the Software is furnished to do so, subject to
the following conditions:

The above copyright notice and this permission notice shall be
included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED 'AS IS', WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY
CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT,
TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

[Guard]: https://github.com/guard/guard
[Guards]: https://github.com/guard
[Guard Team]: https://github.com/guard/guard/contributors
[Ariya Hidayat]: https://twitter.com/#!/AriyaHidayat
[PhantomJS]: http://www.phantomjs.org/
[the PhantomJS download section]: http://code.google.com/p/phantomjs/downloads/list
[PhantomJS build instructions]: http://code.google.com/p/phantomjs/wiki/BuildInstructions
[Brad Phelan]: https://twitter.com/#!/bradgonesurfing
[Jasminerice]: https://github.com/bradphelan/jasminerice
[jasmine-rails]: https://github.com/searls/jasmine-rails
[Pivotal Labs]: http://pivotallabs.com/
[Jasmine]: http://pivotal.github.com/jasmine/
[the Jasmine Gem]: https://github.com/pivotal/jasmine-gem
[Jeremy Ashkenas]: https://twitter.com/#!/jashkenas
[CoffeeScript]: http://coffeescript.org/
[Rails 3.1 asset pipeline]: http://guides.rubyonrails.org/asset_pipeline.html
[Homebrew]: http://brew.sh/
[PhantomJS script]: https://github.com/netzpirat/guard-jasmine/blob/master/lib/guard/jasmine/phantomjs/guard-jasmine.coffee
[Guard::CoffeeScript]: https://github.com/guard/guard-coffeescript
[Sinon.JS]: http://sinonjs.org
[Rails asset pipeline]: http://guides.rubyonrails.org/asset_pipeline.html
[guard-konacha]: https://github.com/alexgb/guard-konacha
[konacha]: https://github.com/jfirebaugh/konacha
[Karma]: https://github.com/karma-runner/karma
[guard-teabag]: https://github.com/modeset/guard-teaspoon
[Teabag]: https://github.com/modeset/teaspoon
