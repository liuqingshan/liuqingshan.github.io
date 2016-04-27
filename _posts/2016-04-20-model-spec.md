---
layout: post
title: 在Rails中使用Rspec测试model
time: 2016年04月19日 星期二
location: 西安
---

### 预测试如下model

```ruby
class User

  validates :name, presence: true
  validates :email, presence: true, uniqueness: true

  def self.match_name(name)
    where("name like ?", "#{name}%").order(:name)
  end

  def to_s
    name
  end
end
```

### 建立Facories 文件: spec/factories/user.rb

```ruby
FactoryGirl.define do
  factory :user do
    name  { FFaker::Name.name }
    email { FFaker::Internet.email }
  end
end
```

### 引入factorygirl的简单语法模式, 打开 spec/rails_helper.rb
```ruby
RSpec.configure do |config|
   ##加入下面这一行
   config.include FactoryGirl::Syntax::Methods
end
```

### 建立测试文件: spec/models/user_spec.rb

```ruby
rails "rails_helper"

describe User do
  it "is valid with a name, email"
  it "is invalid without a name"
  it "is invalid without a email"
  it "is invalid without a duplicate email address"
  it "returns a sorted array of results that match"
  it "returns name when call to_s"
end

```

### 运行测试
```bash
  bin/rspec 或者 bundle exec rspec
```

### 实现测试代码
```ruby
require "rails_helper"

describe User do

  it "is valid with a name, email" do
    expect(build(:user)).to be_valid
  end

  it "is invalid without a name" do
    user = build(:user, name: nil)

    user.valid?
    expect(user.errors[:name]).to include("can't be blank")
  end

  it "is invalid without a email" do
    user = build(:user, email: nil)

    user.valid?
    expect(user.errors[:email]).to include("can't be blank")
  end

  it "is invalid without a duplicate email address" do
    create(:user, name: "zhangsan", email: "zhangsan@test.com")
    user = build(:user, name: "lisi", email: "zhangsan@test.com")

    user.valid?
    expect(user.errors[:email]).to include("has already been taken")
  end

  it "returns string when call to_s" do
    zhangsan = create(:user, name: "zhangsan")
    expect(zhangsan.to_s).to eq "zhangsan"
  end

  describe "filter user by name" do
    before :each do
      @zhangsan = create(:user, name: "zhangsan")
      @zhangming = create(:user, name: "zhangming")
      @xiaoming = create(:user, name: "xiaoming")
    end

    context "with matching name" do
      it "returns a sorted array of results that match" do
        expect(User.match_name("zhang")).to eq [@zhangming, @zhangsan]
      end

      it "omits results that do not match" do
        expect(User.match_name("zhang")).not_to include @xiaoming
      end
    end
  end
end

```

### 再次运行测试, 查看结果是否能测试通过
```bash
  bin/rspec 或者 bundle exec rspec
```

### 参考
  - factorygirl 更多的expect 方法: <a href="https://github.com/rspec/rspec-expectations#user-content-built-in-matchers" target="_blank">参考</a>
  - factorygirl 简洁语法: <a href="http://www.rubydoc.info/gems/factory_girl/file/GETTING_STARTED.md" target="_blank">参考</a>
  - ffaker: <a href="https://github.com/ffaker/ffaker" target="_blank">参考<a>
