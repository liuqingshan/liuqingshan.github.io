---
layout: post
title: 在Rails中使用Rspec feature
time: 2016年04月28日 星期四
location: 西安
---

### 本文以devise 用户注册/登录/退出为例, 需要首先在项目中集成 <a href="https://github.com/plataformatec/devise">devise</a>
```bash
  1. 添加 gem "devise"
  2. rails g devise:install
  3. rails g devise user
  4. rails g devise:views
  5. bundle exec rake db:migrate
```

### 新建文件 spec/features/signing_up_spec.rb
```ruby
require "rails_helper"

RSpec.feature "Users can sign up" do
  scenario "when input email and password to sign up" do
    visit "/"
    click_link "Sign up"
    fill_in "Email", with: "user1@test.com"
    fill_in "user_password", with: "password"
    fill_in "password_confirmation", "password"
    click_button "Sign up"
    expect(page).to have_content "You have signed up successfully."
  end
end
```

### 然后实现注册功能

### 添加登录feature测试 spec/features/signing_in_spec.rb
```ruby
require "rails_helper"

Rspec.feature "User can sign in" do
  let!(:user) { create(:user) }

  scenario "with valid credentials" do
    visit "/"
    click_link "Sign in"
    fill_in "Email", with: user.email
    fill_in "Password", with: "password"
    click_button "Log in"

    expect(page).to have_content "Signed in successfully."
    expect(page).to have_content "Signed in as #{user.email}"
  end
end
```

### 添加用户种子数据 spec/factories/user_factory.rb
```ruby
FactoryGirl.define do
  factory :user do
    sequence(:email) { |n| "test#{n}@test.com" }
    password "password"
  end
end
```

### 实现登录功能

### 用户登出 spec/features/signing_out_spec.rb
```ruby
RSpec.feature "User can sign out" do
  let(:user) { create(:user) }

  before do
    login_as(user)
  end

  scenario do
    visit "/"
    click_link "Sign out"
    expect(page).to have_content "Signed out successfully."
  end
end
```

### 在spec/rails_helper.rb中引入warden的test_helper方法
```ruby
  config.include Warden::Test::Helpers, type: :feature
  config.after(type: :feature) { Warden.test_reset! }
```

### 实现用户退出功能

### bundle exec rspec . 实现上面的所有功能并测试通过



