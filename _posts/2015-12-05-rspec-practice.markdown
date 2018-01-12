---
layout: post
title: "Rspec 实践总结"
date: 2015-12-05 11:11
comments: true
categories: [Rails, 测试, Ruby]
---

### 一、代码规范

#### describe 和 context

- 正确的描述
  * 实例方法：方法前面需要以 # 开头
  * 类方法：方法前面需要以 . 开头
  * action: action 前面需要增加 HTTP method

``` ruby
# 实例方法
describe '#login' do
end

# 类方法
describe '.login' do
end

# controller action
describe 'GET login' do
end

describe 'POST login' do
end

describe 'PUT login' do
end
```

- 两者的区别
  - describe
    - 用来表达独立的功能
  - context 
    - 用来表达独立功能下的一个场景

``` ruby
describe 'GET binding_phone' do
  context '当没有登陆时' do
    it '重定向到登陆页' do
      get :binding_page
      expect(response).to redirect_to(zhaoshang_login_path)
    end
  end
  
  context '当已经登陆时' do
    context '当已经绑定手机时' do
      it '渲染 binded_phone' do
        account_login(controller)
        get :binding_phone
        expect(response).to render_template('binded_phone')
      end
    end
    
    context '当没有绑定手机时' do
      it '渲染 binding_phone' do
      	controller.stub(:current_account).and_return(nophone_account)
        get :binding_page
        expect(response).to render_template('binding_page')
      end
    end
  end
end
```



#### let 和 let!

- let 是延迟执行（lazy-evaluated），当第一次显示调用是才会执行
- let! 在每个用例执行前调用

#### 慎用 before(:all)

before(:all) 中对数据库的操作不在每个用例的事务中，每个用例执行后不会自动清理，需要在 after(:all) 中来清理数据



#### 使用 expect

``` ruby
# BAD
describe 'GET verify_phone' do
  it '渲染 verify_phone' do
    account_login(controller)
    get :verify_phone
    response.should render_template('verify_phone')
  end
end

# GOOD
describe 'GET verify_phone' do
  it '渲染 verify_phone' do
    account_login(controller)
    get :verify_phone
    expect(response).to render_template(:verify_phone)
  end
end
```



#### 测试所有的可能

#### 合理的 it

- 保证简洁
- 一个 it 一个断言

``` ruby
# BAD
before(:each) do
  Account.any_instance.stub(:phone_confirmed?).and_return(true)
end

it '当验证码正确时，将手机号存放在 cache 中，返回 status 为1' do
  post :retrieve_pw_by_phone_commit, phone_number: account.phone_number, code: '1234'
  
  expect(
    Rails.cache.read(controller.send :phone_confirm_code_key)
  ).to eq(account.phone_number)
  expect(response_json['status']).to eq(1)
end

# GOOD
context '当验证码正确时' do
  before(:each) do
    Account.any_instance.stub(:phone_confirmed?).and_return(true)
  end
  
  it '将手机号存在 cache 中' do
    post :retrieve_pw_by_phone_commit, phone_number: account.phone_number, code: '1234'
    
    expect(
      Rails.cache.read(controller.send :phone_confirm_code_key)
    ).to eq(account.phone_number)   
  end
  
  it '返回 status 为1' do
    post :retrieve_pw_by_phone_commit, phone_number: account.phone_number, code: '1234'
    
    expect(response_json['status']).to eq(1)
  end
end
```



### 二、controller 测试

#### 测试要点

- 变量的定义

``` ruby
# controller 代码
def create_success
  @phone_number = params['phone_number']
end

# rspec 测试
describe 'GET create_success' do
  it '定义 @phone_number' do
    get :create_success, phone_number: '133****3622'
    expect(assigns(:phone_number)).to eq('133****3622')
  end
end
```

- 期望的行为

``` ruby
it '保存时执行 change_account_quota' do
  account.daily_quota = 1000
  
  account.should_receive(:change_account_quota)
  account.save
end
```

- 响应的处理

``` ruby
describe 'GET login' do
  it_behaves_like 'redirect_when_login', :get, :login
  
  context '当没有登陆时' do
    it '渲染登陆页' do
      get :login
      expect(response).to render_template('login')
    end
  end
end
```

- 数据的改变

``` ruby
it 'check refunding_to_refused' do
  expect {
    post :check, { id: refund.id, status: 'refused', reason: reason }  
  }.to change(Pay::DetailLog, :count).from(1).to(2)
end
```

#### 测试规范

``` ruby
# GET 请求
describe 'GET login' do
end
# POST 请求
describe 'POST login' do
end
# PUT 请求
describe 'PUT login' do
end
# PATCH 请求
describe 'PATCH login' do
end
```

### 三、TIPS

#### helper

``` ruby
module Support
  module Helper
    def response_json
      JSON.parse(response.body)
    end
    
    def route_to_zhaoshang(url, options = {})
      route_to(url, options.merge(subdomain: 'zhaoshang'))
    end
  end
end
```

#### better output

![https://s3.amazonaws.com/media-p.slid.es/uploads/kimigao/images/900763/output.png](https://s3.amazonaws.com/media-p.slid.es/uploads/kimigao/images/900763/output.png)

