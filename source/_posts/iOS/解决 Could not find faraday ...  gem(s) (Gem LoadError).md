---
title: 解决 Could not find 'faraday' ...  gem(s) (Gem::LoadError)
date: 2020-08-29 17:27:00
categories: "iOS"
tags:
  - iOS

---

<Excerpt in index | 首页摘要> 

`to_specs': Could not find 'faraday' (>= 0.7.4) among 77 total gem(s) (Gem::LoadError)

+<!-- more -->

<The rest of contents | 余下全文>

## 一、遇到的问题
报错内容如下：
```shell
/Users/lxf/.fastlane/bin/bundle/lib/ruby/site_ruby/2.2.0/rubygems/dependency.rb:319:in `to_specs': Could not find 'faraday' (>= 0.7.4) among 77 total gem(s) (Gem::LoadError)
Checked in 'GEM_PATH=/Users/lxf/.fastlane/bin/bundle/bin:/Users/lxf/.fastlane/bin/bundle/lib/ruby/gems/2.2.0', execute `gem env` for more information
	from /Users/lxf/.fastlane/bin/bundle/lib/ruby/site_ruby/2.2.0/rubygems/specification.rb:1439:in `block in activate_dependencies'
	from /Users/lxf/.fastlane/bin/bundle/lib/ruby/site_ruby/2.2.0/rubygems/specification.rb:1428:in `each'
	from /Users/lxf/.fastlane/bin/bundle/lib/ruby/site_ruby/2.2.0/rubygems/specification.rb:1428:in `activate_dependencies'
	from /Users/lxf/.fastlane/bin/bundle/lib/ruby/site_ruby/2.2.0/rubygems/specification.rb:1410:in `activate'
	from /Users/lxf/.fastlane/bin/bundle/lib/ruby/site_ruby/2.2.0/rubygems/specification.rb:1442:in `block in activate_dependencies'
	from /Users/lxf/.fastlane/bin/bundle/lib/ruby/site_ruby/2.2.0/rubygems/specification.rb:1428:in `each'
	from /Users/xxx/.fastlane/bin/bundle/lib/ruby/site_ruby/2.2.0/rubygems/specification.rb:1428:in `activate_dependencies'
	from /Users/lxf/.fastlane/bin/bundle/lib/ruby/site_ruby/2.2.0/rubygems/specification.rb:1410:in `activate'
	from /Users/lxf/.fastlane/bin/bundle/lib/ruby/site_ruby/2.2.0/rubygems.rb:196:in `rescue in try_activate'
	from /Users/lxf/.fastlane/bin/bundle/lib/ruby/site_ruby/2.2.0/rubygems.rb:193:in `try_activate'
	from /Users/lxf/.fastlane/bin/bundle/lib/ruby/site_ruby/2.2.0/rubygems/core_ext/kernel_require.rb:125:in `rescue in require'
	from /Users/lxf/.fastlane/bin/bundle/lib/ruby/site_ruby/2.2.0/rubygems/core_ext/kernel_require.rb:40:in `require'
	from /Users/lxf/.fastlane/bin/bundle/lib/ruby/gems/2.2.0/gems/fastlane-2.28.3/bin/fastlane:9:in `<top (required)>'
	from /Users/lxf/.fastlane/bin/bundle/bin/fastlane:22:in `load'
	from /Users/lxf/.fastlane/bin/bundle/bin/fastlane:22:in `<main>'
```

在执行 `fastlane` 命令时遇到了上面那个错误

## 二、解决方案：
执行如下几行命令即可，解决方案出自: [链接](https://github.com/fastlane/fastlane/issues/15740#issuecomment-560277535)
```shell
rm -rf $HOME/.fastlane/bin/bundle/lib/ruby/gems/2.2.0/gems/faraday-0.*
rm -rf $HOME/.fastlane/bin/bundle/lib/ruby/gems/2.2.0/specifications/faraday-0.*
gem install faraday -v 0.17.0 --install-dir $HOME/.fastlane/bin/bundle/lib/ruby/gems/2.2.0
```

接着再执行 `fastlane` 命令试试看吧~