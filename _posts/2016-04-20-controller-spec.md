---
layout: post
title: 在Rails中使用Rspec测试controller
time: 2016年04月27日 星期三
location: 西安
---

### 设置测试数据 spec/factories/book.rb

```ruby
FactoryGirl.define do
  factory :book do
    title FFaker::Name.name
    isbn FFaker::Identification.drivers_license

    after(:build) do |book|
      # 可添加关系数据
      #book.authors << Author.new(name:xxx)
    end

    factory :invalid_book do
      title nil
    end
  end
end
```

### 创建controller测试类: spec/controller/books_controller_spec.rb

```ruby
require 'rails_helper'

RSpec.describe BooksController, :type => :controller do

  describe "GET index" do
    it "assigns all books as @books" do
      book = create(:book)
      get :index
      expect(assigns(:books)).to eq([book])
    end
  end

  describe "GET show" do
    it "assigns the requested book as @book" do
      book = create(:book)
      get :show, id: book.id
      expect(assigns(:book)).to eq(book)
    end
  end

  describe "GET new" do
    it "assigns a new book as @book" do
      get :new
      expect(assigns(:book)).to be_a_new(Book)
    end
  end

  describe "GET edit" do
    it "assigns the requested book as @book" do
      book = create(:book)
      get :edit, {:id => book.id }
      expect(assigns(:book)).to eq(book)
    end
  end

  describe "POST create" do
    describe "with valid params" do

      it "creates a new Book" do
        expect {
          post :create, {:book => attributes_for(:book) }
        }.to change(Book, :count).by(1)
      end

      it "assigns a newly created book as @book" do
        post :create, {:book => attributes_for(:book) }
        expect(assigns(:book)).to be_a(Book)
        expect(assigns(:book)).to be_persisted
      end

      it "redirects to the created book" do
        post :create, {:book => attributes_for(:book) }
        expect(response).to redirect_to(Book.last)
      end
    end

    describe "with invalid params" do

      before :each do
        post :create, { :book => attributes_for(:invalid_book) }
      end

      it "assigns a newly created but unsaved book as @book" do
        expect(assigns(:book)).to be_a_new(Book)
      end

      it "re-renders the 'new' template" do
        expect(response).to render_template("new")
      end
    end
  end

  describe "PUT update" do
    describe "with valid params" do

      before :each do
        @book = create(:book)
        put :update, {:id => @book.to_param, :book => attributes_for(:book, title: "new_title", isbn: "new_isbn")}
      end

      it "updates the requested book" do
        @book.reload
        expect(@book.title).to eq("new_title")
        expect(@book.isbn).to eq("new_isbn")
      end

      it "assigns the requested book as @book" do
        expect(assigns(:book)).to eq(@book)
      end

      it "redirects to the book" do
        expect(response).to redirect_to(@book)
      end
    end

    describe "with invalid params" do
      before :each do
        @book = create(:book)
        put :update, {:id => @book.id, :book => attributes_for(:invalid_book) }
      end
      it "assigns the book as @book" do
        expect(assigns(:book)).to eq(@book)
      end

      it "re-renders the 'edit' template" do
        expect(response).to render_template("edit")
      end
    end
  end

  describe "DELETE destroy" do
    it "destroys the requested book" do
      book = create(:book)
      expect {
        delete :destroy, {:id => book.id }
      }.to change(Book, :count).by(-1)
    end

    it "redirects to the books list" do
      book = create(:book)
      delete :destroy, {:id => book.id }
      expect(response).to redirect_to(books_url)
    end
  end
end
```

### 实现测试类的功能 app/controller/bookscontroller.rb

```ruby
class BooksController < ApplicationController
  before_action :set_book, only: [:show, :edit, :update, :destroy]

  def index
    @books = Book.all
  end

  def show
  end

  def new
    @book = Book.new
  end

  def edit
  end

  def create
    @book = Book.new(book_params)

    respond_to do |format|
      if @book.save
        format.html { redirect_to @book, notice: 'Book was successfully created.' }
      else
        format.html { render :new }
      end
    end
  end

  def update
    respond_to do |format|
      if @book.update(book_params)
        format.html { redirect_to @book, notice: 'Book was successfully updated.' }
      else
        format.html { render :edit }
      end
    end
  end

  def destroy
    @book.destroy
    respond_to do |format|
      format.html { redirect_to books_url, notice: 'Book was successfully destroyed.' }
    end
  end

  private
    def set_book
      @book = Book.find(params[:id])
    end

    def book_params
      params.require(:book).permit(:title, :isbn, :publish_at)
    end
end
```

### 运行 bin/rspec . 或者 bundle exec rspec . 查看结果
