# Slack Night Mode
A user style for easy Slack theming. [CC0](http://creativecommons.org/publicdomain/zero/1.0/).

## Usage

### Sidebar Theme

#363636,#222222,#222222,#c5c8c6,#222222,#c5c8c6,#38978D,#38978D

### Ruby Switcher

```
#!/usr/bin/env ruby
require 'open-uri'
require 'sass'

JS_ERROR_MSG = "Possible js code injection found in remote css... check your repo!".freeze
remote_repo = 'https://raw.githubusercontent.com/reganjohnson/slack-night-mode/master/css/raw/black.css'
remote_css = open(remote_repo) { |f| f.read }

# Check for possible js code injection in css
abort(JS_ERROR_MSG + "\n#{remote_repo}") if remote_css.index('<script>')

# double check if ruby sass compiles correctly. Will raise if any html or non css
begin
  compsass = Sass::Engine.new(remote_css, :syntax => :scss, :style => :compressed).render
rescue Sass::SyntaxError => e
  puts e
  abort(JS_ERROR_MSG)
end

static_path = '/Applications/Slack.app/Contents/Resources/app.asar.unpacked/src/static/'
target_css_file =  static_path + 'black.css'

File.open(target_css_file, 'w') {|f| f.write remote_css}
slack_pid = `ps -axcf -S -O%cpu | grep Slack | awk '{print $2}' | head -n 1`.strip
unless slack_pid.empty?
  `kill -9 #{slack_pid}`
end

js_code = <<-JS
document.addEventListener('DOMContentLoaded', function() {
  $.ajax({
    url: 'https://raw.githubusercontent.com/reganjohnson/slack-night-mode/master/css/raw/black.css',
    success: function(css) {
      const fs = require('fs');
      fs.readFile("#{target_css_file}", 'utf8', function (err, data) {
        if (err) {
          throw err;
        } else if (css == data) {
          $('<style></style>').appendTo('head').html(data);
        }
      });
    }
  });
});
JS

@file_target = static_path + 'ssb-interop.js'
`cp #{@file_target} #{@file_target}.bak`
@current_file = File.read(@file_target)
@normal_mode = @day_mode_set ? @current_file : @current_file.split(js_code).first
@night_mode = @normal_mode + js_code

use_day_mode = ARGV[0] == '-d'
def set_mode(mode)
  if mode == 'd'
    File.open(@file_target, 'w'){|f| f.puts @normal_mode}
		puts "Slack Normal (day) mode has been set!"
  else
    File.open(@file_target, 'w'){|f| f.puts @night_mode}
		puts "Slack Normal night mode has been set!"
  end
end

use_day_mode ? set_mode('d') : set_mode('n')

`open -a "Slack"`
```
