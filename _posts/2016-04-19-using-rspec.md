---
layout: post
title: 在Rails中引入Rspec测试环境
time: 2016年04月19日 星期二
location: 西安
---

### Rails 根目录下 Gemfile 中添加如下内容: (gem 版本仅供参考，依赖于Rails版本)
```ruby
  group :development, :test do
    gem "rspec-rails", "~> 3.1.0"
    gem "factory_girl_rails", "~> 4.4.1"
  end

  group :test do
    gem "ffaker", "~> 2.2.0"
    gem "capybara", "~> 2.4.3"
    gem "database_cleaner", "~> 1.3.0"
    gem "launchy", "~> 2.4.2"
    gem "selenium-webdriver", "~> 3.43.0"
  end
```

### 控制台中运行: (安装你刚在Gemfile中添加的Gem)
```bash
  bundle
```

### 控制台中运行如下命令: (生成rspec相关初始化文件, 你的测试文件需要用到这些文件)
```bash
   rails g rspec:install
```

### 修改上一步生成的.rspec 文件, 打开此文件，添加如下代码: (更友好的显示测试结果)
```ruby
  --format document
  --color
```

### 打开 config/application.rb， 添加如下内容: (自定义rails g 何时生成测试相关的文件)
```ruby
  config.generators do |g|
    g.test_framework :rspec,
       fixtures: true,
       view_specs: false,
       helper_specs: false,
       routing_specs: false,
       controller_specs: true,
       request_specs: false
    g.fixture_replement :factory_girl, dir: "spec/factories"
  end
```

### 生成rspec-stub: (生成依赖于此项目环境的rspec stub, 不然你就会多敲几个字符，比如 bundle exec rspec .) <a href="http://mislav.net/2013/01/understanding-binstubs/" target="_blank">为什么要有这一步，更详细一点的介绍点这个</a>
```bash
  bundle binstubs rspec-core
```

### 运行rspec 查看输出: (验证你配置对了没有)
```bash
  rspec
```

### 基本环境搭建完毕: (看到绿字，0个啥的，就说明搭建完毕，可以开始测试自己写的代码了)
