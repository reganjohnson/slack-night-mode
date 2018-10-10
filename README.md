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

### Browser

This theme requires that you use [a user styles extension](https://github.com/openstyles/stylus/wiki/Stylish-Alternatives) for your browser, such as Stylus (available for [Firefox](https://addons.mozilla.org/en-US/firefox/addon/styl-us/), [Chrome](https://chrome.google.com/webstore/detail/stylus/clngdbkpkpeebahjckkjfobafhncgmne), and [Opera](https://addons.opera.com/en/extensions/details/stylus/)).

### Desktop App

No official support. Workarounds exist.

**🛑 READ FIRST:** Most workarounds will request the compiled CSS file from this repository. You are strongly discouraged from using a remote CSS file. It's recommended that you create your own copy. An XSS attack could put your Slack client at risk.

[![Chat on Gitter](https://badges.gitter.im/laCour/slack-night-mode.png)](https://gitter.im/slack-night-mode/Lobby?utm_source=share-link&utm_medium=link&utm_campaign=share-link) ([previous discussion](https://github.com/laCour/slack-night-mode/issues/73#issuecomment-242707078))

## Themes

### Supported

#### Black ([source](scss/main.scss) - [build](css/black.css) - [install](https://userstyles.org/styles/117475/slack-night-mode-black))

The primary supported theme. This is an excellent theme if you use a program like f.lux or redshift.

![Black Screenshot](https://userstyles.org/style_screenshots/117475_after.png)

#### Aubergine ([source](scss/themes/_aubergine.scss) - [build](css/variants/aubergine.css) - [install](https://userstyles.org/styles/101971/slack-night-mode))

This is based on Slack's aubergine/maroon style. It's the original theme.

![Aubergine Screenshot](https://userstyles.org/style_screenshots/101971_after.png)

### Variants

* **Arc ([source](scss/themes/_arc-dark.scss) - [build](css/variants/arc-dark.css))** _by [@Lemmmy](https://github.com/Lemmmy)_
* **Midnight Blue ([source](scss/themes/_midnight-blue.scss) - [build](css/variants/midnight-blue.css))** _by [@matt-h](https://github.com/matt-h)_
* **Tomorrow Dark (base16) ([repository](https://github.com/danarnold/slack-night-mode))** _by [@danarnold](https://github.com/danarnold)_

### Extensions

Variants can have extensions which add additional changes.

#### Monospaced ([source](scss/themes/_monospaced.scss))

Replaces the messaging font stack with a monospace font stack.
